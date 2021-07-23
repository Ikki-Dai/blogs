### 栈上分配内存

#### 产生逃逸


```java
public static StringBuffer createStringBuffer(String s1, String s2) {
   StringBuffer sb = new StringBuffer();
   sb.append(s1);
   sb.appedn(s2);
   return sb;
}
```

#### 优化

```java
public static String createStringBuffer(String s1, String s2) {
   StringBuffer sb = new StringBuffer();
   sb.append(s1);
   sb.appedn(s2);
   return sb.toString();
}
```


###### 产生逃逸

```java
public class EscapeAnalysis {
    
    public Object obj;
    // 方法外使用到， 发生逃逸
    public Object getInstance() {
        return obj==null? new Object(): obj;
    }
    // 如果obj设置为static, 仍然是发生逃逸
    public void setObj() {
        this.obj = new Object();
    }

    // 对象作用域仅在当前方法内，没有发生逃逸
    public void useObj() {
        Object obj = new Object();
    }

    // 发生逃逸, 变量在栈上, 对象在堆上
    public void useObject() {
        Object obj = getInstance():
    }
}

```


`-XX:+DoEscapeAnalysis` 显式开启逃逸分析
`-XX:+PrintEscapeAnalysis` 查看逃逸分析结果

###### 结论
- 能使用局部变量，就不要在方法外定义

- 栈上分配
  - 将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针不会逃逸，对象可能是栈分配，而不是堆分配。减少GC

- 消除同步
  - 如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步

```java
public void f() {
    Object hollis = new Object();
    synchronized(hollis) {
        System.out.println(hollis);
    }
}
```
==> 

```java
public void f() {
    Object hollis = new Object();
    System.out.println(hollis);
}
```
  - 该对象只在方法内部使用, 不会被其他线程访问到， JIT编译会被优化掉
  - 字节码还是会存在`monitorenter` 和 `monitoryexit`, 运行时会去掉

- 分离对象和标量替换
  - 有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分(或全部)可以不存储在内存，而是存储在CPU寄存器中

  **标量(Scalar)**: 无法分解成更小的数据的数据。java中的原始数据类型就是标量
  相对的，还可以分解的数据叫做**聚合量**， 对象就是聚合量，因为可以分解成其他聚合量和标量
 
 ```java
 public void static main(String[] args) {
    alloc();
 }

private static void alloc() {
    Point point = new Point(1,2);
    System.out.println(""+point.x +""+ point.y);
}
```
```java
private static void alloc() {
    int x = 1;
    int y = 2;
    System.out.println(""+point.x +""+ point.y);
}
```
  - 标量替换参数: `-XX:+EliminateAllocations` 默认开启
  - 默认开启了 `-server`, 在server 模式下开启了逃逸分析

- 逃逸分析不成熟
 - 99年论文， jdk1.6 才实现
 - 无法保证逃逸分析的性能消耗一定高于他的消耗
 - 分析之后，发现没有逃逸，分析过程浪费
 - Oracle HotSpot 没有使用 (仅实现了标量替换)
 - 对象实例分配在堆上

