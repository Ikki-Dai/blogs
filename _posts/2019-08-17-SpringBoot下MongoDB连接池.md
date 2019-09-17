# SpringBoot下MongoDB 连接池设置

## 常见设置

​		通常情况下SpringBoot 使用MongoDB 是这么设置的

```properties
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=test
spring.data.mongodb.username=username
spring.data.mongodb.password=password
```

## 如何使用连接池？

​		同使用mysql 数据库配置连接池一样，如何设置MongoDB的连接池

- 我百度到的方法基本上是 自定义配置，手动实现一个Pool，去实现的，实现过程较为繁琐

## 正确的打开姿势

### 先看看[mongoDB官方文档](https://docs.mongodb.com/manual/reference/connection-string/#connection-pool-options)是怎么说的

在连接的URI后面有 Options 参数，其中包括Connection Pool Options

| Connection Option | Description |
| ----------------- | --------------- |
| maxPoolSize       | The default value is 100 |
| minPoolSize       | The default value is 0 |
| maxIdleTimeMS     | This option is not supported by all drivers. |
| waitQueueMultiple | This option is not supported by all drivers. |
| waitQueueTimeoutMS | This option is not supported by all drivers. |

所以， 在使用 MongoDB 官方 Driver情况下，驱动已经实现了 一个最大100 的连接池

而 Spring Data MongoDB 底层依赖的也正是 MongoDB 的官方驱动 !

那么我们应该怎么手动设置这个连接池的大小呢

```properties
spring.data.mongodb.uri=mongodb://username:password@IP1:port1,IP2:port2/
test?maxPoolSize=20&minPoolSize=5
```

这样就是轻松实现了一个最小连接数为5，最大连接数为10 的连接池

