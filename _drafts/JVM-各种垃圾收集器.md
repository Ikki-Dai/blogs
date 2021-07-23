## GC分类和性能指标
  - 吞吐量: 用户代码时间/(用户代码时间 + 垃圾收集时间)
  - 垃圾收集开销
  - 暂停时间: 影响用户体验
  - 收集频率
  - 内存占用
  - 快速：对象诞生到回收时间

现在标准: 在最大吞吐量优先的情况下，降低暂停时间

## 概述
  - 串行式和并行式：
  - 并发式和独占式: 
  - 压缩式和非压缩：压缩式(指针碰撞)
  - 年轻代和老年代：

  - 常见的: CMS, ParallelGC
    - 新生代: Serial, Parnew, Parallel Scavenge
    - 老年代: Serial Old, Parallel Old, CMS
    - 整堆：G1

  - 组合关系: 

|      | 新生代            | 老年代                | 备注                                                         |
| ---- | ----------------- | --------------------- | ------------------------------------------------------------ |
|      | Serial GC         | Serial Old GC         | 单CPU，还保留                                                |
|      |                   | CMS GC                | JDK8 废弃，jdk9 移除                                         |
|      | ParNew GC         | CMS GC                | 保留                                                         |
|      |                   | Serial Old GC         | - Serial Old GC 后备方案， 需要STW<br/>- JDK8废弃，jdk9 移除 |
|      | Parallel Scavenge | Serial Old GC         | JDK14 废弃                                                   |
|      |                   | Parallel Scavenge Old | JDK 8 默认                                                   |
| 重要 | CMS               |                       | JDK14 中彻底删除                                             |
|| G1 || JDK9 默认启用|

```bash
-XX:+PrintCommandLineFlags
jinfo -flag UseG1GC/UseParallelGC <pid>
```


## Serial 回收器：串行回收
  - JDK1.3 之前新生代唯一选择， 客户端模式默认
  - 复制算法，串行回收， STW 机制
  - Serial GC: 年轻代, 标记复制
  - Serial GC Old: 老年代, 标记压缩, 作为老年代 CMS 后备方案
  - `-XX:+UseSerialGC` 年轻代和老年代 同时指定


## ParNew 回收器:并行回收
  - ParNew 是Serial 多线程版本
  - 和 Serial 几乎无区别
  - 复制算法, 并行回收, STW,
  - 老年代配合 Serial Old  和 CMS
  - 单CPU 下，效率不比 Serial 高
  - `-XX:+UseParNewGC` `-XX:ParallelGCThreads` 



## Parallel : 吞吐量优先
  - 适合后台运算，不需要太多交互
  - Parallel Scavenge
    - 复制算法, 并行回收, STW
    - 和ParNew不同: 可控制的吞吐量，
    - 自适应调节策略: 
  - Parallel Old
    - JDK1.6 替代 Serial Old
    - 标记，压缩，并行回收 STW
  - `-XX:+UseParallelGC` `-XX:+UsePrarallelOldGC` JDK8 默认, 可以相互激活
  - `-XX:ParallelGCThreads` =< 8 CPU 个数 >8: 3+[[5* cpu]/8]
  - `-XX:MaxGCPauseMillis` 最大停顿延迟
  - `-XX:GCTimeRatio` 垃圾收集时间占总时间比例 0~99 默认99， 停顿不超过1%
  - `-XX:+UseAdaptiveSizePolicy` 设置Parallel Scavenge 

