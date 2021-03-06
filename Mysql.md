### MySQL 的事务隔离级别有哪些

| 隔离级别                     | 脏读 | 不可重复读 | 幻读 |
| :--------------------------- | ---- | ---------- | ---- |
| 读未提交（Read Uncommitted） | 有   | 有         | 有   |
| 读已提交（Read Committed）   | 无   | 有         | 有   |
| 可重复读（Repeatable Read）  | 无   | 无         | 有   |
| 串行化（Serializable）       | 无   | 无         | 无   |

### 分别用于解决什么问题？

主要用于解决脏读、不可重复读、幻读。

脏读：一个事务读取到另一个事务还未提交的数据。

不可重复读：在一个事务中多次读取同一个数据时，结果出现不一致。

幻读：在一个事务中使用相同的 SQL 两次读取，第二次读取到了其他事务新插入的行。

不可重复读注重于数据的修改，而幻读注重于数据的插入。

### MySQL 的可重复读怎么实现的？

使用 MVCC 实现的，即 Mutil-Version Concurrency Control，多版本并发控制。关于 MVCC，比较常见的说法如下，包括《高性能 MySQL》也是这么介绍的。

InnoDB 在每行记录后面保存两个隐藏的列，分别保存了数据行的创建版本号和删除版本号。每开始一个新的事务，系统版本号都会递增。事务开始时刻的版本号会作为事务的版本号，用来和查询到的每行记录的版本号对比。在可重复读级别下，MVCC是如何操作的：

SELECT：必须同时满足以下两个条件，才能查询到。1）只查版本号早于当前版本的数据行；2）行的删除版本要么未定义，要么大于当前事务版本号。

INSERT：为插入的每一行保存当前系统版本号作为创建版本号。

DELETE：为删除的每一行保存当前系统版本号作为删除版本号。

UPDATE：插入一条新数据，保存当前系统版本号作为创建版本号。同时保存当前系统版本号作为原来的数据行删除版本号。

MVCC 只作用于 RC（Read Committed）和 RR（Repeatable Read）级别，因为 RU（Read Uncommitted）总是读取最新的数据版本，而不是符合当前事务版本的数据行。而 Serializable 则会对所有读取的行都加锁。这两种级别都不需要 MVCC 的帮助。

### 常见的索引类型有哪些？

常见的索引类型有：hash、b树、b+树。

hash：底层就是 hash 表。进行查找时，根据 key 调用hash 函数获得对应的 hashcode，根据 hashcode 找到对应的数据行地址，根据地址拿到对应的数据。

B树：B树是一种多路搜索树，n 路搜索树代表每个节点最多有 n 个子节点。每个节点存储 key + 指向下一层节点的指针+ 指向 key 数据记录的地址。查找时，从根结点向下进行查找，直到找到对应的key。

B+树：B+树是b树的变种，主要区别在于：B+树的非叶子节点只存储 key + 指向下一层节点的指针。另外，B+树的叶子节点之间通过指针来连接，构成一个有序链表，因此对整棵树的遍历只需要一次线性遍历叶子结点即可。

### 为什么MySQL数据库要用B+树存储索引？而不用红黑树、Hash、B树？

红黑树：如果在内存中，红黑树的查找效率比B树更高，但是涉及到磁盘操作，B树就更优了。因为红黑树是二叉树，数据量大时树的层数很高，从树的根结点向下寻找的过程，每读1个节点，都相当于一次IO操作，因此红黑树的I/O操作会比B树多的多。

hash 索引：如果只查询单个值的话，hash 索引的效率非常高。但是 hash 索引有几个问题：1）不支持范围查询；2）不支持索引值的排序操作；3）不支持联合索引的最左匹配规则。

B树索引：B树索相比于B+树，在进行范围查询时，需要做局部的中序遍历，可能要跨层访问，跨层访问代表着要进行额外的磁盘I/O操作；另外，B树的非叶子节点存放了数据记录的地址，会导致存放的节点更少，树的层数变高。


### 简述在MySQL数据库中引擎MyISAM和InnoDB的区别？
存储结构方面：MyISAM每个MyISAM在磁盘上存储成三个文件.frm文件存储表定义、数据文件的扩展名为.MYD、索引文件的扩展名是.MYI；InnoDB所有的表都保存在同一个数据文件中，表的大小只受限于操作系统文件的大小，一般为2GB。

存储空间方面：MyISAM可被压缩，存储空间较小。支持三种不同的存储格式：静态表、动态表、压缩表；InnoDB需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。

事务支持方面：MyISAM强调的是性能，每次查询具有原子性,其执行数度比InnoDB类型更快，但是不提供事务支持；InnoDB提供事务支持事务。

表锁差异方面：MyISAM只支持表级锁；InnoDB支持事务和行级锁。
表主键：MyISAM允许没有任何索引和主键的表存在，索引都是保存行的地址。InnoDB如果没有设定主键或者非空唯一索引，就会自动生成一个6字节的主键。

增删改查操作方面：如果执行大量的SELECT，MyISAM是更好的选择。如果执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表。
外键方面：MyISAM不支持；InnoDB支持。

### MySQL中有哪几种锁？
表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。

行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

 ### 简单描述 MySQL 中索引、主键、唯一索引、联合索引的区别，对数据库的性能有什么影响（从读写两方面）？
索引是一种特殊的文件，包含着对数据表里所有记录的引用指针。索引可以极大的提高数据的查询速度，但是会降低插入、删除、更新表的速度，因为在执行这些写操作时，还要操作索引文件。
普通索引的唯一任务是加快对数据的访问速度，允许被索引的数据列包含重复的值。
如果能确定某个数据列将只包含彼此各不相同的值，在为这个数据列创建索引的时候就应该用关键字 UNIQUE把它定义为一个唯一索引，唯一索引可以保证数据记录的唯一性。
主键是一种特殊的唯一索引，在一张表中只能定义一个主键索引，主键用于唯一标识一条记录，使用关键字PRIMARY KEY来创建。
联合索引可以覆盖多个数据列，如像INDEX(columnA, columnB)索引。
