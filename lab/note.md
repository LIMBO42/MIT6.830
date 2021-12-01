### MIT6.830 

### 

#### LAB1



1. 实现TupleDesc的相关接口：

   TupleDesc中最关键的成员是TDItem[]，TDItem记录了Type和Name，就是相当于表中第一行，设定每一列对应的Type和Name。

2. 实现Tuple相关接口：

   Tuple对应着数据库中的一行，私有成员变量有当前的TupleDesc，存储一行数据的Field[]等。

3. 实现Catalog接口：

   Catalog记录了所有的表格和相关的信息。

   利用hashmap<Integer, Table>来记录id和table的映射关系；

   table是自己建立的一个类，其中有成员变量DbFile(用来存储table的contents)，tableName，pKFeild(主键)。

4. BufferPool：

   将活跃的最近从硬盘读出的pages保存在内存中。读写操作通过buffer pool读写硬盘上不同文件，在之后的lab中需要实现淘汰机制。

   <img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211031151821397.png" alt="image-20211031151821397" style="zoom:67%;" />

   getPage的目的是实现一个类似于cache的东西，如果不存在，加入hashmap并返回，如果内存不够，需要淘汰机制淘汰掉某些page。当然这里利用Database.getCatalog().getDatabaseFile获得了相关的信息。

5. heapPage：

   页面大小固定，数据库中每个表对应一个HeapFile对象，**HeapFile包含一组页面**，每一页中的slot对应于一行。物理页还有一个Header，Header的作用就是bitmap，每一位映射一行，代表这一行是否有效。

   每个Tuple对应于一个slot，每个Tuple所需要的Tuple size * 8bits+1bit，因此一页所能拥有的Slot/Tuple的个数为：

   _tuples per page_ = floor((_page size_ * 8) / (_tuple size_ * 8 + 1))

   而header所需要的大小为：

   *headerBytes = ceiling(tupsPerPage/8)*

   除以8的原因是bits到bytes的转化。

6. HeapFile：

   File是物理意义的file，通过java的File接口读取，怎么读取呢？根据pageNO计算对应的位置，读取pagesize大小的数据。

   iterator()需要创建一个迭代器，能够遍历heapFile的所有的tuples（这个难度会大一些）
   
   需要实现Opiterator接口：hasNext和Next，这里的Next获得下一个Tuple，而hashNext在Next之前都会调用，所以利用它去移动HeapPage的页数。
   
7. SeqScan：

   包装了一下HeapFile，本质上还是去读Tuples



![image-20211112140128511](https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112140128511.png)



![image-20211112140146449](https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112140146449.png)



![image-20211112140204233](https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112140204233.png)

#### Exercise 1：

一个Tuple由三部分组成：

<img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112140342718.png" alt="image-20211112140342718" style="zoom:67%;" />

1. 其中TupleDesc为表中第一列的信息（表头）：

<img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112140758051.png" alt="image-20211112140758051" style="zoom: 50%;" />

​	由TDitem[]构成，其中TDitem存储当前列的相关信息（fieldType，fieldName）



2. Field[]数组存储字段信息。

   RecorID由两部分组成：![image-20211112140528419](https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112140528419.png)

   

3. 即该Tuple所属的pageID和在当前页面的tupleno。



#### Exercise 2：

Catalog存储若干表（hashmap），每张表由DbFile存储，这里就是实现一个Table类。

<img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112141459835.png" alt="image-20211112141459835" style="zoom: 67%;" />

可以看到 一张 Table需要Dbfile存储内容，tablename，和keyField作为主键。

这里的hashmap<>使用DbFile的getId进行散列。



#### Exercise 3：

BufferPool在内存中存了20页Page

<img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112143022077.png" alt="image-20211112143022077" style="zoom: 67%;" />

主要是实现getPage方法。根据PageId来查找BufferPool中有无该数据页，如果有直接返回，如果没有就从磁盘中获取。

`Page page=Database.getCatalog().getDatabaseFile(pid.getTableId()).readPage(pid);`

存入BufferPool中。其中BufferPool的容器是用map来存Page的，以pid的hashcode与Page建立映射关系。



#### Exercise 4：

主要涉及HeapPage，即Tuples是如何组织的。

一页Page大小固定，通过Tuple的大小计算该页能存储的Tuple是的数目。

一条Tuple在HeapPage中表示为一个slot，使用header以bitmap的形式来存储，表示第i个slot是否被使用。

