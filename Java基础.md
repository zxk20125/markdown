

#                                                 JAVA基础

![image-20211027000827192](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211027000827192.png)

其实，s1 += 1 相当于 s1 = (short)(s1 + 1)，有兴趣的可以自己编译下这两行代码的字节码，你会发现是一模一样的。



![image-20211027000856675](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211027000856675.png)

答案是：false，true。

执行 Integer a = 128，相当于执行：Integer a = Integer.valueOf(128)，基本类型自动转换为包装类的过程称为自动装箱（autoboxing）。

在 Integer 中引入了 IntegerCache 来缓存一定范围的值，IntegerCache 默认情况下范围为：-128~127。

本题中的 127 命中了 IntegerCache，所以 c 和 d 是相同对象，而 128 则没有命中，所以 a 和 b 是不同对象。

但是这个缓存范围时可以修改的，可能有些人不知道。可以通过JVM启动参数：-XX:AutoBoxCacheMax=<size> 来修改上限值，如下图所示：
![image-20211027000948307](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211027000948307.png)

## String 是 Java 基本数据类型吗？

答：不是。Java 中的基本数据类型只有8个：byte、short、int、long、float、double、char、boolean；除了基本类型（primitive type），剩下的都是引用类型（reference type）。

基本数据类型：数据直接存储在栈上

引用数据类型区别：数据存储在堆上，栈上只存储引用地址


## String和StringBuilder、StringBuffer的区别？
String：String 的值被创建后不能修改，任何对 String 的修改都会引发新的 String 对象的生成。

StringBuffer：跟 String 类似，但是值可以被修改，使用 synchronized 来保证线程安全。

StringBuilder: StringBuffer 的非线程安全版本，没有使用 synchronized，具有更高的性能，推荐优先使用。

## == 和 equals 的区别是什么？

==：运算符，用于比较基础类型变量和引用类型变量。

对于基础类型变量，比较的变量保存的值是否相同，类型不一定要相同。

```java
short s1 = 1; long l1 = 1;
// 结果：true。类型不同，但是值相同
System.out.println(s1 == l1);
```

对于引用类型变量，比较的是两个对象的地址是否相同。

```java
Integer i1 = new Integer(1);
Integer i2 = new Integer(1);
// 结果：false。通过new创建，在内存中指向两个不同的对象
System.out.println(i1 == i2);
```

equals：Object 类中定义的方法，通常用于比较两个对象的值是否相等。

equals 在 Object 方法中其实等同于 ==，但是在实际的使用中，equals 通常被重写用于比较两个对象的值是否相同。

```java
Integer i2 = new Integer(1);
// 结果：true。两个不同的对象，但是具有相同的值
System.out.println(i1.equals(i2));
 
// Integer的equals重写方法
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        // 比较对象中保存的值是否相同
        return value == ((Integer)obj).intValue();
    }
    return false;
```



## Java 静态变量和成员变量的区别。

```java
public class Demo {
    /**
     * 静态变量：又称类变量，static修饰
     */
    public static String STATIC_VARIABLE = "静态变量";
    /**
     * 实例变量：又称成员变量，没有static修饰
     */
    public String INSTANCE_VARIABLE = "实例变量";
}

```

成员变量存在于堆内存中。静态变量存在于方法区中。

成员变量与对象共存亡，随着对象创建而存在，随着对象被回收而释放。静态变量与类共存亡，随着类的加载而存在，随着类的消失而消失。

成员变量所属于对象，所以也称为实例变量。静态变量所属于类，所以也称为类变量。

成员变量只能被对象所调用 。静态变量可以被对象调用，也可以被类名调用。

## try、catch、finally 请指出下面程序的运行结果。
```java
    public static void main(String[] args) {
        System.out.println(test1());
    }
    public static int test1() {
        int i = 0;
        try {
            i = 2;
            return i;
        } finally {
            i = 3;
        }
    }
}
```

执行结果：2。

## JDK1.8之后有哪些新特性？
接口默认方法：Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 default关键字即可

Lambda 表达式和函数式接口：Lambda 表达式本质上是一段匿名内部类，也可以是一段可以传递的代码。Lambda 允许把函数作为一个方法的参数（函数作为参数传递到方法中），使用 Lambda 表达式使代码更加简洁，但是也不要滥用，否则会有可读性等问题，《Effective Java》作者 Josh Bloch 建议使用 Lambda 表达式最好不要超过3行。

