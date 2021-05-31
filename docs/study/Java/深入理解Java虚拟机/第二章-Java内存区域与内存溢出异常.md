# 深入理解Java虚拟机 第二章
# Java内存区域与内存溢出异常
## 运行时数据区域
- Java 虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。

### 程序计数器
- 程序计算器（Program Counter Register）是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。

> 线程私有：每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储。

- 如果一个线程正在执行的是一个Java的方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址

- 如果正在执行的是Native方法，这个计数器值则为空（Undefined）

### Java虚拟机栈
- Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期和线程相同。

- 虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链表、方法出口等信息。
- 每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈和出栈的过程。

- 局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double），其中64位长度的long和double类型的数据会占用2个局部变量空间（slot），其余数据类型只占用1个。

- StackOverflowError异常：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。
- OutOfMemoryError异常：如果扩展时无法申请到足够的内存，就会抛出 OutOfMemoryError异常。

### 本地方法栈
- 虚拟机栈为虚拟机提供Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。

### Java堆
- Java堆（Java Heap）是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。

- 从内存回收的角度来看，由于现在收集器基本都采用分代收集算法，所以Java堆中还可以细分为：新生代 和 老年代，再细致一点的有Eden空间、From Survivor空间、To Survivor空间等。

### 方法区 
- 方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

### 运行时常量池
- 运行时常量池（Runtime Constant Pool）是方法区的一部分。

## HotSpot虚拟机对象探秘
### 直接内存
- 直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分。
- 在JDK 1.4中新加入NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过存储在Java堆中的DirectByteBuffer对象作为
这块内存的引用进行操作。

### 对象的创建
- 虚拟机遇到一条new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化。如果没有，那必须执行相应的类加载过程。
- 在类加载通过后，接下来虚拟机为新生对象分配内存。对象所需的内存大小在类加载完成后便可完全确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来。
- 指针碰撞： 假设Java堆中内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针指向空闲空间那边挪动一段与对象大小相等的距离。
- 空闲列表： 假设Java堆中内存并不是绝对规整的，已使用的内存和空闲的内存相互交错，此时，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表的记录。

- 使用Serial、ParNew等带Compact过程的收集器时，系统采用的分配算法是指针碰撞。
- 使用CMS这种基于Mark-Sweep算法的收集器时，通常采用空闲列表。

- 并发修改指针问题： 一种是对分配内存空间的动作进行同步处理-实际上虚拟机采用CAS配上失败重试的方式保证更新操作的原子性。另一种是把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在Java堆中预先分配一小块内存，称为
  本地线程分配缓冲（Thread Local Allocation Buffer TLAB），哪个线程要分配内存就在哪个线程的TLAB上分配，只有TLAB用完并分配新的TLAB时，才需要同步锁定。
  内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），如果使用TLAB，这一工作过程也可以提前至TLAB分配时进行。这一步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用，程序能访问到这些字段
  的数据类型所对应的零值。
  
### 对象内存布局
- 在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）

对象头（Header）：
- 存储对象自身的运行时数据，（如HashCode、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等） MarkWord：上述部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit。如果对象处于未锁定的状态下，MarkWord的32bit空间中的25bit用于存储对象的哈希码，4bit用于存储对象分代年龄，2bit用于存储锁标志位，1bit固定为0。
- 对象头的另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

实例数据（Instance Data）：
- 实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。这部分的存储顺序会受到虚拟机分配策略参数（FieldsAllocationStyle）和字段在Java源码中定义顺序的影响。

对象填充（Padding）：
- 仅仅起着占位符的作用。

### 对象的访问定位
建立对象是为了使用对象，我们的Java程序需要通过栈上的reference数据来操作堆上的具体对象。
目前主流的访问方式有两种：
- 使用句柄访问：在Java堆中将会划为一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息。

- 使用直接指针访问：在Java堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference中存储的直接就是对象地址。

这两种对象访问方式各有优势
- 使用句柄来访问的最大好处就是reference中存储的是稳定的句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而reference本身不需要修改。
- 使用直接指针访问方式的最大好处就是速度更快，它节省了一次指针定位的时间开销。

## 实战： OutOfMemoryError 异常
### Java堆溢出
- 设置IDE中的 VM arguments： -verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8

- 运行如下代码
```java
public class HeapOOM {

    static class OOMObject{

    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while (true){
            list.add(new OOMObject());
        }
    }

}
```

