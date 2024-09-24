# Chapter3 Storage and retrieval
> turns to the internals of storage engines and looks at how databases lay
out data on disk. Different storage engines are optimized for different workloads,
and choosing the right one can have a huge effect on performance.

how we can store the data that we're given, and how we can find it again when we're asked for it.
how the database handles storage and  retrieval  internally? 

两种storage engine：log-structured engines, and page-oriented engines such as B-trees

log: an append-only sequence of records

trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes. You can choose the indexes that give your application the greatest benefit, without introducing more overhead than necessary.
Bitcask适合存储key-value，假如有很多写入，但是distinct key少，append-only的写入方式会很快耗尽磁盘空间，一个好的解决方法是break the log into segments of a certain size,当这个segment file的size到达一定值的时候，subsequent write to a new segment file.然后我们可以perform compaction on these segments, 在这里意味着throw away duplicate keys in the log, keeping only the most recent update for each key.
因为compaction会让segments变小，我们可以在compaction的同时merge several segments together. 这个merging和compaction的步骤可以在background thread完成，当它还在进行中我们继续在原来的segments中读和写，当它们完成之后我们把读和写操作转换到新segments，删除旧的segments。

P74



