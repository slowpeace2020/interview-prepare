# Chapter3 Storage and retrieval
> turns to the internals of storage engines and looks at how databases lay
out data on disk. Different storage engines are optimized for different workloads,
and choosing the right one can have a huge effect on performance.

how we can store the data that we're given, and how we can find it again when we're asked for it.
how the database handles storage and  retrieval  internally? 

两种storage engine：log-structured engines, and page-oriented engines such as B-trees

log: an append-only sequence of records

## Hash index
trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes. You can choose the indexes that give your application the greatest benefit, without introducing more overhead than necessary.
Bitcask适合存储key-value，假如有很多写入，但是distinct key少，append-only的写入方式会很快耗尽磁盘空间，一个好的解决方法是break the log into segments of a certain size,当这个segment file的size到达一定值的时候，subsequent write to a new segment file.然后我们可以perform compaction on these segments, 在这里意味着throw away duplicate keys in the log, keeping only the most recent update for each key.
因为compaction会让segments变小，我们可以在compaction的同时merge several segments together. 这个merging和compaction的步骤可以在background thread完成，当它还在进行中我们继续在原来的segments中读和写，当它们完成之后我们把读和写操作转换到新segments，删除旧的segments。

append-only log的优势是什么？
append和segment merge是sequential write, 这比random write快，concurrency和crash recovery也比较简单，因为segment files是append-only或者immutable，通过合并old segment避免数据文件碎片化的问题。

但是hash table index也有limitation：
hash table必须是内存能装下的，不然performance表现也不好。对范围搜索的支持不好。

那有什么index Structure可以突破这些limitation呢？


## SSTables and LSM-Trees
对于segment file的变化在于the sequence of key-value pairs is sorted by the key.
怎么保证这个要求呢？用 Sorted String Table(SSTable)。
它和hash index相比的优势是什么？
合并segments简单且高效，即使文件比available memory大也没关系，它通过mergesort实现。
如果要查找一个key，你不需要把所有key的index加载到内存，你可以通过二分查找缩小范围查找。
把index放在内存，文件存在磁盘，还可以压缩文件减少I/O bandwidth use.

### Constructing and maintaining SSTables
可以利用树实现。我们的存储引擎工作流程如下：
写入的时候，把它加入到一个in-moemory balanced tree data structure，这个in-memory tree有时候也叫memtable。
当memtable大到超过阈值，比如说几兆，就把它写入disk. 因为tree已经保持顺序了，这个写入过程是很高效的。
对于读的请求，是从最近的segment开始找。
后台定期合并压缩segment，丢弃overritten或者已删除的值。

这个机制有一个问题就是，数据库崩溃的时候最近的写操作还在内存，会丢失。为了解决这个问题，我们可以在硬盘上保存一个单独的日志，它是immediately append。这个日志不需要sorted，当memtable写入硬盘SSTable之后对应的日志就可以丢弃了。

### Making and LSM-tree out of SSTables
LevelDB和RocksDB，
LevelDB是另外一种应用，Cassandra和HBase有类似的storage engine.
Log-Structured Merge-Tree
Lucene是一个indexing engine 用于全文搜索，ElasticSearch和Solr就用了它。

它对不存在于数据库的key搜索有点慢，可以结合布隆过滤器一起用。

对于SSTable的压缩合并操作有不同的策略。LevelDB和RocksDB，使用leveled compaction，HBase用size-tiered，Cassandra两种都支持。

## B-Tree
最广泛使用的indexing structure.经受住了时间的考验，和SSTable一样，也是sorted by key，支持高效的搜索和范围查找。
B-tree是down into fixed-size blocks or pages, 基本上大小是4KB，一次读写one page，这种设计更接近底层硬件，因为磁盘也是组织为固定大小的block。

one page is designated as the root of the B-tree, 当你寻找key的时候，page包含key和child page的reference，reference之间的key就是它们的边界。
child page的reference数量叫做branching factor，它取决于存储空间和边界范围，一百来说是几百。

一个有n个key的B-tree，深度为logN, 大部分数据库可以放进一个深度为3或4的B-tree，A four-level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB。
第 1 层到第 3 层的节点只是用于存储指针的中间节点，并不包含实际的数据。
第 4 层（叶子节点）才是实际存储数据的独立页面，每个页面大小为 4 KB。
因此，有效的数据存储容量完全取决于第 4 层的叶子节点数量和每个叶子节点的大小。
leaf node数量：500*500*500*500，再乘以4KB≈256TB

