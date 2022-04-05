# **MySQL**索引详解
## 1. 索引是什么

官方介绍索引是帮助MySQL高效获取数据的数据结构。简单来讲，索引类似于一本书的目录，可以帮助我们快速高效的查询数据。

## 2. 索引的优缺点

**优点**

1. 可以提高数据的检索的效率，降低数据库的IO成本。
2. 可以通过索引对数据进行排序，降低数据排序的成本 ，降低CPU的消耗。

- 注
  - 被索引的列会进行自动的排序，包括“单列索引”和“组合索引”，组合索引的排序会比较复杂。
  - 如果按索引的顺序进行排序，对应的`order by`语句的速度会提高很多。

**缺点**

1. 索引会占磁盘的空间（一般来说索引本身也很大，不可能全部存储在内存中，因此**索引往往是存储在磁盘上的文件中的**。可能存储在单独的索引文件中，也可能和数据一起存储在数据文件中）。
2. 索引虽然可以提高数据的查询效率，但是索引也需要维护，会降低表更新的效率。每次对数据库表进行增删改操作时，MySQL不仅要保存或更新数据，还要保存或更新对应的索引。

## 3. 索引的类型

**主键索引**

主键索引的列中的值必须保证唯一，不允许有空值。

 **唯一索引**

唯一索引的列中的值必须保证唯一，可以有空值。

**普通索引**

MySQL中的基本索引类型，没什么限制，可以有重复值，也可以存在空值。

**全文索引**

只能在文本类型CHAR,VARCHAR,TEXT类型字段上创建全文索引。字段长度比较大时，如果创建普通索引，在进行like模糊查询时效率比较低，这时可以创建全文索引。MyISAM和InnoDB中都可以使用全文索引。（一般不会使用，基本都使用Elasticsearch）。

**前缀索引**

在文本类型如CHAR,VARCHAR,TEXT类列上创建索引时，可以指定索引列的长度，但是数值类型不能指定。

## 4. 索引的数据结构
### Hash索引

Hash索引的底层就是Hash表。Hash表是键值对的集合，我们使用Hash表存储表数据时，Key可以存储索引列，Value可以存储行记录或者磁盘地址。Hash在等值查询时效率很高，时间复杂度为O(1)。但是不支持范围的快速查找，范围查找时只能通过全表扫描的方式。

**显然这种并不适合作为经常需要查找和范围查找的数据库索引使用。**

### 二叉查找树

![二叉树结构](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\二叉树数据结构.png)

二叉树的特点就是每个节点最多有两个分叉，左子树和右子树数据顺序**左小右大**。这个特性就是为了每次每次都可以进行折半查找减少IO的次数，但是二叉树的结构取决于根节点，在极端的情况下会转换成一种线性结构。

![不分叉的二叉树](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\不分叉的二叉树.png)

**显然这种情况不稳定的我们再选择设计上必然会避免这种情况的**

### 平衡二叉树

平衡二叉树是采用二分法思维，平衡二叉查找树除了具备二叉树的特点之外，最主要的特点就是左右子树的高度差的绝对值最大相差1。在插入或者删除数据时通过左旋\右旋保持二叉树的平衡，不会出现左子树很高，右子树很矮的情况。

使用平衡二叉查找树的性能近似于二分查找法，时间复杂度为O(log2n)。查询id为8的只需要进行2次IO。

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\二叉树数据结构.png)

虽然平衡二叉查找树相对于二叉查找树有明显的优势，但是依然存在一些将问题：

1. 时间复杂度和树的高度有关。树有多高就要检索多少次，每个节点的读取都对应一次IO操作。树的高度就等于每次查找数据时磁盘IO的操作次数。磁盘每次寻道时间为10ms，在表数据量很大时，查询的性能会很差。（1百万的数据量，log2n约等于20次磁盘IO，时间20*10=0.2s）。
2. 平衡二叉树不支持范围查询快速查找，范围查询时需要从根节点多次遍历，查询效率不高。

###  B-Tree

我们知道，MySQL的数据是存储在磁盘文件中，在查询处理数据的时候，需要先把磁盘中的数据加载到内存中，磁盘的IO操作时非常的耗时的，所以我们要优化的就是尽量减少磁盘的IO操作。访问二叉树的每个节点就是进行一次IO操作，如果想要减少磁盘的IO操作，就要尽量降低树的高度。所以，我们要考虑的就是如何降低树的高度？

