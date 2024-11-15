# B+树索引

1.首先每一个索引对应一个文件，这里是Btreefile

根索引页在该文件的第一块上，记录的是根结点在哪以及根结点的类型:

```
BTreeRootPtrPage rootPtr = getRootPtrPage(tid, dirtypages);
```

根索引页还有int类型的变量header，记录的是树头页在文件中的第几块

2.在索引文件中还有树头页，记录B+树中的页的占用情况，比如byte[] header数组，记录的就是某一页是否被使用，如果是0就是没被用，如果是1就是被用

```
BTreeHeaderPage(BTreePageId id, byte[] data)
```

树头页不止一个，当不够用的时候还会新建，新建的树头页会连接在最初的树头页

3.insertTuple(TransactionId tid, Tuple t)函数

首先判断这个叶子结点是否满：

```
if(leafPage.getNumEmptySlots() == 0) {
    leafPage = splitLeafPage(tid, dirtypages, leafPage, t.getField(keyField)); 
}
```

如果满就进行分裂，返回的是t应该插入的页，然后调用叶页的插入函数

```
leafPage.insertTuple(t);
```

该函数的实现逻辑为：

首先找到空槽的位置就是没有插入元组的位置emptyslot

```
int emptySlot = -1;
for (int i=0; i<numSlots; i++) {
    if (!isSlotUsed(i)) {
       emptySlot = i;
       break;
    }
}
```

然后找到元组应该在的位置（按序排列）

```
for (int i=0; i<numSlots; i++) {
    if(isSlotUsed(i)) {
       if(tuples[i].getField(keyField).compare(Predicate.Op.LESS_THAN_OR_EQ, key))
          lessOrEqKey = i;
       else
          break; 
    }
}
```

然后进行移动：

```
int goodSlot = -1;
if(emptySlot < lessOrEqKey) {
    for(int i = emptySlot; i < lessOrEqKey; i++) {
       moveRecord(i+1, i);
    }
    goodSlot = lessOrEqKey;
}
else {
    for(int i = emptySlot; i > lessOrEqKey + 1; i--) {
       moveRecord(i-1, i);
    }
    goodSlot = lessOrEqKey + 1;
}
```

移动的原因：

好的，让我们通过一个具体的例子来说明为什么要移动记录。

### 假设：

我们有一个 `BTreeLeafPage` 页面，其中记录按照 `keyField` 字段排序。页面上可以存储多个 `Tuple`（例如，表示数据库中的一行记录），每个记录都有一个 `key` 字段。我们需要插入一个新的记录，并保持所有记录的顺序。

#### 初始状态：

假设页面上的记录已经排好序，包含以下 5 个 `Tuple`：

| Slot | Tuple（key） |
| ---- | ------------ |
| 0    | (3)          |
| 1    | (5)          |
| 2    | (7)          |
| 3    | (9)          |
| 4    | (12)         |

页面一共有 5 个插槽（`numSlots = 5`），当前有 5 条记录，所有插槽都已占用。

现在，我们要插入一个新的 `Tuple`，它的 `key` 为 `6`。

### 步骤 1：找到空插槽

首先，代码会找到页面中第一个空的插槽（`emptySlot`），因为我们已经有 5 条记录，页面没有空插槽。假设页面有更多插槽，我们会看到 `emptySlot = 5`，即新的记录将插入到页面的末尾。

### 步骤 2：找到插入点（排序位置）

接下来，我们要找到插入位置。为了保持排序，必须找到**小于或等于**我们要插入的 `Tuple` 的最后一个记录。

在这个例子中，我们要插入的记录是 `key = 6`，我们从左到右扫描页面中的记录，找到 `key = 5`（索引 1）小于 `6`，而下一个记录 `key = 7`（索引 2）大于 `6`。因此，`lessOrEqKey` 就是 1，即我们要插入的位置是 **`key = 6` 应该排在 `key = 5` 和 `key = 7` 之间**。

### 步骤 3：移动记录

接下来，我们需要调整记录的位置。由于空插槽（`emptySlot = 5`）在最后，而插入点（`lessOrEqKey = 2`）比空插槽之前的插槽更靠前，我们需要将记录向后移动，腾出空间来插入新记录。

#### 具体操作：
1. 由于 `emptySlot < lessOrEqKey`，我们需要将插槽 2 到插槽 4 之间的记录向后移动一位。具体来说：
   - 将插槽 4 的记录（`key = 12`）移动到插槽 5。
   - 将插槽 3 的记录（`key = 9`）移动到插槽 4。
   - 将插槽 2 的记录（`key = 7`）移动到插槽 3。

现在页面看起来是这样的：

| Slot | Tuple（key） |
| ---- | ------------ |
| 0    | (3)          |
| 1    | (5)          |
| 2    | (7)          |
| 3    | (9)          |
| 4    | (12)         |
| 5    | 空           |

### 步骤 4：插入新记录

现在空插槽 `5` 是可用的，而且我们知道 `key = 6` 应该排在 `key = 5` 和 `key = 7` 之间，所以我们将新记录插入到 `goodSlot = 2` 位置。

#### 最终页面：

| Slot | Tuple（key） |
| ---- | ------------ |
| 0    | (3)          |
| 1    | (5)          |
| 2    | (6)          |
| 3    | (7)          |
| 4    | (9)          |
| 5    | (12)         |

### 总结：

- **为什么要移动记录**：因为 `Tuple` 是有序的，我们需要保证新记录插入后，页面的顺序不被破坏。通过移动记录，我们确保 `key = 6` 排在 `key = 5` 和 `key = 7` 之间，保证了页面中记录的顺序。
- **移动的过程**：从插槽 2 到插槽 4 的记录被后移，为新记录腾出位置。

最后进行插入：

```
tuples[goodSlot] = t;
```

4.deleteTuple(TransactionId tid, Tuple t)函数

首先直接删除

```
page.deleteTuple(t);
```

然后根据结点的类型以及兄弟结点的内容的数目进行决定是进行合并还是从兄弟哪里借内容

```
if(page.getNumEmptySlots() > maxEmptySlots) { 
    handleMinOccupancyPage(tid, dirtypages, page);
}
```

handleMinOccupancyPage根据具体的情况处理：

```
if(page.getId().pgcateg() == BTreePageId.LEAF) {
    handleMinOccupancyLeafPage(tid, dirtypages, (BTreeLeafPage) page, parent, leftEntry, rightEntry);
}
else { // BTreePageId.INTERNAL
    handleMinOccupancyInternalPage(tid, dirtypages, (BTreeInternalPage) page, parent, leftEntry, rightEntry);
}
```