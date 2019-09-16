# Disruptor CPU 问题

## 起因

在某项目中使用了 disruptor 框架, maven如下
```xml
    <dependency>
        <groupId>com.lmax</groupId>
        <artifactId>disruptor</artifactId>
        <version>3.4.2</version>
    </dependency>
```

在初始化RingBuffer 时，代码如下
```java
disruptor = new Disruptor<>(eventFactory, ringBufferSize, (ThreadFactory) threadPoolExecutor, ProducerType.MULTI, new SleepingWaitStrategy())
```


在开发环境下, 没有发现任何问题 

在部署到测试环境Linux后, CPU 飙升到200% 

## 排查
- 使用 ps -mp 查看jvm 进程下线程占用CPU 时间排序，找出线程号

```shell
# ps -mp 521 -o THREAD,tid,time | sort -rn | more

PID USER PRI ... TIME+ COMMAND
253 root  20   1:12.24    java
254 root  20   1:11.52    java
252 root  20   1:13.01    java
255 root  20   1:12.48    java
```

- 找出对应的线程 16 进制线程号

```shell
# printf "%x\n" 252
fc
```

- 使用jstack 打印线程信息

```shell
# jstack -l 521 | grep 0xfc -A 30
```

确定线程名是 `TASK-1`, 可以确认是 threadPoolExecutor 池中的线程

这个线程池只配给了 Disruptor 框架使用，所以基本确定是 Disruptor 的问题

- 查看disruptor 使用方式, 初步怀疑是 `WaitStrategy` 使用的问题, 将 `SleepingWaitStrategy` 修改为 `BlockWaitStrategy`, 果然 CPU 使用率降低了


## 结果
- 查看源码, SleepingWaitStrategy

```java
    private int applyWaitMethod(final SequenceBarrier barrier, int counter)
        throws AlertException {
        barrier.checkAlert();
        if (counter > 100) {
            --counter;
        } else if (counter > 0) {
            --counter;
            Thread.yield();
        }
        else {
            LockSupport.parkNanos(sleepTimeNs);
        }

        return counter;
    }

```
- 可以看出 `LockSupport.parkNanos(sleepTimeNs)` 使用了 `sun.misc.Unsafe` 的 `park` 方法。
 Unsafe 类中是一些不安全的底层方法, 包括内存的操作等, 甚至常见的`AtomicInteger` 也是调用其中的 `compareAndSwapInt` 实现的。 
 `park` 方法是一个系统层级的精确时间统计, 在不同平台上会有不同的效果。

- 其中`SleepingWaitStrategy()`默认sleep 时间是 200 纳秒， 根据官方回复：
在新一些的 linux 内核上, 如果sleep 时间太短，系统可能不会真正的park 一个线程, 所以这个是导致CPU 狂飙的根本原因

- 修复方案, 在使用SleepWaitStrategy 时，自定义sleep 时间 为 200 微秒
```java
new SleepingWaitStrategy(200, 200_000L);
```

- 重新部署后，CPU 使用率在 5% 左右


