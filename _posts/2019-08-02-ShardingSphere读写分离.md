# 基于shardingsphere 读写分离方案

## shardingsphere 

  ShardingSphere 源于 Sharding-JDBC, 是当当网自研的 java 框架, 用于实现读写分离, 分布式事务, 分库分表等更高级的功能。

  ShardingSphere 现已进入Apache 孵化器, 从 `4.0.0` 版本开始, 由Apache 发布

## 数据库架构

  主流的 数据库架构方案中, 要实现 读写分离 或 分库分表的功能, 有 2 种方案:

  - 在应用层实现

     优点是: 依赖应用层的数据库驱动, 可以实现对任意数据库的操作; 没有额外的网络性能消耗
     缺点是: 对代码有一定的侵入性; 当部署多个应用实例时, 数据库连接数会成倍增加

  - 在代理层实现

     优点是: 对应用层没有侵入, 无需代码变动, 支持异构语言; 需要的连接数不会增加
     缺点是: 支持的数据单一, 有额外的网络性能损耗。


## 基于 Spring Boot Starter 的读写方案的分离


### 引入jar包
```xml

    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
        <version>4.0.0-RC1</version>
    </dependency>

    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-namespace</artifactId>
        <version>4.0.0-RC1</version>
    </dependency>

```

### 配置 properties

```properties

spring.shardingsphere.datasource.names=master,slave

#使用 Spring Boot 默认连接池
spring.shardingsphere.datasource.master.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.master.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.master.jdbc-url=jdbc:mysql://localhost:3306/sharding_test?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=false&useAffectedRows=true
#配置用户名和密码
spring.shardingsphere.datasource.master.username=username
spring.shardingsphere.datasource.master.password=password
spring.shardingsphere.datasource.master.pool-name=master-write

spring.shardingsphere.datasource.slave.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.slave.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.slave.jdbc-url=jdbc:mysql://localhost:3306/sharding_test?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true&useSSL=false&useAffectedRows=true
spring.shardingsphere.datasource.slave.username=username
spring.shardingsphere.datasource.slave.password=password
spring.shardingsphere.datasource.slave.pool-name=slave-read

spring.shardingsphere.masterslave.name=ms
#配置读库和写库
spring.shardingsphere.masterslave.master-data-source-name=master
spring.shardingsphere.masterslave.slave-data-source-names=slave

```
- 这里有一点要注意: datasource 名称比如 `master`,`slave` 后面的 配置项的key 根据所使用的 连接池 来配置
比如 配置 jdbc 地址, 在 hikari 中叫 `jdbc-url`, 在 dbcp 里叫 `url`