## CMS : 低延迟
  - `-XX:+UseConcMarkSweepGC`
    - ParNew + CMS + SerialOld 激活

  - 标记压缩, 并发回收, STW, 减少停顿时间
  - 初始标记, 并发标记, 重新标记, 并发清理，重置线程
    - 初始标记 STW : 仅仅标记GC Roots 能直接关联的对象 
    - 并发标记: 从 GC Roots 直接关联对象开始遍历整个对象图, 耗时长，不需要停顿用户线程
    - 重新标记 STW : 修正并发标记期间，因用户程序运作导致标记产生变动的那一部分对象的标记记录， 停顿比初始标记稍长. 比 并发标记时间短 
    - 并发清理： 清理标记阶段判定死亡的对象， 不需要移动对象，可以与用户线程同时并发
  - CMS 堆内存使用率达到一定阈值, 开始进行回收, 内存不够会出现 *Concurrent Mode Failure*, 零时启用 Serial Old
  - 产生内存碎片, 空闲列表(Free List) 执行内存分配, 无法分配大对象
  - CPU 资源敏感: 占用线程, 总吞吐量降低
  
  - CMS 无法处理浮动垃圾: 并发标记阶段产生的垃圾，无法标记，导致无法回收。只能下次GC 释放
  - `-XX:ParallelCMSThreads` (CPU + 3) / 4
  - `-XX:CMSlnitiatingOccupanyFraction` 堆内存使用阈值
    - =< JDK5 68% 频率高
    - => JDK6 92% 频率低, 
  - `-XX:+UseCMSCompactAtFullCollection` 启用要设置下面的参数
  - `-XX:CMSFullGCsBeforeCompaction` 执行多少次full GC 堆内存进行整理，`=3` 3 次GC 后压缩整理
  - 

## G1: 区域化分代式
  - 延迟可控尽可能高吞吐量
  - 并行回收, 划分region, 分代(不要求各分带是连续的)
  - 避免整个java 堆进行全区域的垃圾回收, 根据region 垃圾堆回收价值大小(获得的空间大小和 回收所需时间)
  - 针对多核核大容量
  - `-XX:+UseG1GC` 
  - 空间整合: G1 将内存划分为region, 内存回收以region 为单位. region 之间是复制算法, 整体上看是标记压缩，避免内存碎片。分配大对象不会因为无法找到内存空间 提前触发GC
  - 可预测的停顿模型: 预测停顿时间模型, 选取部分区域进行内存回收, 缩小回收范围。跟踪各region 垃圾堆积价值大小(回收所获得的空间大小以及回收所需时间经验值)，后台维护一个优先列表。 软实时
  - 相较于CMS, G1 不具备全方位，压倒性优势. 产生的内存占用 和运行时的额外负载 都比 CMS 高
  - 小内存 CMS 表现优于 G1, G1 在大内存发挥优势， 平衡点在 6~8G 之间
  - `-XX:G1HeapRegionSize` 每个region大小, 1MB ~ 32MB (2G ~ 64G) 默认是堆的 1/2000, 划分 2048 个 region
  - `-XX:MaxGCPauseMillis` 期望最大停顿时间，默认200ms
  - `-XX:ParallelGCThreads` STW 线程数, 最多8
  - `-XX:ConcGCThreads`: 并发标记线程数. 将n设置为并行垃圾回收线程数(ParallelGCThreads) 1/4 左右
  - `-XX:InitiatingHeapOccupancyPercent`: 设置触发并发GC周期java 堆占用内存阈值。超过触发阈值，默认45%

  - 常见调优, G1 简化调优
    - 开启G1
    - 设置堆最大内存
    - 设置停顿时间
  - 设置 H 区的原因
    - 大对象超过1.5 个region 放到 H区(Humongous)
    - 寻找连续的 H区 来存储
    - G1 大多数时候把H区当作老年代看待
  - region 区在回收后角色可变 (Eden, survivor, old, humongous)
  - Bump-the-pointer: 指针碰撞
  - TLAB : 提高内存分配效率
  - G1 回收
    - 年轻代GC
    - 老年代 并发标记
    - 混合回收: G1 老年代 回收和年轻代一起回收
    - 单线程，独占式，Full GC 还是会存在,  评估失败，强力回收
    - 

### Remembered Set 记忆集

- 一个对象被不同区域引用的问题
- 一个region中的对象可能被其他任意Region 中对象引用
- 使用记忆集避免全堆扫描

- 写屏障, 是在不同的region

### G1 具体过程

- 年轻代GC
  - Eden 空间耗尽，启动年轻代垃圾回收 Eden 和 Survivor
  - 扫描根 -> 更新RSet -> 处理RSet -> 复制对象 -> 处理引用
- 并发标记
  - 初始标记 -> 根区域扫描 -> 并发标记 -> 再次标记(SATB) -> 独占清理(STW) -> 并发清理
- 混合回收
  - `-XX:G1MixedGCCountTarget`
  - ``

- Full GC






## 垃圾回收器总结

## GC日志分析

## 垃圾回收器新发展
