---
title: Java后端面试整理
date: 2021-03-04 18:28:05
tags:
---

## 语言

Java通过编译生成.class字节码文件，这种字节码必须用Java解释器执行，因此Java可以说是“编译与解释”共存的语言。

### Java数据类型

#### 基本数据类型（8个）

|数据类型|位大小|最大值|最小值|默认值|
|-|-|-|-|-|
|boolean|1|||false|
|byte|8|2^7-1|-2^7|0|
|short|16|2^15-1|-2^15|0|
|int|32|2^31-1|-2^31|0|
|long|64|2^63-1|-2^63|0|
|float|32|||0.0f|
|double|64|||0.0d|
|char|16|\uffff|\u0000|\u0000|

### Java内存模型

（这里所说的内存模型和Java 内存区域的 Java 堆、栈、方法区不是同一层次内存划分）
Java内存模型规定了所有变量都存储在主内存，每条线程有自己的工作内存。线程的所有操作都必须在工作内存中进行，工作内存之间无法访问。
主内存和工作内存的交互，有8种原子操作：

1. lock(锁定):作用于主内存的变量，把一个变量标识为一条线程独占状态。
2. unlock(解锁):作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的 变量才可以被其他线程锁定。**lock和unlock必须成对出现**
3. read(读取):作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中， 以便随后的 load 动作使用
4. load(载入):作用于工作内存的变量，它把 read 操作从主内存中得到的变量值放入 工作内存的变量副本中。**read和load必须成对出现**
5. use(使用):作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每 当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
6. assign(赋值):作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作 内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。**不允许一个线程丢弃它最近assign的操作，也不允许不assign而直接同步回祝内存**
7. store(存储):作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中， 以便随后的 write 的操作。
8. write(写入):作用于主内存的变量，它把 store 操作从工作内存中一个变量的值传送 到主内存的变量中。**store和write必须成对出现**

### JVM

Java 虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM 有针对不同系统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们都会给出相同的结果。