Stream API：用函数式编程方式在集合类上进行复杂操作的工具，配合Lambda表达式可以方便的对集合进行处理。Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。简而言之，Stream API 提供了一种高效且易于使用的处理数据的方式。

方法引用：方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

日期时间API：Java 8 引入了新的日期时间API改进了日期时间的管理。

Optional 类：著名的 NullPointerException 是引起系统失败最常见的原因。很久以前 Google Guava 项目引入了 Optional 作为解决空指针异常的一种方式，不赞成代码被 null 检查的代码污染，期望程序员写整洁的代码。受Google Guava的鼓励，Optional 现在是Java 8库的一部分。

新工具：新的编译工具，如：Nashorn引擎 jjs、 类依赖分析器 jdeps。

## wait() 和 sleep() 方法的区别
来源不同：sleep() 来自 Thread 类，wait() 来自 Object 类。

对于同步锁的影响不同：sleep() 不会该表同步锁的行为，如果当前线程持有同步锁，那么 sleep 是不会让线程释放同步锁的。wait() 会释放同步锁，让其他线程进入 synchronized 代码块执行。

使用范围不同：sleep() 可以在任何地方使用。wait() 只能在同步控制方法或者同步控制块里面使用，否则会抛 IllegalMonitorStateException。

恢复方式不同：两者会暂停当前线程，但是在恢复上不太一样。sleep() 在时间到了之后会重新恢复；wait() 则需要其他线程调用同一对象的 notify()/nofityAll() 才能重新恢复。

## 线程的 sleep() 方法和 yield() 方法有什么区别？
线程执行 sleep() 方法后进入超时等待（TIMED_WAITING）状态，而执行 yield() 方法后进入就绪（READY）状态。

sleep() 方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程运行的机会；yield() 方法只会给相同优先级或更高优先级的线程以运行的机会。

### 线程的状态流转

![image-20211101214932125](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211101214932125.png)

NEW：新建但是尚未启动的线程处于此状态，没有调用 start() 方法。

RUNNABLE：包含就绪（READY）和运行中（RUNNING）两种状态。线程调用 start() 方法会会进入就绪（READY）状态，等待获取 CPU 时间片。如果成功获取到 CPU 时间片，则会进入运行中（RUNNING）状态。

BLOCKED：线程在进入同步方法/同步块（synchronized）时被阻塞，等待同步锁的线程处于此状态。

WAITING：无限期等待另一个线程执行特定操作的线程处于此状态，需要被显示的唤醒，否则会一直等待下去。例如对于 Object.wait()，需要等待另一个线程执行 Object.notify() 或 Object.notifyAll()；对于 Thread.join()，则需要等待指定的线程终止。

TIMED_WAITING：在指定的时间内等待另一个线程执行某项操作的线程处于此状态。跟 WAITING 类似，区别在于该状态有超时时间参数，在超时时间到了后会自动唤醒，避免了无期限的等待。

TERMINATED：执行完毕已经退出的线程处于此状态。

线程在给定的时间点只能处于一种状态。这些状态是虚拟机状态，不反映任何操作系统线程状态。

## 为什么要使用线程池？直接new个线程不是很舒服？
如果我们在方法中直接new一个线程来处理，当这个方法被调用频繁时就会创建很多线程，不仅会消耗系统资源，还会降低系统的稳定性，一不小心把系统搞崩了，就可以直接去财务那结帐了。

如果我们合理的使用线程池，则可以避免把系统搞崩的窘境。总得来说，使用线程池可以带来以下几个好处：

降低资源消耗。通过重复利用已创建的线程，降低线程创建和销毁造成的消耗。
提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
增加线程的可管理型。线程是稀缺资源，使用线程池可以进行统一分配，调优和监控。
## 线程池的核心属性有哪些？
threadFactory（线程工厂）：用于创建工作线程的工厂。

corePoolSize（核心线程数）：当线程池运行的线程少于 corePoolSize 时，将创建一个新线程来处理请求，即使其他工作线程处于空闲状态。

workQueue（队列）：用于保留任务并移交给工作线程的阻塞队列。

maximumPoolSize（最大线程数）：线程池允许开启的最大线程数。

handler（拒绝策略）：往线程池添加任务时，将在下面两种情况触发拒绝策略：

1）线程池运行状态不是 RUNNING；2）线程池已经达到最大线程数，并且阻塞队列已满时。

keepAliveTime（保持存活时间）：如果线程池当前线程数超过 corePoolSize，则多余的线程空闲时间超过 keepAliveTime 时会被终止