假如key为bigint=8字节，每个节点有两个指针，每个指针为4个字节，一个节点占用的空间就是8+4+4=16字节。

MySQL的InnoDB存储引擎中，一次IO会读取一页的数据量（默认一页16k数据量）。那么二叉树每次IO读取的数据量就是16个字节，空间利用率非常的低。为了可以以最大化利用一次IO的空间，最简单的做法就是在一个节点中存储多个元素，在每个节点中存储尽量多的数据。每个节点可以存储1000个索引（16k/16=1000），这样二叉树就该造成了多叉树，通过增加树的叉数，从高瘦变成了矮胖树。构建100万条数据，树的高度只需要两层就可以（1000*1000=100000），也就是说只需要2次磁盘IO就可以查询到数据。磁盘IO次数变少了，查询数据的效率也就提高了。

这就是我们要说的多路平衡查找树B-Tree。B-Tree具有以下的特点：

1. B树每个节点中存储多个元素，每个节点有多个分叉。
2. 节点中的元素包含键值（key）和数据（data），节点中的键值从大到小排列。也就是说所有的节点及存储索引也存储数据。
3. 父节点中的元素不会出现在子节点中。
4. 所有的叶子节点都位于同一层，叶子节点具有相同的深度叶子节点之间互相独立。

![B-Tree](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\B-Tree结构.png)

举个例子，在b树中查询数据的情况：

假如我们查询值等于10的数据。查询路径磁盘块1->磁盘块2->磁盘块5。

第一次磁盘IO：将磁盘块1加载到内存中，在内存中从头遍历比较，10<15，走左路，到磁盘寻址磁盘块2。

第二次磁盘IO：将磁盘块2加载到内存中，在内存中从头遍历比较，7<10，到磁盘中寻址定位到磁盘块5。

第三次磁盘IO：将磁盘块5加载到内存中，在内存中从头遍历比较，10=10，找到10，取出data，如果data存储的行记录，取出data，查询结束。如果存储的是磁盘地址，还需要根据磁盘地址到磁盘中取出数据，查询终止。

相比二叉平衡查找树，在整个查找过程中，虽然数据的比较次数并没有明显减少，但是磁盘IO次数会大大减少。同时，由于我们的比较是在内存中进行的，比较的耗时可以忽略不计。B树的高度一般2至3层就能满足大部分的应用场景，所以使用B树构建索引可以很好的提升查询的效率。

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\B-Tree查询过程.png)

看到这里一定觉得B树就很理想了，但是前辈们会告诉你依然存在可以优化的地方：

> 1. B树不支持范围查询的快速查找，你想想这么一个情况如果我们想要查找10和35之间的数据，查找到15之后，需要回到根节点重新遍历查找，需要从根节点进行多次遍历，查询效率有待提高。
> 2. 如果data存储的是行记录，行的大小随着列数的增多，所占空间会变大。这时，一个页中可存储的数据量就会变少，树相应就会变高，磁盘IO次数就会变大。

### B+Tree

B+树，作为B树的升级版，在B树基础上，MySQL在B树的基础上继续改造，使用B+树构建索引。B+树和B树最主要的区别在于**非叶子节点是否存储数据**的问题。

- B树：非叶子节点和叶子节点都会存储数据。
- B+树：只有叶子节点才会存储数据，非叶子节点至存储键值。叶子节点之间使用双向指针连接，最底层的叶子节点形成了一个双向有序链表。

![B+Tree结构](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\B+Tree结构.png)

B+树的最底层叶子节点包含了所有的索引项。从图上可以看到，B+树在查找数据的时候，由于数据都存放在最底层的叶子节点上，所以每次查找都需要检索到叶子节点才能查询到数据。

所以在需要查询数据的情况下每次的磁盘的IO跟树高有直接的关系，但是从另一方面来说，由于数据都被放到了叶子节点，放索引的磁盘块锁存放的索引数量是会跟这增加的，相对于B树来说，B+树的树高理论上情况下是比B树要矮的。

也存在索引覆盖查询的情况，在索引中数据满足了当前查询语句所需要的全部数据，此时只需要找到索引即可立刻返回，不需要检索到最底层的叶子节点。

