### 介绍下 HashMap 的底层数据结构吧。

在 JDK 1.8，HashMap 底层是由 “数组+链表+红黑树” 组成，如下图所示，而在 JDK 1.8 之前是由 “数组+链表” 组成，就是下图去掉红黑树。

![image-20211102221501259](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211102221501259.png)

### 为什么要改成“数组+链表+红黑树”？
通过上题可以看出，“数组+链表” 已经充分发挥了这两种数据结构的优点，是个很不错的组合了。

但是这种组合仍然存在问题，就是在定位到索引位置后，需要先遍历链表找到节点，这个地方如果链表很长的话，也就是 hash 冲突很严重的时候，会有查找性能问题，因此在 JDK1.8中，通过引入红黑树，来优化这个问题。

使用链表的查找性能是 O(n)，而使用红黑树是 O(logn)。
### 那在什么时候用链表？什么时候用红黑树？
对于插入，默认情况下是使用链表节点。当同一个索引位置的节点在新增后超过8个（阈值8）：如果此时数组长度大于等于 64，则会触发链表节点转红黑树节点（treeifyBin）；而如果数组长度小于64，则不会触发链表转红黑树，而是会进行扩容，因为此时的数据量还比较小。

对于移除，当同一个索引位置的节点在移除后达到 6 个（阈值6），并且该索引位置的节点为红黑树节点，会触发红黑树节点转链表节点（untreeify）。
### 为什么链表转红黑树的阈值是8？
我们平时在进行方案设计时，必须考虑的两个很重要的因素是：时间和空间。对于 HashMap 也是同样的道理，简单来说，阈值为8是在时间和空间上权衡的结果。红黑树节点大小约为链表节点的2倍，在节点太少时，红黑树的查找性能优势并不明显，付出2倍空间的代价作者觉得不值得。

理想情况下，使用随机的哈希码，节点分布在 hash 桶中的频率遵循泊松分布，按照泊松分布的公式计算，链表中节点个数为8时的概率为 0.00000006（就我们这QPS不到10的系统，根本不可能遇到嘛），这个概率足够低了，并且到8个节点时，红黑树的性能优势也会开始展现出来，因此8是一个较合理的数字

### 那为什么转回链表节点是用的6而不是复用8？

如果我们设置节点多于8个转红黑树，少于8个就马上转链表，当节点个数在8徘徊时，就会频繁进行红黑树和链表的转换，造成性能的损耗。

### HashMap 有哪些重要属性？分别用于做什么的？
除了用来存储我们的节点 table 数组外，HashMap 还有以下几个重要属性：

1）size：HashMap 已经存储的节点个数；

2）threshold：1）扩容阈值（主要），当 HashMap 的个数达到该值，触发扩容。2）初始化时的容量，在我们新建 HashMap 对象时， threshold 还会被用来存初始化时的容量。HashMap 直到我们第一次插入节点时，才会对 table 进行初始化，避免不必要的空间浪费。

3）loadFactor：负载因子，扩容阈值 = 容量 * 负载因子。

### HashMap 的默认初始容量是多少？HashMap 的容量有什么限制吗？

默认初始容量是16。HashMap 的容量必须是2的N次方，HashMap 会根据我们传入的容量计算一个“大于等于该容量的最小的 2 的 N 次方”，例如传 16，容量为16；传17，容量为32。

### 总结下 JDK 1.8 主要进行了哪些优化？
JDK 1.8 的主要优化刚才我们都聊过了，主要有以下几点：

1）底层数据结构从“数组+链表”改成“数组+链表+红黑树”，主要是优化了 hash 冲突较严重时，链表过长的查找性能：O(n) -> O(logn)。

2）计算 table 初始容量的方式发生了改变，老的方式是从1开始不断向左进行移位运算，直到找到大于等于入参容量的值；新的方式则是通过“5个移位+或等于运算”来计算。

```java
public HashMap(int initialCapacity, float loadFactor) {
    // 省略
    // Find a power of 2 >= initialCapacity
    int capacity = 1;
    while (capacity < initialCapacity)
        capacity <<= 1;
    // ... 省略
}
// JDK 1.8.0_191
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

3）优化了 hash 值的计算方式，老的通过一顿瞎JB操作，新的只是简单的让高16位参与了运算。

4）扩容时插入方式从“头插法”改成“尾插法”，避免了并发下的死循环。

5）扩容时计算节点在新表的索引位置方式从“h & (length-1)”改成“hash & oldCap”，性能可能提升不大，但设计更巧妙、更优雅。

### Hashtable 是怎么加锁的 ？
Hashtable 通过 synchronized 修饰方法来加锁，从而实现线程安全

```java
public synchronized V put(K key, V value) {
       // ...
}
public synchronized V put(K key, V value) {
    // ...
} 
```

### LinkedHashMap 和 TreeMap 排序的区别？

LinkedHashMap 和 TreeMap 都是提供了排序支持的 Map，区别在于支持的排序方式不同：

LinkedHashMap：保存了数据的插入顺序，也可以通过参数设置，保存数据的访问顺序。

TreeMap：底层是红黑树实现。可以指定比较器（Comparator 比较器），通过重写 compare 方法来自定义排序；如果没有指定比较器，TreeMap 默认是按 Key 的升序排序（如果 key 没有实现 Comparable接口，则会抛异常）。
### HashMap 和 Hashtable 的区别？
HashMap 允许 key 和 value 为 null，Hashtable 不允许。

HashMap 的默认初始容量为 16，Hashtable 为 11。

HashMap 的扩容为原来的 2 倍，Hashtable 的扩容为原来的 2 倍加 1。

HashMap 是非线程安全的，Hashtable 是线程安全的，使用 synchronized 修饰方法实现线程安全。

HashMap 的 hash 值重新计算过，Hashtable 直接使用 hashCode。

HashMap 去掉了 Hashtable 中的 contains 方法。

HashMap 继承自 AbstractMap 类，Hashtable 继承自 Dictionary 类。

HashMap 的性能比 Hashtable 高，因为 Hashtable 使用 synchronized 实现线程安全，还有就是 HashMap 1.8 之后底层数据结构优化成 “数组+链表+红黑树”，在极端情况下也能提升性能。

### 介绍下 ConcurrenHashMap，要讲出 1.7 和 1.8 的区别？
ConcurrentHashMap 是 HashMap 的线程安全版本，和 HashMap 一样，在JDK 1.8 中进行了较大的优化。

JDK1.7：底层结构为：分段的数组+链表；实现线程安全的方式：分段锁（Segment，继承了ReentrantLock），如下图所示。

![image-20211110212818782](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211110212818782.png)

JDK1.8：底层结构为：数组+链表+红黑树；实现线程安全的方式：CAS + Synchronized

![image-20211110212828749](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211110212828749.png)

区别：

1、JDK1.8 中降低锁的粒度。JDK1.7 版本锁的粒度是基于 Segment 的，包含多个节点（HashEntry），而 JDK1.8 锁的粒度就是单节点（Node）。

concurrentHashmap用的是单节点锁，HashTable用的是全表锁

2、JDK1.8 版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用 synchronized 来进行同步，所以不需要分段锁的概念，也就不需要 Segment 这种数据结构了，当前还保留仅为了兼容。

3、JDK1.8 使用红黑树来优化链表，跟 HashMap 一样，优化了极端情况下，链表过长带来的性能问题。

4、JDK1.8 使用内置锁 synchronized 来代替重入锁 ReentrantLock，synchronized 是官方一直在不断优化的，现在性能已经比较可观，也是官方推荐使用的加锁方式。
