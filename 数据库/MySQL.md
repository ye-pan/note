## MySQL

> 存储引擎

* InnoDB  MySQL默认事务型存储引擎，被设计用来处理大量短期事务。采用MVCC支持高并发，实现了4个标准的隔离级别，默认级别为REPEATABLE READ，通过间隙锁策略防止幻读。
* MyISAM 
* memory

> 转换表的存储引擎

* alter table table_name engine = InnoDB;
* 导出->导入
* create table new_engine_table like old_engine_table;

​       alter table new_engine_table engine=new_engine;

​       insert into new_engine_table select * from old_engine_table; 

## 数据库scheme优化

### 数据类型

- 更小的通常更好，占用更少的磁盘，内存和CPU缓存，处理时需要的CPU周期也更少
- 简单就好，简单的数据类型的操作通常需要更少的CPU周期
- 尽量避免NULL，如果计划在列上建索引，就应该尽量避免设计成为为NULL的列。

整数类型：tinyint，smallint，mediumint，int，bigint分别使用8，16，24，32，64位存储。可以加unsigned修饰，将数字上限提高一倍。可以加位数，对大多数应用这是没有意义的，比如对于存储和计算来说，int(1)和int(20)是相同的。

实数：float，double不精确的类型，decimal精确的类型。

字符串类型：

### scheme问题

- 太多的列
- 太多的关联，MySQL限制每个关联操作最多只能有61张表，最好使用12个表以内。
- 全能的枚举
- 变相的枚举
- 非此发明的NULL，避免NULL但在确实需要表示未知值时就应该使用它。

### 范式和反范式

范式化中每个事实数据会出现并且只出现一次，反范式中，信息时冗余的。

范式优点：

* 范式化的更新操作通常比反范式化要快
* 当数据较好的范式化时，就只有很少或者没有重复数据，所以只需要修改更少的数据
* 范式化的表通常更小，可以更好的放在内存中，所以执行操作会更快
* 很少有多余的数据意味着检索列表数据时更少需要DISTINCT或者GROUP BY语句

范式缺点：通常需要关联，稍微复杂一些的查询语句在符合范式的scheme上都可能需要至少一次关联。

反范式：

* 反范式化的schema因为所有数据都在一张表中，可以很好的避免关联
* 单独的表也能使用更有效的索引策略

> 如果不需要关联表，则对大部分查询最差的情况，即使表没有使用索引——使用全表扫描，当数据比内存大时这可能比关联要快的多，因为这样避免了随机IO。
>
> 全表扫描基本上时顺序IO，但也不是100%的，跟引擎的实现有关。

> 更快的读，更慢的写。为了提升读查询的速度，经常会需要建一些额外索引，增加冗余列，甚至是创建缓存表和汇总表。这些方法会增加写查询的负担，也需要额外的维护任务，但在设计高性能数据库时，这些都是常见的技巧：虽然写操作变得更慢了，但显著的提高了读操作的性能。

### 缓存表和汇总表

有时为了满足特定检索需求，需要创建一张完全独立的汇总表或缓存表。

物化视图：类似汇总表，但会主动抓取数据的变更，不需要自己通过查询来维护。

计数器表：

> 更快的读 更慢的写
>
> 为了提升查询速度，经常会需要建一些额外索引，增加冗余列，甚至是创建缓存表和汇总表，这些方法会增加写的负担。

### 加快ALTER TABLE



## 索引

索引时存储引擎用于快速找到记录的一种数据结构，索引对于良好的性能非常关键。索引优化应该是对查询性能优化最有效的手段了，索引能够轻易将查询性能提高几个数量级。

### 索引基础

例：

```mysql
select first_name from sakila.actor where actor_id = 5;
```

如果actor_id列上建有索引，则MySQL将使用该索引找到actor_id为5的行，也就是说MySQL先在索引上按值进行查找，然后返回所有包含该值的数据行。