**等值查询**

假如我们查询值等于9的数据。查询路径磁盘块1->磁盘块2->磁盘块6。

第一次磁盘IO：将磁盘块1加载到内存中，在内存中从头遍历比较，9<15，走左路，到磁盘寻址磁盘块2。

第二次磁盘IO：将磁盘块2加载到内存中，在内存中从头遍历比较，7<9<12，到磁盘中寻址定位到磁盘块6。

第三次磁盘IO：将磁盘块6加载到内存中，在内存中从头遍历比较，在第三个索引中找到9，取出data，如果data存储的行记录，取出data，查询结束。如果存储的是磁盘地址，还需要根据磁盘地址到磁盘中取出数据，查询终止。（这里需要区分的是在InnoDB中Data存储的为行数据，而MyIsam中存储的是磁盘地址。）

过程如图：

![B+Tree等值查询](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\B+Tree查询过程.png)

**范围查询：**

假如我们想要查找9和26之间的数据。查找路径是磁盘块1->磁盘块2->磁盘块6->磁盘块7。

首先查找值等于9的数据，将值等于9的数据缓存到结果集。这一步和前面等值查询流程一样，发生了三次磁盘IO。

查找到15之后，底层的叶子节点是一个有序列表，我们从磁盘块6，键值9开始向后遍历筛选所有符合筛选条件的数据。

第四次磁盘IO：根据磁盘6后继指针到磁盘中寻址定位到磁盘块7，将磁盘7加载到内存中，在内存中从头遍历比较，9<25<26，9<26<=26，将data缓存到结果集。

主键具备唯一性（后面不会有<=26的数据），不需再向后查找，查询终止。将结果集返回给用户。

![B+Tree范围查询](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\B+Tree范围查询.png)

**可以看到B+树可以保证等值和范围查询的快速查找，MySQL的索引就采用了B+树的数据结构。**

**总结：B-Tree和B+Tree的区别**

- B 树的所有节点既存放键(key) 也存放 数据(data)，而 B+树只有叶子节点存放 key 和 data，其他内节点只存放 key。
- B 树的叶子节点都是独立的;B+树的叶子节点有一条引用链指向与它相邻的叶子节点。
- B 树的检索的过程相当于对范围内的每个节点的关键字做二分查找，可能还没有到达叶子节点，检索就结束了。而 B+树的检索效率就很稳定了，任何查找都是从根节点到叶子节点的过程，叶子节点的顺序检索很明显。

## 5. MySQL的索引实现

介绍完了索引数据结构，那肯定是要带入到Mysql里面看看真实的使用场景的，所以这里分析Mysql的两种存储引擎的索引实现：**MyISAM索引**和**InnoDB索引**

### MyISAM

以一个简单的user表为例。user表存在两个索引，id列为主键索引，age列为普通索引

```SQL
CREATE TABLE `user_myisam`(
  `id`       int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age`      int(11)     DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_age` (`age`) USING BTREE
) ENGINE = MyISAM AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8;

INSERT INTO `user_myisam`(`id`, `username`, `age`) VALUES (8, 'zhangyi', 27), (12, 'zhanger', 23), (18, 'zhangsan', 58), (28, 'zhangsi', 33), (22, 'zhangwu', 47), (36, 'zhangliu', 31), (47, 'zhangqi', 25);
```

![image-20220405211302549](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\t_user.png)

MyISAM存储引擎中，数据文件和索引文件的分开的。MyISAM使用B+Tree构建索引树时，叶子节点中存储的键值为索引列的值，数据为索引所在行的磁盘地址。

`user`表中的数据文件存储在user.MYD文件中，索引文件则存储在user.MYI文件中。

#### 主键索引

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\MyISM-主键索引.png)

简单分析下查询时的磁盘IO情况：

**根据主键等值查询数据：**

```sql
select * from user where id = 28;
```

1. 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路。（1次磁盘IO）
2. 将左子树节点加载到内存中，比较16<28<47，向下检索。（1次磁盘IO）
3. 检索到叶节点，将节点加载到内存中遍历，比较16<28，18<28，28=28。查找到值等于30的索引项。（1次磁盘IO）
4. 从索引项中获取磁盘地址，然后到数据文件user.MYD中获取对应整行记录。（1次磁盘IO）
5. 将记录返给客户端。

