## System.gc()
- 通过System.gc() 或 Runtime.getRuntime().gc() 会显式触发 FullGC
- 免责声明，无法保证堆垃圾收集器的调用 ： 重写finalize() 方法，不是每次调用都会触发GC
- 无需手动触发, 性能基准程序可用考虑使用

## 内存溢出和内存泄漏
- JVM堆内存设置不够
- 创建了大量对象，并且长时间不能被垃圾收集器收集
- 只有对象不会被程序再用到了，但是GC 又不能回收他们的情况，才叫内存泄漏

## StopTheWorld
## 垃圾回收的并行与并发
- 并行: Serial
- 并发: 

## 安全点和安全区域


## 强引用(Strong Reference)
- 代码中最普遍的引用复制 `Object obj = new Object()`
- 照成内存泄漏的主要原因之一

## 软引用(Soft Reference) (缓存)
- 在系统要发生内存溢出之前, 会把这些对象列入回收范围之内进行二次回收
- 回收后还没有足够的内存，OOM

## 弱引用(Weak Reference) (缓存)
- 被弱引用关联的对象只能生存到下一次垃圾收集之前。
- 当垃圾收集器工作时，无论内存是否足够，都会回收
- 发现即回收，被弱引用关联的对象只能生存到下一次垃圾回收发生为止
- 弱引用不需要使用检查引用算法
- WeakHashMap WeakReference

## 虚引用(Phantom Reference)
- 垃圾收集时，收到一个系统通知
- 无法获得虚引用对象实例
- PhantomReference(T, ReferenceQueue)

## 终结器引用
- FinalReference
- 无需编码