索引可以包含一个或多个列的值，如果索引包含多个列，那么列的顺序也十分重要，因为MySQL只能高效的使用索引的最左前缀列。

### 索引类型

索引有很多种类型，可以为不同的场景提供更好的性能，在MySQL中，索引是在存储引擎层而不是服务器层实现的。

#### B-Tree索引

InnoDB使用的时B+树。存储引擎以不同的方式使用B-Tree索引，性能也各有不同。如，MyISAM使用前缀压缩技术使得索引更小通过数据的物理位置引用被索引的行，但InnoDB则按照原数据格式进行存储，根据主键引用被索引的行。下图展示了B-Tree索引的抽象表示，大概反映了InnoDB索引时如何工作的。

![1566260241762](assets/1566260241762.png)

B-Tree索引能够加快访问数据的速度，因为存储引擎不再需要进行全表扫描来获取需要的数据，取而代之的是从索引的根节点开始进行搜索根节点的槽总存放了指向子节点的指针，存储引擎根据这些指针向下层查找，通过比较节点页的值和要查找的值可以找到合适的指针进入下层子节点， 这些指针实际上定义了子节点页中值的上限和下限，最终存储引擎要么是找到对应的值，要么该记录不存在。叶节点比较特别，它们的指针指向的是被索引的数据。根节点和叶子节点之间可能有很多层节点页，树的深度和表的大小直接相关。B-Tree对索引是顺序组织存储的，所以很适合查找范围数据。

B+Tree索引并不能找到一个给定键值的具体行，B+Tree索引能找到的知识被查找数据行所在的页，然后数据库通过把页读入内存，再在内存中进行查找，得到查找的数据。

当取出的数据量超过表中数据的20%，优化器就不会使用索引，而是进行全表扫描。

例：

```mysql
create table people (
	last_name varchar(50) not null,
	first_name varchar(50) not null,
	dob date not null,
	gender enum('m', 'f') not null,
	key sample_idx (last_name, first_name, dob)
);
```

索引sample_idx的结构如图

![1566261226248](assets/1566261226248.png)

B-Tree索引适用于全键值，键值范围或键前缀查找。能够有效使用sample_idx的查询如下：

1. 全值匹配，查询条件和索引的所有列匹配
2. 匹配最左前缀，查询last_name = Aleen的人，只使用索引的第一列
3. 匹配列前缀，查询last_name以'J'开头的，只使用了索引第一列
4. 匹配范围值，查询last_name范围为Allen和Barrymore之间的，只使用了第一列
5. 精确匹配某一列并范围匹配另外一列，
6. 只访问索引的查询，即需要的数据列和查询条件列全部被索引覆盖，存储引擎只需通过访问索引就可以的到所有数据。

因为索引树种的节点是有序的，所以除了按值查找外，还可用于查询中的ORDER BY操作。

限制：

* 如果不是按照索引的最左列开始查找，则无法使用索引
* 不能跳过索引中的列
* 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引优化查找

索引失效：

* 查询列中有函数计算
* 查询列中有模糊匹配
* 如果查询条件中有or，索引会失效，除非所有条件都加上索引
* 使用不等于（!= 或 <>）
* is null 或者 is not null
* 字符串不加引号会导致索引实效
* 联合索引中不符合最左匹配原则

索引列的顺序非常重要，上面关于索引有效性的限制都和索引顺序有关。

#### 哈希索引

基于哈希表实现，只有精确匹配索引的所有列。存储引擎对索引的列建立hash code，并存储在哈希索引中，同时保存了指向每个数据行的指针。MySQL中只有Memory引擎显示支持哈希索引，它也支持B-Tree索引，且Memory使用哈希索引作为默认索引，用链接法解决了哈希冲突。

例：

```mysql
create table testhash (
 fname varchar(50) not null,
 lname varchar(50) not null,
 key hash_idx(fname) using hash
) engine=memory;
```