**磁盘IO次数：3次索引检索+记录数据检索。**

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\MyISAM-主键索引-等值查询.png)

**根据主键范围查询数据：**

```sql
select * from user where id between 28 and 47;
```

1. 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路。（1次磁盘IO）

2. 将左子树节点加载到内存中，比较16<28<47，向下检索。（1次磁盘IO）

3. 检索到叶节点，将节点加载到内存中遍历比较16<28，18<28，28=28<47。查找到值等于28的索引项。

   根据磁盘地址从数据文件中获取行记录缓存到结果集中。（1次磁盘IO）

   我们的查询语句时范围查找，需要向后遍历底层叶子链表，直至到达最后一个不满足筛选条件。

4. 向后遍历底层叶子链表，将下一个节点加载到内存中，遍历比较，28<47=47，根据磁盘地址从数据文件中获取行记录缓存到结果集中。（1次磁盘IO）

5. 最后得到两条符合筛选条件，将查询结果集返给客户端。

**磁盘IO次数：4次索引检索+记录数据检索。**

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\MyISAM-主键索引-范围查找.png)

**备注**：以上分析仅供参考，MyISAM在查询时，会将索引节点缓存在MySQL缓存中，而数据缓存依赖于操作系统自身的缓存，所以并不是每次都是走的磁盘，这里只是为了分析索引的使用过程。

#### 辅助索引

在 MyISAM 中,辅助索引和主键索引的结构是一样的，没有任何区别，叶子节点的数据存储的都是行记录的磁盘地址。只是主键索引的键值是唯一的，而辅助索引的键值可以重复。

查询数据时，由于辅助索引的键值不唯一，可能存在多个拥有相同的记录，所以即使是等值查询，也需要按照范围查询的方式在辅助索引树中检索数据。

### InnoDB

#### 主键索引（聚簇索引）

每个InnoDB表都有一个聚簇索引 ，聚簇索引使用B+树构建，叶子节点存储的数据是整行记录。一般情况下，聚簇索引等同于主键索引，当一个表没有创建主键索引时，InnoDB会自动创建一个ROWID字段来构建聚簇索引。InnoDB创建索引的具体规则如下：

> 1. 在表上定义主键PRIMARY KEY，InnoDB将主键索引用作聚簇索引。
> 2. 如果表没有定义主键，InnoDB会选择第一个不为NULL的唯一索引列用作聚簇索引。
> 3. 如果以上两个都没有，InnoDB 会使用一个6 字节长整型的隐式字段 ROWID字段构建聚簇索引。该ROWID字段会在插入新行时自动递增。

除聚簇索引之外的所有索引都称为辅助索引。在中InnoDB，辅助索引中的叶子节点存储的数据是该行的主键的值。在检索时，InnoDB使用此主键值在聚簇索引中搜索行记录。

这里以user_innodb为例，user_innodb的id列为主键，age列为普通索引。

```sql
CREATE TABLE `user_innodb`(
  `id`       int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age`      int(11)     DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_age` (`age`) USING BTREE
) ENGINE = InnoDB;

INSERT INTO `user_innodb`(`id`, `username`, `age`) VALUES (8, '张一', 27), (12, '张二', 23), (18, '张三', 58), (28, '张四', 33), (22, '张五', 47), (36, '张六', 31), (47, '张七', 25);
```

![image-20220405211344385](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\t_user_innodb.png)

InnoDB的数据和索引存储在同一个文件中，t_user_innodb.ibd中。InnoDB的数据组织方式是聚簇索引。

主键索引的叶子会存储数据行，辅助索引只存储主键的值，所以在使用辅助索引的时候可能需要回表操作。

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\InnoDB-主键索引.png)

**等值查询数据：**

```sql
select * from user_innodb where id = 28;
```

先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路。（1次磁盘IO）

将左子树节点加载到内存中，比较16<28<47，向下检索。（1次磁盘IO）

检索到叶节点，将节点加载到内存中遍历，比较16<28，18<28，28=28。查找到值等于28的索引项，直接可以获取整行数据。将改记录返回给客户端。（1次磁盘IO）

**磁盘IO数量：3次。**

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\InnoDB-主键索引-等值查找.png)

#### 辅助索引