## Stream

- ### 为什么需要Stream ?

　　Stream作为Java8的一大亮点，它与java.io包里的InputStream和OutputStream是完全不同的概念。它也不同于StAX对XML解析的Stream，也不是Amazon Kinesis对大数据实时处理的Stream。Java8中的Stream是对容器对象功能的增强，它专注于对容器对象进行各种非常便利、高效的 **聚合操作（aggregate operation）**，或者大批量数据操作 (bulk data operation)。Stream API借助于同样新出现的Lambda表达式，极大的提高编程效率和程序可读性。同时，它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用fork/join并行方式来拆分任务和加速处理过程。通常，编写并行代码很难而且容易出错, 但使用Stream API无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。所以说，Java8中首次出现的 **java.util.stream是一个函数式语言+多核时代综合影响的产物。**

- ### 什么是聚合操作

　　在传统的J2EE应用中，Java代码经常不得不依赖于关系型数据库的聚合操作来完成诸如：

- 客户每月平均消费金额
- 最昂贵的在售商品
- 本周完成的有效订单（排除了无效的）
- 取十个数据样本作为首页推荐

这类的操作。但在当今这个数据大爆炸的时代，在数据来源多样化、数据海量化的今天，很多时候不得不脱离 RDBMS，或者以底层返回的数据为基础进行更上层的数据统计。而Java的集合API中，仅仅有极少量的辅助型方法，更多的时候是程序员需要用Iterator来遍历集合，完成相关的聚合应用逻辑，这是一种远不够高效、笨拙的方法。在Java7中，如果要发现type为grocery的所有交易，然后返回以交易值降序排序好的交易ID集合，我们需要这样写：

```java
List<Transaction> groceryTransactions = new Arraylist<>();
for(Transaction t: transactions){
 if(t.getType() == Transaction.GROCERY){
 groceryTransactions.add(t);
 }
}

Collections.sort(groceryTransactions, new Comparator(){
 public int compare(Transaction t1, Transaction t2){
 return t2.getValue().compareTo(t1.getValue());
 }
});

List<Integer> transactionIds = new ArrayList<>();
for(Transaction t: groceryTransactions){
 transactionsIds.add(t.getId());
}
```

　　而在 Java 8 使用 Stream，代码更加简洁易读；而且使用并发模式，程序执行速度更快。

```java
List<Integer> transactionsIds = transactions.parallelStream()
.filter(t -> t.getType() == Transaction.GROCERY)
.sorted(comparing(Transaction::getValue).reversed())
.map(Transaction::getId).collect(toList());
```

### 什么是流？

　　Stream不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的Iterator。原始版本的Iterator，用户只能显式地一个一个遍历元素并对其执行某些操作；高级版本的Stream，用户只要给出需要对其包含的元素执行什么操作，比如，“过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，Stream会隐式地在内部进行遍历，做出相应的数据转换。Stream就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。

　　而和迭代器又不同的是，Stream可以并行化操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方式去遍历时，每个item读完后再读下一个item。**而使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。**Stream的并行操作依赖于Java7中引入的Fork/Join框架（JSR166y）来拆分任务和加速处理过程。Stream 的另外一大特点是，数据源本身可以是无限的。

### 流的构成

当我们使用一个流的时候，通常包括三个基本步骤：获取一个数据源（source）→ 数据转换 → 执行操作获取想要的结果。**每次转换原有Stream对象不改变，返回一个新的Stream对象（可以有多次转换）**，这就允许对其操作可以像链条一样排列，变成一个管道，如下图所示:

![image-20211101223725090](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211101223725090.png)

### 为什么要改成“数组+链表+红黑树”？

主要是为了提升在 hash 冲突严重时（链表过长）的查找性能，使用链表的查找性能是 O(n)，而使用红黑树是 O(logn)。

### 那在什么时候用链表？什么时候用红黑树？
对于插入，默认情况下是使用链表节点。当同一个索引位置的节点在新增后超过8个（阈值8）：如果此时数组长度大于等于 64，则会触发链表节点转红黑树节点（treeifyBin）；而如果数组长度小于64，则不会触发链表转红黑树，而是会进行扩容，因为此时的数据量还比较小。

对于移除，当同一个索引位置的节点在移除后达到 6 个，并且该索引位置的节点为红黑树节点，会触发红黑树节点转链表节点（untreeify）。

### HashMap 的默认初始容量是多少？HashMap 的容量有什么限制吗？

