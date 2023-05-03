# Redis



## 高性能



### 单线程+多路IO复用

使用一个线程执行，避免了多线程下并发问题导致开锁解锁，使用多路IO复用避免了由于等待IO设备而阻塞（原本是使用多线程，在一个线程阻塞的时候切换到另一个线程执行），导致无法处理其他的请求。

多路IO复用实现原理：使用epoll系统调用，在内核态中保存有当前线程所处理的文件描述符（使用红黑树结构），当某个文件描述符上的设备有动作时（传输数据，断开连接等等），操作系统将这个文件描述符统一放在一个就绪链表中，等待时机（？不确定，到时再看看）把这些就绪的描述符返回到用户态，在此期间，用户态不必等待在某个文件描述符上。



### 数据存储在内存中

主要操作的内容都在内存中，不需要大量的IO操作，效率高

---





## 基本数据类型



### String



#### 二进制安全

~~~
二进制安全是一种主要用于字符串操作函数相关的计算机编程术语。一个二进制安全功能（函数），其本质上将操作输入作为原始的、无任何特殊格式意义的数据流。对于每个字符都公平对待，不特殊处理某一个字符。 
~~~

~~~
先来解释一下什么叫二进制安全。
在C语言中的字符串必须以'\0'结尾而且字符串里面不能包含空字符，这样使得c字符串只能保存文本，不能保存图片、视频、压缩文件这样的二进制数据。
redis字符串是因为结构体中设置了len，长度确定，所以是二进制安全字符串；可以存储任意二进制数据；
~~~



#### 大小

一个Redis字符串value最多可以是512M



#### 常用命令

~~~
set   <key><value>添加键值对
 
*NX：当数据库中key不存在时，可以将key-value添加数据库
*XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥
*EX：key的超时秒数
*PX：key的超时毫秒数，与EX互斥

get   <key>查询对应键值
append  <key><value>将给定的<value> 追加到原值的末尾
strlen  <key>获得值的长度
setnx  <key><value>只有在 key 不存在时    设置 key 的值

incr  <key>
将 key 中储存的数字值增1，只能对数字值操作，如果为空，新增值为1
decr  <key>
将 key 中储存的数字值减1，只能对数字值操作，如果为空，新增值为-1

incrby / decrby  <key><步长>将 key 中储存的数字值增减。自定义步长。

mset  <key1><value1><key2><value2>  ..... 
同时设置一个或多个 key-value对  
mget  <key1><key2><key3> .....
同时获取一个或多个 value  
msetnx <key1><value1><key2><value2>  ..... 
同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
原子性，有一个失败则都失败
~~~







#### 数据结构