除聚簇索引之外的所有索引都称为辅助索引，InnoDB的辅助索引只会存储主键值而非磁盘地址。

以表user_innodb的age列为例，age索引的索引结果如下图。

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\InnoDB-辅助索引.png)

底层叶子节点的按照（age，id）的顺序排序，先按照age列从小到大排序，age列相同时按照id列从小到大排序。

使用辅助索引需要检索两遍索引：首先检索辅助索引获得主键，然后使用主键到主索引中检索获得记录。

**画图分析等值查询的情况：**

```sql
select * from t_user_innodb where age=19;
```

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\InnoDB-辅助索引-等值查找.png)

根据在辅助索引树中获取的主键id，到主键索引树检索数据的过程称为**回表**查询。

**磁盘IO数：辅助索引3次+获取记录回表3次**

#### 组合索引

还是以自己创建的一个表为例：表 abc_innodb，id为主键索引，创建了一个联合索引idx_abc(a,b,c)。

```sql
CREATE TABLE `abc_innodb`
(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a`  int(11)     DEFAULT NULL,
  `b`  int(11)     DEFAULT NULL,
  `c`  varchar(10) DEFAULT NULL,
  `d`  varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_abc` (`a`, `b`, `c`)
) ENGINE = InnoDB;

select * from abc_innodb order by a, b, c, id;
```

![图片](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\abc_innodb)

组合索引的数据结构：

![](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\组合索引数据结构.png)

**组合索引的查询过程：**

**最左匹配原则：**

最左前缀匹配原则和联合索引的**索引存储结构和检索方式**是有关系的。

在组合索引树中，最底层的叶子节点按照第一列a列从左到右递增排列，但是b列和c列是无序的，b列只有在a列值相等的情况下小范围内递增有序，而c列只能在a，b两列相等的情况下小范围内递增有序。

就像上面的查询，B+树会先比较a列来确定下一步应该搜索的方向，往左还是往右。如果a列相同再比较b列。但是如果查询条件没有a列，B+树就不知道第一步应该从哪个节点查起。

可以说创建的idx_abc(a,b,c)索引，相当于创建了(a)、（a,b）（a,b,c）三个索引。、

**组合索引的最左前缀匹配原则：使用组合索引查询时，mysql会一直向右匹配直至遇到范围查询(>、<、between、like)就停止匹配。**

#### 覆盖索引

覆盖索引并不是说是索引结构，**覆盖索引是一种很常用的优化手段。\**因为在使用辅助索引的时候，我们只可以拿到主键值，相当于获取数据还需要再根据主键查询主键索引再获取到数据。但是试想下这么一种情况，在上面abc_innodb表中的组合索引查询时，如果我只需要abc字段的，那是不是意味着我们查询到组合索引的叶子节点就可以直接返回了，而不需要回表。这种情况就是\**覆盖索引**。

可以看一下执行计划：

**覆盖索引的情况：**

![图片](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\覆盖索引.png)

使用到覆盖索引

**未使用到覆盖索引：**

![图片](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\覆盖索引-未使用)

## 6. 索引下推

**索引下推**（index condition pushdown ）简称ICP，在**Mysql5.6**的版本上推出，用于优化查询。

在不使用ICP的情况下，在使用非主键索引（又叫普通索引或者二级索引）进行查询时，存储引擎通过索引检索到数据，然后返回给MySQL服务器，服务器然后判断数据是否符合条件 。

在使用ICP的情况下，如果存在某些被索引的列的判断条件时，MySQL服务器将这一部分判断条件传递给存储引擎，然后由存储引擎通过判断索引是否符合MySQL服务器传递的条件，只有当索引符合条件时才会将数据检索出来返回给MySQL服务器 。

**索引条件下推优化可以减少存储引擎查询基础表的次数，也可以减少MySQL服务器从存储引擎接收数据的次数**

- 在开始之前先先准备一张用户表(user)，其中主要几个字段有：id、username、age。建立**联合索引（username，age）**。

```sql
CREATE TABLE `user`(
  `id`       int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age`      int(11)     DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_username_age` (`username`,`age`) USING BTREE
) ENGINE = InnoDB;
```

![image-20220405214540565](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\t_user_pulloff.png)

- 假设有一个需求，要求匹配姓名第一个为陈的所有用户，sql语句如下：