[JVM白话](https://snailclimb.gitee.io/javaguide/#/docs/java/jvm/[%E5%8A%A0%E9%A4%90]%E5%A4%A7%E7%99%BD%E8%AF%9D%E5%B8%A6%E4%BD%A0%E8%AE%A4%E8%AF%86JVM)

#### Java虚拟机运行时内存

线程共享堆，私有虚拟机栈、本地方法栈、程序计数器。

- 程序计数器
可以理解为执行的行号，例如循环判断时会跳跃。
程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。

- 虚拟机栈
存储局部变量。调用方法时压入栈帧，返回时弹出。栈帧包括了局部变量表、操作数栈、动态链接、方法出口等信息。每个方法被执行时都会创建一个栈帧。

- 本地方法栈
虚拟机使用到的 Native 方法

- 堆
存储对象实例。垃圾回收在此区域进行。

- 方法区
用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

#### HotSpot

- 对象的创建

1. 类加载检查：new一个新对象时，检查这个类名对应的对象，并执行相应的类加载过程。
2. 分配内存：根据类确定所需内存大小，并在堆中为其分配。
3. 初始化零值：内存分配完后，将内存空间都初始化为零值
4. 设置对象头：虚拟机要对对象进行必要的设置，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 这些信息存放在对象头中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。
5. 执行init方法：把对象按照程序员的意愿进行初始化

- 内存分配的两种方式：

1. 指针碰撞：适用于内存比较规整的情况。把用过的内存放到一边，没用过的放另一边，中间是分界指针，只需要指针一直往没用过的地方走即可。
2. 空闲列表：内存不规整。维护一个列表，记录哪些内存块可用，分配的时候找一个足够大的内存块分给对象实例，然后更新列表。

#### 垃圾回收

##### 堆内存分配

堆空间分为新生代和老年代。
大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。
对象优先分配在eden区域。当 eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC。而大对象（字符串，数组）直接进入老年代（避免为大对象分配内存时由于分配担保机制带来的复制而降低效率）
当对象经历了一次Minor GC仍然存活，会被移动到Survivor，并且年龄为1。后续每一次MinorGC未被清理都会年龄+1。年龄到达一定程度，将进入老年代。
>Hotspot 遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了 survivor 区的一半时，取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值

GC常用的是新生代收集（Minor GC / Young GC）和整堆收集 (Full GC)，如果进行新生代收集之前发现统计平均晋升大小大于老年代剩余空间，则直接进行FullGC。

- 对象死亡计算

1. 引用计数法。有一个引用就+1，引用失效就-1，为0的时候证明不可能被使用。简单但是**不使用**，因为难以解决循环引用
2. 可达性分析算法。以“GC Roots”为起点，向下搜索，走过的路为引用链。如果引用链无法到达某个对象，则对象不可用。

- 作为GC Roots的主要对象：

1. 虚拟机栈(栈桢中的本地变量表)中的引用的对象
2. 方法区中的类静态属性引用的对象
3. 方法区中的常量引用的对象
4. 本地方法栈中 JNI(即一般说的 Native 方法)的引用的对象

- 引用：

1. 强引用。最普遍的引用，如果一个对象具有强引用，那就类似于必不可少的生活用品，垃圾回收器绝不会回收它。
2. 软引用。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。
3. 弱引用。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。
4. 虚引用。虚引用在任何时候都可能被垃圾回收。

实际使用中，软引用比弱引用和虚引用多。因为软引用可以加速 JVM 对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出问题。

##### 垃圾收回算法

1. 标记-清除算法：标记出所有不需要回收的对象，然后统一清理所有未被标记的对象。最基础的方法，存在效率和空间问题（产生碎片）
2. 标记-复制算法：内存分两块，一半空，另一半标记完之后，把标记的复制过去，然后清理掉所有使用的空间。
3. 标记-整理算法：针对老年代特点，类似标记-清除，但是不是直接回收，而是存活对象统一向一端移动，然后清理掉边界以外的对象。
4. 分代收集算法（当前使用）：新生代每次收集都会死很多，标记复制的成本很小。老年代存活概率大，使用”标记-清除”或“标记-整理“。

##### 垃圾收集器

1. Serial串型收集器，单线程，最古老，进行垃圾收集工作的时候必须暂停其他所有的工作线程，新生代采用标记-复制算法，老年代采用标记-整理算法。
2. ParNew是Serial的多线程版本，除了多线程进行工作以外都一样。新生代采用标记-复制算法，老年代采用标记-整理算法。
3. Parallel Scavenge。关注吞吐量，几乎和parNew一样。新生代采用标记-复制算法，老年代采用标记-整理算法。JDK8默认收集器。
4. Serial Old。Serial的老年代版本。
5. Parallel Old，老年代版本。使用多线程和“标记-整理”算法。在注重吞吐量以及 CPU 资源的场合，都可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。
6. CMS收集器。一种以获取最短回收停顿时间为目标的收集器。它非常符合在注重用户体验的应用上使用。是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。
**初始标记：** 暂停所有的其他线程，并记录下直接与 root 相连的对象，速度很快 ；
**并发标记：** 同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
**重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
**并发清除：** 开启用户线程，同时 GC 线程开始对未标记的区域做清扫。
缺点：对 CPU 资源敏感；无法处理浮动垃圾；"标记-清除"会有大量空间碎片产生。
7. G1收集器。面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征.整体看是整理算法，局部是复制算法。
8. ZGC收集器。JDK 11中推出的一款低延迟垃圾回收器，适用于大内存低延迟服务的内存管理和回收。

[新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)

### 泛型

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

[JAVA 泛型中的通配符](https://juejin.cn/post/6844903917835419661)

### 字符串

- String、StringBuffer与StringBuilder

String对象不可变，final修饰
StringBuffer线程安全，多线程操作使用
StringBuilder不是线程安全的，单线程操作使用

- ==和 equals 的区别

== 比较的是值，即基础数据类型比较的是具体的值，引用数据类型比较的是地址。
如果要比较引用类型的值，需要用a.equals(b)

### 集合

#### ArrayList

ArrayList是一个动态数组，容量能够自动增长。ArrayList**不是线程安全**的，线程安全的AL可以通过`Collections.synchronizedList(List l)`获取
支持序列化`write/readObj()`、支持下标`get()`访问、支持克隆

ArrayList包含两个私有属性：elementData和size，分别存储当前包含的元素和元素数量。无参数创建时默认大小为10
插入新元素时，size+1，如果size超过当前数组大小，则动态扩容`ensureCapacity`。申请一个新的数组，大小为1.5倍，并将旧的数组复制到新的数组。旧的数组在调用`Arrays.copyOf()`进行复制前，会通过`Object oldData[] = elementData;`进行一次引用，保证copy过程的安全。拷贝使用`System.arraycopy()`方法，底层使用C语言实现，效率更高。

#### LinkedList

LinkedList是一个双向链表，对于插入和删除频繁的list，使用LinkedList会优于ArrayList。LinkedList**不是线程安全**的
LinkedList包括三个成员变量：头结点`first`，尾结点`last`，容量`size`。每一个结点也都是双向结点，包括`prev`和`next`。链表的插入删除时间复杂度都为O(1)。下标查询方式为从头/从尾遍历。占用内存更大。

#### HashMap

[HashMap详解](https://blog.csdn.net/woshimaxiao1/article/details/83661464)

HashMap基于数组+链表实现，解决哈希冲突的方法是链地址法。HashMap不是线程安全的。
HashMap的Entry数据结构包括四个属性：key，value，next（存储下一个entry），hash（key计算出的哈希值）。
哈希表本身还包括几个重要属性：size（实际存储的kv数量），threshold（阈值，默认16），loadFactor（负载因子，默认0.75），modCount（哈希表被改变的次数）

扩容操作：当插入元素后size大于阈值的时候，新建一个大小为双倍的数组，并将目前已有的数据都拷贝过去
插入操作：根据key计算出哈希code：h，然后index = h&(length-1)。因为哈希表的length固定为2的n次幂，因此-1后高位为0，其余为1，只要h对应的最左边的那一个差异位为0，就能保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)
>HashMap 的长度为什么是 2 的幂次方: 取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；）。采用二进制位操作 &，相对于%能够提高运算效率。
查询操作：根据key获取具体的index，找到对应数组下标，判断是否是链表，如果是链表则遍历，直到找到**hash值相同的并且key**相同的entry，然后输出。

JDK1.8对HashMap进行了优化，把链表换成了红黑树，能够保证更快的检索速度。（链表节点大于等于8且数组长度大于等于64）

如果需要线程安全的HashMap，可以使用ConcurrentHashMap（Hashtable也是线程安全的，但是是遗留类，不建议使用）ConcurrentHashMap内部封装了多个HashMap，可以分段加锁而不影响效率。
>不建议使用Hashtable是因为：synchronized会造成线程饥饿和死锁，且进行遍历时其他线程也能对其进行put等操作

重写equals时必须重写hashCode方法。如果两个对象相等，则 hashcode 一定也是相同的。两个对象相等,对两个对象分别调用 equals 方法都返回 true。但是，两个对象有相同的 hashcode 值，它们也不一定是相等的 。因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖。否则，当对象作为Map的键会导致两个equals的对象的hashCode不同而存储为两个键值对

- LinkedHashMap通过双向链表结构保存了插入的顺序

#### 线程安全的容器

- Collections.synchronizedX
Collections.synchronizedX函数返回线程安全的List/Map，底层通过synchronized关键字修饰实现了线程同步，锁属性Mutex在初始化时默认是this
- ConcurrentHashMap
针对HashMap专门提供了一个线程安全的Map，在JDK6的版本它通过segment分段加锁，可以同时操作多个segment，更加高效。扩容时也是对segment进行扩容，而不是整个Map。此外还使用volatile进行修饰共享变量，即使两个线程同时修改和获取volatile变量，get操作也能拿到最新的值。
而在JDK8中，摒弃了segment的理念，转而采用CAS(compareAndSwap)算法。通过volatile修饰tabel数组，tabel数组存储了具体的数据。
CAS的语义是“我认为V的值应该为A，如果是，那么将V的值更新为B，否则不修改并告诉V的值实际为多少”，CAS是靠硬件实现的，当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。是一条CPU的原子指令。

```java
if(v == a) {
  v = b;
} else {
  //什么都不做
}
```

CAS存在三个问题：

1. ABA问题：如果一个元素为A，被修改为B，又被修改为A，CAS无法感知变化。解决方法：加版本号。AtomicStampedReference原子标记类能够保证，但是带来额外开销，使用synchronized可能更好
2. 循环时间长开销大：因为CAS操作是通过死循环执行，如果长时间不成功，CPU开销较大。如果 JVM 能支持处理器提供的 pause 指令那么效率会有一定的提升。
3. 只能保证一个共享变量的原子操作：JDK5之后AtomicReference支持引用对象的原子性，可以把多个变量放在一个对象里进行CAS操作，本质还是加锁volatile

### IO

javaIO框架把不同的输入、输出源抽象称作stream，即“流”。通过抽象，代码能够统一对流进行数据处理，而不必关心具体的输入输出形式。

#### 字节流与字符流

底层设备永远是通过字节流传输。字符流是对字节流的包装，直接接收字符串，方便处理。

#### AIO/BIO/NIO

- BIO（同步阻塞I/O模式）
  数据的读取写入必须阻塞在一个线程内等待其完成。
  直到水烧开以前不做任何别的事。

- NIO（同步非阻塞I/O模型）
  使用一个线程去轮询IO操作，发现改变后及时处理。
  有一个线程单独监控水的状态，一旦烧开则通知

- AIO（异步非阻塞I/O模型）
  异步非阻塞无需一个线程去轮询所有IO操作的状态改变，在相应的状态改变后，系统会通知对应的线程来处理。
  每一个水壶都装了开关，水烧开了开关自动通知
  
### 多线程

[Java并发面试基础](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/2020%E6%9C%80%E6%96%B0Java%E5%B9%B6%E5%8F%91%E5%9F%BA%E7%A1%80%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93)

[Java并发面试进阶](https://snailclimb.gitee.io/javaguide/#/docs/java/multi-thread/2020%E6%9C%80%E6%96%B0Java%E5%B9%B6%E5%8F%91%E8%BF%9B%E9%98%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93)

一个 Java 程序的运行是 main 线程和多个其他线程同时运行。

线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反。

#### 实现多线程的方式

1. 继承Thread类：实现run()方法
2. 实现Runnable接口：实现run()方法
3. 实现Callable接口：实现call()方法。可以有返回值，可以抛异常，需要借助FutureTask类
4. 使用线程池：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁、实现重复利用。

多线程的好处:

1. 提高CPU的利用率。从磁盘上读取文件的时候，大多数的CPU时间都会花费在等待磁盘来读取数据。在这个时候CPU是相当空闲的。在这个时候它可以干点别的事情。通过改变操作的顺序，CPU可以得到更好的利用。
2. 防止阻塞主线程,提高吞吐量。使用线程可以把占据时间长的程序中的任务放到后台去处理
3. 程序的运行效率可能会提高，提升程序的响应速度。

```java
Thread t = new Thread();
t.start()
```

通过extends实现自己的线程，通过`start()`方法启动线程，会执行线程内部实现的`run()`方法
>调用 start() 方法方可启动线程并使线程进入就绪状态，由线程自动执行run()。直接执行 run() 方法的话不会以多线程的方式执行。
线程执行是无序不等待的，可以通过`join()`方法等待线程执行完毕
通过`synchronized`获取锁
通过`this.wait()`等待，必须在`synchronized`内部才能调用
通过`this.notify()`唤醒。

#### synchronized

synchronized 修饰方法（实例锁）、静态方法（类锁）、代码块（指定锁）

- 构造方法不能使用 synchronized 关键字修饰。

[Java 6对synchronized的优化](https://www.cnblogs.com/wuqinglong/p/9945618.html)

#### volatile

[volatile详解](https://www.cnblogs.com/wangwudi/p/12303772.html)
多线程执行时，各个线程会将共享变量从主内存中拷贝到工作内存，然后执行引擎会基于工作内存中的数据进行操作处理。而将工作内存的变量写回主内存的时机则不确定。对于volatile修饰的变量，java虚拟机有特殊的约定：被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象。

volatile针对变量，而Synchronized针对对象和类

#### 乐观锁和悲观锁

乐观锁：读的时候允许写锁后入，导致读的数据可能是不一致的（例如获取锁以后，读取某x的值，但是此时另一个线程获取了写锁，改写了x的值，此时x的值已被更改）
悲观锁：读的过程不允许写，必须等待。
乐观锁的并发效率更高，但是需要有检测机制，检查乐观读锁后是否有新的写锁，如果有，则需要加悲观锁进行读（避免再次被修改）

- 如何避免死锁

1. 破坏互斥条件 ：这个条件我们没有办法破坏，因为我们用锁本来就是想让他们互斥的（临界资源需要互斥访问）。
2. 破坏请求与保持条件 ：一次性申请所有的资源。
3. 破坏不剥夺条件 ：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
4. 破坏循环等待条件 ：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

### 反射

在运行过程中动态获取类的信息和调用对象方法。

优点： 运行期类型的判断，动态加载类，提高代码灵活度。
缺点： 1,性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的 java 代码要慢很多。2,安全问题，让我们可以动态操作改变类的属性同时也增加了类的安全隐患。

反射是框架设计的灵魂。日常开发少用，但是 Spring／Hibernate 等框架大量使用到了反射机制，IOC，AOP，JDBC等都用到

### Stream

Java 8引入Stream对数据进行链式处理。

```java
//10 个随机数进行排序并输出
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);
```

[Stream](https://www.runoob.com/java/java8-streams.html)

### Maven

### Spring

Spring 是一种轻量级开发框架，我们一般说 Spring 框架指的都是 Spring Framework，包括：核心容器、数据访问/集成,、Web、AOP、工具、消息和测试模块

- IoC
  IoC（Inverse of Control:控制反转）是一种设计思想，就是 将原本在程序中手动创建对象的控制权，交由Spring框架来管理。IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。（实际项目一个Service可能依赖多个类，如果每一个构造函数都需要理解则会很复杂。PHP的Laravel框架也是IoC）
- AOP
  AOP将与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

#### Spring bean

Spring bean的作用域：

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。

Spring bean单例存在线程安全问题。多个线程操作同一个对象的时候，对这个对象的成员变量的写操作会存在线程安全问题。但是常用的Controller、Service、Dao这些bean没有状态，不会有影响。如果有需要，可以使用prototype，或是建立一个成员变量ThreadLocal，储存可能会变化的数据。

- @Component 和 @Bean 的区别

1. 作用对象不同: @Component 注解作用于类，而@Bean注解作用于方法。
2. @Component通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 @ComponentScan 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。@Bean 注解通常是我们在标有该注解的方法中定义产生这个 bean,@Bean告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. @Bean 注解比 Component 注解的自定义性更强，而且很多地方我们只能通过 @Bean 注解来注册bean。比如当我们引用第三方库中的类需要装配到 Spring容器时，则只能通过 @Bean来实现。

- 常见的bean注解

1. @Component ：通用的注解，可标注任意类为 Spring 组件。如果一个Bean不知道属于哪个层，可以使用@Component 注解标注。
2. @Repository : 对应持久层即 Dao 层，主要用于数据库相关操作。
3. @Service : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
4. @Controller : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

- bean的生命周期

1. Bean 容器找到配置文件中 Spring Bean 的定义。
2. Bean 容器利用 Java Reflection API 创建一个Bean的实例。
3. 如果涉及到一些属性值 利用 set()方法设置一些属性值。
4. 如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName()方法，传入Bean的名字。
5. 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoader对象的实例。
6. 与上面的类似，如果实现了其他 *.Aware接口，就调用相应的方法。
7. 如果有和加载这个 Bean 的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessBeforeInitialization() 方法
8. 如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。
9. 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
10. 如果有和加载这个 Bean的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessAfterInitialization() 方法
11. 当要销毁 Bean 的时候，如果 Bean 实现了 DisposableBean 接口，执行 destroy() 方法。
12. 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

#### Spring MVC

分层： Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制层，返回数据给前台页面)。

处理流程：
客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。
HandlerAdapter 会根据 Handler 来调用真正的处理器来处理请求，并处理相应的业务逻辑。
处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
ViewResolver 会根据逻辑 View 查找实际的 View。
DispaterServlet 把返回的 Model 传给 View（视图渲染）。
把 View 返回给请求者（浏览器）

[Spring boot 装配](https://www.cnblogs.com/javaguide/p/springboot-auto-config.html)

### MyBatis

[MyBatis 常见面试题总结](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/mybatis/mybatis-interview)

### Netty

### ZooKeeper

ZooKeeper 是一个开源的分布式协调服务.
ZooKeeper 为我们提供了高可用、高性能、稳定的分布式数据一致性解决方案，通常被用于实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

>分布式和集群：简单的加机器是集群，而将服务进行拆分是分布式。分布式需要处理很多因为拆分导致的问题

#### 特点

顺序一致性： 从同一客户端发起的事务请求，最终将会严格地按照顺序被应用到 ZooKeeper 中去。
原子性： 所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，也就是说，要么整个集群中所有的机器都成功应用了某一个事务，要么都没有应用。
单一系统映像 ： 无论客户端连到哪一个 ZooKeeper 服务器上，其看到的服务端数据模型都是一致的。
可靠性： 一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖。

#### 典型应用场景

分布式锁 ： 通过创建唯一节点获得分布式锁，当获得锁的一方执行完相关代码或者是挂掉之后就释放锁。
命名服务 ：可以通过 ZooKeeper 的顺序节点生成全局唯一 ID
数据发布/订阅 ：通过 Watcher 机制 可以很方便地实现数据发布/订阅。当你将数据发布到 ZooKeeper 被监听的节点上，其他机器可通过监听 ZooKeeper 上节点的变化来实现配置的动态更新。

总之，Zookeeper用于分布式保存数据，但是不适合保存大量数据。
>Kafka : ZooKeeper 主要为 Kafka 提供 Broker 和 Topic 的注册以及多个 Partition 的负载均衡等功能。

ZK存储数据的最小单元是znode，每个结点最大1M。

#### Watcher

ZooKeeper 允许用户在指定节点上注册一些 Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去，该机制是 ZooKeeper 实现分布式协调服务的重要特性

#### 集群

ZK不是传统的Master/Slave机制，而是有三个角色：

Leader：为客户端提供读和写的服务，负责投票的发起和决议，更新系统状态。
Follower：为客户端提供读服务，如果是写服务则转发给 Leader。在选举过程中参与投票。
Observer：为客户端提供读服务器，如果是写服务则转发给 Leader。不参与选举过程中的投票，也不参与“过半写成功”策略。在不影响写性能的情况下提升集群的读性能。此角色于 ZooKeeper3.3 系列新增的角色。

Leader的选择包括以下几个阶段：

1. Leader election（选举阶段）：节点在一开始都处于选举阶段，只要有一个节点得到超半数节点的票数，它就可以当选准 leader。
2. Discovery（发现阶段） ：在这个阶段，followers 跟准 leader 进行通信，同步 followers 最近接收的事务提议。
3. Synchronization（同步阶段） :同步阶段主要是利用 leader 前一阶段获得的最新提议历史，同步集群中所有的副本。同步完成之后 准 leader 才会成为真正的 leader。
4. Broadcast（广播阶段） :到了这个阶段，ZooKeeper 集群才能正式对外提供事务服务，并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。

>ZK集群机器最好是单数台。因为剩下的机器数量要大于宕机的数量。例如3台机器，最多宕1台，4台机器也是1台（因为宕机2台后2=2不满足大于），因此没有必要多增加一台机器

#### 总结

1. ZooKeeper 本身就是一个分布式程序（只要半数以上节点存活，ZooKeeper 就能正常服务）。
2. 为了保证高可用，最好是以集群形态来部署 ZooKeeper，这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么 ZooKeeper 本身仍然是可用的。
3. ZooKeeper 将数据保存在内存中，这也就保证了 高吞吐量和低延迟（但是内存限制了能够存储的容量不太大，此限制也是保持 znode 中存储的数据量较小的进一步原因）。
4. ZooKeeper 是高性能的。 在“读”多于“写”的应用程序中尤其地明显，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景。）
5. ZooKeeper 有临时节点的概念。 当创建临时节点的客户端会话一直保持活动，瞬时节点就一直存在。而当会话终结时，瞬时节点被删除。持久节点是指一旦这个 znode 被创建了，除非主动进行 znode 的移除操作，否则这个 znode 将一直保6存在 ZooKeeper 上。
6. ZooKeeper 底层其实只提供了两个功能：① 管理（存储、读取）用户程序提交的数据；② 为用户程序提供数据节点监听服务。

### Dubbo

[SpringBoot+Dubbo搭建](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484809&idx=1&sn=a789eba40404e6501d51b24345b28906&source=41#wechat_redirect)

Dubbo 是一个Java RPC框架，支持：面向接口的远程方法调用，智能容错和负载均衡。
> RPC：远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。

- 优势

1. **负载均衡**——同一个服务部署在不同的机器时该调用那一台机器上的服务。
2. **服务调用链路生成**——随着系统的发展，服务越来越多，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。Dubbo 可以为我们解决服务之间互相是如何调用的。
3. **服务访问压力以及时长统计、资源调度和治理**——基于访问压力实时管理集群容量，提高集群利用率。
4. **服务降级**——某个服务挂掉之后调用备用服务。

- ZK宕掉Dubbo直连

假如zookeeper注册中心宕掉，一段时间内服务消费方还是能够调用提供方的服务的，实际上它使用的本地缓存进行通讯，这只是dubbo健壮性的一种体现。

1. 监控中心宕掉不影响使用，只是丢失部分采样数据
2. 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
3. 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
4. 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
5. 服务提供者无状态，任意一台宕掉后，不影响使用
6. 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

## 数据结构

线性数据结构：数组、链表、栈、队列

### 二叉树

树是一种常见的数据结构，所有结点的度均不超过2的有序树为二叉树。二叉树是一种最简单且最重要的树，所有的树都能转化为二叉树。
[二叉树介绍](https://www.jianshu.com/p/bf73c8d50dc2)

#### 二叉树的遍历

通常使用递归调用的方法进行二叉树的遍历，并根据递归的输出顺序，可以简单地分为：前序遍历，中序遍历，后序遍历。

- 前序遍历
从二叉树的根结点出发，当第一次到达结点时就输出结点数据。按照先左后右的顺序，只要能向左前进，就一直向左，否则返回上层，向右前进，而后继续尝试向左。如果右侧也为叶子结点，则返回上层。
技巧：按照遍历顺序输出
遍历代码：

```java
/*前序遍历*/
void PreOrderTraverse(BiTree T)
{
    if(T == NULL) {
        return;
    }
    System.out.println("%c", T.data);  /*先输出*/
    PreOrderTraverse(T.lchild);    /*再遍历左*/
    PreOrderTraverse(T.rchild);    /*最后遍历右*/
}
```

>对于二叉树来说，前序遍历即深度优先搜索

- 中序遍历
从二叉树的根结点出发，当第二次到达结点时就输出结点数据。按照先左后右的顺序，先向左前进，如果前进后为空（即原结点为叶子结点）则返回原结点，此时第二次访问，输出。后续继续向右前进
技巧：对于每一个子树，输出顺序均为左中右
遍历代码：

```java
/*中序遍历*/
void InOrderTraverse(BiTree T)
{
    if(T == NULL) {
        return;
    }
    InOrderTraverse(T.lchild);    /*先遍历左*/
    System.out.println("%c", T.data);  /*再输出*/
    InOrderTraverse(T.rchild);    /*最后遍历右*/
}
```

- 后序遍历
从二叉树的根结点出发，当第三次到达结点时就输出结点数据。按照先左后右的顺序，先向左前进，如果前进后为空（即原结点为叶子结点）则返回原结点，再向右侧出发，如果前进后为空，则返回原结点，此时第三次访问，输出。
技巧：对于每一个子树，输出顺序均为左右中
遍历代码：

```java
/*后序遍历*/
void PostOrderTraverse(BiTree T)
{
    if(T == NULL) {
        return;
    }
    PostOrderTraverse(T.lchild);    /*先遍历左*/
    PostOrderTraverse(T.rchild);    /*后遍历右*/
    System.out.println("%c", T.data);  /*最后输出*/
}
```

- **重点考点：根据前序遍历/后序遍历和中序遍历顺序，确定一颗二叉树**
首先根据前序/后续遍历获取根节点：前序遍历的第一个结点和后序遍历的最后一个结点，都必定是根节点。
然后根据根结点，从中序遍历中获取左子树和右子树：位于中序遍历中根结点前的均为左子树，根结点后的均为右子树。
重复上述过程，继续对子树进行分析，即可得到结果

- 层次遍历
从二叉树的根结点出发，按照从上到下，从左到右的方式依次进行输出，为层次遍历

>对于二叉树来说，层次遍历即为广度优先搜索

### 红黑树

[红黑树介绍](https://zhuanlan.zhihu.com/p/79980618)
[红黑树介绍](https://tech.meituan.com/2016/12/02/redblack-tree.html)

红黑树是一种平衡二叉查找树。二叉查找树是一种左子节点的值比父节点的值要小，右节点的值要比父节点的值大的二叉树。

红黑树定义如下：

1. 任何一个节点都有颜色，黑色或者红色。
2. 根节点是黑色的。
3. 父子节点之间不能出现两个连续的红节点。
4. 任何一个节点向下遍历到其子孙的叶子节点，所经过的黑节点个数必须相等。
5. 空节点被认为是黑色的。

如果父节点和新节点为红色，则按照以下规则进行修复：
左左：提绳法，提起父节点并变色
左右：旋转新节点，然后复用左左
右右：提起父节点并变色
右左：旋转新节点，然后复用右右

## 操作系统

### 基础

操作系统本质是一个运行在计算机上的软件，它屏蔽了硬件的复杂性，统一进行管理。

- 内核

操作系统的内核（Kernel）是操作系统的核心部分，它负责系统的内存管理，硬件设备的管理，文件系统的管理以及应用程序的管理。
操作系统的内核是连接应用程序和硬件的桥梁，决定着操作系统的性能和稳定性。

- 系统调用

根据进程访问资源的权限，可以分为用户态和系统态。用户态的进程只能读取用户内存中的数据，涉及到系统级别的资源访问（设备管理，文件管理，进程控制，进程通信，内存管理），需要向操作系统发出请求（系统调用），由操作系统代为完成。

### 进程

- 进程的状态转换

[操作系统之进程的状态和转换详解](https://blog.csdn.net/qwe6112071/article/details/70473905)

通常将进程分为五个状态：

1. 创建状态(new) ：进程正在被创建，尚未到就绪状态。
2. 就绪状态(ready) ：进程已处于准备运行状态，即进程获得了除了处理器之外的一切所需资源，一旦得到处理器资源(处理器分配的时间片)即可运行。
3. 运行状态(running) ：进程正在处理器上上运行(单核 CPU 下任意时刻只有一个进程处于运行状态)。
4. 阻塞状态(waiting) ：又称为等待状态，进程正在等待某一事件而暂停运行如等待某资源为可用或等待 IO 操作完成。即使处理器空闲，该进程也不能运行。
5. 结束状态(terminated) ：进程正在从系统中消失。可能是进程正常结束或其他原因中断退出运行。

- 进程 vs 线程

线程是进程划分成的更小的运行单位,一个进程在其执行的过程中可以产生多个线程。

和多线程相比，多进程的缺点在于：
创建进程比创建线程开销大，尤其是在Windows系统上；
进程间通信比线程间通信要慢，因为线程间通信就是读写同一个变量，速度很快。

而多进程的优点在于：
多进程稳定性比多线程高，因为在多进程的情况下，一个进程崩溃不会影响其他进程，而在多线程的情况下，任何一个线程崩溃会直接导致整个进程崩溃。

- 进程通信

常用的方法：

1. 管道：父子进程/兄弟进程之间通信
2. 信号量：多个进程对于共享数据访问，通过信号量达到同步的目的，避免竞争
3. 消息队列：消息队列是消息的链表,具有特定的格式,存放在内存中并由消息队列标识符标识。
4. 共享内存：使得多个进程可以访问同一块内存空间，不同进程可以及时看到对方进程中对共享内存中数据的更新。（需要注意上锁）
5. 套接字：常见的TCP/IP通信

- 进程的调度算法

1. 先到先服务(FCFS)调度算法 : 从就绪队列中选择一个最先进入该队列的进程为之分配资源，使它立即执行并一直执行到完成或发生某事件而被阻塞放弃占用 CPU 时再重新调度。
2. 短作业优先(SJF)的调度算法 : 从就绪队列中选出一个估计运行时间最短的进程为之分配资源，使它立即执行并一直执行到完成或发生某事件而被阻塞放弃占用 CPU 时再重新调度。
3. 时间片轮转调度算法 : 时间片轮转调度是一种最古老，最简单，最公平且使用最广的算法，又称 RR(Round robin)调度。每个进程被分配一个时间段，称作它的时间片，即该进程允许运行的时间。
4. 多级反馈队列调度算法 ：前面介绍的几种进程调度的算法都有一定的局限性。如短进程优先的调度算法，仅照顾了短进程而忽略了长进程 。多级反馈队列调度算法既能使高优先级的作业得到响应又能使短作业（进程）迅速完成。，因而它是目前被公认的一种较好的进程调度算法，UNIX 操作系统采取的便是这种调度算法。
5. 优先级调度 ： 为每个流程分配优先级，首先执行具有最高优先级的进程，依此类推。具有相同优先级的进程以 FCFS 方式执行。可以根据内存要求，时间要求或任何其他资源要求来确定优先级。

### 内存

- 内存管理

包括内存的分配、回收等，常见的管理方式包括：

1. 页式管理
   把主存分为大小相等且固定的一页一页的形式进行管理，提高利用率，降低碎片
2. 段式管理
   页式无实际意义，而段式有。段式更小，例如程序段，数据段，栈段
3. 段页式管理
   结合了段式和页式，把主存先分成若干段，每个段又分成若干页。段与段之间以及段的内部的都是离散的。

快表：当拿到一个页号要寻找对应的物理地址时，先查快表（类似缓存），快表没有再查页表。快表将满时则进行淘汰。

分段和分页的共同点与区别：

1. 共同点：都是为了提高内存利用率，减少碎片。都是离散存储，内部连续。
2. 区别：页的大小固定，由操作系统决定。段的大小不固定，由程序决定。分段是带有逻辑信息的，例如代码段，数据段。

逻辑地址和物理地址：通常编程只关心逻辑地址，例如指针中存储的数值就是逻辑地址。物理地址是内存真正的地址。
>如果没有逻辑地址，直接操作物理地址，会造成数据混乱。例如两个程序同时修改物理地址为1的数据。因此使用虚拟地址进行隔离。

- 虚拟内存

虚拟内存是计算机系统内存管理的一种技术，我们可以手动设置自己电脑的虚拟内存。它定义了一个连续的虚拟地址空间，并且**把内存扩展到硬盘**空间。

- 局部性原理

1. 时间局部性 ：如果程序中的某条指令一旦执行，不久以后该指令可能再次执行；如果某数据被访问过，不久以后该数据可能再次被访问。产生时间局部性的典型原因，是由于在程序中存在着大量的循环操作。
2. 空间局部性 ：一旦程序访问了某个存储单元，在不久之后，其附近的存储单元也将被访问，即程序在一段时间内所访问的地址，可能集中在一定的范围之内，这是因为指令通常是顺序存放、顺序执行的，数据也一般是以向量、数组、表等形式簇聚存储的。

- 页面置换算法

当发生缺页中断，即内存中不存在所需要的页，需要操作系统从外存将页面load到内存。此时内存中没有空闲的页，必须要进行淘汰，即页面置换算法。常见的包括以下几种：

1. OPT 页面置换算法（最佳页面置换算法） ：最佳(Optimal, OPT)置换算法所选择的被淘汰页面将是以后永不使用的，或者是在最长时间内不再被访问的页面,这样可以保证获得最低的缺页率。但由于人们目前无法预知进程在内存下的若千页面中哪个是未来最长时间内不再被访问的，因而该算法**无法实现**。一般作为衡量其他置换算法的方法。
2. FIFO（First In First Out） 页面置换算法（先进先出页面置换算法） : 总是淘汰最先进入内存的页面，即选择在内存中驻留时间最久的页面进行淘汰。
3. LRU （Least Currently Used）页面置换算法（最近最久未使用页面置换算法） ：LRU算法赋予每个页面一个访问字段，用来记录一个页面自上次被访问以来所经历的时间 T，当须淘汰一个页面时，选择现有页面中其 T 值最大的，即最近最久未使用的页面予以淘汰。
4. LFU （Least Frequently Used）页面置换算法（最少使用页面置换算法） : 该置换算法选择在之前时期使用最少的页面作为淘汰页。

### Linux常见操作命令

目录相关：cd（目录切换）、mkdir（新建目录）、ls（显示目录）、pwd（当前绝对路径）
文件相关：rm（删除）、mv（移动）、cp（拷贝）、touch（新建）
修改权限：chmod（1=执行，2=写，4=读）
其他：ps aux（查看当前系统进程）、grep（搜索）、kill（杀死进程）

## 网络

### 描述一次Http请求

[一次Http请求](https://segmentfault.com/a/1190000006879700)

- DNS解析
  根据域名查询对应的IP地址，涉及到DNS缓存（常用的:浏览器缓存>路由器缓存>服务器缓存）,DNS负载均衡，CDN。
- 建立TCP连接
  三次握手确认连接建立
- 发送HTTP请求
  客户端组装报文并发送
- 服务器处理并返回HTTP报文
  后端代码对请求进行相应处理，并返回数据
- 浏览器解析渲染页面
  返回的报文单纯是字符串则直接输出（例如json）如果是HTML页面，浏览器还会进行渲染

### Session与Cookie

1. cookie数据存放在客户的浏览器上，session数据放在服务器上。
2. cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗，考虑到安全应当使用session。
3. session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用COOKIE。
4. 单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。

### GET和POST

1. get是从服务器上获取数据，post是向服务器传送数据。
2. get是把参数数据队列加到提交表单的ACTION属性所指的URL中，值和表单内各个字段一一对应，在URL中可以看到。post是通过HTTP post机制，将表单内各个字段与其内容放置在HTML HEADER内一起传送到ACTION属性所指的URL地址。用户看不到这个过程。
3. get传送的数据量较小，不能大于2KB。post传送的数据量较大，一般被默认为不受限制。
4. get安全性非常低，post安全性较高。但是执行效率却比Post方法好。

### http状态码

12345法则

- 1** 消息 100 客户端应当继续发送请求。
- 2** 成功 200 成功
- 3** 重定向 301 永久重定向，例如http定向到https 302 临时重定向，例如js跳转
- 4** 请求错误 403 forbidden 拒绝请求。 404 not found 找不到请求的网页。
- 5** 服务器错误 500 Internal Server Error 服务器内部错误，例如代码错误

### http版本

- 1.0
**无状态、无连接** HTTP1.0规定浏览器和服务器保持短暂的连接，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器处理完成后立即断开TCP连接（无连接），服务器不跟踪每个客户端也不记录过去的请求（无状态）。
**队头阻塞** HTTP1.0规定下一个请求必须在前一个请求响应到达之前才能发送。假设前一个请求响应一直不到达，那么下一个请求就不发送，同样的后面的请求也给阻塞了。

- 1.1
**长连接** HTTP1.1增加了一个Connection字段，通过设置Keep-Alive可以保持HTTP连接不断开，避免了每次客户端与服务器请求都要重复建立释放建立TCP连接，提高了网络的利用率。如果客户端想关闭HTTP连接，可以在请求头中携带Connection: false来告知服务器关闭请求。
**管道化** 基于HTTP1.1的长连接，使得请求管线化成为可能。管线化使得请求能够“并行”传输。
**新的字段** 如cache-control，支持断点传输，以及增加了Host字段（使得一个服务器能够用来创建多个Web站点）。

- 2.0
**二进制分帧** HTTP2.0通过在应用层和传输层之间增加一个二进制分帧层，突破了HTTP1.1的性能限制、改进传输性能。
**多路复用（连接共享）** HTTP2.0实现了真正的并行传输，它能够在一个TCP上进行任意数量HTTP请求。而这个强大的功能则是基于“二进制分帧”的特性。
**头部压缩** HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。高效的压缩算法可以很大的压缩header，减少发送包的数量从而降低延迟。
**服务器推送** 服务器除了对最初请求的响应外，服务器还可以额外的向客户端推送资源，而无需客户端明确的请求。

### 网络模型及常见协议

[四层模型及OSI七层参考模型](https://blog.csdn.net/guoguo527/article/details/52078962)
[三次握手四次挥手](https://www.cnblogs.com/Jessy/p/3535612.html)

五层结构：

1. 应用层 应用进程交互层，基于应用层协议。常见应用层协议：DNS，HTTP，SMTP（邮件）应用层之间通过报文交互。
2. 运输层 两台主机之间的数据传输。以TCP/UDP方式。
3. 网络层 网络层的任务就是选择合适的网间路由和交换结点，确保数据及时传送。例如IP协议
4. 数据链路层 通常以帧为单位进行数据传输，丢帧的检错和纠错也是在这一层
5. 物理层 以比特为单位传输数据

TCP协议简略快速回忆版：

三次握手 客户端：我要和你通信(syn-sent) 服务端：你的请求已收到，发送确认(syn-rcvd) 客户端：你的确认已收到，连接建立(est)

四次挥手 客户端：我没有东西了，准备关闭(fin-wait) 服务端：你的关闭我收到了，但我还有点东西没传完(close-wait) ……一段时间后…… 服务端：我的东西传完了，可以关闭了(last-ack) 客户端：收到关闭通知，你也可以关闭了(time-wait)

UDP协议：无需建立连接，不需要确认，不可靠，但高效。常用于即时通讯、语音、直播

滑动窗口：TCP使用滑动窗口协议实现流量控制。

## Mysql

### InnoDB

InnoDB是事务性数据库引擎，支持事务和行级锁

### 字符集

字符集是字符的集合，常见的包括：ASCII、GBK、UTF8等。
Mysql数据库一般使用utf8mb4字符集（mysql默认的utf8最大为3字节，而emoji表情、不常见的汉字等字符都是4字节的）

### 索引

MySQL索引使用的数据结构主要有BTree索引 和 哈希索引 。对于哈希索引来说，底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快；其余大部分场景，建议选择BTree索引。

- InnoDB和MyISAM的B+Tree：

1. MyISAM：非聚簇索引。检索时先根据key找到对应的data域，data域存储的是实际数据所在的地址，根据地址读取对应的数据
2. InnoDB：聚簇索引。索引文件和数据文件分离。数据文件本身是一个按照B+Tree存储的主索引，每一个叶子结点都直接存储了完整的数据记录。其余索引都是辅助索引，存储的是主键的值而非地址。在根据主索引搜索时，直接找到key所在的节点即可取出数据；在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。
  
>MyISAM存地址，InnoDB存真实数据，辅助索引结点存主键

- B树和B+树的主要区别

1. B+Tree左闭右合：关键字的搜索采用的是左闭合区间，之所以采用左闭合区间是因为他要最好的去支持自增id，这也是mysql的设计初衷。即，如果id = 1命中，会继续往下查找，直到找到叶子节点中的1。
2. B+Tree只有叶子结点存数据：即只有叶子节点中的关键字数据区才会保存真正的数据内容或者是内容的地址。而在B树种，如果根节点命中，则会直接返回数据。
3. B+Tree叶子节点不会去保存子节点的引用。
4. B+Tree叶子节点是顺序排列的，并且相邻的节点具有顺序引用的关系，叶子节点之间有指针相连接。

### 事务

- 事务的四大特性（ACID）

1. 原子性（Atomicity）： 事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；
2. 一致性（Consistency）： 执行事务后，数据库从一个正确的状态变化到另一个正确的状态；
3. 隔离性（Isolation）： 并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
4. 持久性（Durability）： 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

### 数据库优化

数据库优化主要围绕以下几个方向：

1. SQL和索引优化
2. 表结构优化
3. 系统配置
4. 硬件

具体优化方案：
首先是从代码优化，这是最根本的优化方式，找到瓶颈。尤其注意循环中查询
然后从sql语句上优化，explain查看具体的索引。
其他方向可以考虑读写分离。
>主从同步的方式

[全局唯一增长id生成-美团](https://tech.meituan.com/2017/04/21/mt-leaf.html)

[mysql高性能优化规范](https://www.cnblogs.com/huchong/p/10219318.html)

## Redis

Redis是一种单线程内存型数据库，常用来做缓存、分布式锁、消息队列等。支持的数据类型包括：kv、list、hash等

### 单线程设计

Redis基于Reactor模式设计了高性能单线程模型。

1. 使用单线程模型能带来更好的可维护性，方便开发和调试；
2. 使用单线程模型也能并发的处理客户端的请求；
3. Redis 服务中运行的绝大多数操作的性能瓶颈都不是 CPU；

>I/O 多路复用：多个客户端连接共用一个I/O（多个应用程序通过一根网线传输数据）

如果redis的吞吐量不能达到要求，更好的方案是用多个redis服务器，分片存储数据，而不是在同一个redis服务中使用多线程。

多线程redis用于删除操作：同步删除键，异步删除值

### IO多路复用

IO多路复用程序根据系统中支持的具体函数库作为底层，实现相同的IO相关的API。
客户端与服务端建立套接字，I/O多路复用程序会监听多个套接字的事件
redis有多个文件事件处理，包括命令请求、命令回复、连接应答等。
当套接字产生对应的事件（例如客户端连接服务器）会引发对应的事件处理器执行，并执行相应的套接字应答操作。

基于epoll实现，调用epoll_create时内核会创建一个file结点，用红黑树存储事件，还会有一个双向链表存储准备就绪的时间。epoll_wait调用时观察双向链表有没有数据即可。
I/O中断事件来临时，会激活内核的回调函数，将IO事件放到链表中，由用户程序进行操作。

具体而言，redis通过epoll监听客户端与服务器的套接字事件。
当客户端发起连接请求，会生成IO事件，产生中断，内核会激活回调函数将事件放到链表中。此时redis能够感知到这个文件，并将其指定给对应的文件事件处理器，此时为连接应答处理器，创建客户端套接字，关联套接字事件到命令请求处理器。
当客户端发送命令请求，客户端套接字产生新的可读事件，引发命令请求处理器执行，处理器读取客户端的命令内容，然后传给相关程序去执行。命令处理完成，服务器会将客户端的可写事件关联到命令回复处理器。
当客户端尝试读取命令回复时，客户端套接字产生新的可写事件，触发命令回复处理器，将回复内容写入套接字，然后关闭可写事件到处理器的关联。

### 过期

设置过期时间后，对应的key会在过期时间后失效，但不会立即删除。
删除策略主要包括：

1. 惰性删除 ：只会在取出key的时候才对数据进行过期检查。这样对CPU最友好，但是可能会造成太多过期 key 没有被删除。
2. 定期删除 ： 每隔一段时间抽取一批 key 执行删除过期key操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。

但是删除策略依旧无法保证所有的key都会自动过期，因此Redis有内存淘汰机制。当内存已满时，除非选择“禁止驱逐数据”的模式（这种情况下满内存写入会报错），否则会根据一定策略进行删除。可以参考操作系统的内存管理机制（经典LRU）

### 缓存相关问题

- 缓存穿透

缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。

解决方案：使用布隆过滤器。

[布隆过滤器](https://www.cnblogs.com/liyulong1982/p/6013002.html)

- 缓存雪崩

缓存在同一时间大面积的失效，后面的请求都直接落到了数据库上，造成数据库短时间内承受大量请求。

解决办法：注意不要把缓存设定到集中某个时间（例如凌晨0点）按时过期。

- 缓存读写策略

1. 旁路缓存模式 写入时先写DB，再删cache（如果先删，可能会在写入完成前被别的客户端读取从而导致写入旧数据）读取时先读cache，读不到再读DB并回写入cache
   缺点：第一次读取必定不在cache（解决：热数据可以提前写入）。写操作频繁时cache删除频繁
2. 读写穿透 读写都在cache，由固定的服务将cache改动同步到DB（很少见）
3. 异步缓存写入 只更新缓存，不直接更新 DB，而是改为异步批量的方式来更新DB（非常适合一些数据经常变化又对数据一致性要求没那么高的场景：例如微博点赞量，网页访问量，每次cache自动+1，固定时间取出写入DB存储）

## Kafka

## ES

ElasticSearch

## 常见概念

### CAP理论

分布式重点：

1. 一致性（Consistence） : 所有节点访问同一份最新的数据副本
2. 可用性（Availability）: 非故障的节点在合理的时间内返回合理的响应（不是错误或者超时的响应）。
3. 分区容错性（Partition tolerance） : 分布式系统出现网络分区的时候，仍然能够对外提供服务。

CAP 理论中分区容错性 P 是一定要满足的，在此基础上，只能满足可用性 A 或者一致性 C。

>原因：发生网络分区（存在系统无法联通）时，当一个分区在写时，为了保证C，必须禁止其他分区的读写，因此A不能保证
如果系统没有发生“分区”的话，节点间的网络连接通信正常的话，也就不存在 P 了。这个时候，我们就可以同时保证 C 和 A 了。因此，如果系统发生“分区”，我们要考虑选择 CP 还是 AP。如果系统没有发生“分区”的话，我们要思考如何保证 CA 。

ZooKeeper保证了CP

[CAP应用](https://juejin.cn/post/6844903936718012430)

### BASE理论

BASE是CAP理论的延伸，Basically Available（基本可用） 、Soft-state（软状态） 和 Eventually Consistent（最终一致性） 三个短语的缩写。

1. 基本可用：允许损失部分可用性，例如响应时间、系统部分功能不可用
2. 软状态：允许系统中的数据存在中间状态（CAP 理论中的数据不一致），并认为该中间状态的存在不会影响系统的整体可用性
3. 最终一致性：系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性。
    **强一致性** ：系统写入了什么，读出来的就是什么。
    **弱一致性** ：不一定可以读取到最新写入的值，也不保证多少时间之后读取到的数据是最新的，只是会尽量保证某个时刻达到数据一致的状态。
    **最终一致性** ：弱一致性的升级版，系统会保证在一定时间内达到数据一致的状态。

## 算法

### 常用java操作函数

#### Collections

Arrays.sort(T[]); 默认升序，降序需要实现Comparator

#### List-ArrayList

list.add(value);
list.get(index);
list.size();
list.remove(index);
list.isEmpty();

#### Deque-LinkedList

stack.isEmpty();
stack.peek();
stack.pop();
stack.push(value);

queue.offer(value);
queue.poll();
queue.peek();

#### Map-HashMap

map.push(key, value);
map.containsKey(key);
map.containsValue(value);
map.get(key);
map.remove(key);
map.size();
`Set<String> keySet = map.keySet();`
`Collection<Integer> values = map.values();`

#### Set-HashSet

set.add(value);
set.remove(value);
set.contains(value);
set.size();
set.clear();

#### StringBuffer

s.append(char);
s.deleteCharAr(index);
s.toString();

### 解题方法

#### 回溯搜索

有的时候无法提前知道循环的次数，需要用递归进行回溯。
回溯的过程通常需要找到边界条件，即已经搜索到结果输出。
例如：输入一个数，给出可能的相加组合。因为不知道参与相加的数字会有多少。全排列。

#### 双指针

分为左右指针和快慢指针，左右指针是从左右两侧开始向中间移动，快慢指针是朝一个方向移动，但是速度或是布长不同。常用来解决寻找两个边界的问题。

#### 哈希表

当寻找某个相同特征的元素时，使用HashMap存储，可以直接通过map.get获取对应的特征。
例如：寻找多个字符串中，元素相同的。

#### 动态规划

当dp(i) 与 dp(i-1)相关，并且有变化时使用。定义转移量。

#### 滑动窗口

## 一些面试总结

[Java Guide](https://snailclimb.gitee.io/javaguide/)

[各大公司Java后端开发面试题总结](https://blog.csdn.net/sinat_35512245/article/details/59056120)