- 抛出如下异常  java.lang.OutOfMemoryError: Java heap space
```log
[GC (Allocation Failure) [PSYoungGen: 8192K->1014K(9216K)] 8192K->1499K(19456K), 0.0013164 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 9206K->995K(9216K)] 9691K->6856K(19456K), 0.0069342 secs] [Times: user=0.07 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 995K->0K(9216K)] [ParOldGen: 5860K->6469K(10240K)] 6856K->6469K(19456K), [Metaspace: 3288K->3288K(1056768K)], 0.0779819 secs] [Times: user=0.48 sys=0.01, real=0.08 secs] 
[Full GC (Ergonomics) [PSYoungGen: 6665K->1445K(9216K)] [ParOldGen: 6469K->9887K(10240K)] 13135K->11333K(19456K), [Metaspace: 3301K->3301K(1056768K)], 0.1170625 secs] [Times: user=0.79 sys=0.00, real=0.12 secs] 
[Full GC (Ergonomics) [PSYoungGen: 8192K->7156K(9216K)] [ParOldGen: 9887K->8800K(10240K)] 18079K->15956K(19456K), [Metaspace: 3306K->3306K(1056768K)], 0.1483900 secs] [Times: user=1.11 sys=0.01, real=0.15 secs] 
[Full GC (Ergonomics) [PSYoungGen: 7908K->7831K(9216K)] [ParOldGen: 8800K->8800K(10240K)] 16708K->16632K(19456K), [Metaspace: 3306K->3306K(1056768K)], 0.1127290 secs] [Times: user=1.05 sys=0.00, real=0.11 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 7831K->7831K(9216K)] [ParOldGen: 8800K->8782K(10240K)] 16632K->16614K(19456K), [Metaspace: 3306K->3306K(1056768K)], 0.1247204 secs] [Times: user=1.12 sys=0.00, real=0.13 secs] 
Heap
 PSYoungGen      total 9216K, used 8016K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 8192K, 97% used [0x00000007bf600000,0x00000007bfdd4150,0x00000007bfe00000)
  from space 1024K, 0% used [0x00000007bff00000,0x00000007bff00000,0x00000007c0000000)
  to   space 1024K, 0% used [0x00000007bfe00000,0x00000007bfe00000,0x00000007bff00000)
 ParOldGen       total 10240K, used 8782K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  object space 10240K, 85% used [0x00000007bec00000,0x00000007bf493aa8,0x00000007bf600000)
 Metaspace       used 3340K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 364K, capacity 388K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at com.jueshan.pulsar.test.oom.HeapOOM.main(HeapOOM.java:15)

```
- 要解决这个区域的异常，一般的手段是先通过内存映象分析工具对Dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底
  是出现了内存泄露（Memory Leak）还是内存溢出（Memory Overflow）
- 如果是内存泄露，可进一步通过工具查看泄露对象到GCRoots引用链。于是就能找到泄露对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们的。
- 如果不存在泄露，那么应当检查虚拟机的堆参数（-Xmx 与 -Xms）


### 虚拟栈和本地方法栈溢出
如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverFlow异常
如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常

> Tips：
> 操作系统分配给每个进程的内存是有限制的，譬如32位的Windows限制为2GB。虚拟机提供了参数来控制Java堆和方法区的这两部分内存的最大值。剩余的内存为
> 2GB（操作系统限制）减去Xmx（最大堆容量），再减去MaxPermSize（最大方法区容量），程序计数器消耗内存很小，忽略不计。
> 开发多线程的应用时特别注意，如果出现StackOverflowError异常有错误堆栈可以阅读，相对来说，比较容易找到问题的所在，但是如果是建立多线程导
> 致的内存溢出，在不能减少线程数或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程。
                                                               
### 方法区和运行时常量池溢出
String.intern()是一个Native方法，它的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象。
否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。

在JDK 1.6中，intern() 方法会把首次遇到的字符串实例复制到永久代中，返回的也是永久代中这个字符串实例的引用，而由StringBuilder创建的字符串实例在
Java堆上，所以必然不是同一个引用，将返回false。

在JDK 1.7中，intern() 方法实现不会再复制实例，只是在常量池中记录首次出现的实例引用，因此intern()返回的引用和由StringBuilder创建的字符串实例
是同一个。

方法区用于存放Class的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。

方法区溢出也是一种常见的内存溢出异常，一个类要被垃圾收集器回收掉，判定条件是比较苛刻的。在经常动态生成大量的Class的应用中，需要特别注意类的回收状况。


### 本机直接内存溢出
DirectMemory容量可通过-XX：MaxDirectMemorySize指定，如果不指定，则默认与Java堆最大值（-Xmx指定）一样。

由DirectMemory导致的内存溢出，一个明显的特性是在Heap Dump文件中不会看见明显的异常。

