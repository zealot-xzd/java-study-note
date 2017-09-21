# 安装MAT内存分析工具，是eclipse的一个插件 
# 获取堆转储文件 
- 增加JVM参数，-XX:+HeapDumpOnOutOfMemoryError，JVM 就会在发生内存泄露时抓拍下当时的内存状态，也就是我们想要的堆转储文件
- JMap -dump:format=b,file=dumpfile pid
# shallow haep vs. retained heap
- shallow heap: 
    比较好理解（好理解不代表好计算），直译就是浅层堆，其实就是这个对象实际占用的堆大小
    shallow heap = 类定义引用大小 + 父类的成员变量所占的空间 + 当前类的成员变量所占的空间 + 结构体对齐
其中类定义引用的大小是固定（注意不同架构下这个定义是不一样的），在32位架构下为4个字节。对象引用变量也是4个字节（也是跟体系架构有关），最后结构体对齐是为了对齐 8字节的倍数。

举个栗子：
```java
class A {
     int a;
     byte b;
     Integer c = new Integer(1);
}
class B extends A {
     Date d;
     String e;
     List f;
}

如果有对象：


A a = new A();
B b = new B();
```
下面来计算a和b的shallow heap。
a的shallow heap = 8个字节类定义引用 + 4个字节的int成员 + 1个字节的byte成员 + 4个字节的Integer引用成员 + 7个字节的结构体对齐 = 24Byte
（注意这里不用管 new Integer(1), 这是A类引用了Integer类，这份空间会计算在他的retained heap里面。）

b的shallow heap = 8个字节的类定义引用 + 9 个字节的父类成员空间 + 4个字节的Date引用 + 4个字节的String 引用 + 4 个字节的 List 引用 + 3个字节的结构体对齐 = 32B

一个对象的的shallow heap只需要对齐一次，所以这里不需要在父类对齐一次，然后在子类也再对齐一次。
上面之所以要先定义好是在32bit x86架构下，是因为不同架构确实有不同的定义：
64bit JVM的 类定义引用字节是 12Byte
通过设置JVM flag参数和heap的大小，对象引用的大小也可以从4字节变成8字节
另外还有一些架构每个对象都有自己的对齐规则的。 

- retained heap:比较难理解，直译过来是保留堆，一般会大于或者等于shallow heap，那么retained heap如何理解呢？ 
    其实最简单的理解就是，如果这个对象被删除了（GC回收掉），能节省出多少内存，这个值就是所谓的retained heap。而GC算法中，是否回收一个对象，主要是判断一个对象是否存在引用（还有一些系统级别或特定对象不在此列），至于标记还是引用计数算法，最终都是为了判断是否被引用。简单理解，如果一个对象没有被引用了，就可以回收了。 