B-tree是原地overwrite,这意味着moving the disk head to the right place, waiting for the  right  position  on  the  spinning  platter  to  come  around,  and  then  overwriting  the appropriate sector with new data. 如果是写入新数据，可能更麻烦，需要分页，also overwrite their parent page to update the references to the two child pages. This is a dangerous operation, because if the data‐base  crashes  after  only  some  of  the  pages  have  been  written,  you  end  up  with  a  corrupted index。

### making B-tree reliable
为了make the database resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a write-ahead log (WAL, alsoknown as a redo log)，这就是redolog的来源和作用. 它是append-only file，在对B-tree进行操作之前必须先写入redo log. 当数据库崩溃恢复时, redo log用来将B-tree还原到一致的状态。原地写还有一个并发的问题，这就需要一个latch(lightweight locks).

### optimization
-  copy-on-write  scheme 代替WAL，有点类似于多版本并发控制，将更改的page写入新的location，新版本的parent pages指向它。
-  一页中存入更多的key，更高的branch factor
-  尝试让叶子结点的页保存在磁盘相近的位置
-  叶子结点同时也存姐妹节点的指针
  

## B-trees vs. LSM-Trees
B-tree读快，LSM-Tree写快
B-tree要写两次，一次是WAL，一次是写入磁盘的树中，
LSM也要写几次，因为重复的压缩合并SSTables,这就是写放大，write amplification.在SSD中这是一个问题，因为它的重写次数有限。对performance有影响：the more that a storage engine writes to disk, the fewer writes per second it can handle within the available disk bandwidth。
通常来说，LSM Tree比B Tree有更高的write throughput，write amplification的问题在B-Tree中更严重，因为碎片的问题导致有些磁盘空间闲置。
lower  write amplification  and  reduced  fragmentation  are  still  advantageous  on  SSDs:  representing data more compactly allows more read and write requests within the available I/O
bandwidth.

A  downside  of  log-structured  storage  is  that  the  compaction  process  can  sometimes interfere  with  the  performance  of  ongoing  reads  and  writes. it  can  easily  happen  that  a  request  needs  to wait  while  the  disk  finishes  an  expensive  compaction  operation. the response time of queries to log-structured storage  engines  can  sometimes  be  quite  high,  and  B-trees  can  be  more  predictable. the bigger the database gets, the more disk bandwidth is required for compaction. If write throughput is high and compaction is not configured carefully, it can happen that  compaction  cannot  keep  up  with  the  rate  of  incoming  writes. In  this  case,  the number  of  unmerged  segments  on  disk  keeps  growing  until  you  run  out  of  disk space, and reads also slow down because they need to check more segment files. SSTable based engine不会监控写入速度，需要你留心。

对于B-Tree来说每个key仅仅存在于一个位置，它可以提供strong  transactional  semantics，可以用锁。

### 其他的indexing structure
secondary index
storing values within the index,对于写频繁的情况表现不太好

clustered  index
In some situations, the extra hop from the index to the heap file is too much of a performance  penalty  for  reads,  so  it  can  be  desirable  to  store  the  indexed  row  directly
within  an  index.  This  is  known  as  a  clustered  index.
比如MySQL, The  primary  key  of a  table  is  always  a  clustered  index,  and secondary indexes refer to the primary key (rather than a heap file location)
这段内容讲解了**聚簇索引（Clustered Index）**的概念及其在数据库中的应用，特别是以 **MySQL** 为例，描述了其在表结构中的特点和工作原理。我们来逐句分析这段内容，并详细解释其中的关键概念。

### 1. **概念理解：聚簇索引（Clustered Index）**
- **索引（Index）**是数据库中用来加速数据检索的结构，类似于书籍的目录。通过索引，可以更快速地定位到我们想要的数据行，而不需要全表扫描。
- **聚簇索引（Clustered Index）**和**非聚簇索引（Non-clustered Index）**的区别在于：聚簇索引中的**索引项与实际数据存储在一起**，而非聚簇索引中的索引项与实际数据是分离的。

### 2. **“额外的跳转”与性能影响**


> “在某些情况下，从索引跳转到堆文件（heap file）的额外开销对读取性能有很大的影响。”