然后是实现一个迭代器，依次返回有效记录。

<img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211112143911158.png" alt="image-20211112143911158" style="zoom:67%;" />



#### Exercise 5：

这里涉及到HeapFile：HeapFile要求我们实现一个readPage方法，根据PageId去从磁盘中读取数据。这里主要的难点是计算偏移量然后读取数据，这里需要先计算出offset，然后用seek移动指针，然后再开始读取。

然后需要实现一个迭代器，遍历HeapFile中所有的Tuples。这里需要先从BufferPool中读取，读取不到再去磁盘中读。







#### LAB2

- [ ] 实现Filter和Join操作符

- [ ] 实现IntegerAggregator和StringAggregator。编写逻辑，计算输入元组序列中多个组的特定字段的聚合。使用整数除法计算平均值，因为SimpleDB只支持整数。StringAggegator只需要支持COUNT聚合，因为其他操作对字符串没有意义。

- [ ] 实现Aggregate操作符。与其他操作符一样，聚合实现了OpIterator接口，这样它们就可以放在SimpleDB查询计划中。注意，对于每次调用next()， Aggregate操作符的输出是整个组的聚合值，而聚合构造函数接受聚合和分组字段。

- [ ] 在BufferPool中实现与元组插入、删除和页面回收相关的方法。不需要担心事务。

- [ ] 实现插入和删除操作符。与所有操作符一样，Insert和Delete操作符实现OpIterator，接受元组流来插入或删除，并输出带有整数字段的单个元组，该字段指示插入或删除的元组数量。这些操作符需要调用BufferPool中实际修改磁盘上页面的适当方法。




**Filter：**

按照规则Predict过滤满足条件的Tuple，实现主要是通过tuple iterator遍历，筛选出满足条件的。

**Join：**

表的联结，本质上是两个表做笛卡尔积，然后挑选出满足条件的tuple。

实现是固定表一的某个tuple，遍历表二的所有tuple，根据joinpredict进行处理。



**IntegerAggregate：**

两个函数：mergeTupleIntoGroup和iterator

创建hashmap，按照group的元素进行分类；每次merge一个tuple处理一次。

iterator将hashmap中的内容重新输入进arraylist



**Aggregate：**

需要注意的是：

<img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211110210854157.png" alt="image-20211110210854157" style="zoom:67%;" />

child是迭代器，依次喂入数据，因此aggregator才是一个动态的过程，每次输入一个数，改变对应的hashmap中对应的值。

![image-20211111082450099](https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211111082450099.png)



**HeapPage中插入删除元组：**

HeapPage中有tuples[]，用来存储页面所有的tuple

<img src="https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211111090939205.png" alt="image-20211111090939205" style="zoom:67%;" />



![image-20211111091023669](https://typora-1306385380.cos.ap-nanjing.myqcloud.com/img/image-20211111091023669.png)







**加锁的思路：**

锁管理器**pageLocks = hashmap<PageID, page存储锁的hashmap >**

每个页面存储锁的结构是 **pageLock = hashmap<TransactionID, PageLock>**：

即不同的交易会在同一个Page上施加锁，但是有一个原则，一个Page只能有一个X锁，S锁倒是可以有多个。

1. 如果当前页面的pageLock为空 或者锁管理器为空，此时需要什么锁就加什么锁，分别加入pageLock和pageLocks。

2. 获取当前页面的hashmap：**pageLock**，获取transaction对应的Lock。

3. 如果**Lock != null**，说明当前Transaction有锁：

   - 如果需要S，但是Lock.type为X，满足条件，如果Lock.type为S也满足条件：X>S。
   - 如果需要X，由于一个页面只能有一个X，那么去看pageLock的大小，如果大于1，说明全都是S，（如果有X锁，pageLock的大小只能为1）此时死锁（这里的死锁有点不知道为什么）；如果pageLock==1，那么说明pageLock该页面的唯一的锁就是当前Transaction的锁，去看这个锁的type，如果是X，满足，如果是S，需要升级该锁。

4. 如果**lock==null**，当前Transaction没有锁，需要加锁：

   - 如果需要S锁，去看pageLock大小，如果大于1，随便加，如果等于1，去看pageLock的那唯一一个锁，如果那个锁为S，随便加，如果为X，需要wait。
   - 如果需要X锁，多个S，一个其他Transaction的X，这两种情况都需要wait。

   

