# 数据库索引

基本回答：

索引是为了加快数据查询的一种数据结构。

从数据结构角度出发，索引分为B树索引，B+树索引，哈希索引和位图索引。
在MySQL上，主要是采用B+树索引，B树索引在NoSQL上使用较多，哈希索引在KV数据库上较为常见。

从形态上来说，可以分成覆盖索引，前缀索引，全文索引，联合索引，唯一索引和主键索引。（下面的定义，可以直接一股脑说出来，也可以等面试官问）
1. 覆盖索引其实是指我们查询的列全部命中了索引；
2. 前缀索引是指只利用了数据前几个字符的索引，如果前面几个字符区分度不好的话，不建议使用前缀索引；
3. 全文索引现在比较少用，一般推荐使用别的中间件来完成，例如ES（小心这一步，可能咔嚓把话题引过去了ES上）；
4. 联合索引是指多个列组成一个索引。创建的时候我们会考虑把区分度好的索引放在前面，因为MySQL遵循最左前缀匹配原则；（这里可能会问你啥是最左前缀匹配原则）
5. 唯一索引是指数据库里面要求该索引值必须要唯一，我们一般用于业务唯一性保证；
6. 主键索引是比较特殊的索引，一般它的叶子节点要么存储了数据，要么存储了指向数据的指针。MySQL的innodb引擎存储的是数据，MyISAM放的是数据的地址；（这里也会引过去聚簇索引与非聚簇索引）

从是否存储数据的角度，又可以分为聚簇索引和非聚簇索引，MySQL的主键就是聚簇索引，每张表唯一一个，非聚簇索引的数据本质上存储的是主键。（面试官可能从这里被引导过去聚簇索引与非聚簇索引）

而对于MySQL的innodb来说，它的行锁是利用索引来实现的，所以如果查询的时候没有索引，那么会导致表锁。（这一句可能引导面试官问你锁和事务的问题，如果不熟悉锁和事务，请不要回答）。

![知识点](./img/index.png)

## 扩展点

### 什么是最左匹配原则

分析：单单回答最左前缀匹配原则是很简单，但是没有亮点。亮点在，最左前缀匹配大概是如何运作的。之所以只需要回答”大概“如何运作，是因为详细回答太难，面试官没读过源码也搞不清楚，犯不着。

答案：最左前缀匹配原则是指，MySQL会按照联合索引创建的顺序，从左至右开始匹配。例如创建了一个联合索引（A，B，C)，那么本质上来说，是创建了A，（A，B），（A，B，C）三个索引。之所以如此，因为MySQL在使用索引的时候，类似于多重循环，一个列就是一个循环。在这种原则下，我们会优先考虑把区分度最好的放在最左边，而区分度可以简单使用不同值的数量除以总行数来计算（distinct(a, b, c)/count(*)）。

### 数据库支持哈希索引吗

分析：很少有面试官会在数据库面试里边聊哈希索引，因为这个东西很罕见，用法也比较奇诡。不过这也是一个优雅装逼的点。

答案：哈希索引是利用哈希表来实现的，适用于等值查询，如等于，不等于，IN等，对范围查询是不支持的。我们惯常用的innodb引擎是不支持用户自定义哈希索引的，但是innodb有一个优化会建立自适应哈希索引。
所谓的自适应哈希索引，是指innodb引擎，如果发现二级索引（除了主键以外的别的索引）被经常使用，那么innodb会给这个索引建立一个哈希索引，加快查询。所以从本质上来说，innodb的自适应哈希索引是一个对索引的哈希索引。

关键：等值查询，对索引的哈希索引

#### 如何引导
1. 在前面回答了哈希索引之后，就直接跳过来这里，例如“哈希索引在KV数据库上比较常见，不过innodb引擎支持自适应哈希索引，它是..."

### 聚簇索引和非聚簇索引的区别

分析：这其实是一个很简单的问题，但也是一个很能装逼的问题。聚簇索引和非聚簇索引的区别，只需要回答，他们叶子节点是否存储了数据。但是要答出亮点，就要多回答两个点：第一，MySQL的非聚簇索引存储了主键；第二，覆盖索引不需要回表。

答案：聚簇索引是指叶子节点存储了数据的索引。MySQL整张表可以看做是一个聚簇索引。因为非聚簇索引没有存储数据，所以一般是存储了主键。于是会导致一个回表的问题。即如果我们查询的列包含不在索引上的列，这会引起数据库先根据非聚簇索引找出主键，而后拿着主键去聚簇索引里边捞出来数据。而根据主键找数据会引起磁盘IO，性能大幅度下降。这就是我们推荐使用覆盖索引的原因。

关键点：聚簇索引存了数据，非聚簇索引要回表

#### 如何引导过来这里？
1. 聊到了覆盖索引与回表的问题，话术可以是”一般用覆盖索引，在不使用覆盖索引的时候，会引起回表查询，这是因为MySQL的非聚簇索引...“；
2. 聊到如何计算一次查询的开销。这个比较少见，因为一般的面试官也讲不清楚一次MySQL查询时间开销会在哪里；
3. 前面基本回答，回答了聚簇索引之后直接回答这部分
4. 聊到了B+树的叶子节点可以存放什么，或者聊到了索引的叶子节点可以存放什么
5. 是不是查询一定会引起回表？这其实是考察覆盖索引，所以在谈及了覆盖索引之后可以聊这个聚簇索引和非聚簇索引的点

