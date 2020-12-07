---
title: Seata AT 模式
date: 2020-04-25 11:00:00
tags: 'Seata'
categories:
  - ['开发', '框架']
permalink: seata-at-mode
---

## 简介

AT 模式是一种无侵入的分布式事务解决方案. 在 AT 模式下, 用户只需关注自己的“业务 SQL”, 用户的 “业务 SQL” 作为一阶段, Seata 框架会自动生成事务的二阶段提交和回滚操作

### 一阶段

在一阶段, Seata 会拦截“业务 SQL”, 首先解析 SQL 语义, 找到“业务 SQL”要更新的业务数据, 在业务数据被更新前, 将其保存成“before image”, 然后执行“业务 SQL”更新业务数据, 在业务数据更新之后, 再将其保存成“after image”, 最后生成行锁. 以上操作全部在一个数据库事务内完成, 这样保证了一阶段操作的原子性.

### 二阶段提交

二阶段如果是提交的话, 因为“业务 SQL”在一阶段已经提交至数据库,  所以 Seata 框架只需将一阶段保存的快照数据和行锁删掉, 完成数据清理即可.

### 二阶段回滚

二阶段如果是回滚的话, Seata 就需要回滚一阶段已经执行的“业务 SQL”, 还原业务数据. 回滚方式便是用“before image”还原业务数据, 但在还原前要首先要校验脏写, 对比“数据库当前业务数据”和 “after image”, 如果两份数据完全一致就说明没有脏写, 可以还原业务数据, 如果不一致就说明有脏写, 出现脏写就需要转人工处理.

AT 模式的一阶段、二阶段提交和回滚均由 Seata 框架自动生成, 用户只需编写“业务 SQL”, 便能轻松接入分布式事务, AT 模式是一种对业务无任何侵入的分布式事务解决方案

<!-- more -->

## 简单使用

### 服务端

启动 seata-server, seata-server 主要作为事务协调者, 维护全局和分支事务的状态, 驱动全局事务提交或回滚.

```sh
docker run --name seata-server -p 8091:8091 seataio/seata-server:latest
```

or

```yaml
version: '3.1'
services:

  seata-server:
    image: seataio/seata-server:latest
    hostname: seata-server
    ports:
      - 8091:8091
    environment:
      - SEATA_PORT=8091
    expose:
      - 8091
    depends_on:
      - nacos
```

可以看到相应的输出表示启动成功

```sh
root@seata-server:/seata-server# tail -f /root/logs/seata/seata-server.log
2020-04-17 02:41:55,364 INFO The server is running in container.

2020-04-17 02:41:55,593 INFO The configuration file used is registry.conf

2020-04-17 02:41:55,798 INFO The configuration file used is file.conf

2020-04-17 02:41:56,951 INFO Server started ...
```

### 配置

项目依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-alibaba-seata</artifactId>
  <version>2.2.0.RELEASE</version>
</dependency>
```

配置文件中使用 file 模式连接

```yaml
seata:
  enabled: true
  application-id: ${spring.application.name}
  tx-service-group: seataGroup-${seata.application-id}
  service:
    vgroup-mapping: default
    grouplist: 127.0.0.1:8091
    disable-global-transaction: false
  config:
    type: file
    file:
      name: file.conf
  registry:
    type: file
    file:
      name: file.conf
```

### 使用

AT 模式可以使用自带的注解实现简单的无侵入应用, 下面新建一个远程服务, 使用注解实现 AT 模式

```java
@Slf4j
@Service
public class AtServiceImpl implements AtService {

    @Autowired
    AtDAO atDAO;

    @Override
    @GlobalTransactional(timeoutMills = 60000 * 2)
    public String insert(Map<String, String> params) {
        log.info("------------------> xid = " + RootContext.getXID());
        atDAO.insert(params);
        return "success";
    }
}

@RestController
public class AtController {

    @Autowired
    AtService atService;

    @PostMapping("/at-insert")
    public String insert(@RequestBody Map<String, String> params) {
        return atService.insert(params);
    }
}
```

下面在本地服务中调用远程事务并模拟执行过程出错进行全局事务回滚

```java
/**
  * 请求远程服务插入一条记录
  * 再请求本地事务插入一条记录
  *
  * @param params - 业务参数
  * @return String
  */
