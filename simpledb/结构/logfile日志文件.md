日志文件，是数据库在执行过的操作的记录

## 主要变量：

RandomAccessFile raf 要写的日志

Map<Long,Long> tidToFirstLogRecord当前活跃的事务

## 主要函数：

logXactionBegin(TransactionId tid)

事务开始函数，当一个事务开始的时候调用此函数

logCheckpoint()

检查点函数，存储当前活跃的事务以及它们在日志文件中的偏移

writePageData(RandomAccessFile raf, Page p)

readPageData(RandomAccessFile raf)

从日志中读写Page格式的数据

logCommit(TransactionId tid)

事务提交函数，当事务提交时调用此函数

logWrite(TransactionId tid, Page before,
                                   Page after)

事务更新函数，当修改时调用此函数，before是该page修改前的数据，after是要修改的数据

## 调用关系

- ### 一个事务t，执行插入操作并提交

1. t.start()事务开始在start函数中调用logXactionBegin()函数，在日志文件中写下begin日志
2. 执行insert（）函数，调用缓冲池的插入元组，缓冲池利用多肽特性找到是索引还是堆文件的插入，假如是Heapfile，那么调用Heapfile的insert函数
3. 调用flushAllPages()刷盘，根据是否是脏页进行，如果是脏页那么调用Database.getCatalog().getDatabaseFile(pid.getTableId()).writePage(target);写入磁盘，并调用

​       logWrite(tid, before,target)在日志文件中更新，写下更新日志

   4.调用commit函数，然后调用事务的transactionComplete（）函数，然后调用缓冲池的transactionComplete       函数，在事务的transactionComplete（）函数中调用logcommit函数在日志中写下提交日志。

- ### 一个事务t，执行插入操作并回滚

t.start()调用logXactionBegin()，调用insert（），调用t.abort（），然后调用flushAllPages()将修改内容刷盘，调用logAbort(t.getId())，在logAbort中调用rollback，在rollback中将旧数据刷盘，然后调用缓冲池的transactionComplete(t.getId(), false)。