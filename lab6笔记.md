# Lab6事务故障与恢复

1. steal/no-force策略

   lab6这个实验中，我们实现的是steal/no-force策略，两种策略的区别如下:

steal/no-steal: 是否允许一个uncommitted的事务将修改更新到磁盘，如果是steal策略，那么此时磁盘上就可能包含uncommitted的数据，因此系统需要记录undo log，以防事务abort时进行回滚（roll-back）。如果是no steal策略，就表示磁盘上不会存在uncommitted数据，因此无需回滚操作，也就无需记录undo log；
force/no-force:force策略表示事务在committed之后必须将所有更新立刻持久化到磁盘，这样会导致磁盘发生很多小的写操作（更可能是随机写）。no-force表示事务在committed之后可以不立即持久化到磁盘， 这样可以缓存很多的更新批量持久化到磁盘，这样可以降低磁盘操作次数（提升顺序写），但是如果committed之后发生crash，那么此时已经committed的事务数据将会丢失（因为还没有持久化到磁盘），因此系统需要记录redo log，在系统重启时候进行前滚（roll-forward）操作。

 2.本实验中的日志记录结构

![img](https://i-blog.csdnimg.cn/blog_migrate/60f63b263a18dd97edb73effc8969e24.png)

只需要注意checkpoint的结构是：
日志类型  事务号 当前活跃的事务的个数 活跃的事务号 事务开始在文件中的偏移，比如1234、2345、3456是事务号，12222、13333是该事务的start记录相对文件开始的偏移

 5               1234               3                          2345                         12222                   3456                   13333

注意活跃的事务号和偏移不止一个

update的结构是：

日志类型  事务号   beforedata   afterdata  偏移

3               1234      pagebefore  pageafter   13333

## 练习1 rollback

![fbcd3b584192ef89173ac624102d98f](C:\Users\hjl\Documents\WeChat Files\wxid_x5uzy38kcowr32\FileStorage\Temp\fbcd3b584192ef89173ac624102d98f.jpg)

回滚很简单，就是遍历日志，找到对应事务的update记录，将beforedata刷回磁盘，注意Map<Long,Long> tidToFirstLogRecord = new HashMap<>()存放了当前活跃的事务号和偏移，可以通过tid直接找到其对应的事务开始begin记录，然后从begin开始遍历日志记录，找到update后将旧数据刷盘即可（教材上说的是反向扫描，但是我的实现是正向扫描，也都能过测试，反向扫描可以先正向扫描找出每个日志记录的偏移，然后反向遍历实现）

```
public void rollback(TransactionId tid)
    throws NoSuchElementException, IOException {
    synchronized (Database.getBufferPool()) {
        synchronized(this) {
            preAppend();
            // some code goes here
            //计算事务号为tid的事务在日志文件中的偏移，并移到事务号为tid的事务开始的地方
            if(tidToFirstLogRecord.containsKey(tid.getId()))
            {
                long offset = tidToFirstLogRecord.get(tid.getId());
                raf.seek(offset);
                //顺序读取日志记录，找到该事务的修改日志
                while(true)
                {
                    try{
                        int type = raf.readInt();
                        long record_tid = raf.readLong();
                        switch (type)
                        {
                            case UPDATE_RECORD:

                                Page before = readPageData(raf);
                                Page after = readPageData(raf);
                                raf.readLong();
                                if(tid.getId() == record_tid)
                                {
                                    //如果是目标事务则将旧数据刷盘
                                    Database.getCatalog().getDatabaseFile(before.getId().getTableId()).writePage(before);
                                    logWrite(tid,after,before);
                                    Database.getBufferPool().getnewpage(before.getId());
                                }
                                break;
                            case CHECKPOINT_RECORD:
                                int numTransactions = raf.readInt();
                                raf.seek(raf.getFilePointer()+(numTransactions+1)*8);
                                break;
                            default:
                                raf.readLong();
                                break;
                        }
                        //读下一条指令
                    } catch (EOFException e) {
                        //e.printStackTrace();
                        break;
                    }
                }
            }
            else {
                throw new NoSuchElementException();
            }
        }
    }
}
```

## 练习2recover

​	注意事务的恢复是在发生系统崩溃的时候发生的，也就是说此时能用的只有日志文件，日志文件是写在磁盘上的，而程序中的变量是在内存上的，当系统崩溃时这些变量是会损坏的，这也就是为什么要有检查点日志的原因，防止崩溃时找不到事务。因此在本实验中变量tidToFirstLogRecord是不能用的，因此在函数开始需要新建一个logfile文件，变量名为raf，通过读raf来进行恢复。

​	恢复算法：

重做阶段：主要任务就是将update日志中的afterdata恢复并维护undolist，undolist初始化为检查点中的事务，若有事务begin，那么将该事务假如undolist，若有事务提交和回滚那么将该事务从undolist移除

本质就是重新执行一遍无论这些数据有没有刷到磁盘，如果它们提交或回滚过就不用撤销了，为什么回滚过也不用撤销了呢？因为在rollback中会往日志中写一个更新logwrite（after，before）就是回滚的时候的写after是before就是把after数据改成了before，这条日志在事务恢复的时候会执行，因此相当于没做。

![e824d9c8062d31db124b78e6272e514](C:\Users\hjl\Documents\WeChat Files\wxid_x5uzy38kcowr32\FileStorage\Temp\e824d9c8062d31db124b78e6272e514.jpg)

撤销阶段：处理undolist的事务，反向遍历日志将在undolist中的事务更新恢复到before，当遇到在undolist中的事务的start后表示该事务的重做结束，将事务从undolist删除。当undolist为空的时候撤销结束

![47d0f271e327d226b329ab3094c4038](C:\Users\hjl\Documents\WeChat Files\wxid_x5uzy38kcowr32\FileStorage\Temp\47d0f271e327d226b329ab3094c4038.jpg)