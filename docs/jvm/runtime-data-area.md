
### I JDK7 JVM运行时数据区
<img src="_img/jvm/runtime-data-area/JVM运行时数据区.jpg" width="60%"/>

### II JDK8 JVM运行时数据区
<img src="_img/jvm/runtime-data-area/JVM运行时数据区jdk8.jpg" width="60%"/>

### 1 程序计数器

记录当前线程所执行的字节码的行号，获取下一条需要执行的字节码指令。

- 线程私有内存区域
- 没有规定任何`OutOfMemoryError`


### 2 虚拟机栈

描述Java方法执行的内存模型。由若干个栈帧组成，每一个方法在执行的同时都会创建一个栈帧。栈帧用于存储`局部变量表`、`操作数栈`、`动态链接`、`方法出口`等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

- 线程私有内存区域
- 异常：`StackOverflowError`和`OutOfMemoryError`


### 3 本地方法栈

与虚拟机栈功能类似，描述Native方法执行的内存模型。Sun HotSpot虚拟机将本地方法栈和虚拟机栈合二为一。

- 线程私有内存区域
- 异常：`StackOverflowError`和`OutOfMemoryError`

### 4 堆

几乎所有的对象创建都是在这个区域进行内存分配。

这块区域也是垃圾回收器重点管理的区域，由于大多数垃圾回收器都采用`分代回收算法`，堆内存又分为`新生代`、`老年代`，可以方便垃圾的准确回收。

- 线程共享内存区域
- 异常：`OutOfMemoryError`
- 可以通过参数`-Xmx`、`-Xms`控制堆大小


### 5 方法区

用于存储已被虚拟机加载的`类信息`、`常量`、`静态变量`、`即时编译器编译后的代码`等数据。JDK7及以前版本，把GC分代收集扩展至方法区，又叫`永久代`。


- 线程共享内存区域
- 异常：`OutOfMemoryError`
- 可以通过参数`-XX:PermSize`、`-XX:MaxPermSize`控制方法区大小


### 6 元数据区

在`JDK8`中，用元数据区（`Metaspace`）取代了`JDK7`及以前版本的永久代。元数据区和永久代本质上都是JVM规范中方法区的实现。默认情况下元数据区会根据使用情况动态调整，避免了在`JDK7`中由于加载类过多从而出现` java.lang.OutOfMemoryError: PermGen`。

元数据区并不在虚拟机中，而是使用本地内存。因此，元数据区的大小仅受本地内存限制，但可以通过以下参数来指定元空间的大小：
- `-XX:MetaspaceSize`：初始化的Metaspace大小，控制元空间发生GC的阈值，达到该值就会触发垃圾收集进行类型卸载。GC后，同时GC会对该值进行调整，动态增加或降低MetaspaceSize。如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。在默认情况下，这个值大小根据不同的平台在12M到20M浮动。使用Java -XX:+PrintFlagsInitial命令查看本机的初始化参数
- `-XX:MaxMetaspaceSize`：限制Metaspace增长的上限，防止因为某些情况导致Metaspace无限的使用本地内存，影响到其他程序。在本机上该参数的默认值为4294967295B（大约4096MB）。最大空间，默认是没有限制的。
- `-XX:MinMetaspaceFreeRatio`：在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集。当进行过Metaspace GC之后，会计算当前Metaspace的空闲空间比，如果空闲比小于这个参数（即实际非空闲占比过大，内存不够用），那么虚拟机将增长Metaspace的大小。默认值为40，也就是40%。设置该参数可以控制Metaspace的增长的速度，太小的值会导致Metaspace增长的缓慢，Metaspace的使用逐渐趋于饱和，可能会影响之后类的加载。而太大的值会导致Metaspace增长的过快，浪费内存。
- `-XX:MaxMetasaceFreeRatio`：在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集。当进行过Metaspace GC之后， 会计算当前Metaspace的空闲空间比，如果空闲比大于这个参数，那么虚拟机会释放Metaspace的部分空间。默认值为70，也就是70%。
- `MinMetaspaceExpansion`：Metaspace增长时的最小幅度。在本机上该参数的默认值为340784B（大约330KB）。
- `MaxMetaspaceExpansion`：Metaspace增长时的最大幅度。在本机上该参数的默认值为5452592B（大约为5MB）。

### 7 运行时常量池

运行时常量池是方法区的一部分，用来存放Class文件中的符号引用。

### 8 直接内存

直接内存又称为`Direct Memory（堆外内存）`，它并不属于虚拟机管理的内存区域。

NIO可以使用`Native`函数直接分配堆外内存。通过存储在堆中的`DirectByteBuffer` 对象作为这块内存的引用进行操作，避免了堆内存和堆外内存来回复制数据。

既然是内存，那也得是可以被回收的。但由于堆外内存不直接受`JVM`管理，所以常规`GC` 操作并不能回收堆外内存。它是借助于老年代产生的`fullGC`顺便进行回收。同时也可以显式调用 `System.gc()`方法进行回收（前提是没有使用`-XX:+DisableExplicitGC`参数来禁止该方法）。

**值得注意的是**：由于堆外内存也是内存，是由操作系统管理。如果应用有使用堆外内存则需要平衡虚拟机的堆内存和堆外内存的使用占比。避免出现堆外内存溢出。

### 常用参数

<img src="_img/jvm/runtime-data-area/1.jpg" />

通过上图可以直观的查看各个区域的参数设置。

常见的如下：

- `-Xms64m`：最小堆内存64m.
- `-Xmx128m`：最大堆内存128m.
- `-XX:NewSize=30m`：新生代初始化大小为30m.
- `-XX:MaxNewSize=40m`：新生代最大大小为40m.
- `-Xss=256k`：线程栈大小。
- `-XX:+PrintHeapAtGC`：当发生 GC 时打印内存布局。
- `-XX:+HeapDumpOnOutOfMemoryError`：发送内存溢出时dump内存。
- `-XX:NewRatio=2`：设置老年代/新生代的比例。新生代和老年代的默认比例为 1:2，也就是说新生代占用 1/3的堆内存，而老年代占用 2/3 的堆内存。



