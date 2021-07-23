#

## 底层存储结构
- 1.8 char[] 16bit
- 1.9 byte[]

修改原因: 
1. 堆主要成分
2. latin-1, 一个byte足够
3. + encoding-flag

节约空间

## 不可变性

- 重新赋值时， 重新指定内存区域，不能使用原有value 赋值
- 连接操作时， ~~~
- replace 修改时，~~~

字面量赋值，在常量池中

## 常量池
`-XX:StringTableSize` 设置StringTable  长度 是一个Hashtable

- jdk6 默认长度: 1009
- jdk7 中默认值: 60013; 
- jdk8 1009 是可设置的最小值


## 在内存中的分配
- 字面量会直接存储在常量池中
- intern() 分配在常量池

- jdk6: 常量池放在永久代
- jdk7: 堆
  - permSize 小，容易OOM
  - 永久代垃圾回收频率低

## 拼接 “+”

- 常量拼接，放在常量池， 编译期优化
- 只要有一个变量，放在堆中
- "+" 底层使用StringBuilder  == false
- final 修饰的String 就是常量(字面量), 没有使用StringBuilder 拼接 == true

- StringBuilder 效率高
- 只有一个StringBuilder 对象， 没有大量的StringBuilder 和 String 对象， 减少GC
- StringBuilder 预估 容量，减少扩容次数

## native intern()
- 没有则放入常量池，并使用常量池引用
- 等同于字面量赋值
  - `new String("ab")` 创建了 几个对象 : 2个:  常量池一个指令：ldc, new 关键字在堆空间创建的
  - `new String("a") + new String(""b)` :
    - StringBuilder： 1 个
    - new String : 2 个
    - 字面量 a和 b : 2个

```java
String s = new String("1")
s.intern(); //调用之前，常量池已存在1
String s2 = "1";
System.out.println(s == s2); // false
```

```java
String s3 = new String("1") + new String("1");// s3 = new String("11")， 没有进入常量池;
s3.intern(); // 在常量池 生成 "11": jdk6: 创建新对象, jdk7： 常量池没有创建，而是创建引用
String s4 = "11";
System.out.println(s3 == s4); //jdk6: false jdk7: true
```

```java
String s3 = new String("1") + new String("1");// s3 = new String("11")， 没有进入常量池;
String s4 = "11";// 在常量池 生成 "11"
s3.intern(); // 在常量池 生成 "11": jdk6: 创建新对象, jdk7： 常量池没有创建，而是创建引用
System.out.println(s3 == s4); //jdk6: false jdk7: false
```

jdk6 vs jdk7/8
- jdk6: 如果没有, 放入常量池，并返回串池对象地址
- jdk7/8: 如果没有，复制引用地址，放出串池，并返回串池引用地址

`-XX:+PrintStringTableStatistics` 统计StringTable 常量池信息

## G1 对StringTable 的优化
- openjdk.java.net/jeps/192
- 去重操作: 去除堆中的String
  - 堆中数据占了25%
  - 堆存活数据集合里 重复的有13.5%
  - 平均长度 45

- `UseStringDeduplicaiton (bool)` `PrintStringDeduplicationStatistics (bool)` `StringDeduplicationAgeThreshold (uintx)`



