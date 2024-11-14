# Lab1笔记

## 1.exersize1

Tuple是元组，是表中的一行数据，tupledes相当于表头，tuple中用集合fields记录一行数据的具体值，集合的每一个元素是一个Field，Field有两种，一种是int一种是string，int只有一个域，string还有一个int型的表示长度的域。

Tupledesc中有一个TDitem类代表表头每一个属性的集合，每一个TDitem域有fieldType(Type)和fieldName(String)两个属性变量,Type有INT_TYPE和STRING_TYPE。

![img](https://pic3.zhimg.com/80/v2-85dce761b78e452f5f0c45c93bc43a60_720w.webp)

## 2.exersize2

Catalog是数据库中表的目录，能够使用其中的方法按照表名字和表的id来获取相关的table，table是定义的一个内部类，表示一张表需要的信息。

主要的数据结构是：

```
ConcurrentHashMap<Integer,Table> tableIdMap;

ConcurrentHashMap<String,Integer> tableNameMap;
```

```
比如这张表在磁盘上的位置信息file(DbFile)，

表的名字name(String),

表的主键值pkeyField(String)
```

Catalog的实现的主要方法为：

添加一张表：

```
public void addTable(DbFile file, String name, String pkeyField)
```

根据表的阿id，表明查找相关的信息，比如：

```
public TupleDesc getTupleDesc(int tableid)
```

## 3.exersize3

