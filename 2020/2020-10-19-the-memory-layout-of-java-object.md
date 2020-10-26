Java对象所占的内存空间
---
一篇引经据典的考究文

## 8种基本类型
|Primitive Type| Memory(Byte)|
|----|----|
| boolean| 1 |
| byte | 1 |
| short | 2 |
| char | 2 |
| int | 4 |
| float | 4 |
| long | 8 |
| double | 8 |
| oops(ordinary object pointers) | 4(32,64+UseCompressedOops) / 8(64) |

## 堆栈关系

最基本的指针关系，

`String a = new("sssss");`
```
|  main() stack |                | heap |
╔═══════════════╗ class pointer
║   String a    ║   -------->     "sssss"
╚═══════════════╝
```


## Java Object Layout JVM工具
官方的查看Java对象内存结构的工具，不过内存结构具体取决于JVM的实现。以下以`64位 Hotspot JVM`为例。
### Maven依赖
```xml
<dependency>
  <groupId>org.openjdk.jol</groupId>
  <artifactId>jol-core</artifactId>
  <version>0.14</version>
</dependency>
```

### 使用

```java
public class JOLtest {
    static class A {
        char a;
        String b;
        String[] c = {"你", "a"};
    }

    public static void main(String[] args) {
        A a = new A();

        System.out.println(VM.current().details());
        System.out.println(ClassLayout.parseClass(A.class).toPrintable());
        System.out.println(ClassLayout.parseInstance(a.c[1]));
    }
}
```

`VM.current()`会输出以下信息。
```shell
# Running 64-bit HotSpot VM.
# Using compressed oop with 3-bit shift.
# Using compressed klass with 3-bit shift.
# Objects are 8 bytes aligned.
# Field sizes by type: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
# Array element sizes: 4, 1, 1, 2, 2, 4, 4, 8, 8 [bytes]
```
可以看到当前运行的是64位HotSpot VM, 启用了指针压缩，对象以8字节对齐。后面两行是引用类型`oops(ordinary object pointers)` + 8种基本数据类型的大小（byte）。(开启了指针压缩(compressed oops) oops只占4字节，否则占8字节)

然后`ClassLayout`会打印类和对象的内存layout，如下
```shell
com.effective.memory.JOLtest$A object internals:
 OFFSET  SIZE                 TYPE   DESCRIPTION                             VALUE
      0    12                        (object header)                         N/A
     12     2                 char   A.a                                     N/A
     14     2                        (alignment/padding gap)                 
     16     4     java.lang.String   A.b                                     N/A
     20     4   java.lang.String[]   A.c                                     N/A
Instance size: 24 bytes
Space losses: 2 bytes internal + 0 bytes external = 2 bytes total

java.lang.String.value @12 (byte[], 4b)
java.lang.String.hash @16 (int, 4b)
java.lang.String.coder @20 (byte, 1b)
size = 24
```
可以看到实例a因为字节对齐损失2字节，其中`object header`是每个java对象都有的对象头。`(alignment/padding gap) `是指为了实现字节对齐的填充。

注意在类中的`A.c`是指空对象，只有一个`klass pointer`，所以只占4字节。
而第二段`parseInstance(A.c[1])`是指实例化后的`A.c[1]`，一共24字节，12字节的`object header`，4字节的值：2个char`'a'`,`'\0'`,4字节的hashcode，1字节记录编码方式coder。

## 字节对齐

**64位计算机一次处理的数据是64位(即字长，计算机进行数据处理时，一次存取、加工和传送的数据长度称为字`word`)，所以为了配合CPU的效率，内存按8字节使用最高效。当然也会有浪费空间(Padding)，这是一种取舍。**