@Override
@GlobalTransactional(timeoutMills = 60000 * 2)
public String insertAt(Map<String, String> params) {
    log.info("------------------> xid = " + RootContext.getXID());
    String res = atFeign.insertAT(params);
    log.info(res);
    tmDAO.insert(params);
    throw new RuntimeException("AT 服务测试回滚");
}
```

启动本地和远程服务后会发现 seata-server 已经接入了

```sh
2020-04-17 03:01:52,745 INFO RM register success,message:RegisterRMRequest{resourceIds='jdbc:mysql://127.0.0.1:3306/seata_test', applicationId='service-tm', transactionServiceGroup='service-tm-seata-service-group'},channel:[id: 0xf171ea08, L:/172.19.0.5:8091 - R:/172.19.0.1:54952]

2020-04-17 03:01:55,350 INFO TM register success,message:RegisterTMRequest{applicationId='service-tm', transactionServiceGroup='service-tm-seata-service-group'},channel:[id: 0x6bc59083, L:/172.19.0.5:8091 - R:/172.19.0.1:54956]

2020-04-17 03:02:06,675 INFO RM register success,message:RegisterRMRequest{resourceIds='jdbc:mysql://127.0.0.1:3306/seata_test', applicationId='service-at', transactionServiceGroup='service-at-seata-service-group'},channel:[id: 0x0bbf5a53, L:/172.19.0.5:8091 - R:/172.19.0.1:54980]

2020-04-17 03:02:07,302 INFO TM register success,message:RegisterTMRequest{applicationId='service-at', transactionServiceGroup='service-at-seata-service-group'},channel:[id: 0xd6bba9af, L:/172.19.0.5:8091 - R:/172.19.0.1:54984]
```

### 调用方法

通过访问本地服务发起请求

```sh
curl http://{{ip}}:7700/insert-at?name=az2
```

当执行到可以看到相应的日志输出

```sh
2020-04-17 11:03:47.412  INFO 5150 --- [nio-7700-exec-1] c.t.s.service.impl.TmServiceImpl         : ------------------> xid = 172.19.0.5:8091:2009254517
2020-04-17 11:03:47.415  INFO 5150 --- [lector_TMROLE_1] i.s.c.r.netty.NettyClientChannelManager  : return to pool, rm channel:[id: 0xe9a71eed, L:/127.0.0.1:54887 ! R:/127.0.0.1:8091]
2020-04-17 11:03:47.415  INFO 5150 --- [lector_RMROLE_1] i.s.c.r.netty.NettyClientChannelManager  : return to pool, rm channel:[id: 0xf9752dbe, L:/127.0.0.1:54878 ! R:/127.0.0.1:8091]
```

可以在 undo 表中看到如下数据

```
1  2009254519  172.19.0.5:8091:2009254518  serializer=jackson  {"@class":"io.seata.rm.datasource.undo.BranchUndoLog","xid":"172.19.0.5:8091:2009254518","branchId":2009254519,"sqlUndoLogs":["java.util.ArrayList",[{"@class":"io.seata.rm.datasource.undo.SQLUndoLog","sqlType":"INSERT","tableName":"service_at","beforeImage":{"@class":"io.seata.rm.datasource.sql.struct.TableRecords$EmptyTableRecords","tableName":"service_at","rows":["java.util.ArrayList",[]]},"afterImage":{"@class":"io.seata.rm.datasource.sql.struct.TableRecords","tableName":"service_at","rows":["java.util.ArrayList",[{"@class":"io.seata.rm.datasource.sql.struct.Row","fields":["java.util.ArrayList",[{"@class":"io.seata.rm.datasource.sql.struct.Field","name":"id","keyType":"PrimaryKey","type":4,"value":4},{"@class":"io.seata.rm.datasource.sql.struct.Field","name":"NAME","keyType":"NULL","type":12,"value":"az2"}]]}]]}}]]}  0  2020-04-17 03:08:22  2020-04-17 03:08:22
```

解析成 json 后可以发现这就是 seata 自动将业务转换为补偿操作的镜像缓存

```json
{
  "@class": "io.seata.rm.datasource.undo.BranchUndoLog",
  "xid": "172.19.0.5:8091:2009254518",
  "branchId": 2009254519,
  "sqlUndoLogs": [
    "java.util.ArrayList",
    [
      {
        "@class": "io.seata.rm.datasource.undo.SQLUndoLog",
        "sqlType": "INSERT",
        "tableName": "service_at",
        "beforeImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords$EmptyTableRecords",
          "tableName": "service_at",
          "rows": [
            "java.util.ArrayList",
            []
          ]
        },
        "afterImage": {
          "@class": "io.seata.rm.datasource.sql.struct.TableRecords",
          "tableName": "service_at",
          "rows": [
            "java.util.ArrayList",
            [
              {
                "@class": "io.seata.rm.datasource.sql.struct.Row",
                "fields": [
                  "java.util.ArrayList",
                  [
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "id",
                      "keyType": "PrimaryKey",
                      "type": 4,
                      "value": 4
                    },
                    {
                      "@class": "io.seata.rm.datasource.sql.struct.Field",
                      "name": "NAME",
                      "keyType": "NULL",
                      "type": 12,
                      "value": "az2"
                    }
                  ]
                ]
              }
            ]
          ]
        }
      }
    ]
  ]
}
```

注意：**当请求超时后服务端会自动回滚**

```sh
2020-04-17 03:05:29,661 INFO Global transaction[172.19.0.5:8091:2009254517] is timeout and will be rolled back.

