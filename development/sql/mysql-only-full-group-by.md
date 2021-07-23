---
title: mysql-only-full-group-by
date: 2021-03-07 14:00:00
tags: 'SQL'
categories:
  - ['开发', 'SQL']
permalink: mysql-only-full-group-by
---

https://www.cnblogs.com/Wayou/p/mysql_group_by_issue.html

# [MySQL GROUP BY 的问题](https://www.cnblogs.com/Wayou/p/mysql_group_by_issue.html)

<table class="d-block"> <tbody class="d-block"> <tr class="d-block"> <td class="d-block comment-body markdown-body  js-comment-body"> <p> 拿 <code> employee</code> 示例数据库为例, 当进行如下操作时会报错.</p> <div class="highlight highlight-source-shell"> <pre> mysql<span class="pl-k">> </span> SELECT <span class="pl-k">*</span> FROM   employees GROUP  BY gender<span class="pl-k">; </span>
ERROR 1055 (42000): Expression <span class="pl-c"> <span class="pl-c">#</span>1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'employees.employees.emp_no' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode= only_full_group_by</span> </pre> </div> <p> 其中 <code> employee</code> 的表结构为: </p> <div class="highlight highlight-source-shell"> <pre> mysql<span class="pl-k">> </span> DESCRIBE employees<span class="pl-k">; </span>
+------------+---------------+------+-----+---------+-------+
<span class="pl-k"> |</span> Field      <span class="pl-k"> |</span> Type          <span class="pl-k"> |</span> Null <span class="pl-k"> |</span> Key <span class="pl-k"> |</span> Default <span class="pl-k"> |</span> Extra <span class="pl-k"> |</span>
+------------+---------------+------+-----+---------+-------+
<span class="pl-k"> |</span> emp_no     <span class="pl-k"> |</span> int(11)       <span class="pl-k"> |</span> NO   <span class="pl-k"> |</span> PRI <span class="pl-k"> |</span> NULL    <span class="pl-k"> |</span>       <span class="pl-k"> |</span>
<span class="pl-k"> |</span> birth_date <span class="pl-k"> |</span> date          <span class="pl-k"> |</span> NO   <span class="pl-k"> |</span>     <span class="pl-k"> |</span> NULL    <span class="pl-k"> |</span>       <span class="pl-k"> |</span>
<span class="pl-k"> |</span> first_name <span class="pl-k"> |</span> varchar(14)   <span class="pl-k"> |</span> NO   <span class="pl-k"> |</span>     <span class="pl-k"> |</span> NULL    <span class="pl-k"> |</span>       <span class="pl-k"> |</span>
<span class="pl-k"> |</span> last_name  <span class="pl-k"> |</span> varchar(16)   <span class="pl-k"> |</span> NO   <span class="pl-k"> |</span>     <span class="pl-k"> |</span> NULL    <span class="pl-k"> |</span>       <span class="pl-k"> |</span>
<span class="pl-k"> |</span> gender     <span class="pl-k"> |</span> enum(<span class="pl-s"> <span class="pl-pds">'</span> M<span class="pl-pds">'</span> </span>, <span class="pl-s"> <span class="pl-pds">'</span> F<span class="pl-pds">'</span> </span>) <span class="pl-k"> |</span> NO   <span class="pl-k"> |</span>     <span class="pl-k"> |</span> NULL    <span class="pl-k"> |</span>       <span class="pl-k"> |</span>
<span class="pl-k"> |</span> hire_date  <span class="pl-k"> |</span> date          <span class="pl-k"> |</span> NO   <span class="pl-k"> |</span>     <span class="pl-k"> |</span> NULL    <span class="pl-k"> |</span>       <span class="pl-k"> |</span>
+------------+---------------+------+-----+---------+-------+
6 rows <span class="pl-k"> in</span> <span class="pl-c1"> set</span> (0.01 sec) </pre> </div> <h2> 原因 </h2> <p> SQL 标准中不允许 SELECT 列表, HAVING 条件语句, 或 ORDER BY 语句中出现 GROUP BY 中未列表的可聚合列. 而 MySQL 中有一个状态 <a href="https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by" rel="nofollow"> ONLY_FULL_GROUP_BY</a> 来标识是否遵从这一标准, 默认为开启状态.</p> <p> 所以这样的语句是不可以的, </p> <div class="highlight highlight-source-sql"> <pre> <span class="pl-c"> <span class="pl-c">#</span> 🚨</span>
<span class="pl-k"> SELECT</span> gender,
       last_name