针对*select lname from testhash where fname = 'Peter'*，MySQL先计算查询条件的哈希值，并查找索引中对应的记录，找到后，再比较值是否相同。

限制：

* 哈希索引只包含哈希值和行指针，而不存储字段值，所以不能使用索引中的值来避免读取行。
* 哈希索引数据并不是按照索引值顺序存储的，所以也就无法用于排序
* 哈希索引也不支持部分索引列匹配查找，因为哈希索引始终是使用索引列的全部内容来计算哈希值的
* 哈希索引只支持等值比较查询，包括=，in()，<=>，不支持任何范围查询
* 访问哈希索引的数据非常快，除非有很多哈希冲突
* 如果哈希冲突很多的话，一些索引维护操作的代价也会很高

> 因为这些限制，哈希索引只适用于某些特定的场合，而一旦适合哈希索引，则它带来的性能提升非常显著。
>
> InnoDB引擎有一个特殊的功能叫做自适应哈希索引，当InnoDB注意到某些索引值被使用得非常频繁时，它会在内存中基于B-Tree索引之上在创建一个哈希索引，这样就让B-Tree索引也具有一些哈希索引的优点了。

#### 空间数据索引

MyISAM表支持空间索引，可以用作地理数据存储。

#### 全文索引

他查找的时文本中的关键词，而不是直接比较索引中的值，全文索引适用于MATCH AGAINST操作，而不是WHERE条件操作。

#### 联合索引



### 索引的优点

B-Tree索引可以用于加速查询，用来做ORDER BY和GROUP BY，如果查询的字段只包括索引列，则直接通过索引就可以查询所需数据。

* 索引大大减少了服务器需要扫描的数据量
* 索引可以帮助服务器避免排序和临时表
* 索引可以将随机IO变为顺序IO

### 高性能索引策略

#### 独立地列

索引是独立的列，索引列不能是表达式的一部分，也不能是函数的参数。

例中，索引列其实会失效：

```mysql
select actor_id from actor where actor_id + 1 = 5;
select  ... where to_days(current_date) - to_days(date_col) <= 10;
```

#### 前缀索引和索引选择性

如果需要索引很长的字符串，这会让索引大且慢，可以通过模拟哈希索引来避免索引整个字符串，或者使用前缀索引——索引字段开始的部分字符串。

索引选择性：不重复的索引值和记录总数的比值，返回从1/#T到1之间，索引的选择性越高则查询效率越高。比如，唯一索引的选择性是1。

前缀索引的例子：

```mysql
#初始化数据
create table city_demo(city varchar(50) not null);
insert into city_demo(city) select city from city;
insert into city_demo(city) select city from city_demo;
update city_demo set city = (select city from city order by rand() limit 1);

#尝试对city字段选择性做预估
select count(*) as cnt, city 
from city_demo group by city order by cnt desc limit 10;
select count(*) as cnt, left(city, 7) as pref 
from city_demo group by pref order by cnt desc limit 10;

#计算city字段索引选择性
select count(distinct city)/count(*) from city_demo;

#计算前缀索引选择性
select count(distinct left(city, 3))/count(*) as sel3,
count(distinct left(city, 4))/count(*) as sel4,
count(distinct left(city, 5))/count(*) as sel5,
count(distinct left(city, 6))/count(*) as sel6,
count(distinct left(city, 7))/count(*) as sel7
from city_demo;
```

索引选择性计算如图，我们可以得出，前缀长度用6就够了。

![1566346615829](assets/1566346615829.png)



![1566346646159](assets/1566346646159.png)

有时后缀索引也有用途，但MySQL不支持，可以把字符串反转后存储，并基于此建立前缀索引。

#### 多列索引

MySQL再5.0以后的版本引入了一种叫做“索引合并”的策略用于，再多个列上单独创建了索引这种情况。适用于：OR或AND条件查询。索引合并策略有时候是一种优化的结果，但实际上更多时候说明了表上的索引建的很糟糕：