2020-04-17 03:05:30,658 INFO Successfully rollback global, xid = 172.19.0.5:8091:2009254517

2020-04-17 03:05:38,376 INFO SeataMergeMessage xid=172.19.0.5:8091:2009254517,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/seata_test,lockKey=service_at:3
,clientIp:172.19.0.1,vgroup:service-at-seata-service-group

2020-04-17 03:05:38,389 ERROR Catch TransactionException while do RPC, request: xid=172.19.0.5:8091:2009254517,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/seata_test,lockKey=service_at:3

io.seata.core.exception.GlobalTransactionException: Could not found global transaction xid = 172.19.0.5:8091:2009254517
```

注意：**当其他分支数据被修改后会触发异常, 当前分支事务不会正常回滚**

```sh
2020-04-17 16:49:11.596  INFO 17658 --- [tch_RMROLE_1_16] i.s.core.rpc.netty.RmMessageListener     : onMessage:xid=172.19.0.5:8091:2009254554,branchId=2009254555,branchType=AT,resourceId=jdbc:mysql://127.0.0.1:3306/seata_test,applicationData=null
2020-04-17 16:49:11.614  INFO 17658 --- [tch_RMROLE_1_16] io.seata.rm.AbstractRMHandler            : Branch Rollbacking: 172.19.0.5:8091:2009254554 2009254555 jdbc:mysql://127.0.0.1:3306/seata_test
2020-04-17 16:49:11.790  INFO 17658 --- [tch_RMROLE_1_16] i.s.r.d.undo.AbstractUndoExecutor        : Field not equals, name NAME, old value az3, new value az3111
2020-04-17 16:49:11.801  INFO 17658 --- [tch_RMROLE_1_16] i.seata.rm.datasource.DataSourceManager  : [stacktrace]branchRollback failed. branchType:[[AT, 172.19.0.5:8091:2009254554, 2009254555, jdbc:mysql://127.0.0.1:3306/seata_test, null, Branch session rollback failed and try again later xid = 172.19.0.5:8091:2009254554 branchId = 2009254555 Has dirty records when undo.]], xid:[{}], branchId:[{}], resourceId:[{}], applicationData:[{}]. stacktrace:[{}]

io.seata.core.exception.BranchTransactionException: Branch session rollback failed and try again later xid = 172.19.0.5:8091:2009254554 branchId = 2009254555 Has dirty records when undo.
```

## 参考

- [Seata 分布式事务实践和开源详解 | GIAC 实录](https://www.sofastack.tech/blog/seata-distributed-transaction-deep-dive/)
