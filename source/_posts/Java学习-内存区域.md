---
title: Java学习-内存区域
categories:
  - Java
tags:
  - 内存区域
date: 2020-02-03 16:12:21
---

### 目录
* [1.缘由](#1-缘由)
* [2.目的](#2-目的)
* [3.具体实施](#3-具体实施)
    * [3.1 事前准备](#3-1事前准备)
    * [3.2 进入正题](#3-2进入正题)

## 1.缘由

巩固Java知识图谱

## 2.目的

强化基础，避免大楼不稳

## 3.具体实施

### 3.1事前准备

注意区别于内存模型，JMM即内存模型，是指线程和主内存之前的抽象关系，定义了JVM在RAM中的工作方式，用于并发编程。

内存区域指的是JVM运行时将数据分区域存储，是对内存空间的划分。

### 3.2进入正题

jdk8之后JVM内存区域有几大块：

* 本地方法栈(Native Method Stacks)
* 程序计数器(Program Counter Register)
* 虚拟机栈(JVM Stacks)
* 虚拟机堆(JVM Heap)
* 元数据区(Metaspace)

### 3.2.1本地方法栈


为虚拟机使用到的native方法服务，Sun HotSpot虚拟机把本地方法栈和虚拟机栈合二为一，都会跑出对应SOF、OOM异常。

由于是调用操作系统特性功能，复用非Java代码，所以不受JVM约束，可以通过JNI来访问运行时的数据区，或调用寄存器等，不过使用JNI同样会使得系统丧失跨平台特性。

### 3.2.2程序计数器

程序计数器是一块较小的内存空间，可以看作为当前线程所执行的字节码的行号指示器。

值得一提的是JVM的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器内核都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，这类内存区域为"线程私有"的内存。

若线程执行的是一个Native方法，这个计数器值则为空(Undefined)。此内存区域是唯一一个在JVM中没有规定OOM的区域。

### 3.2.3虚拟机栈

和程序计数器一样，为线程私有，生命周期于线程相同。

虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧(Stack Frame,方法运行时的基础数据结构)用于存储局部变量、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

在活动线程中，只有位于栈顶的帧才是有效的，称为当前栈帧。正在执行的方法称为当前方法，栈帧是方法运行的基本结构。在执行引擎运行时，所有指令都只能针对当前栈帧进行操作。


1. 局部变量表
局部变量表是存放方法参数和局部变量的区域。 局部变量没有准备阶段， 必须显式初始化。如果是非静态方法，则在 index[0] 位置上存储的是方法所属对象的实例引用，一个引用变量占 4 个字节，随后存储的是参数和局部变量。字节码指令中的 STORE 指令就是将操作栈中计算完成的局部变呈写回局部变量表的存储空间内。

虚拟机栈规定了两种异常状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 StackOverflowError 异常；如果虚拟机栈可以动态扩展（当前大部分的 Java 虚拟机都可动态扩展），如果扩展时无法申请到足够的内存，就会抛出 OutOfMemoryError 异常。

2. 操作栈
操作栈是个初始状态为空的桶式结构栈。在方法执行过程中， 会有各种指令往
栈中写入和提取信息。JVM 的执行引擎是基于栈的执行引擎， 其中的栈指的就是操
作栈。字节码指令集的定义都是基于栈类型的，栈的深度在方法元信息的 stack 属性中。

i++ 和 ++i 的区别：

i++：从局部变量表取出 i 并压入操作栈，然后对局部变量表中的 i 自增 1，将操作栈栈顶值取出使用，最后，使用栈顶值更新局部变量表，如此线程从操作栈读到的是自增之前的值。
++i：先对局部变量表的 i 自增 1，然后取出并压入操作栈，再将操作栈栈顶值取出使用，最后，使用栈顶值更新局部变量表，线程从操作栈读到的是自增之后的值。
之前之所以说 i++ 不是原子操作，即使使用 volatile 修饰也不是线程安全，就是因为，可能 i 被从局部变量表（内存）取出，压入操作栈（寄存器），操作栈中自增，使用栈顶值更新局部变量表（寄存器更新写入内存），其中分为 3 步，volatile 保证可见性，保证每次从局部变量表读取的都是最新的值，但可能这 3 步可能被另一个线程的 3 步打断，产生数据互相覆盖问题，从而导致 i 的值比预期的小。

3. 动态链接
每个栈帧中包含一个在常量池中对当前方法的引用， 目的是支持方法调用过程的动态连接。

4.方法返回地址
方法执行时有两种退出情况：

正常退出，即正常执行到任何方法的返回字节码指令，如 RETURN、IRETURN、ARETURN 等；
异常退出。
无论何种退出情况，都将返回至方法当前被调用的位置。方法退出的过程相当于弹出当前栈帧，退出可能有三种方式：

返回值压入上层调用栈帧。
异常信息抛给能够处理的栈帧。
PC计数器指向方法调用后的下一条指令。


### 3.2.4虚拟机堆

#### Java堆

对于大多数应用来说，Java 堆（Java Heap）是 Java 虚拟机所管理的内存中最大的一块。Java 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。

堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC堆”（Garbage Collected Heap）。从内存回收的角度来看，由于现在收集器基本都采用
//todo
![分代收集算法](./GC-分代收集算法.md)，所以 Java 堆中还可以细分为：新生代和老年代；再细致一点的有 Eden 空间、From Survivor 空间、To Survivor 空间等。从内存分配的角度来看，线程共享的 Java 堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer,TLAB）。

Java 堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，当前主流的虚拟机都是按照可扩展来实现的（通过 -Xmx 和 -Xms 控制）。如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出 OutOfMemoryError 异常。

### 3.2.5元数据区

#### 方法区

方法区（Method Area）与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然
Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。

Java 虚拟机规范对方法区的限制非常宽松，除了和 Java 堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。垃圾收集行为在这个区域是比较少出现的，其内存回收目标主要是针对常量池的回收和对类型的卸载。当方法区无法满足内存分配需求时，将抛出 OutOfMemoryError 异常。

JDK8 之前，Hotspot 中方法区的实现是永久代（Perm），JDK8 开始使用元空间（Metaspace），以前永久代所有内容的字符串常量移至堆内存，其他内容移至元空间，元空间直接在本地内存分配。

为什么要使用元空间取代永久代的实现？

字符串存在永久代中，容易出现性能问题和内存溢出。
类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
将 HotSpot 与 JRockit 合二为一。



运行时常量池

运行时常量池（Runtime Constant Pool）是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

一般来说，除了保存 Class 文件中描述的符号引用外，还会把翻译出来的直接引用也存储在运行时常量池中。

运行时常量池相对于 Class 文件常量池的另外一个重要特征是具备动态性，Java 语言并不要求常量一定只有编译期才能产生，也就是并非预置入 Class 文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是 String 类的 intern() 方法。

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。

直接内存

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是 Java 虚拟机规范中定义的内存区域。

在 JDK 1.4 中新加入了 NIO，引入了一种基于通道（Channel）与缓冲区（Buffer）的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆中来回复制数据。

显然，本机直接内存的分配不会受到 Java 堆大小的限制，但是，既然是内存，肯定还是会受到本机总内存（包括 RAM 以及 SWAP 区或者分页文件）大小以及处理器寻址空间的限制。服务器管理员在配置虚拟机参数时，会根据实际内存设置 -Xmx 等参数信息，但经常忽略直接内存，使得各个内存区域总和大于物理内存限制（包括物理的和操作系统级的限制），从而导致动态扩展时出现 OutOfMemoryError 异常。

//todo

疑问1：是执行native方法时才没有OOM还是所有方法计数器都没有OOM？

疑问2：JVM堆中方法区jdk8使用元数据的原因以及产生有点的原理？

疑问3：直接内存的使用场景？