* 当出现服务器对多个索引做相交操作时（通常多个AND），通常意味着需要一个包含所有相关列的多列索引，而不是多个独立的单列索引
* 当服务器需要对多个索引做联合操作时（通常多个OR），通常需要消耗大量CPU和内存资源在算法的缓存，排序和合并操作上。特别是当其中有些索引的选择性不高，需要合并扫描返回的大量数据的时候
* 更重要的是，优化器不会把这些计算到查询成本中，优化器只关心随机页面读取。

#### 聚簇索引

聚簇索引并不是一种单独的索引类型，而是一种数据存储方式。聚簇表示数据行和相邻的键值紧凑的存储在一起。因为无法同时把数据行存放在两个不同的地方，所以一个表只能由一个聚簇索引。

InnoDB通过主键聚集数据，如果没有主键则选择一个唯一的非空索引代替，如果没有，则会隐式的定义一个主键来作为聚簇索引。

聚簇索引的一些优点：

* 可以把相关数据保存在一起。
* 数据访问更快。
* 使用覆盖索引扫描的查询可以直接使用叶节点中的主键值。

聚簇索引的一些缺点：

* 聚簇数据最大限度地提高了I/O密集型应用的性能，但如果数据全部都放在内存中，则访问的顺序就没那么重要了，聚簇索引也就没什么优势了。
* 插入速度严重依赖于插入顺序
* 更新聚簇索引列的代价很高，因为会强制InnoDB将每个被更新的行移动到新的位置。
* 基于聚簇索引的表在插入新行，或者主键被更新导致需要移动行的时候，可能面临页分裂。
* 聚簇索引可能导致全表扫描变慢，尤其是行比较稀疏，或者由于页分裂导致数据存储不连续的时候。
* 二级索引可能比想象的要更大，因为在二级索引的叶子节点包含了引用行的主键列。
* 二级索引访问需要两次索引查找，而不是一次。二级索引叶子节点保存的不是指向行的物理位置的指针，而是行的主键。

#### 覆盖索引

MySQL可以使用索引来直接获取列的数据，这样就不再需要读取数据行，这种包含所有需要查询的字段的值，我们就称之为覆盖索引。

覆盖索引能极大的提高性能，好处：

* 索引条目通常远小于数据行大小，这样只读取索引，可以极大的减少数据访问量。
* 因为索引是按照列顺序存储的，所以对于I/O密集型的范围查询会比随机从磁盘读取每一行数据的I/O要少的多。
* 一些存储引擎如MyISAM在内存中只缓存索引，数据则依赖于操作系统来缓存，因此要访问数据需要一次系统调用。
* 由于InnoDB的聚簇索引，覆盖索引对InnoDB表特别有用，InnoDB的二级索引在叶子节点中保存了行的主键值，所以如果二级索引能够覆盖查询，则可以避免对于主键索引的二次查询。

覆盖索引必须存储列的值，而哈希索引，空间索引和全文索引等都不存储索引列的值，所以MySQL只能使用B-Tree索引做覆盖索引。

#### 使用索引扫描来做排序

MySQL有两种方式生成有序结果：通过排序操作，或者按索引顺序扫描。MySQL可以使用同一个索引既满足排序有用于查询，如果可能设计索引是应该尽可能地同时满足这两种任务。

## 查询性能优化

库表结构优化，索引优化，查询优化对于高性能必不可少。查询优化的是响应时间，如果把查询看作是一个任务，那么它由一系列子任务组成，每个子任务都会消耗一定的时间。优化查询，实际上要优化其子任务，要么消除其中一些子任务，要么减少子任务的执行次数，要么让子任务运行的更快。

### 慢查询基础：优化数据访问

查询性能低下最基本的原因是访问的数据太多，大部分性能低下的查询都可以通过减少访问的数据量的方式进行优化，下面两个步骤对分析来说很有用：

* 确认应用程序是否在检索大量超过需要的数据
* 确认MySQL服务器层是否在分析大量查过需要的数据行

