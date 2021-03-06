# 数据库索引设计与优化

## 不合适的索引

不合适的所以是性能低下的最常见原因。最普遍的问题似乎是没有索引足够多的列来支持where自居中的所有谓词。包括，表上没有足够多的索引，一些select语句可能没有有效的索引，有时索引上包含了正确的列，但列的顺序却不对。

## 表和索引结构

### 索引页和表页

表和索引行都是被存储在页中，页大小一般为4KB，页的大小仅仅决定了一个页可以存储多少索引行，表行，以及一共需要多少页来存储表或者索引。

### 索引行

一个索引行等同于叶子页中的一个索引条目，字段的值从表中复制到索引上，并加上一个指向表中记录的指针，对于非唯一索引，每个索引条目后面都有多个指针

### 索引结构

索引通常以B树实现，其中非叶子页通常包含着键值，以及一个指向下一层级页的指针，该键值是下一层级页中的最大键值，多个索引层级按照这一方式逐层建立，直到只剩下一个页，这被称作根页

### 表行

每个索引行都指向表中相对应的一行记录，指针通常标识了记录所存放的页及它在页中的位置。当加载表或者向表中插入记录的时候，表中记录的顺序可以被定义成和它的某一个索引记录相同的顺序（对表建立索引，并按索引来访问），这种情况下，当所索引行被按顺序处理时，对应的表行也将依照相同的顺序被逐个处理，索引和表都按相同的顺序被访问，这是一个效率很高的处理过程。

索引已经关联的表行，及如果我们按照索引去访问表记录，实际上是走的按索引去访问表记录，以查询为例，按索引去查询某些记录，实际上我们是通过索引访问（B树遍历）效率是远大于顺序访问整个表的。

### 缓冲池和磁盘I/O

关系型数据库管理系统提供了缓存池的概念来最小化磁盘I/O。索引或表页在或不在缓冲池中，访问的成本是不同的。

所以当一个索引或表页被请求时，它的理想位置是在数据库缓冲池，如果它不再那儿，那么下一个最佳的位置实在磁盘服务器的读缓冲区中，如果它也不在那儿，那么就必须从磁盘进行一次很慢的读取，这一过程可能要花费很长的时间等待磁盘设备空闲下来。

从DBMS缓冲池进行读取到了索引或表页，那么唯一的成本就是去处理这些索引或者表记录。

顺序读取优于随机读，顺序读取的两个优势：

* 同时读取多个页意味着平均读取每个页的时间将会减少
* 由于DBMS事先直到需要读取哪些页，所以可以在页被真正请求之前就提前将其读取进来，即预读

辅助式随机读，数据库或者系统提供的用于优化随机读取的方式：

1. 自动跳跃式顺序读，如果不连续的行被按照同一个方向扫描，那么访问模式将会是跳跃式顺序的。带来的好处类似顺序读
2. 列表预读，在表和索引行顺序不一致的情况下，主动创造跳跃式顺序访问，即访问所有满足条件的索引行，在按照表行的顺序排序后再访问表行，DB2
3. 数据块预读，从索引片上搜集指针，然后再进行多重随机I/O来并行的读取表行，Oracle

### DBMS特性

1. 页，表页的大小限定了表行的最大长度，
2. 表聚簇，
3. 索引行，
4. 表行，
5. 索引组织表，
6. 页邻接

### B树索引的替代品

1. 位图索引
2. 散列
3. 聚簇索引

## SQL处理过程

 ### 谓词

where子句由一个或者多个谓词组成，又被称为条件表达式或者真值表达式。谓词表达式是索引设计的主要入手点，如果一个索引能够满足select查询语句的所有谓词表达式，那么优化器就很有可能建立起一个搞笑的访问路径。

### 优化器及访问路径

优化器决定了访问数据的方式，不同关系型系统的优化器各不相同，但他们都是在系统收集的统计信息的基础上，尽可能以最高效的方式访问数据。优化器是SQL处理过程的核心。

访问路径，在SQL语句能够被真正执行之前，优化器必须首先确定如何访问数据，包括：应该使用哪一个索引，索引的访问方式如何等等。

### 索引片及匹配列

索引的一个窄的片段将会被顺序扫描，其上索引行的值在100至110之间，相应的表行将通过同步读从表中读取，除非该页已在缓冲池中，所以访问路路径的成本很大程度上取决于索引片的厚度，即谓词表达式确定的值域范围，索引片越厚，需要顺序扫描的索引页就越多需要处理的索引记录也就越多，而最大开销还是来自于增加的对表的同步读操作，如果索引片比较窄，就会显著减少索引访问的那部分开销，但主要的成本节省还是在更少的对表的同步读取上。索引片的另一种描述，定义索引匹配列的数量。