```sql
SELECT * from user where  username like '陈%'
```

- 根据 "最佳左前缀" 的原则，这里使用了联合索引（name，age）进行了查询，性能要比全表扫描肯定要高。
- 问题来了，如果有其他的条件呢？假设又有一个需求，要求匹配姓名第一个字为陈，年龄为20岁的用户，此时的sql语句如下：

```sql
SELECT * from user where  username like '陈%' and age=20
```

- 这条sql语句应该如何执行呢？下面对Mysql5.6之前版本和之后版本进行分析。

### Mysql5.6之前的版本

- 5.6之前的版本是没有索引下推这个优化的，因此执行的过程如下图：



![image-20220405213707583](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\索引下推-5.6之前.png)



- 会忽略age这个字段，直接通过name进行查询，在(name,age)这课树上查找到了两个结果，id分别为2,1，然后拿着取到的id值一次次的回表查询，因此这个过程需要**回表两次**。

### Mysql5.6及之后版本

- 5.6版本添加了索引下推这个优化，执行的过程如下图：

![image-20220405213726401](E:\workSpace\IdeaProjects\ITenderL.github.io\docs\database\mysql\images\index-introduce\索引下推-5.6之后.png)



- InnoDB并没有忽略age这个字段，而是在索引内部就判断了age是否等于20，对于不等于20的记录直接跳过，因此在(name,age)这棵索引树中只匹配到了一个记录，此时拿着这个id去主键索引树中回表查询全部数据，**这个过程只需要回表一次**。

### 总结

- 索引下推在**非主键索引**上的优化，可以有效减少回表的次数，大大提升了查询的效率。
- 关闭索引下推可以使用如下命令，配置文件的修改不再讲述了，毕竟这么优秀的功能干嘛关闭呢：

```sql
set optimizer_switch='index_condition_pushdown=off';
```

## 7. 总结

看到这里，你是不是对于自己的sql语句里面的索引的有了更多优化想法呢。

比如：

### 避免回表

在InnoDB的存储引擎中，使用辅助索引查询的时候，因为辅助索引叶子节点保存的数据不是当前记录的数据而是当前记录的主键索引，索引如果需要获取当前记录完整数据就必然需要根据主键值从主键索引继续查询。这个过程我们成位回表。想想回表必然是会消耗性能影响性能。那如何避免呢？

使用索引覆盖，举个例子：现有User表（id(PK),name(key),sex,address,hobby...）

如果在一个场景下，`select id,name,sex from user where name ='zhangsan';`这个语句在业务上频繁使用到，而user表的其他字段使用频率远低于它，在这种情况下，如果我们在建立 name 字段的索引的时候，不是使用单一索引，而是使用联合索引（name，sex）这样的话再执行这个查询语句是不是根据辅助索引查询到的结果就可以获取当前语句的完整数据。

这样就可以有效地避免了回表再获取sex的数据。

**这里就是一个典型的使用覆盖索引的优化策略减少回表的情况。**

### 联合索引的使用

**联合索引**，在建立索引的时候，尽量在多个单列索引上判断下是否可以使用联合索引。联合索引的使用不仅可以节省空间，还可以更容易的使用到索引覆盖。

试想一下，索引的字段越多，是不是更容易满足查询需要返回的数据呢。比如联合索引（a_b_c），是不是等于有了索引：a，a_b，a_b_c三个索引，这样是不是节省了空间，当然节省的空间并不是三倍于（a，a_b，a_b_c）三个索引，因为索引树的数据没变，但是索引data字段的数据确实真实的节省了。

**联合索引的创建原则**，在创建联合索引的时候因该把频繁使用的列、区分度高的列放在前面，频繁使用代表索引利用率高，区分度高代表筛选粒度大，这些都是在索引创建的需要考虑到的优化场景，也可以在常需要作为查询返回的字段上增加到联合索引中，如果在联合索引上增加一个字段而使用到了覆盖索引，那我建议这种情况下使用联合索引。

**联合索引的使用**

1. 考虑当前是否已经存在多个可以合并的单列索引，如果有，那么将当前多个单列索引创建为一个联合索引。
2. 当前索引存在频繁使用作为返回字段的列，这个时候就可以考虑当前列是否可以加入到当前已经存在索引上，使其查询语句可以使用到覆盖索引。