是否向数据库请求了不需要的数据：

* 查询不需要的记录
* 多表关联时返回全部列
* 总是取出全部列
* 重复查询相同数据

MySQL是否在扫描额外的记录：

最简单的衡量查询的3个指标：

* 响应时间，指服务时间和排队时间
* 扫描的行数和返回的行数，理想情况下扫描的行数和返回的行数应该是相同的
* 扫描的行数和访问类型，

一般MySQL能够以如下三种方式应用where，从好到坏：

* 在索引中使用where条件来过滤不匹配的记录，这是在存储引擎层完成的。
* 使用索引覆盖扫描（在extra列中出现了using index）来返回记录，直接从索引中过滤不需要的记录来返回命中的结果。这是在MySQL服务器层完成的，但无须再回表查询记录
* 从数据表中返回数据，然后过滤不满足条件的记录（再extra列中出现using where）。这在MySQL服务器层完成，MySQL需要先从数据表读出记录然后过滤。

如果发现查询需要扫描大量的数据但只返回少数的行，那么通常可以尝试下面技巧去优化它：

* 使用索引覆盖扫描，把所有需要用的列都放到索引中
* 改库表结构
* 重写这个复杂的查询，让MySQL优化器能够以更优化的方式执行这个查询

### 查询执行的基础

![1566778914159](assets/1566778914159.png)

show full processlist，查看所有线程，线程状态：

* sleep
* query
* locked
* analyzing and statistics
* copying to tmp table 
* sorting result
* sending data

SQL->查询缓存->解析器->解析树->预处理器->解析树->查询优化器->查询执行计划->查询执行引擎->存储引擎->查询缓存->返回

## explain

![1569196503105](../assets/1569196503105.png)

* select_type：
  * SIMPLE：简单查询，不包括子查询，关联查询
  * PRIMARY：查询中如果有复杂的部分，最外层的查询将被标记为PRIMARY
  * UNION：union中第二个或后面的select语句
  * DEPENDENT UNION：union中的第二个或后面的select语句，取决于外面的查询
  * UNION RESULT：union的结果，union语句中第二个select开始后面所有的select
  * SUBQUERY：子查询中的第一个，结果不依赖于外部查询
  * DEPENDENT SUBQUERY：子查询中的第一个SELECT，依赖于外部查询
  * DERIVED：派生表的SELECT, FROM 子句的子查询
  * UNCACHEABLE SUBQUERY：一个子查询的结果不能被缓存，必须重新评估外连接的第一行
* type，查询性能一次递增：
  * ALL：全表扫描，最耗性能，遍历全表
  * index：全索引列表扫描，遍历索引树
  * range：对单个索引列进行范围查找
  * index_merge：多个索引合并查询
  * ref：根据单个索引查找，表示上述表的连接匹配条件
  * eq_ref：连接时使用primary key 或者 unique类型
  * constant：常量，MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL将能将该查询转换为一个常量
  * system：系统，是constant的特例，当查询只有一行的情况下，使用system
  * NULL：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引例去最小值可以通过单独索引查找完成
* possible_key，可能使用的索引
* key，真实使用的索引
* rows，扫描的行数
* extra，包含MySQL为了解决查询的详细信息：
  * using index：将使用覆盖索引，以避免表访问
  * using where：MySQL服务器将在存储殷勤检索后再进行过滤，暗示：查询可受益于不同的索引
  * using temporary：意味着MySQL在对查询结果排序时会使用一个临时表
  * using filesort：
  * range checked for each record 

关于explain：

* explain不会告诉你关于触发器，存储过程的信息或用户自定义对查询的影响情况
* explain不考虑各种cache
* explain不能显示MySQL在执行查询时所作的优化工作
* 部分统计信息是估算的，并非精确
* explain只能解释select操作，其它操作要重写为select后查看执行计划

## 锁

