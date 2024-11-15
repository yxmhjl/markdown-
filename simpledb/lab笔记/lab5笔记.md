# Lab5笔记

## 1.exersize1-B+树的查找

```
private BTreeLeafPage findLeafPage(TransactionId tid, Map<PageId, Page> dirtypages, BTreePageId pid, Permissions perm,
                            Field f)
```

查找索引值为f的叶子结点，当前查找的页pid进行递归查询

如果当前的结点是叶节点，那么直接返回当前结点

**（1）**如果当前的结点的索引是非叶结点，那么根据相应的条目来进行递归索引

![image-20241111130111787](C:\Users\hjl\AppData\Roaming\Typora\typora-user-images\image-20241111130111787.png)

比如这个条目要查找的field

从头开始遍历，找到条目的field大于f的条目，然后调用：

```
findLeafPage(tid, dirtypages, bTreeEntry.getLeftChild(), perm, f);
```

**（2)**意思说如果f小于该条目的值那么f在当前结点的左孩子中，

**(3)**如果直到这个条目的最后仍然没有找到条目的值大于f，那么要找的f在最后一个条目的右孩子中。

**(4)**还有就是f为空的情况，则每次返回左孩子，情况可归于(2)

## 2.exersize2B+树的插入

这一部分原本的代码有相应的插入逻辑，整体的思路是先查找，找到位置后判断当前结点是否满，如果满则分裂页结点或中间结点

1. ```
   splitLeafPage(TransactionId tid, Map<PageId, Page> dirtypages, BTreeLeafPage page, Field field)
          throws DbException, IOException, TransactionAbortedException {
   ```

就是说现在field要插入page页中，但是page已经满了，现在需要将page分裂，以便插入field

整体思路：

- 首先获取一个空页，以便于将page的内容移过来一部分
- 将新页和page连接起来就是更新左右指针

- 计算要移动多少，一般page保留n/2，移到新页n-n/2
- 利用反向迭代器从page取出（n-n/2）个内容放到新页
- 将新页的第一个内容的值提到父条目，该条目的左孩子是page，右孩子是新页
- 更新脏页
- 根据field与新页的第一个内容的值的大小关系决定返回page还是新页

2.

```
splitInternalPage(TransactionId tid, Map<PageId, Page> dirtypages,
       BTreeInternalPage page, Field field) 
```

- 首先获取一个空的中间结点

- 计算要移动多少，一般page保留n/2，移到新页n-n/2

- 将新页的父亲id设为page的父亲id，更新指针关系

- 利用反向迭代器将page的后半部分条目移到新页

- 创建一个新的条目，值为新页的第一个值，左孩子为page的最后一个条目的右孩子，有孩子为新页的第一个条目的左孩子。

  ![img](https://i-blog.csdnimg.cn/blog_migrate/f8cd4cd8a6fa93fd09304e6e7a40c81d.png)

- 更新脏页，根据field的值与上一步的条目的值的大小关系返回page或新页

## Exersize3删除操作

主要是删除后防止不平衡从兄弟结点借内容的实现

```
stealFromLeafPage(BTreeLeafPage page, BTreeLeafPage sibling,
       BTreeInternalPage parent, BTreeEntry entry, boolean isRightSibling)
```

从兄弟结点借元组

- 首先计算需要借几个元组，方法就是page的元组数加兄弟的元组数除以2-page的元组数
- 判断是左兄弟还是右兄弟，左兄弟则是反向迭代器取出元组，右兄弟则是正向迭代器取出元组，插入page
- 更新条目，将右边的页的第一个元组的索引值放到父条目

```
stealFromLeftInternalPage()(TransactionId tid, Map<PageId, Page> dirtypages,
       BTreeInternalPage page, BTreeInternalPage leftSibling, BTreeInternalPage parent,
       BTreeEntry parentEntry)
```

![redist_internal](C:\Users\hjl\Desktop\redist_internal.png)

这个由于每一个中间结点都是唯一的，所以移动的时候会将父条目一起移动，比如把6移到了

右边变为6和10而不改变8，那么违反了B+树右边的都比当前结点大的原则，因此要把8也移到右边

- 首先计算需要移动几个条目，计算方法和上面类似
- 首先将父条目的值插入page，注意左孩子和右孩子与练习2相同
- 利用反向迭代器从左兄弟中取相应数目的条目插入到page
- 将page中最小的条目移到父节点，具体的做法就是将父条目的值改为page中最小的条目的值，删除page中的该条目
- 更新脏页和一些父节点

## Exersize4合并

```
mergeLeafPages(TransactionId tid, Map<PageId, Page> dirtypages,
       BTreeLeafPage leftPage, BTreeLeafPage rightPage, BTreeInternalPage parent, BTreeEntry parentEntry)
```

![merging_leaf](C:\Users\hjl\Desktop\merging_leaf.png)

很简单，比如上图假如是叶子结点，那么只需要做：

- 将右页的元组挪到左页，用正向迭代器即可
- 处理兄弟指针，就是将左页的右指针改为右页的右兄弟等一系列操作、
- 将右页置为空页
- 删除父条目，比如这个6
- 增加脏页

```
mergeInternalPages(TransactionId tid, Map<PageId, Page> dirtypages,
       BTreeInternalPage leftPage, BTreeInternalPage rightPage, BTreeInternalPage parent, BTreeEntry parentEntry)
```

比较简单

- 新建条目（值和父条目相同）插入左边的条目，原因和练习2一样，记得左孩子和右孩子是谁
- 删除父条目
- 将右页的条目移到左页
- 将右页设置为空页
- 更新指针和脏页