<span class="pl-k"> FROM</span>   employees
GROUP  BY gender </pre> </div> <p> 将 <code> last_name</code> 从 SELECT 中移除或将其添加到 GROUP BY 中都可以修复.</p> <div class="highlight highlight-source-shell"> <pre> <span class="pl-c"> <span class="pl-c">#</span> ✅</span>
SELECT gender,
FROM   employees
GROUP  BY gender
<p> <span class="pl-c"> <span class="pl-c">#</span> ✅</span> <br>
SELECT gender, <br>
last_name<br>
FROM   employees<br>
GROUP  BY gender, <br>
last_name  </p> </pre> </div> <p> </p> <p> 但这样的修改查询出来就可能就不是想要的结果了.</p> <h2> 解决 </h2> <p> 三种方式来解决.</p> <h3> 关闭 ONLY_FULL_GROUP_BY</h3> <p> 可以选择关掉 MySQL 的 <a href="https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by" rel="nofollow"> ONLY_FULL_GROUP_BY</a> 模式.</p> <p> 有两种方式, 通过昨晚设置 <code> sql_mode</code> 来关闭.</p> <p> 首先查看变更前的 <code> sql_mode</code>: </p> <div class="highlight highlight-source-shell"> <pre> mysql<span class="pl-k">> </span> SELECT @@sql_mode<span class="pl-k">; </span>
+-----------------------------------------------------------------------------------------------------------------------+
<span class="pl-k"> |</span> @@sql_mode                                                                                                            <span class="pl-k"> |</span>
+-----------------------------------------------------------------------------------------------------------------------+
<span class="pl-k"> |</span> ONLY_FULL_GROUP_BY, STRICT_TRANS_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_ENGINE_SUBSTITUTION <span class="pl-k"> |</span>
+-----------------------------------------------------------------------------------------------------------------------+
1 row <span class="pl-k"> in</span> <span class="pl-c1"> set</span> (0.00 sec) </pre> </div> <p> 通过以下脚本关闭 : </p> <div class="highlight highlight-source-sql"> <pre> <span class="pl-k"> SET</span> SESSION sql_mode<span class="pl-k">= </span> (<span class="pl-k"> SELECT</span> REPLACE(@@sql_mode, <span class="pl-s"> <span class="pl-pds">'</span> ONLY_FULL_GROUP_BY, <span class="pl-pds">'</span> </span>, <span class="pl-s"> <span class="pl-pds">'</span> <span class="pl-pds">'</span> </span>)); </pre> </div> <p> 再次查询 <code>@@sql_mode</code> 返回中应该已经没有该模式了.</p> <div class="highlight highlight-source-shell"> <pre> mysql<span class="pl-k">> </span> SELECT @@sql_mode<span class="pl-k">; </span>
+----------------------------------------------------------------------------------------------------+
<span class="pl-k"> |</span> @@sql_mode                                                                                         <span class="pl-k"> |</span>
+----------------------------------------------------------------------------------------------------+
<span class="pl-k"> |</span> STRICT_TRANS_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_ENGINE_SUBSTITUTION <span class="pl-k"> |</span>
+----------------------------------------------------------------------------------------------------+
1 row <span class="pl-k"> in</span> <span class="pl-c1"> set</span> (0.00 sec) </pre> </div> <p> 第二种是找到 MySQL 配置文件修改并保存.</p> <p> MySQL 的配置文件名为 <code> my.cnf</code>, 可通过以下命令查看你位置: </p> <div class="highlight highlight-source-shell"> <pre>$ mysql --help <span class="pl-k"> |</span> grep cnf
                      order of preference, my.cnf, <span class="pl-smi">$MYSQL_TCP_PORT</span>,
/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf <span class="pl-k">~</span>/.my.cnf</pre> </div> <p> 找到后编辑并保存, 重启 MySQL 后生效.</p> <div class="highlight highlight-source-diff"> <pre> [mysqld]
<span class="pl-md"> <span class="pl-md">-</span> sql_mode= ONLY_FULL_GROUP_BY, STRICT_TRANS_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_ENGINE_SUBSTITUTION</span>
<span class="pl-mi1"> <span class="pl-mi1">+ </span> sql_mode= STRICT_TRANS_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_ENGINE_SUBSTITUTION</span> </pre> </div> <p> 如果文件中没有 <code> sql_mode</code> 配置项可手动添加上.</p> <p> 因为 <code> ONLY_FULL_GROUP_BY</code> 更加符合 SQL 标准, 所以不建议关掉.</p> <h3> ANY_VALUE() </h3> <p> 还可以通过 <a href="https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_any-value" rel="nofollow"> ANY_VALUE() </a> 来改造查询语句以避免报错.</p> <p> 使用 <code> ANY_VALUE() </code> 包裹的值不会被检查, 跳过该错误. 所以这样是可以的: </p> <div class="highlight highlight-source-diff"> <pre> SELECT gender,
<span class="pl-md"> <span class="pl-md">-</span>       last_name</span>
<span class="pl-mi1"> <span class="pl-mi1">+ </span>       ANY_VALUE(last_name) </span>
FROM   employees
GROUP  BY gender </pre> </div> <h3> 添加列间的依赖 </h3> <p> 像这个示例中, </p> <div class="highlight highlight-source-sql"> <pre> <span class="pl-c"> <span class="pl-c">#</span> 🚨</span>
<span class="pl-k"> SELECT</span> gender,
       last_name
<span class="pl-k"> FROM</span>   employees
GROUP  BY gender </pre> </div> <p> 假如我们让 <code> gender</code> 变成不重复的主键, <code> last_name</code> 便与 <code> gender</code> 产生了一种关系, 即 <code> gender</code> 可唯一确定 <code> last_name</code>. 此时便可进行 <code> GROUP BY</code> 了. 因为, 之所以报错是因为在进行聚合的时候有不能确定的列参与了进来.</p> <h2> 总结 </h2> <p> 一般 GROUP BY 会与另外的聚合函数配合使用, 比如 COUNT(), SUM() 等. 查询所有列无差别地进行 GROUP BY 的情况并不是正常的使用姿势.</p> <h2> 相关资源 </h2> <ul> <li> <a href="https://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html" rel="nofollow">12.20.3 MySQL Handling of GROUP BY</a> </li> <li> <a href="https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by" rel="nofollow"> ONLY_FULL_GROUP_BY</a> </li> <li> <a href="https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html" rel="nofollow">12.21 Miscellaneous Functions</a> </li> <li> <a href="https://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html" rel="nofollow">12.20.1 Aggregate (GROUP BY) Function Descriptions</a> </li> </ul> </td> </tr> </tbody> </table>

posted @ 2019-05-16 00:59  [刘哇勇](https://www.cnblogs.com/Wayou/)  阅读 (9892)  评论 (1)  [编辑](https://i.cnblogs.com/EditPosts.aspx? postid=10873151)  [收藏](javascript: void(0))