锁机制用于管理对共享资源的并发访问，提供数据的完整性和一致性。InnoDB存储引擎会在行级别上对表数据上锁。MyISAM是表锁，可以并发读，不支持并发写。

### InnoDB中的锁

InnoDB实现了两种标准行级锁：

* 共享锁，允许事务读一行数据
* 排他锁，允许事务删除或者更新一行数据

当一个事务获得了行r的共享锁，那么另外的事务可以立即获得行r的共享锁，这也叫锁兼容。如果有事务想获得行r的排他锁，则它必须等待事务释放行r的共享锁，这叫锁不兼容。

![1566782159974](assets/1566782159974.png)

为了支持不同粒度上进行加锁操作，InnoDB存储引擎支持一种额外的锁方式，我们称之为意向锁，意向锁是表级别的锁，其设计目的主要是为了在一个事务中揭示下一行将被请求的锁的类型

* 意向共享锁，IS lock，事务想要获得一个表中某几行的共享锁
* 意向排他锁，IX lock，事务想要获得一个表中某几行的排他锁

因为innodb支持的是行级别的锁，所以意向锁其实不会阻塞除全表扫描以外的任何请求

### 一致性的非锁定读

指InnoDB通过行多版本的方式来读取当前执行时间数据库中行的数据，如果读取的行正在执行delete，update操作，这是读取操作不会因此而会等待行上锁的释放，相反InnoDB会去读取行的一个快照数据。

快照数据是指该行之前版本的数据，该实现是通过undo段来实现的，而undo段用来在事务中回滚数据，因此快照数据本省是没有额外开销。

非锁定读机制大大提供高了数据读取的并发行

select ... for update：对读取的行加一个X锁，其它事务想在这些行上加任何锁都会被阻塞

select ... lock in share mode：对读取行加一个S锁，其它事务可以向被锁定的记录加S锁，但是对于加X锁，则会被阻塞

对于一致性非锁定读，即使读取的行已被使用select  ... for update，也是可以进行读取的。另外，select ... for update，select ... lock in share mode必须在一个事务中，当事务提交了，锁也就释放了。

## 事务

InnoDB的事务完全符合ACID特性：

* 原子性，atomicity
* 一致性，consistency
* 隔离性，isolation
* 持久性，durability

#### 事务实现

redo，在InnoDB种，事务日志通过重做日志文件和InnoDB存储引擎的日志缓冲来实现。当开始一个事务时，回记录该事务的一个LSN（log Sequence Nubmer），当事务执行时，会往InnoDB存储引擎的日志缓冲力插入事务日志，当事务提交时，必须将InnoDB存储引擎的日志缓冲写入磁盘。也就是在写数据前，需要先写日志，这种方式称为预写日志方式。

#### 事务隔离级别

脏读

不可重复读

幻读

## SQL编写

### CASE表达式

```mysql
	#简单CASE表达式
    CASE sex WHEN '1' THEN '男' 
    		 WHEN '2' THEN '女'
    		 ELSE '其它' END
    #搜索CASE表达式
    CASE WHEN sex = '1' THEN '男'
         WHEN sex = '2' THEN '女'
         ELSE '其它' END
```

发现WHEN子句为真的时候，CASE表达式的判断就会结束。

注意点：

* 统一个分支返回的数据类型
* 不要忘了写END
* 养成写ELSE的习惯

### 自连接的用法

针对相同的表进行的连接被称为自连接。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        

## MySQL操作

常用命令：show databases，show tables，describe tablename，show variables like '%'， show create database db_name，

导入*.sql文件：source *.sql

迁库/备份（innodb）：

在innodb模式下只有数据库的data文件，之前没有任何备份措施，恢复数据库。步骤：

1. 删掉现有mysql服务的data文件夹（记得备份）；

2. 将需要恢复的data文件夹放在对应的位置；

3. 修改my.cnf，加入innodb_force_recovery=4或者6；

4. 直接启动，应该可以直接使用了；

5. mysqldump备份数据并且恢复。

