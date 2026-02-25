# NoSQL存储引擎核心原理解析（面试必看！）

DB分为日志型和页面型，日志型追加写，页面型覆盖写；大都NoSQL都采用日志型，MySQL采用页面型。这篇讲讲日志型 DB。

## 1） 原始DB

我们从读取、写入、故障恢复三个角度来讲下原理。

写入时，顺序将数据写入文件末尾，因为是顺序写入性能非常高。读取时，直接从文件末尾开始读key。

文件到一定规模后，创建新文件继续写。每个文件称为段。定时重写数据到新段，可以去除冗余key，最后再将旧段压缩归档。系统崩溃时通过读取最新的段恢复。

每次读取从文件末尾开始遍历性能差，改造下。

## 2） Hash

内存中维护key的磁盘偏移量，读取时先查询hash表，找到偏移后再读磁盘。写入时需要更新内存索引。

此方案需要全内存缓存，并且只能点查不能范围查找，继续改造下。

## 3） LSM

在内存中引入红黑树结构维护有序结构，称之为memtable，内存更新到一定程度后整块顺序写入磁盘；为了在写入磁盘过程中保持可写，在写入前把这个memtable变成只读，称为immutable memtable；

写入磁盘后的数据称之为sort string table（sstable），需要和磁盘中已有sstable做merge，因为每个文件都有序，整个merge过程就是归并排序，充分利用磁盘顺序写。

串起来：内存表memtable->只读内存表immutable memtable->sstable->全量 sstable，这就叫 LSM（log-structed merge-tree）。

再看看写入、读取、故障恢复。

写入，先写日志wal，再写memtable，后台程序定时做merge。

读取，先查内存表、再查只读内存表、再查磁盘sstable、最后查全量sstable；为了加快查询，在内存中维护sstable索引，因为有序，只需保存key范围。读取时传入不存在的key不停的洞穿这个流程，引入bloom filter（告诉你有不一定，告诉你没有一定没有）。

故障恢复，掉电丢失的就是内存memtable，从日志中故障恢复即可。如果担心恢复速度太慢，定时对memtable做dump，恢复时从最新dump中恢复，然后开始追进度，这就是checkpoint机制。

恭喜大家，掌握了bitcast 、levelDB、rocksDB、hbase等NoSQL存储原理了。