[Memory access granularity](https://developer.ibm.com/technologies/systems/articles/pa-dalign/)


此时内存结构如下，`A.a`占2字节，可以补在`class pointer`后面.而``占4字节，为保证内存对齐补不进去了。所以只能放在下一个字。
```
0              ------------->           7
╔═══════════════════════════════════════╗
║               Mark word               ║
╠═══════════════════════════════════════╣
║ klass pointer ║  A.a (char) ║ Pad 2   ║
╠═══════════════════════════════════════╣
║ A.b(String) ║ A.c(String) ║           ║
╚═══════════════════════════════════════╝
16                          20
```

显然怎么保证字节对齐的同时充分利用空间跟*对象存放的顺序*有关，使用JVM指令`-XX:FieldsAllocationStyle`可以改变排序方式。为1时：先放入基本变量类型（顺序：longs/doubles、ints、shorts/chars、bytes/booleans），然后放入oops（普通对象引用指针），JDK 8默认为1。

## 关于对象头：

`Object header`一共12字节，包括一个`word` size（8 byte）的`mark word`和一个`klass pointer`,也是`word` size，但如果JVM开启了指针压缩(compressed oops)则只有4字节。所以一共12字节。

如果是对象是数组，则对象头还会多一个4byte 的`length`。

有段[Commented code](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/markOop.hpp)解释得很好：

`/* HotSpot居然是C++写的，很烦我很讨厌C++ */`
```cpp
// Bit-format of an object header (most significant first, big endian layout below):
//
//  64 bits:
//  --------
//  unused:25 hash:31 -->| unused:1   age:4    biased_lock:1 lock:2 (normal object)
//  JavaThread*:54 epoch:2 unused:1   age:4    biased_lock:1 lock:2 (biased object)
//  PromotedObject*:61 --------------------->| promo_bits:3 ----->| (CMS promoted object)
//  size:64 ----------------------------------------------------->| (CMS free block)
//
//  unused:25 hash:31 -->| cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && normal object)
//  JavaThread*:54 epoch:2 cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && biased object)
//  narrowOop:32 unused:24 cms_free:1 unused:4 promo_bits:3 ----->| (COOPs && CMS promoted object)
//  unused:21 size:35 -->| cms_free:1 unused:7 ------------------>| (COOPs && CMS free block)
```

* `biased_lock`：对象是否启用`偏向锁`标记，1 bit。为1时表示对象启用偏向锁，为0时表示对象没有偏向锁。

* `age`：4 bit的Java对象年龄。在GC中，如果对象在`Survivor`区复制一次，年龄增加1。当对象达到设定的阈值时，将会晋升到老年代。默认情况下，并行GC的年龄阈值为15，并发GC的年龄阈值为6。由于age只有4 bit，所以最大值为15，这就是`-XX:MaxTenuringThreshold`选项最大值为15的原因。

* `identity_hashcode`：31 bit的对象标识Hash码，采用延迟加载技术。调用方法`System.identityHashCode()`计算，并会将结果写到该对象头中。当对象被锁定时，该值会移动到管程Monitor中。在对象调用`hashCode()`前，整个31bit为0，调用`hashCode()`后会写入mark word。

* `thread`：持有偏向锁的线程ID。

* `epoch`：偏向时间戳。

* `ptr_to_lock_record`：指向栈中锁记录的指针。

* `ptr_to_heavyweight_monitor`：指向管程Monitor (`Sychronized`) 即重型锁的指针。

更多并发相关的资料： https://wiki.openjdk.java.net/display/HotSpot/Synchronization

## 继承

* 在继承关系C->B->A中，父类的对象必然排在子类的对象之前,无论是否有指定排序顺序都不会利用padding的空间。

* `Hotspot JVM`会通过Padding的的方式将每个类自身定义的实例域总空间填充为引用大小`(4 Bytes/8 Bytes)`的整数倍

```java
    public static class A {
        char a;
    }
    public static class B extends A {
        char b;
    }
    public static class C extends B {
        char c;
    }
```

如图，即使重排序可以节省8 byte，也不会填充。
```
╔═══════════════════════════════════════╗
║               Mark word               ║
╠═══════════════════════════════════════╣
║   klass pointer   ║ A.a ║ Padding 2   ║
╠═══════════════════════════════════════╣
║ B.b ║ Padding 2   ║ C.c ║ Padding 2   ║
╚═══════════════════════════════════════╝
```

## Class 元数据

元数据就是类`Class`相关的信息。

`System.out.println(ClassLayout.parseClass(Class.class).toPrintable());`

```
java.lang.Class object internals:
 OFFSET  SIZE                                              TYPE DESCRIPTION                               VALUE
      0    12                                                   (object header)                           N/A
     12     4                     java.lang.reflect.Constructor Class.cachedConstructor                   N/A
     16     4                                   java.lang.Class Class.newInstanceCallerCache              N/A
     20     4                                  java.lang.String Class.name                                N/A
     24     4                                                   (alignment/padding gap)
     28     4                       java.lang.ref.SoftReference Class.reflectionData                      N/A
     32     4   sun.reflect.generics.repository.ClassRepository Class.genericInfo                         N/A
     36     4                                java.lang.Object[] Class.enumConstants                       N/A
     40     4                                     java.util.Map Class.enumConstantDirectory               N/A
     44     4                    java.lang.Class.AnnotationData Class.annotationData                      N/A
     48     4             sun.reflect.annotation.AnnotationType Class.annotationType                      N/A
     52     4                java.lang.ClassValue.ClassValueMap Class.classValueMap                       N/A
     56    32                                                   (alignment/padding gap)
     88     4                                               int Class.classRedefinedCount                 N/A
     92     4                                                   (loss due to the next object alignment)
Instance size: 96 bytes
```
在56的位置开始有32 Bytes的的内存空间，JVM会向该内存区域注入元数据（meta-information）
[http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/tip/src/share/vm/classfile/javaClasses.cpp]

参考文章:[http://zhongmingmao.me/2016/07/01/jvm-jol-tutorial-1/]

## 伪共享

首先了解一下**cache**：

cache，中译名高速缓冲存储器，其作用是为了更好的利用局部性原理，减少CPU访问主存的次数。简单地说，CPU正在访问的指令和数据，其可能会被以后多次访问到，或者是该指令和数据附近的内存区域，也可能会被多次访问。因此，第一次访问这一块区域时，将其复制到cache中，以后访问该区域的指令或者数据时，就不用再从主存中取出。

今天的CPU不再是按字节访问内存，而是以64字节为单位的块(chunk)拿取，称为一个缓存行(`cache line`)。cache分成多个组，每个组分成多个行（`cache line`），linesize是cache的基本单位.从主存向cache迁移数据都是按照linesize为单位替换的。 

注意多个变量可以放在同一个cache line，所以在多线程情况下，如果需要修改“共享同一个缓存行的变量”，就会无意中影响彼此的性能，这就是伪共享（False Sharing）。比如A，B都在一个cache line， 则修改A的时候，B就不能被访问了，很影响并发效率。

最简单的解决方法仍然是padding，保证一行只有一个变量即可。

Java8引入`@sun.misc.Contended`注解，自动进行缓存行填充,运行时需要设置JVM启动参数`-XX:-RestrictContended`
[https://www.jianshu.com/p/c3c108c3dcfd]