String的数据结构为简单动态字符串(Simple Dynamic String,缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配. 

![1657769410986](D:\文档\学习笔记\专业方向\中间件\Redis.assets\1657769410986.png)

如图中所示，内部为当前字符串实际分配的空间，总容量capacity一般要高于实际字符串长度len。



#### 扩容机制

当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。 

---





### List



#### 简介

Redis列表（List）存储方式是单键多值，相当于Java中的list集合。

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

![1657772371286](D:\文档\学习笔记\专业方向\中间件\Redis.assets\1657772371286.png)



#### 常用命令

~~~
lpush/rpush  <key><value1><value2><value3> .... 从左边/右边插入一个或多个值。
lpop/rpop  <key>  从左边/右边吐出一个值。值在键在，值光键亡。

rpoplpush  <key1><key2>从<key1>  从<key1>列表右边吐出一个值，插到<key2>列表左边。

lrange <key><start><stop>  按照索引下标获得元素(从左到右)
lrange mylist 0 -1   0左边第一个，-1右边第一个，（0-1表示获取所有）
lindex <key><index>  按照索引下标获得元素(从左到右)
llen <key>  获得列表长度 

linsert <key>  before <value><newvalue>  在值为<value>的元素前面插入<newvalue>插入值
lrem <key><n><value>  从左边删除n个值为value的元素(从左到右)
lset<key><index><value>  将列表key下标为index的元素值替换成value
~~~



#### 数据结构

List的数据结构为quickList。

在元素数量较少的情况下，使用一块连续的内存存储元素，相当于数组，在Redis中称为ziplist（压缩列表），当数据量大的时候，将多个连续的ziplist连接起来，形成链表结构，这样就不会出现链表的指针占用内存较多的情况。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next，只有三分之一的空间是存储数据的空间。

![1657773196118](D:\文档\学习笔记\专业方向\中间件\Redis.assets\1657773196118.png)

Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。 



---



### Set



#### 简介

Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的**复杂度都是O(1)**。



#### 常用命令

~~~
sadd <key><value1><value2> ..... 
将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
smembers <key>  取出该集合的所有值。
sismember <key><value>  判断集合<key>是否为含有该<value>值，有1，没有0
scard<key>  返回该集合的元素个数。
srem <key><value1><value2> ....   删除集合中的某个元素。
spop <key>  随机从该集合中吐出一个值。
srandmember <key><n>  随机从该集合中取出n个值。不会从集合中删除 。
smove <source><destination>value  把集合中一个值从一个集合移动到另一个集合
sinter <key1><key2>  返回两个集合的交集元素。
sunion <key1><key2> 返回两个集合的并集元素。
sdiff <key1><key2>  返回两个集合的差集元素(key1中的，不包含key2中的)
~~~



#### 数据结构

Set数据结构是dict字典，字典是用哈希表实现的。

Java中HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。



---





### Hash



#### 简介

Redis中的hash相当于一个键对应一个hash表，然后hash表还可以有自己的键值对。

通过 key(用户ID) + field(属性标签) 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题。

![1657779072027](D:\文档\学习笔记\专业方向\中间件\Redis.assets\1657779072027.png)





#### 常用命令

~~~
hset <key><field><value>  给<key>集合中的  <field>键赋值<value>
hget <key1><field>  从<key1>集合<field>取出 value 
hmset <key1><field1><value1><field2><value2>...   批量设置hash的值
hexists<key1><field>  查看哈希表 key 中，给定域 field 是否存在。 
hkeys <key>  列出该hash集合的所有field
hvals <key>  列出该hash集合的所有value
hincrby <key><field><increment>  为哈希表 key 中的域 field 的值加上增量 1   -1
hsetnx <key><field><value>  将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .
~~~



#### 数据结构

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。 

---





### Zset



#### 简介

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分**（score）,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复的 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。



#### 常用命令

~~~
zadd  <key><score1><value1><score2><value2>…
将一个或多个 member 元素及其 score 值加入到有序集 key 当中。

zrange <key><start><stop>  [WITHSCORES]   
返回有序集 key 中，下标在<start><stop>之间的元素
带WITHSCORES，可以让分数一起和值返回到结果集。

zrangebyscore key min max [withscores] [limit offset count]
返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 

zrevrangebyscore key max min [withscores] [limit offset count]               
同上，改为从大到小排列。 

zincrby <key><increment><value>      为元素的score加上增量

zrem  <key><value>  删除该集合下，值为value的元素 

zcount <key><min><max>  统计该集合，分数区间内的元素个数 

zrank <key><value>  返回该值在集合中的排名，从0开始。
~~~



#### 数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

---





### Bitmaps



#### 简介

合理地使用操作位能够有效地提高内存使用率和开发效率。

Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

（1）Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。

（2）Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做偏移量。

![1657789735942](D:\文档\学习笔记\专业方向\中间件\Redis.assets\1657789735942.png)



#### 常用命令

~~~
setbit <key><offset><value>  设置Bitmaps中某个偏移量的值（0或1） 

getbit<key><offset>  获取Bitmaps中某个偏移量的值，获取键的第offset位的值（从0开始算）

bitcount<key>[start end]   统计字符串从start字节到end字节比特值为1的数量
注意：redis的setbit设置或清除的是bit位置，而bitcount计算的是byte位置。

bitop  and(or/not/xor) <destkey> [key…]  
bitop是一个复合操作， 它可以做多个Bitmaps的and（交集） 、 or（并集） 、 not（非） 、 xor（异或） 操作并将结果保存在destkey中。
~~~

---





### HyperLogLog



#### 简介

在工作当中，我们经常会遇到与统计相关的功能需求，比如统计网站PV（PageView页面访问量）,可以使用Redis的incr、incrby轻松实现。

但像UV（UniqueVisitor，独立访客）、独立IP数、搜索记录数等需要去重和计数的问题如何解决？这种求集合中不重复元素个数的问题称为基数问题。

解决基数问题有很多种方案：

（1）数据存储在MySQL表中，使用distinct count计算不重复个数

（2）使用Redis提供的hash、set、bitmaps等数据结构来处理

以上的方案结果精确，但随着数据不断增加，导致占用空间越来越大，对于非常大的数据集是不切实际的。

能否能够降低一定的精度来平衡存储空间？Redis推出了HyperLogLog



**优点**

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。



**缺点**

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

 

什么是基数?

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。



#### 常用命令

~~~
pfadd <key>< element> [element ...]   添加指定元素到 HyperLogLog 中
将所有元素添加到指定HyperLogLog数据结构中。如果执行命令后HLL估计的近似基数发生变化，则返回1，否则返回0。

pfcount<key> [key ...]   计算HLL的近似基数，可以计算多个HLL，比如用HLL存储每天的UV，计算一周的UV可以使用7天的UV合并计算即可

pfmerge<destkey><sourcekey> [sourcekey ...]  将一个或多个HLL合并后的结果存储在另一个HLL中，比如每月活跃用户可以使用每天的活跃用户来合并计算可得
~~~

---





### Geospatial

#### 简介

Redis 3.2 中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。 



#### 常用命令

~~~
geoadd<key>< longitude><latitude><member> [longitude latitude member...]   添加地理位置（经度，纬度，名称）
两极无法直接添加，一般会下载城市数据，直接通过 Java 程序一次性导入。
有效的经度从 -180 度到 180 度。有效的纬度从 -85.05112878 度到 85.05112878 度。
当坐标位置超出指定范围时，该命令将会返回一个错误。
已经添加的数据，是无法再次往里面添加的。

geopos  <key><member> [member...]  获得指定地区的坐标值

geodist<key><member1><member2>  [m|km|ft|mi ]  获取两个位置之间的直线距离
单位：
m 表示单位为米[默认值]。
km 表示单位为千米。
mi 表示单位为英里。
ft 表示单位为英尺。
如果用户没有显式地指定单位参数， 那么 GEODIST 默认使用米作为单位

georadius<key>< longitude><latitude>radius  m|km|ft|mi   以给定的经纬度为中心，找出某一半径内的元素
~~~