默认初始容量是16。HashMap 的容量必须是2的N次方，HashMap 会根据我们传入的容量计算一个大于等于该容量的最小的2的N次方，例如传 9，容量为16。

### HashMap 的插入流程是怎么样的？

![image-20211102210021866](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211102210021866.png)

### HashMap 的扩容（resize）流程是怎么样的？

![image-20211102210138269](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211102210138269.png)

### 除了 HashMap，还用过哪些 Map，在使用时怎么选择？

![image-20211102210254176](C:\Users\Chick\AppData\Roaming\Typora\typora-user-images\image-20211102210254176.png)

### HashMap 和Hashtable 的区别?
HashMap 允许 key 和 value 为 null，Hashtable 不允许。

HashMap 的默认初始容量为 16，Hashtable 为 11。

HashMap 的扩容为原来的 2 倍，Hashtable 的扩容为原来的 2 倍加 1。

HashMap 是非线程安全的，Hashtable是线程安全的。

HashMap 的 hash 值重新计算过，Hashtable 直接使用 hashCode。

HashMap 去掉了 Hashtable 中的 contains 方法。

HashMap 继承自 AbstractMap 类，Hashtable 继承自 Dictionary 类。

### Java 内存结构（运行时数据区）
程序计数器：线程私有。一块较小的内存空间，可以看作当前线程所执行的字节码的行号指示器。如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Native方法，这个计数器值则为空。

Java虚拟机栈：线程私有。它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

本地方法栈：线程私有。本地方法栈与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。

Java堆：线程共享。对大多数应用来说，Java堆是Java虚拟机所管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。

方法区：与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息（构造方法、接口定义）、常量、静态变量、即时编译器编译后的代码（字节码）等数据。方法区是JVM规范中定义的一个概念，具体放在哪里，不同的实现可以放在不同的地方。

运行时常量池：运行时常量池是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

```String str = new String("hello");```
上面的语句中变量 str 放在栈上，用 new 创建出来的字符串对象放在堆上，而"hello"这个字面量是放在堆中。

### 什么是双亲委派模型？
如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。

### Java虚拟机中有哪些类加载器？
#### 启动类加载器（Bootstrap ClassLoader）：

这个类加载器负责将存放在<JAVA_HOME>\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。

#### 扩展类加载器（Extension ClassLoader）：

这个加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载<JAVA_HOME>\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。

#### 应用程序类加载器（Application ClassLoader）：

这个类加载器由sun.misc.Launcher$AppClassLoader实现。由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，所以一般也称它为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

#### 自定义类加载器：

用户自定义的类加载器。

### 介绍下垃圾收集机制（在什么时候，对什么，做了什么）？
在什么时候？

在触发GC的时候，具体如下，这里只说常见的 Young GC 和 Full GC。

触发Young GC：当新生代中的 Eden 区没有足够空间进行分配时会触发Young GC。

触发Full GC：

- 当准备要触发一次Young GC时，如果发现统计数据说之前Young GC的平均晋升大小比目前老年代剩余的空间大，则不会触发Young GC而是转为触发Full GC。（通常情况）
- 如果有永久代的话，在永久代需要分配空间但已经没有足够空间时，也要触发一次Full GC。
- System.gc()默认也是触发Full GC。
- heap dump带GC默认也是触发Full GC。
- CMS GC时出现Concurrent Mode Failure会导致一次Full GC的产生。
  对什么？

对那些JVM认为已经“死掉”的对象。即从GC Root开始搜索，搜索不到的，并且经过一次筛选标记没有复活的对象。

做了什么？

对这些JVM认为已经“死掉”的对象进行垃圾收集，**新生代使用复制算法，老年代使用标记-清除和标记-整理算法。**

### 垃圾收集有哪些算法，各自的特点？

#### 标记 - 清除算法

首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。它的主要不足有两个：一个是效率问题，标记和清除两个过程的效率都不高；另一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行过程中需要分配较大对象时，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

#### 复制算法

为了解决效率问题，一种称为“复制”（Copying）的收集算法出现了，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。这样使得每次都是对整个半区进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为了原来的一半，未免太高了一点。

#### 标记 - 整理算法

复制收集算法在对象存活率较高时就要进行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

#### 分代收集算法

当前商业虚拟机的垃圾收集都采用“分代收集”（Generational Collection）算法，这种算法并没有什么新的思想，只是根据对象存活周期的不同将内存划分为几块。

一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。

在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。

在老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用标记—清理或者标记—整理算法来进行回收。