### MySQL为什么使用B+树索引

分析：实际上就是为了考察数据结构，B+树的特征，而且能够根据B+树的特征，理解MySQL选择B+树的原因。面试官可能同时希望你能够横向比较B+树、B树、平衡二叉树，红黑树和跳表
直接背这几种树的基本特征是比较难的，所以我们可以只回答关键点。关键点就是三个，和二叉树比起来，B+树是多叉的，高度低；和B树比起来，它的叶子节点组成了一个链表；第三点，是一个角度很清奇的点，就是查询时间稳定，可预测。和跳表的比较比较奇诡，要从MySQL组织B+树的角度出发。

答案：MySQL使用B+树主要就是考虑三个角度：
1. 和二叉树，如平衡二叉树，红黑树比起来，B+树是多叉树，比如MySQL默认是1200叉树，同样数据量，高度要比二叉树低；
2. 和B树比起来，B+树的叶子节点被连接起来，形成了一个链表，这意味着，当我们执行范围查询的时候，MySQL可以利用这个特性，沿着叶子节点前进。而之所以NoSQL数据库会使用B树作为索引，也是因为它们不像关系型数据库那般大量查询都是范围查询；
3. B+树只在叶子节点存放数据，因此和B树比起来，查询时间稳定可预测。（注：这是一个高级观点，就是在工程实践中，我们可能倾向于追求一种稳定可预测，而不是某些数据贼快，某些数据唰一下贼慢）
4. B+树和跳表比起来，MySQL将B+树节点大小设置为磁盘页大小，这样可以充分利用MySQL的预加载机制，减少磁盘IO

关键点：高度低，叶子节点是链表，查询时间可预测性，节点大小等于页大小


#### 如何引导过来这里？
1. 面试官直接问起来；
2. 你们聊起了树结构，聊到了B树和B+树，话术一般是“因为B+树和B树比起来，有...的优点，索引MySQL索引主要是使用B+树的；
3. 聊到了范围查询或者全表扫描，你可以从B+树的角度来说，这种扫描利用到了B+树叶子节点是链表的特征；

### 为什么使用自增主键

分析：这是一个常考点，从根源上来说，是为了考察你对数据库如何组织数据的理解。问题在于，数据库如何组织数据其实是一个很难的问题，所以一般情况下，不需要回答到非常底层的地步。

答案：MySQL的主键是一个**聚簇索引**，即它的叶子节点存放了数据。
在使用自增主键的情况下，会保证树的分裂照着单方向分裂的，这会大概率导致物理页的分裂也是朝着单方向进行的，即连续的。
在不使用自增主键的情况下，如果在已经满的页里面插入，会导致MySQL页分裂，虽然逻辑上页依旧是连续的，但是物理页已经不连续了。
如果在使用机械硬盘的情况下，会导致范围查询经常导致机械硬盘重新定位，性能差。

关键点：单方向增长，物理页连续

#### 如何引导过来这里？
1. 面试官可能直接问你
2. 你在基本回答那里，回答到"聚簇索引"的时候，主动说起，为什么我们要使用自增索引。话术可能是"MySQL的主键索引是聚簇索引，每张表一个，所以我们一般推荐使用自增主键，因为自增主键会保证树单方向分裂..."
3. 聊起数据库表结构设计，你们公司推荐使用自增主键，你可以主动说，我们公司是强制要求使用自增主键的，因为...
4. 聊起数据库表结构设计，你有一些特殊的表，没有使用自增主键，你可以说"我们大多数表都是使用自增主键的，因为...但是这几张表我们没有使用，因为xxx(结合你们的业务特征回答)"，慎用
5. 聊到树结构的特征。比如说面试官其实面你的是数据结构，而不是数据库，但是你们聊到了树，就可以主动提起。因为大部分树，比如说红黑树，二叉平衡树，B树，B+树都有一个调整树结构的过程，所以可以强行引过来；
6. 聊起分库分表设计，主键生成的时候，可以提起生成的主键为什么最好是单调递增的。这个问题其实和为什么使用自增主键，是同一个问题；
7. 评价为什么使用UUID来作为主键生成策略会很糟糕

### 索引有什么缺点

分析：面试官就是为了吓你，出其不意攻其不备。又或者面试官问为啥不在所有列上建立索引，或者问为什么不建立多一点索引

答案：索引的维护是有开销的。在增改数据的时候，数据库都要对应修改索引；而如果索引过多，以至于内存没法装下全部索引，那么会导致访问索引本身都会触发IO。所以索引不是越多越好。比如为了避免数据量过大，某些时候我们会使用前缀索引。

#### 如何引导过来这里
1. 面试官直接问了
2. 你在基本回答那里回答了前缀索引，之后可以说”使用前缀索引是为了节省空间，因为索引本身的维护是有开销的，除了空间开销，在数据更新的时候..."
3. 在回答完什么时候索引之后可以直接说