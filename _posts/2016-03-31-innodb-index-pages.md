---
layout: default
title: "The physical structure of InnoDB index pages"
date: 2016-03-31
categories: MySQL/InnoDB/翻译
---

原文地址:

[The physical structure of InnoDB index pages ](http://blog.jcole.us/2013/01/07/the-physical-structure-of-innodb-index-pages/)

### [](#InnoDB：一切皆索引)InnoDB：一切皆索引
深入讨论物理结构之前，需要明白一件事：InnoDB中一切都是索引

- 每一个表都有一个主键；如果CREATE TABLE没有指定主键，第一个非空(NOT NULL)的唯一键将被作为主键，如果没有非空的唯一键，InnoDB会自动分配一个48位（6个字节）的隐藏主键(ROW ID)。由于隐藏主键对用户不见，仍会占用表空间，建议在建表时明确指定主键。

- 表数据(主键以外的列)存储在主键索引结构中（clustered index）。聚集索引按照主键列构建索引树，行数据（包括一些用于MVCC的额外数据）存在索引页上。

- 二级索引存放在单独的索引结构中，按照键值构建索引树，但是在索引页中存放了主键的信息。

所以，讨论InnoDB表的索引时，指的就是表数据和索引，数据即索引，索引即数据。

### [](#索引数据页结构)索引数据页结构

- FIL header, trailer  一般数据页都会包含的内容。索引页有不同的地方，在页头中的前后页指针分别指向索引树中同级的前后节点，这样在索引中每一级上就形成了双向链表。下文会详细讨论

- FSEG header  索引的root节点中FSEG header存放指向索引文件段的指针，其他的索引页未使用并用0填

- INDEX header  索引页以及记录管理信息。下文讨论。

- System record  每个数据页中都包含2个系统记录：infimum和supremum。由于存放在固定的位置，这2个信息可以直接通过偏移字节数来找到。

- User record  实际数据，每个记录包含一个可变长记录头和实际的列数据。记录头中包含指向下一个数据记录的偏移量，数据记录形成单链表。

- Page directory  数据页目录从数据页尾的起始位置开始。存放一些指向数据页内数据记录的位置信息（每4到8个记录）。

### [](#INDEX-header)INDEX header

- Index ID： 数据页所属的索引ID

- Format Flag： 页内记录格式，以高位（0x8000）存放在 Number of Heap Record列。可选值：COMPACT 和 REDUNDANT。

- Maximun Transaction ID：数据页中对记录更改的最大事务ID

- Number of Heap Records：数据页中的记录总数，包含infimum和supremum2个系统记录以及垃圾数据。

- Heap Top Position：当前已使用空间末尾的偏移量。在heap top到数据页目录之前为可用空间。

- First Garbage Record Offset： 指向第一个垃圾记录的指针。脏记录指针通过在记录头中指向下一个记录的next pointer 形成单链表

- Garbage Space：垃圾记录列表的字节总数

- Last Insert Position： 上次插入记录的字节偏移量

- Page Direction：页方向可选值：LEFT,RIGHT 和 NO_DIRECTION. 这个值可以标识数据正在进行顺序插入还是随机插入。对于每一次插入，获取上一次插入的记录和它的位置，通过比较上一次插入记录的key和当前插入的记录key来决定插入的方向。

- Number of Inserts in Page Direction：一旦页方向被设置，后续的未改变页方向的插入操作将增加这个计数

- Page Level：索引中页的级别数（深度）。页节点级别值为0，从页节点往上增加。例如一个深度为3的B+tree，root节点的级别为2，中间非叶节点级别为1，叶节点为0.

### [](#Record-format)Record format
COMPACT 记录格式是Barracuda表的新格式， REDUNDANT格式则是Antelope表的原始格式之一。COMPACT 主要目的是消除每一个记录都存放的多余信息，这些信息可以从数据字典中获得。例如列的数量，那些列允许为NULL，以及那些列是可变长的。

### [](#Record-pointer)Record pointer
记录指针使用多个不同的地方：

- INDEX header中最后插入位置

- 页目录中的值

- 系统记录和用户记录中的指向下一个记录的指针所有的记录包含一个记录和时间的记录数据。记录指针指向实际数据的第一字节，也就是在记录头和数据之间，这样使得可以通过记录索引往回读取记录头信息，也可以往下继续读取数据信息。

由于系统记录和用户记录中后指针可以通过记录指针回读获得，这样可以高效地读取页内所有的记录而不需要去解析变长字段。

### [](#System-records)System records
每一个索引页包含2条系统记录：infimum和supremum，分别存放在固定偏移位置： 99和112。系统记录结构：

- infimum record 包含一个数据页中最小的键值。他的后指针指向用户记录中最小键值。 它提供了对用户记录遍历的固定入口。

- supremum record 包含一个数据页中最大的键值，他的后指针总是为0，也就是指向无效的地址，用户记录中包含最大键值的记录中的后索引指向supremum record

### [](#User-record)User record
用户记录按照他们插入的顺序保存在数据页中，包括复用已删除的记录空间，并且通过每一个记录头中的后索引来形成键值递增的单链表。单链表以infimum开始，递增链接所有的用户数据，以supremum结尾。通过这个单链表，升序遍历数据页中的所有数据变得非常容易。

再通过INDEX header中的后索引，就可以形成数据页之间的单链表，这样对整个索引树的升序遍历也会变成非常容易，也就是数据表的升序遍历：

- 从索引中包含最小键值的数据页开始

- 读取infimum，接着是后索引

- 如果读取到supremum，跳转第五步，否则继续读取记录

- 根据后索引跳转第三步

- 如果后索引指向NULL，否则跳转第二步，进行下一个数据页

由于结构是单链表，逆序的遍历比升序遍历要复杂一些。

### [](#The-page-directory)The page directory

页目录从FIL trailer向上存放，对每4-8个记录保存一个制证在页目录中，另加对infimum和supremum的指针。该指针为16位的变长数组，其中为数据记录的偏移量。

### [](#Free-space)Free space
在用户记录和页目录之间为可用空间。如果用户记录块和页目录块直接没有空闲空间（通过重组数据页，删除垃圾数据之后），则数据页被占满。
