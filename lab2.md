# Lab2笔记

## Exersize1

1.predicate.java(谓词)

主要属性值：

```
private int field;
private Op op;
private Field operand;
```

主要功能：filter(Tuple t)

就是比较元组t的第field个属性和Field operand做op操作的结果，返回True和False ,比如3 > 4,返回false

实现：借助compare方法实现：compare分为整数型的和字符串型的

![image-20241026201449453](C:\Users\hjl\AppData\Roaming\Typora\typora-user-images\image-20241026201449453.png)

![image-20241026201725310](C:\Users\hjl\AppData\Roaming\Typora\typora-user-images\image-20241026201725310.png)

2 JoinPredicate.java

主要属性：

```
private int field1;
private int field2;
Predicate.Op op;
```

主要功能：filter(Tuple t1,Tuple t2)

就是将元组t1的field1与元组t2的field2进行compare的比较，compare方法与上面相同

3.Filter.java过滤

主要属性：

```
private Predicate p;
private OpIterator[] child;
private TupleDesc td;
```

主要方法fetchNext()，返回的是下一个满足过滤条件的元组

使用举例：

Filter sf1 = new Filter(
                new Predicate(0,
                        Predicate.Op.GREATER_THAN, new IntField(1)), ss1);

SeqScan ss1 = new SeqScan(tid, table1.getId(), "t1");

组合起来filter的功能选处了就是”t1‘表格中第一个域大于1的元组

4.Join.java

主要属性：

```
private JoinPredicate joinPredicate;
private TupleDesc tupleDesc;
private OpIterator[] child;
private Tuple nowTuple;
```

主要功能：有两张表t1和t2，每次返回一个元组，这个元组是t1和t2的合并，并且对应的元组满足相应的操作。相当于用两层循环，外层循环是表1，内层循环表2，一次遍历表1的每一个元组，对于这个元组依次找所有满足连接条件的表2的元组，每次返回一个这样的连接元组 。

使用实例：

        JoinPredicate p = new JoinPredicate(1, Predicate.Op.EQUALS, 1);
        Join j = new Join(p, sf1, ss2);

JoinPredicate p表面连接的谓词是第一个域和第二个域相等的元组，sf1是经过谓词筛选后的表，ss2是表2

SeqScan ss2 = new SeqScan(tid, table2.getId(), "t2");

经过上面四部，可以实现查询：

SELECT *
FROM T1,T2
WHERE t1.field1 = t.field1
  AND T1.id > 1

连接条件用Join，具体用Join谓词实现，谓词逻辑用Filter实现

## Exersize2

聚合实现，包括整形的max、min、sum、count 、字符型的count

聚合分为是否有分组groub by

在Exercise2中要完成的就是SQL中的分组（GROUP BY）与聚合（aggregator）的操作。先简单复习下这两个的概念:

这是一张简单的会员信息的fee表，如果只是对fee进行SUM聚合操作，不进行分组的话则结果应为：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aeb4472ad4b87c973774fe7467d83a45.png)

但是如果以城市进行分组进行计算的话（19700/7000/9300 则分别对应一个Field):

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86df1cb55847f4cecf9c72bed78171da.png)

分组：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba8cdc6de2f0febb20e636b4cfde69fb.png)

整形的聚合：

创建了一个内部类GroupCalResult，里边记录聚合的整形值

同时创建一个hash图：

```
ConcurrentHashMap<Field, GroupCalResult> groupResults
```

```
private int sum;
private int count;
private int min = Integer.MAX_VALUE;
private int max = Integer.MIN_VALUE;

public void addValue(int value) {
    sum += value;
    count++;
    if (value < min) min = value;
    if (value > max) max = value;
}
```

通过addValue进行值的改变

主要函数mergeTupleIntoGroup(Tuple tup)

先判断是否要进行分组聚合，如果不用分组聚合，则把他值为默认值：

```
groupByField = new IntField(NO_GROUPING);
```

这样在非首次查询的时候，利用groupResults.get(groupByField)查询的结果区分是否需要分组聚合，如果是默认值那么就是非分组的聚合，最后把tup.getField(afield).getValue()字段用addValue加入hash表

```
public void mergeTupleIntoGroup(Tuple tup) {
    Field groupByField = gbfield >= 0 ? tup.getField(gbfield) : null;
    IntField aggField = (IntField) tup.getField(afield);
    int value = aggField.getValue();
    if(groupByField == null) {
        groupByField = new IntField(NO_GROUPING);
    }
    GroupCalResult result = groupResults.get(groupByField);
    if (result == null) {
        result = new GroupCalResult();
        groupResults.put(groupByField, result);
    }

    result.addValue(value);
}
```

然后String的聚合与面类似且更简单

Aggregate操作只用根据是否需要分组和整形还是字符型进行分别考虑即可。

最大的收获就是内部类的GroupCalResult的结构，非常的规整好用

## Exersize3

1.一个Heapfile是堆文件和文件的映射，它能通过getid获得一个int型的id，它可以通过构造HeapPageId（HeapPageId（int tableid，int Pagenum），tableid可以用getid（）获得，Pagenum可以用getNums获得）进而通过Database.getBufferPool().getPage获得页面，HeapFile上面还有一个numPages，这样就可以实现堆堆文件上的页面的便利。

## Exersize4

### 1.insert操作