### 索引过滤及过滤列

有时候，列可能即存在于where子句中，也存在于索引中，但这个列却不能参与索引片的定义，不过这些列仍然能够减少回表进行同步读的次数，所以这些列仍扮演着很重要的角色，我们称这些列为过滤列。

书中的例子说明了这个情况，有一个组合索引(A, B, C, D)，但查询条件为where A=:A and B > :B and C = :C，此时索引A B会生效，C的索引不会生效，但C起着过滤的作用因为它减少了读取表页的数量。

例子总结，针对组合索引(A, B, C, D)：

* where A = :A and B > :B and C = :C，索引在A, B上有效
* where A = :A and C = :C，索引在A上生效
* where A = :A and B = :B and C = :C，索引在A, B, C上生效

### 帮助优化器

基于成本的优化器在评估不同访问路径的成本时会fetch所有满足条件的记录——除非有特别说明，如果我们不需要所有的记录集合，那么可以设置仅fetch第前n行

* SQL Server，select语句最后添加options (fast n)
* Oracle，Oracle 9i select /* first_rows(n)*/
* DB2 for z/OS，optimize for n rows

### 何时确定访问路径

在每次SQL语句执行时都进行一次访问路径选择要比仅做一次消耗更多的资源，所以在应用程序开发的过程中，基于成本的优化器进行访问路径选择的处理成本不容忽视。

### 过滤因子

过滤因子描述了谓词的选择性，即表中满足谓词条件的记录行所占的比例。

在评估一个索引是否合适，最差情况下的过滤因子比平均过滤因子更重要，因为最差情况与最差输入相关，即在该输入条件下，基于特定索引的查询将消耗最长的时间。

### 组合谓词的过滤因子

如果组成谓词的列之间非相关，那么组合谓词的过滤因子可以从谓词的过滤因子中推导出来。书中的例子说明了组合谓词的过滤因子可能比两个谓词过滤因子乘积要低的多。

例一：

* cust: 1000000 -> where city = :city，ff = 0.2%
* cust: 1000000->where bd=:bd，ff = 0.27%
* cust: 1000000->where city=:city and bd=:bd，ff = 0.2% * 0.27%

在设计索引结构的时候，需要将组合谓词看作一个整体评估过滤因子，而不能仅仅基于零相关性来进行评估。

过滤因子相似的概念选择率：

选择率 = 100 * 某个键值对应的函数 / 表的总记录数

### 过滤因子对索引设计的影响

需要扫描的索引片的大小对访问路径的性能影响至关重要，在当前的硬件条件下，索引片大小的最重要的度量就是需要扫描的索引记录数，即匹配组合谓词的过滤因子与总行数的乘积。

### 物化结果集

物化结果集意味着执行必要的数据库访问来构建结果集，在最好的情况下，这只需要简单地从数据库缓冲池向应用返回一条记录，在最坏的情况下，数据库管理系统需要发起大量的磁盘读取。

## 为select语句创建理想的索引

 ### 三星索引

三星索引：

* 如果一个查询相关的索引行是相邻的，或至少足够靠近的话，那这个索引就可以被标记上第一颗星，这最小化了必须扫描的索引片的宽度
* 如果索引行顺序与查询语句的需求一直，则索引可以被标记上第二颗星，这排除了排序操作
* 如果索引行包含查询语句中所有列，那么索引就可以被标记上第三颗星，这避免了访问表的操作——仅访问索引就可以了。我们把至少包含第三颗星的索引称为对应查询语句的宽索引

宽索引：指一个至少满足第三颗星的索引，该索引包含了select语句所涉及的所有列，因此能够使得查询只需访问索引而无需访问表。

三星索引的构造：

1. 为了满足第一颗星，取出所有等值谓词的列，吧这些列作为索引最开头的列，以任意顺序都可以，这样必须扫描的索引片宽度将被压缩至最窄
2. 为了满足第二颗星，将order by列加入到索引，不要改变这些列的顺序，
3. 为了满足第三颗星，将查询语句中剩余的列加到索引中去，列在索引中添加的顺序对查询语句的性能没有影响，但是将易变列放在最后能够降低更新的成本。

### 范围谓词和三星索引

书中是通过一个例子来说明范围谓词对三星索引设计的影响的。

例一：

