---
title: spring-boot-external-configuration
date: 2021-03-07 13:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-boot-external-configuration
photo:
---

https://aijishu.com/a/1060000000024101

  SpringBoot 正式环境必不可少的外部化配置 - 极术社区 - 连接 AIoT 开发者与生态服务

! [](https://cdn-static.aijishu.com/v-6049f0b3/public/favicons/wechat-share-icon.png)

[极术社区](/) [](/search) [注册](/login)

*   [首页](/)
*   [专栏](/blogs)
*   [专题](/topics)
*   [问答](/questions)
*   [公开课](/lives)
*   [活动](/events)

*   [注册 ` 登录](/login)

▲

[](javascript:;)

0

* * *

[](javascript:;)

 [! [](https://cdn-avatar.aijishu.com/231/243/231243724-5d5f70fa9bf1d_24) 逸飞兮](/u/lw5946) ` 2019 年 11 月 12 日

# [SpringBoot 正式环境必不可少的外部化配置](/a/1060000000024101)

[Java](/t/java) [服务器](/t/server)

! [SpringBoot 外部化配置 ](/img/bVgqP "SpringBoot 外部化配置 ")

# 前言

[<[源码解析] 凭什么? spring boot 一个 jar 就能开发 web 项目>](https://mp.weixin.qq.com/s?__biz= Mzg2MDIxODU2Nw==&mid=2247483692&idx=1&sn=04fe1bbb5e1d07e0dd591a9da4f485a9&chksm= ce28f484f95f7d92583bdab398c0c2b6256e1387a5cce6f263de77e70ac104921c207c625b1a&mpshare=1&scene=1&srcid=&sharer_sharetime=1573392057076&sharer_shareid= a676e0ed20ad819ca1459ece28973a8a#rd) 中有读者反应:

> 部署后运维很不方便, 比较修改一个 IP 配置, 需要重新打包.

这一点我是深有体会,17 年自学, 并很大胆的直接在生产环境用的时候, 我都是让产品经理 (此时他充当我们的运维, 嘿嘿) 用压缩软件打开 jar, 然后复制出配置, 修改完之后再替换回去. 为什么我这么大胆, 因为当时才入行一年, 而且觉得有架构师兜底, 我就奔放了. 你是不知道, 当时负责这个项目的开发 (c#开发) 一开始不想用 SpringBoot 的.

不过如今看到这个问题, 我有点震惊, 都 9102 年了, 竟然还担心这样的问题. 我想说, 哥们, 这真的不是事儿.SpringBoot 早就提供了方法来解决这个问题.

# SpringBoot 生产特性

SpringBoot 有很多生产特性, 可以在生产环境中使用时更加方便. 其中外部化配置基本都会用到.

> Spring Boot 允许外部化配置, 以便相同的应用在不同的环境中工作.
> 属性值可以在 Spring 环境中使用 @Value 或 @ConfigurationProperties 使用.

此次参考的版本是 `SpringBoot-2.2.0.RELEASE`

# 优先级

外部化配置的优先级顺序如下:

1.  Devtools 全局配置: 当 devtools 启用时,`$HOME/.config/spring-boot`
2.  测试类中的 `@TestPropertySource`
3.  测试中的 `properties` 属性: 在 @SpringBootTest 和 用来测试特定片段的测试注解
4.  命令行参数
5.  `SPRING_APPLICATION_JSON` 中的属性: 内嵌在环境变量或系统属性中的 JSON
6.  `ServletConfig` 初始化参数
7.  `ServletContext` 初始化参数
8.  `java: comp/env` 中的 JNDI 属性
9.  Java 系统属性:`System.getProperties()`
10.  操作系统环境变量
11.  随机值 (`RandomValuePropertySource`):`random.*` 属性
12.  jar 包**外**的指定 profile 配置文件:`application-{profile}.properties`
13.  jar 包**内**的指定 profile 配置文件:`application-{profile}.properties`
14.  jar 包**外**的默认配置文件:`application.properties`
15.  jar 包**内**的默认配置文件:`application.properties`
16.  代码内的 `@PropertySource` 注解: 用于 `@Configuration` 类上
17.  默认属性: 通过设置 `SpringApplication.setDefaultProperties` 指定

注意: 以上用 `properties` 文件的地方也可用 `yml` 文件

# 配置随机值

```
my.uuid=${random.uuid}
```

# 命令行属性

```
java -jar -Ddemo= vm demo.jar --demo= arg
```

*   \-Dxxx 为 vm 参数, 在代码中通过 `System#getProperty` 获取
*   \--xxx 为 spring 命令行参数, 通过 `Environment#getProperty` 获取, 若通过此方法获取不到, 会获取 vm 同名参数
*   xxx.jar 之后的参数都是 arg 参数, 都会在 main 方法中的 arg 数组中获取到

## 示例

```
public static void main(String[] args) {
    ConfigurableApplicationContext context = SpringApplication.run(ArgApplication.class, args);
    LOGGER.info("----------------");
    /* 打印 arg 参数 */
    Arrays.stream(args)
        .forEach(
            arg -> {
              LOGGER.info("arg: {}", arg);
            });
    /* 命令行传参 demo */
    LOGGER.info("System#getProperty: {}", System.getProperty("demo"));
    LOGGER.info("Environment#getProperty: {}", context.getEnvironment().getProperty("demo"));
}
```

输入命令

```
java -jar -Ddemo= vm arg-0.0.1-SNAPSHOT.jar aaa bbb ccc --demo= arg
```

效果如下:

```
----------------
arg: aaa
arg: bbb
arg: ccc
arg:--demo= arg
System#getProperty: vm
Environment#getProperty: arg
```

而如果执行命令是:

```
java -jar -Ddemo= vm arg-0.0.1-SNAPSHOT.jar aaa bbb ccc
```

结果如下:

```
arg: aaa
arg: bbb
arg: ccc
System#getProperty: vm
Environment#getProperty: vm
```

如果执行命令是:

```
java -jar arg-0.0.1-SNAPSHOT.jar aaa bbb ccc --demo= arg
```

结果如下:

```
arg: aaa
arg: bbb
arg: ccc
arg:--demo= arg
System#getProperty: null
Environment#getProperty: arg
```

# 属性文件

优先级:

1.  file:./config/
2.  file:./
3.  classpath:/config/
4.  classpath:/

如果定义了 `spring.config.location`, 如:`classpath:/custom-config/, file:./customr-config/`, 优先级如下:

1.  file:./custom-config/
2.  classpath: custom-config/

如果指定了 `spring.config.additional-location`, 会先加载 **additional** 配置 如:`spring.config.additional-location= classpath:/custom-config/, file:./customr-config/`, 优先级如下:

1.  file:./custom-config/
2.  classpath:/custom-config/
3.  file:./config/
4.  file:./
5.  classpath:/config/
6.  classpath:/

# 指定 profile 的属性

默认的 profile 是 `default`, 当没有指定 `spring.profiles.active` 属性时, 默认会加载 `application-default.properties` 文件. 指定 profiles 文件的加载顺序与上述不指定 profiles 文件的加载一致.` 指定 profile 文件的属性始终覆盖未指定文件的属性 `. 如:`spring.profiles.active= dev`, 则 `application-dev.properties` 文件内的属性会覆盖 `application.properties` 内的同名属性.

> 注意: 如果在 `spring.config.location` 属性中指定了 ` 文件 `, 则此文件对应的特定 profiles 类文件不起作用. 如果想要起作用, 在 `spring.config.location` 中使用 ` 文件夹 `.

# 占位符

配置文件中可以引用之前定义的值, 如下:

```
app.name= MyApp
app.description=${app.name} is a Spring Boot application.
```

可以用此特性创建一些已存在的 Spring Boot 配置的较短, 易于使用的变量. 如下:

```
# nacos 配置示例
spring:
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        namespace: d9a39d78-xxxxxxxx-ea4f282e9d99
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: d9a39d78-xxxxxxxx-ea4f282e9d99
# Discovery 配置示例
nacos:
  plugin:
    namespace: d9a39d78-xxxxxxxx-ea4f282e9d99
```

可改为如下配置

```
spring:
  cloud:
    nacos:
      config:
        server-addr: ${app.server-addr}
        namespace: ${app.namespace}
      discovery:
        server-addr: ${app.server-addr}
        namespace: ${app.namespace}
# Discovery 配置示例
nacos:
  plugin:
    namespace: ${app.namespace}

app:
  server-addr: 127.0.0.1:8848
  namespace: d9a39d78-xxxxxxxx-ea4f282e9d99
```

然后在命令行可以直接通过 `-Dapp.namespace` 或 `--app.namespace` 来传参, 会方便很多. 特别是在多个地方用到同一个属性的时候.

# 属性加密

Spring Boot 不支持属性加密, 但提供钩子节点修改配置属性.`EnvironmentPostProcessor` 接口允许在应用启动前操作 `Environment` .

# yaml

yaml 文件使用的时候非常直观, 方便. 而且在 Spring Boot 中做了处理, 获取 yaml 和 properties 文件中的属性基本是一样的操作.

## 一个文件指定多 pfofile

通过 `spring.profiles` 指示何时使用对应的配置, 使用 `---` 进行配置分隔

```
# application.yml
server:
  address: 192.168.1.100
---
spring:
  profiles: development
server:
  address: 127.0.0.1
---
spring:
  profiles: production & eu-central
server:
  address: 192.168.1.120
```

## yaml 缺点

用 `@PropertySource` 不能加载 yaml 文件, 这种情况下只能使用 properties 文件.

在特定 profile 的 yaml 文件中使用多 profile 配置, 会有意料之外的情况:

```
# application-dev.yml
server:
  port: 8000
---
spring:
  profiles: "! test"
  security:
    user:
      password: "secret"
```

当运行时指定 `--spring.profiles.active= dev` , 启用 dev profile, 其它的 profile 会忽略. 也就是此例中 `spring.security.user.password` 属性会失效.

因此, 不要在指定 profile 的 yaml 文件中使用多种 profile 配置.

# 类型安全的属性配置

## JavaBean 属性绑定

通过 `@ConfigurationProperties` 注解将属性 (properties, yml 文件, 环境变量等) 绑定到类对象中. 与自动配置类类似.

```
@ConfigurationProperties("acme")
public class AcmeProperties{
    private boolean enabled;
    private InetAddress remoteAddress;
    private final Security security = new Security();
    // getter and setter
    public static class Security{
        private String username;
        private String password;
        private List<String> roles = new ArrayList<> (Collections.singleton("USER"));
         // getter and setter
    }
}
```

> 这种安排依赖于默认的无参构造器, getter 和 setter 通常是必需的, 因为绑定就像 Spring MVC 一样是通过标准的 Java Beans 属性描述符进行的. 在下列情况下, 可省略 setter:
>
> *   Maps: 只要被初始化后, getter 必须而 setter 不必须, binder 可以对它们进行修改
> *   Collections 和 数组: 可以通过索引或逗号分隔的值来设定属性. 后者必须有 setter 方法. 建议对于这种情况一直加上 setter. 如果初始化了一个 Collection, 确保它不是不可变类型.
> *   如果初始化了嵌套的 POJO 属性 (如上例中的 Security), setter 不是必须的. 如果需要 binder 通过其默认构造器动态创建实例, 则需要 setter
>
> 注意: 如果使用 Lombok 生成 getter 和 setter, 确保不会生成任何特定的构造器, 不然容器会自动使用它来实例化对象.
> 最后, 只有标准 Java Bean 属性可以这样绑定属性, 静态属性不支持.

## 构造器绑定

上述示例可以改成如下:

```
@ConstructorBinding
@ConfigurationProperties("acme")
public class AcmeProperties{
  private final boolean enabled;
  private final InetAddress remoteAddress;
  private final Security security;

  public AcmeProperties(boolean enabled, InetAddress remoteAddress, Security security) {
      this.enabled = enabled;
      this.remoteAddress = remoteAddress;
      this.security = security;
  }
  // getter and setter

  public static class Security{
      private final String username;
      private final String password;
      private final List<String> roles;
      public Security(String username, String password, @DefaultValue("USER") List<String> roles) {
          this.username = username;
          this.password = password;
          this.roles = roles;
      }
      // getter and setter
  }
}
```

`@ConstructorBinding` 注解表示使用构造函数绑定属性值. 这意味着 `binder` 将期望找到一个包含待绑定参数的构造器.
`@ConstructorBinding` 类的嵌套成员也将通过构造函数绑定属性值.

可以使用 `@DefaultValue` 指定默认值, 转换服务将字符串值强转为缺少属性的目标类型.

> 要使用构造绑定, 类必须允许使用 `@EnableConfigurationProperties` 或 配置属性扫描方式. 不能对由常规 Spring 机制创建的 bean 使用构造函数绑定. 如:@Component Bean, 通过@Bean 方法创建的 Bean 或使用@Import 加载的 Bean
>
> 如果类中有多个构造器, 可以直接将 `@ConstructorBinding` 注解使用在要绑定的构造器上.

## 启用 `@ConfigurationProperties` 注解类型

> Spring Boot 提供了一个基础设施来绑定这些类型并将它们自动注册为 bean.
> 如果应用程序中使用 `@SpringBootsApplication`, 用 `@ConfigurationProperties` 注解的类将被自动扫描并注册为 bean. 默认情况下, 将从声明此注解的类的包中进行扫描. 如果要扫描特定的包, 可以对 ``@SpringBootsApplication` 注解的类显式使用 `@ConfigurationPropertiescan` 注解, 如下例所示:

```
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "org.acme.another" })
public class MyApplication {
}
```

> 有时, 用 `@ConfigurationProperties` 注释的类可能不适合扫描, 例如, 如果正在开发自己的自动配置, 在这些情况下, 可以在任何@Configuration 类上指定要处理的类型列表, 如下例所示:

```
@Configuration(proxyBeanMethods = false) @EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration { }
```

> 注意: 当使用配置属性扫描或通过@EnableConfigurationProperties 注册@ConfigurationProperties bean 时, bean 有一个常规名称:`<prefix>-<fqn>`, 其中 `<prefix>` 是 `@ConfigurationProperties` 注解中指定的环境 key 前缀,`<fqn>` 是 bean 的完全限定名. 如果注解没有提供任何前缀, 则只使用 bean 的完全限定名.
> 上例中 bean name 是 `acme-com.example.AcmeProperties`.

## 使用 `@ConfigurationProperties` 注解类型

这种类型的配置在 SpringApplication 外部 YAML 配置中特别适用, 如下例所示:

```
# application.yml

acme:
  remote-address: 192.168.1.1
  security:
    username: admin
    roles:
      - USER
      - ADMIN
```

`@ConfigurationProperties` bean 可以像其它 bean 一样注入使用. 如下:

```
@Service
public class MyService{
    private final AcmeProperties properties;

    @Autowired
    public MyService(AcmeProperties properties) {
        this.properties = properties;
    }

    // ...
}
```

> 使用 `@ConfigurationProperties` 还可以生成元数据文件, IDE 可以使用这些文件提供代码自动完成功能.

## 第三方配置

除了可以在 ` 类 ` 上使用 `@ConfigurationProperties` 注解, 还可以在 public @Bean ` 方法 ` 上使用它. 如果要将属性绑定到不在控制范围内的第三方组件, 那么这样做特别有用.

要从 `Environment` 属性配置 bean, 将 `@ConfigurationProperties` 添加到其 bean 注册中, 如下例所示:

```
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
    //...
}
```

> 用 `another` 前缀定义的任何 JavaBean 属性都映射到 `AnotherComponent` bean 上, 映射方式类似于前面的 AcmeProperties 示例.

## 松绑定

> Spring Boot 使用一些宽松的规则将 `Environment` 属性绑定到 `@ConfigurationProperties` bean, 因此环境属性名和 bean 属性名之间不需要完全匹配. 常见的包括短划线分隔的环境属性 (例如,`context-path` 绑定到 `contextPath`) 和大写的环境属性 (例如,`PORT` 绑定到 `port`).

```
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {
    private String firstName;
    public String getFirstName() {
        return this.firstName;
    }
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
}
```

对于以上 Java Bean, 可以使用以下属性

! [属性松绑定](/img/bVgqQ " 属性松绑定 ")

> 注意: 注解的前缀值必须是短横线 (小写, 用 - 分隔, 如: acme.my-project.person).

## 放宽每个属性源的绑定规则

! [放宽每个属性源的绑定规则](/img/bVgqR " 放宽每个属性源的绑定规则 ")

> 建议: 如果可能的话, 将属性存储为小写的短横线格式, 例如: my.property-name= acme.

在绑定到 `Map` 属性时, 如果 `key` 包含除小写字母 - 数字字符或 `-` 之外的任何内容, 则需要使用括号符号, 以便保留原始值. 如果 `key` 没有被 `[]` 包围, 则删除任何不是字母数字或 `-` 的字符.

```
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```

上面的属性将绑定到 `Map` 的这些 `key` 中:`/key1`,`/key2`,`key3`

## 合并复杂类型

### List

当在多个位置配置 list 时, 通过**替换**(而非添加) 整个 list 来覆盖.

```
@ConfigurationProperties("acme")
public class AcmeProperties {
    private final List<MyPojo> list = new ArrayList<> ();
    public List<MyPojo> getList() { return this.list;
    }
}
``````
acme:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

当启用 `dev` 配置时,`AcmeProperties.list` 中值包含一个 `MyPojo` 对象 (name 为 `my another name`), 不是添加操作, 而是覆盖操作.

> 当一个 `List` 在多个 profiles 中定义时, 最高优先级的被使用.

### Map

对于 `Map` 属性,**可以**使用从多个属性源获取属性值进行绑定. 但是, 对于多个源中的同一属性, 将使用优先级最高的属性.

```
@ConfigurationProperties("acme")
public class AcmeProperties {
    private final Map<String, MyPojo> map = new HashMap<> ();
    public Map<String, MyPojo> getMap() {
    return this.map;
    }
}
``````
acme:
  map:
    key1:
      name: my name 1
      description: my description 1
---
spring:
  profiles: dev
acme:
  map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2
```

当 dev 配置启用时,`AcmeProperties.map` 中包含两个键值对.`key1` 中 pojo name 为 dev name 1, description 为 my description 1;`key2` 中 pojo name 为 dev name 2, description 为 dev description 2.

**不同属性源的配置进行了合并**

> 以上合并规则适用于所有的属性源

## 属性转换

> Spring Boot 试图在绑定到 `@ConfigurationProperties` bean 时将外部应用程序属性强转为正确的类型. 如果需要自定义类型转换, 可以提供 `ConversionService` bean(带有名为 `ConversionService` 的 bean) 或自定义属性编辑器 (通过 `CustomEditorConfigurer` bean) 或自定义 `Converters` (使用 bean 定义注解 `@ConfigurationPropertiesBinding` ).
>
> 注意: 由于此 bean 在应用程序生命周期的早期被请求, 请确保限制 `ConversionService` 正在使用的依赖项. 通常, 需要的任何依赖项在创建时都可能未完全初始化. 如果自定义的 `ConversionService` 不需要配置 keys 强转, 并且仅依赖于使用 `@ConfigurationPropertiesBinding` 限定的自定义转换器, 则可能需要将它重命名.

## 时间区间转换

SpringBoot 对表示持续时间有专门的支持. 如果暴露 `java.time.Duration` 属性, 则可以用以下格式:

*   常规的 `long` 表示 (除非指定了 `@DurationUnit`, 否则使用毫秒作为默认单位)
*   `java.time.Duration` 使用的标准 ISO-8601 格式
*   一种更可读的格式, 其中值和单位是耦合的 (例如,10s 表示 10 秒)

```
@ConfigurationProperties("app.system")
public class AppSystemProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public void setSessionTimeout(Duration sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }

}
```

要指定 30 秒的 sessionTimeout,30, PT30S 和 30s 都是等效的.500ms 的 readTimeout 可以用以下任何形式指定:500, PT0.5S 和 500ms.
也可以使用以下任何支持的单位:

*   `ns`: 纳秒
*   `us`: 微妙
*   `ms`: 毫秒
*   `s`: 秒
*   `m`: 分
*   `h`: 时
*   `d`: 天

> 默认的单位是毫秒, 可以使用 `@DurationUnit` 指定

## 数据 size 转换

Spring 框架有一个 `DataSize` 类型, 以字节表示大小. 如果暴露一个 `DataSize` 属性, 则可以用以下格式:

*   常规的 `long` 表示 (除非指定了 `@DataSizeUnit`, 否则使用字节作为默认单位)
*   `java.time.Duration` 使用的标准 ISO-8601 格式
*   一种更可读的格式, 其中值和单位是耦合的 (例如,`10MB` 表示 10 兆字节).

```
@ConfigurationProperties("app.io")
public class AppIoProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public void setBufferSize(DataSize bufferSize) {
        this.bufferSize = bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

    public void setSizeThreshold(DataSize sizeThreshold) {
        this.sizeThreshold = sizeThreshold;
    }

}
```

要指定 10 兆字节的 `bufferSize`,`10` 和 `10MB` 是等效的.256 字节的 `sizeThreshold` 可以指定为 `256` 或 `256B` .
也可以使用以下任何支持的单位:
`B`: 字节
`KB`: 千字节
`MB`: 兆字节
`GB`: 千兆字节
`TB`: 兆兆字节

> 默认的单位是字节, 可以使用 `@DataSizeUnit` 指定

# @ConfigurationProperties 校验

每当对 `@ConfigurationProperties` 类使用 Spring 的 `@Validated` 注解时, Spring Boot 就会验证它们. 可以直接在配置类上使用 JSR-303 `javax.validation` 约束注解. 必须确保类路径上有一个兼容的 JSR-303 实现 (如: hibernate-validator), 然后将约束注解添加到字段中.

```
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {
    @NotNull
    private InetAddress remoteAddress;

    // ... getters and setters
}
```

> 注意: 还可以通过注解@Bean 方法来触发验证, 该方法使用@Validated 创建配置属性.
>
> 尽管嵌套属性在绑定时也将被验证, 但最好对关联字段使用 `@Valid`. 这确保即使找不到嵌套属性, 也会触发验证.

```
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    // ... getters and setters

    public static class Security {

        @NotEmpty
        public String username;

        // ... getters and setters

    }

}
```

> 还可以通过创建 `ConfigurationPropertiesValidator` bean 来添加自定义 Spring `Validator`.`@Bean` 方法应该声明为 `static` . 配置属性验证器是在应用程序生命周期的早期创建的, 将@Bean 方法声明为 static 可以创建 Bean, 而无需实例化@configuration 类. 这样做可以避免任何可能由早期实例化引起的问题.
>
> 注意:`spring-boot-actuator` 模块包含一个端点, 该端点暴露所有 `@ConfigurationProperties` bean. 访问 `/actuator/configprops` 可获得相关信息.

# @ConfigurationProperties vs. @Value

`@Value` 注解是一个核心容器特性, 它不提供与 `@ConfigurationProperties` 相同的特性.

! [@ConfigurationProperties vs. @Value](/img/bVgqS "@ConfigurationProperties vs. @Value")

> 如果需要为组件定义了一组配置键, 建议将它们配置到一个 `@ConfigurationProperties` 注解的 POJO 中. 由于 `@Value` 不支持松绑定, 如果需要使用环境变量提供值, 则它不是一个好的选项.
> 虽然可以在 `@Value` 中编写 `SpEL` 表达式, 但此类表达式不会从 properties 文件中处理.

# 使用配置中心

如果项目比较大的话, 分成了好几个 SpringBoot 工程, 可以使用某些 SpringCloud 组件, 比如: 配置中心. 配置中心支持一个地方管理所有的配置, 有些还可以支持修改配置实时生效而不用重启应用, 真的是很棒棒呢. 推荐使用 `nacos`. 如果项目比较小, 你用 `git` 或者 ` 指定文件夹 ` 来作为配置存放的地方也可以.

怎么样? 有了这些用法的支持, 你还会觉得 Springboot 打成一个 `jar` 会在部署的时候很不方便吗?

# 参考资料

[官方文档](https://docs.spring.io/spring-boot/docs/2.2.0.RELEASE/reference/htmlsingle/#boot-features-external-config)
公众号: 逸飞兮 (专注于 Java 领域知识的**深入学习**, 从源码到原理, 系统有序的学习)

! [逸飞兮](/img/bVceG " 逸飞兮 ")

[0](javascript:;) 阅读 2.3k

[0](javascript:;)

[](#)

[举报](#)

##### 推荐阅读

[异常记录——Connection reset](/a/1060000000084336) [走进 JavaWeb 技术世界 7: Tomcat 和其他 WEB 容器的区别](/a/1060000000022252) [Intellij IDEA 基于 Springboot 的远程调试](/a/1060000000020034) [搭建云服务器](/a/1060000000006577) [Java 服务器 -Disruptor 使用注意](/a/1060000000009443) [Linux 系统安装 jdk](/a/1060000000040481)

##### 0 条评论

! [](https://cdn-static.aijishu.com/v-6049f0b3/public/avatar/user.svg)

提交

Loading...

! [...](https://cdn-avatar.aijishu.com/231/243/231243724-5d5f70fa9bf1d_36)

[

##### 逸飞兮

](/u/lw5946)

关注数

500 Error - 极术社区

#### 500

### 当前服务不可用 T^T

We're sorry, the server encountered an internal error and was unable to complete your request.

[联系我们](https://jinshuju.net/f/7NSOGq)