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

P80






