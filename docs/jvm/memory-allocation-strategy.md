

### 1 对象优先在Eden分配

大多数情况下，对象在新生代Eden区分配。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。

### 2 大对象直接进入老年代

所谓大对象是指，需要大量连续内存空间的Java对象，最典型的大对象就是那种很长的字符串以及数组。
大对象对虚拟机的内存分配是一个坏消息，而更坏的消息是，存在很多短命的大对象。

`-XX:PretenureSizeThreshold` 令大于这个设置值的对象直接在老年代分配。避免在eden和两个survivor发生大量的内存赋复制。只对Serial和ParNew有效。该值默认为0，表示不论多大对象，都先在Eden区分配。


### 3 长期存活的对象将进入老年代

如果对象在eden区创建，并经过第一次minor gc后仍然存活，并且能被servivor容纳，将被移动到survivor中，并且年龄设置为1。以后在survivor中每经过一次minor gc ，年龄就加1，当年龄超过一定程度（默认15），就会晋升到老年代中。
可以使用`-XX:MaxTenuringThreshold`设置年龄阈值。



### 4 动态对象年龄判定
survivor中同龄的所有对象大小总和大于survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。无需等到MaxTenuringThreshold中要求的年龄阈值。




### 5 空间分配担保

jdk1.6 u24前
在发生minor gc前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间
- 条件成立，那么minor gc可以确保安全。
- 不成立，则虚拟机会查看HandlePromotionFailure设置值是否允许担保失败。
	- 允许，那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小
		- 大于，将尝试着进行一次minor gc，尽管这次minor gc是有风险的
		- 小于或者HandlePromotionFailure设置不允许冒险，此时改为进行一次Full GC。

jdk1.6 u24后
只要老年代连续空间大于新生代对象总大小或历次晋升的平均大小就会进行minor gc，否则进行Full GC。