这句话的意思是，传统的**非聚簇索引**需要在索引项和实际数据行之间做一次“跳转”，也就是**额外的 I/O 操作**。因为非聚簇索引存储的是数据行在堆文件（或数据页）中的物理地址（或指针），当需要读取实际数据时，数据库会：

1. **首先查找索引项**，定位到该索引项指向的数据行地址。
2. **然后跳转到该地址**，读取实际的数据行。

这个过程需要**两次 I/O 操作**：一次读取索引，一次根据索引找到实际数据行。如果数据库表中数据量很大且频繁访问的情况下，这种额外的跳转可能会显著影响读取性能。

### 3. **聚簇索引如何解决性能问题？**
原文提到：

> “在某些情况下，我们希望将索引行直接存储在索引中，这样就不会有额外的跳转开销。”

这描述了**聚簇索引（Clustered Index）**的优势：

- **聚簇索引把索引项和实际数据行直接存储在一起**，数据行按照索引值的顺序存储在物理磁盘中。
- 这样，当你通过聚簇索引查找数据时，只需要**一次 I/O 操作**即可：通过索引定位时，实际数据行就已经在同一位置了。
- 因此，聚簇索引对于数据读取来说更加高效，因为它将数据和索引紧密结合在一起，避免了“从索引跳转到堆文件”的额外开销。

### 4. **MySQL 中的聚簇索引**


> “在 MySQL 中，表的主键（Primary Key）总是聚簇索引，二级索引（Secondary Index）则指向主键，而不是堆文件地址。”

这是 MySQL 中 InnoDB 存储引擎的一个典型特性：

1. **主键是聚簇索引**：
   - 每个 InnoDB 表都有一个**聚簇索引**，该索引总是基于主键（Primary Key）创建的。
   - 也就是说，数据行**按照主键的顺序**存储在物理磁盘上（或者在数据文件中）。因此，主键索引中的每一个索引项都包含了实际的数据行本身。

2. **二级索引（Secondary Index）指向主键**：
   - 如果表中有其他的索引（非主键），比如二级索引（Secondary Index），这些索引项中不会存储实际的数据地址（不像非聚簇索引）。
   - 相反，这些二级索引中**存储的是对应行的主键值**。查找时，先根据二级索引找到主键值，然后再根据主键值查找聚簇索引，定位到数据行。

### 5. **聚簇索引的优缺点**
了解聚簇索引的好处和限制可以帮助理解其工作原理：

**优点：**
1. **数据读取更快**：由于数据和索引项存储在一起，只需一次 I/O 操作即可定位到数据行，非常适合范围查询或顺序访问。
2. **磁盘空间利用率高**：通过按照主键顺序存储，可以在读取连续数据时减少磁盘寻道时间。

**缺点：**
1. **插入和更新较慢**：因为数据按照主键顺序存储，当插入或更新数据导致数据行位置改变时，可能会发生数据页的分裂和重新组织，影响写入性能。
2. **表只能有一个聚簇索引**：因为聚簇索引决定了数据的物理存储顺序，所以每个表最多只能有一个聚簇索引（通常是主键）。

### 6. **总结**
- 聚簇索引的特点是**将索引和数据行存储在一起**，从而避免了查找数据时的额外跳转开销。
- 在 MySQL 中，主键总是一个聚簇索引，而其他二级索引指向主键值。
- 这种设计在读取大量数据或执行复杂查询时，可以大幅提升查询性能，但在频繁写操作时可能有一定的性能开销。

通过聚簇索引，MySQL 在读写性能和磁盘空间利用率之间取得了较好的平衡，因此在大多数实际场景中，主键作为聚簇索引的设计是非常有效的。

这里就涉及到数据库查询的优化方法了：A compromise between a clustered index (storing all row data within the index) and a nonclustered index (storing only references to the data within the index) is known as a covering index or index with included columns, which stores some of a table’s columns  within  the  index  [33].  This  allows  some  queries  to  be  answered  by  using  the index alone (in which case, the index is said to cover the query) 

### multi-column indexes
concatenated index
Multi-dimensional  indexes  are  a  more  general  way  of  querying  several  columns  at once,  which  is  particularly  important  for  geospatial  data. 

### Full-text search and fuzzy indexes


### Keeping everything in memory
内存数据库


## Transaction Processing or Analytics?

P90