插入一个元组，fetchNext()需要返回的是一个一元元组，元组的值是插入的元组的个数，如果多次被调用则返回null，需要注意的是操作符类的Tupledesc与fetchNext方法返回的Tuple的Tupledesc相同。主要的实现为：

    protected Tuple fetchNext() throws TransactionAbortedException, DbException{
        // some code goes here
        Tuple resultTuple  = new Tuple(new TupleDesc(type));
        type[0] = Type.INT_TYPE;
        //记录插入的个数
        int num = 0;
        if(flagisFirst) {
            try {
                while (child[0].hasNext()) {
                    Tuple tuple = child[0].next();
                    // 将元组插入到缓冲池中
                    Database.getBufferPool().insertTuple(tid,tableId, tuple);
                    num++;
                }
                resultTuple.setField(0, new IntField(num));
                flagisFirst = false;
                return resultTuple;
            } catch (IOException e) {
                // 处理 IOException
                throw new DbException("Error inserting tuple into buffer pool: " + e.getMessage());
            }
        }
        else
        {
            return null;
        }
    }

### 2.delete操作

与上面的类似：

    protected Tuple fetchNext() throws TransactionAbortedException, DbException {
        // some code goes here
        //计数删除个数
        int num = 0;
        if(flagisFirst) {
            try {
                while (child[0].hasNext()) {
                    Tuple tuple = child[0].next();
                    Database.getBufferPool().deleteTuple(tid, tuple);
                    num++;
                }
                Tuple resultTuple = new Tuple(tupleDesc);
                resultTuple.setField(0, new IntField(num));
                flagisFirst = false;
                return resultTuple;
            } catch (IOException e) {
                // 处理 IOException
                throw new DbException("Error inserting tuple into buffer pool: " + e.getMessage());
            }
        }
        else
        {
            return null;
        }
    }

## Exersize5 缓冲池换页操作

一个简单的最近最少使用策略

参考：[全面讲解LRU算法-CSDN博客](https://blog.csdn.net/belongtocode/article/details/102989685)

```
package simpledb.execution;

import simpledb.common.DbException;
import simpledb.common.Type;
import simpledb.storage.Page;
import simpledb.storage.PageId;
import simpledb.storage.Tuple;
import simpledb.storage.TupleDesc;
import simpledb.transaction.TransactionAbortedException;

import java.util.ArrayList;
import java.util.NoSuchElementException;
import java.util.concurrent.ConcurrentHashMap;

public class LRUcache {
        public class node {
            int nodeid;
            PageId pageid;
        }

        //维护一个哈希表，K为页号、V为在假设的栈的位置
        private final ConcurrentHashMap<PageId, node> pages = new ConcurrentHashMap<>();
        private int maxSize;

        public LRUcache(int size) {
            maxSize = size;
        }

        public void setMaxSize(int maxSize) {
            this.maxSize = maxSize;
        }

        public int getMaxSize() {
            return maxSize;
        }

        public void save(PageId pageId) {
            //当前页在栈中，将该页放到最上面，之前在该页上面的下移，下面不动
            if (pages.containsKey(pageId)) {
                down(pageId);
            } else// 不在则要将所有的下移
            {
                node newnode = new node();
                newnode.nodeid=1;
                newnode.pageid = pageId;
                pages.put(pageId, newnode);
                for (PageId temppageId : pages.keySet()) {
                    node node = pages.get(temppageId);
                    node.nodeid = node.nodeid + 1;
                    if (node.nodeid > maxSize) {
                        pages.remove(temppageId);
                    }
                }
            }
        }

        public void gets(PageId pageId) {
            down(pageId);
        }
        public void down(PageId pageId) {
            int index = pages.get(pageId).nodeid;
            pages.get(pageId).nodeid = 1;
            for (PageId temppageId : pages.keySet()) {
                node node = pages.get(temppageId);
                if (node.nodeid < index) {
                    node.nodeid = node.nodeid + 1;
                }
            }
        }
}
```

```
private static class LRUCache {
    int cap,size;
    ConcurrentHashMap<PageId,Node> map ;
    Node head = new Node(null ,null);
    Node tail = new Node(null ,null);

    public LRUCache(int capacity) {
        this.cap = capacity;
        map = new ConcurrentHashMap<>();
        head.next = tail;
        tail.prev = head;
        size = 0;
    }
    private class Node
    {
        Node prev;
        Node next;
        Page value;
        PageId key;
        public Node(Page value,PageId key) {
            this.key = key;
            this.value = value;
        }
    }
    public synchronized int getSize(){
        return this.size;
    }
    public void put(PageId pid,Page value)
    {
        if(map.containsKey(pid))
        {
            map.get(pid).value = value;
            map.get(pid).key = pid;
            down(pid);
        }
        else
        {
            //申请一个结点，放到最上面，并放入map
            Node newNode = new Node(value,pid);
            head.next.prev = newNode;
            newNode.next = head.next;
            newNode.prev = head;
            head.next = newNode;
            map.put(pid,newNode);
            //根据栈是否满决定是否移除页面
            if(size < cap)
            {
                //不满，不移动。
                size++;
            }
            else
            {
                //移动非脏页，首先找到最后使用的非脏页
                Node notdirtynode = tail.prev;
                while(notdirtynode.value.isDirty != null && notdirtynode!=head)
                {
                    notdirtynode = notdirtynode.prev;
                }
                if(notdirtynode != head && notdirtynode != tail)
                {
                    map.remove(notdirtynode.key);
                    notdirtynode.prev.next = notdirtynode.next;
                    notdirtynode.next.prev = notdirtynode.prev;
                }
            }
        }
    }
    public void down(PageId pid)
    {
        map.get(pid).prev.next = map.get(pid).next;
        map.get(pid).next.prev = map.get(pid).prev;
        map.get(pid).next = head.next;
        head.next.prev = map.get(pid);
        head.next = map.get(pid);
        map.get(pid).prev = head;
    }
    public Page get(PageId pid)
    {
        if(map.containsKey(pid))
        {
            down(pid);
            return map.get(pid).value;
        }
        else
        {
            return null;
        }
    }
    public Set<Map.Entry<PageId, Node>> getEntrySet(){
        return map.entrySet();
    }
}
```