```sql
DECLARE CURSOR43 CURSOR FOR
SELECT CNO, FNAME
FROM CUST
WHERE LNAME BETWEEN :LNAME1 AND :LNAME2
      AND CITY = :CITY
      ORDER BY FNAME
```

首先最简单的是第三颗星，按照先前的所述，确保语句中所有列都在所以中就能满足第三颗星，这样不需要访问表，那么同步读也就不会造成问题。

添加ORDER BY 列才能使索引满足第二颗星，但是这个仅在将其放在BETWEEN谓词列LNAME之前的情况才成立 ，如索引(CITY,FNAME,LNAME)，由于CITY值只有1个=谓词，所以使用这个索引可以以FNAME的顺序排列，而不需要额外的排序。但是如果ORDER BY字段加在BETWEEN谓词列LANME后面，如索引(CITY,LNAME,FNAME)，那么索引行不是按FNAME顺序排列的因而就需要进行排序操作。因此，为了满足第二颗星，FNAME必须在BETWEEN谓词列LNAME的前面，如索引(FNAME,...)或索引(CITY,FNAME,...)。

在考虑第一颗星，如果CITY是索引的第一个列，那么我们将会有一个相对较窄的索引片需要扫描，这取决于CITY的过滤因子。但是如果用索引(CITY,LNAME,...)的话，索引片会更窄，这样在有两个匹配列的情况下我们只需要访问真正需要的索引行。但是为了做到这样，并从一个很窄的索引片中获益，其他列(FNAME)就不能放在两列之间。

到现在，设计的索引满足了第三颗星，但在第一，二颗星之间我们先前的设计只能满足其中一颗，如下。通常情况下，第一颗星比第二颗星更重要。

* 避免排序——拥有第二颗星
* 拥有可能的最窄索引片，不仅将需要处理的索引行数降至最低，而且将后续处理量，特别是表中数据行的同步读减小到最少——拥有第一颗星

再考虑一个索引(LNAME,CITY,...)LNAME是范围谓词，这意味着LNAME是参与索引匹配过程的最后一个列，等值谓词CITY不会再匹配过程中被使用，这样做将会导致只有一个匹配列——所以片将会比所用索引(CITY,LNAME,...)更宽。

### 为查询语句设计最佳索引的算法

根据书中描述一个三星索引是理想的索引，但当存在范围谓词时，这是不可能实现的，这是就需要进行设计索引设计的折衷——不得不牺牲第二颗星来满足一个更窄的索引片，这样最佳索引就拥有两颗星。

首先设计一个索引尽可能窄(第一颗星)的宽索引(第三颗星)，如果查询使用这个索引时不需要排序(第二颗星)，那这个索引就是三星索引。否则这个索引只能是二星索引，牺牲第二颗星。或者采用另一种选择，避免排序，牺牲第一颗星保留第二颗星。

### 需要为所有查询语句都设计理想索引吗

在为查询语句设计了一个最佳索引后，去看一下已经存在的索引是很有必要的，有可能某一个已经存在的索引几乎和理想索引差不多好用，特别是打算在这个已有索引的最后添加一些列的情况下。

多余的索引：

* 完全多余的索引，
* 近乎多余的索引，
* 可能多余的索引，

机械性地为每一个查询设计最佳索引时不明智地，因为索引维护可能会使的一些程序太慢或者使磁盘负载超负荷。最佳索引使一个好开端，但是再决定为一个新的查询创建理想索引前，需要先考虑下三种多余的索引。

## 前瞻性索引设计

评估索引对新的应用是否合适，两个可行的方法：BQ(基本问题法)，QUBE(快速上限估算法)

### BQ 基本问题法

基本问题：对于每一个select，是否有一个已存在的或者计划中的索引包含了where子句所引用的所有列(一个半宽索引)？

* 如果答案式否，那么我们应当首先考虑将缺少的谓词列加到一个现有的索引上去。这将产生一个半宽索引，尽管索引的等值匹配过程并不令人满意，但是索引过滤可以确保回表访问只发生在所有查询条件都满足的时候
* 如果这都还没有达到足够的性能，那么下一个选择就是将所有涉及的列都加到索引上，以使访问路径只需访问索引，这将产生一个避免所有表访问的宽索引
* 如果select仍然很慢，就应当使用第4章介绍的两个候选索引算法来设计一个新的索引。

注意：BQ并不能保证足够的性能，BQ的目的只是确保我们至少可以通过索引过滤来最小化对表的访问。然而用BQ可以提早发现大部分的索引问题。

### QUBE 快速上限估算法