允许用户远程访问：grant all privileges on *.* to 'root'@'%' identified by 'password' with #grant option;

​           GRANT ALL PRIVILEGES ON *.* TO ['root'@'%'](mailto: root @ %) IDENTIFIED BY 'youpassword' WITH GRANT OPTION;

登录：mysql -h host -u username -p，mysql -h host -u username -p  databasename

创建用户：create user 'username'@'host' identified by 'password';

赋予权限：grant all privileges on *.* to 'username'@'%' identified by 'password'，grant all privileges on *.* to 'username'@'%';

刷新权限：flush privileges;

Windows启动/停止：net start mysql/net stop mysql

Linux启动：service mysqld start

### 安装部署

Windows忘记root密码解决方法：

方法一：

1. 启用mysqld-net --skip-grant-tables;

2. msyq -uroot

3. mysql> use mysql;

​    mysql> update user set password=password('new_passwd') where user='root';

​    mysql> flush privileges;

​    mysql> exit;

4. 关闭刚刚启动的服务，重新正常模式启动MySQL服务

方法二：

1. 首先在 MySQL的安装目录下 新建一个pwdhf.txt, 输入文本：SET PASSWORD FOR 'root'@'localhost' = PASSWORD('new_passwd'); 

2. 然后运行： mysqld-nt --init-file=../pwdhf.txt 

3. 再重新以正常模式启动MYSQL 即可

压缩版安装：

1. 配置内容：

```
 [client]

  port = 3306

  default-character-set = utf8

  [mysqld]

  port = 3306

  character_set_server = utf8

  basedir =  D:\mysql-5.7.16

  datadir =  D:\mysql-5.7.16\data

  sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

 命令：mysqld --initialize  初始化mysql，生成data中的文件；

命令：mysqld  -install   安装mysql  

命令：net start mysql    启动mysql服务

   其他一些命令：mysqld  -remove   卸载mysql； 

​                 net stop mysql 停止mysql服务；

   mysql和mysqld的区别：mysql为客户端的程序，mysqld为服务器端的程序；

3、配置好以上就可以登录mysql了：

  a、首次登录时由于没有设置root密码，登录会报错，此时在配置文件my.ini中加上skip-grant-tables,保存后，重启mysql服务，在cmd中依次输入：net stop mysql; net start mysql; mysql -uroot -p，回车后就直接登录了；

  b、设置root密码：① 进入mysql数据库：use mysql;

② 设置密：update user set authentication_string=password('xxx') where user='root' and Host = 'localhost';  （5.7版本）

update user set password=password("xxx") where user="root";  （5.5版本）

③ 退出数据库：exit （或者quit）

④ 密码改好后，再进入my.ini，注释掉skip-grant-tables，保存；

⑤ 再重启mysql服务，重新登录即可；

4、再次登录mysql，输入命令：alter user 'root'@'localhost' identified by 'xxx';

退出：quit

至此安装配置完成！可以开始使用mysql了。

Linux压缩版安装：

1. 在解压目录下新建：my.cnf；

2. 内容如下：

[mysqld]

port    = 3306

basedir    =/users/zgyjs/mysql5.6

datadir    =/users/zgyjs/mysql5.6/data

3. 安装配置数据库

./scripts/mysql_install_db --user=linux_user --defaults-file=./my.cnf

4. 启动

./bin/mysqld_safe --defaults-file=./my.cnf --user=linux_user 启动服务

./bin/mysql -uroot

常用简单操作：

检查是否有进程：ps -ef | gerp mysqld

启动：./bin/mysqld_safe --defaults-file=./my.cnf --user=zgyjs 

修改root用户密码：mysql> use mysql;

mysql> update user set password=password('new_passward') where user='root';

mysql> flush privileges;

## 附录.A MySQL

MySQL官方有个自带的测试数据库，叫employees，超过30W用户数据。

具体参考：https://github.com/datacharmer/test_db

## 参考

* 《高性能MySQL》