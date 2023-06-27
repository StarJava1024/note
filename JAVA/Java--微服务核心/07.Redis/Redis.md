# Redis 学前准备



## 1、在Linux上安装gcc

> 查看gcc版本：gcc -v
>
> 安装gcc：yum -y install gcc- c++

## 2、安装Redis

> ①  在 /opt 目录下解压Redis安装包：tar -zxcf redis-7.0.11.tar.gz
>
> ②  进入目录：cd redis-7.0.11
>
> ③  执行：make & make install
>
> ④  进入默认安装目录：/usr/local/bin
>
> ​		redis-benchmark: 性能测试工具，服务启动后运行该命令，看看自己本子性能如何
>
> ​		redis-check-aof: 修复有问题的AOF文件，rdb和aof后面讲
>
> ​		redis-check-dump: 修复有问题的dump.rdb文件
>
> ​		redis-cli: 客户端，操作入口
>
> ​		redis-sentinel: redis集群使用
>
> ​		redis-server: Redis服务器启动命令
>
> ⑤  将默认的`redis.conf`拷贝到自己定义的一个路径下，比如`/myredis`
>
> ​		mkdir /myredis
>
> ​		cp redis.conf /myredis/
>
> ⑥  修改`/myredis`目录下`redis.conf`配置文件做初始化设置
>
> ​		redis.conf配置文件，改完后确保生效，记得重启，记得重启
>
> ​			1 默认daemonize no              改为  daemonize yes
>
> ​			2 默认protected-mode  yes    改为  protected-mode no
>
> ​			3 默认bind 127.0.0.1              改为  直接注释掉(默认bind 127.0.0.1只能本机访问)或改成本机IP地址，否则影响远程IP连接
>
> ​			4 添加redis密码                      改为 requirepass 你自己设置的密码
>
> ⑦  启动服务：`/usr/local/bin`目录下运行`redis-server`，启用`/myredis`目录下的`redis7.conf`文件
>
> ⑧  连接服务：redis-cli -a 111111 -p 6379 		Redis端口为6379
>
> ​							redis-cli -a 111111 --raw			解决中文乱码
>
> ⑨  关闭服务：单实例关闭：redis-cli -a 111111 shutdown
> ​						  多实例关闭，指定端口关闭：redis-cli -p 6379 shutdown
>
> ⑩  卸载Redis：
>
> ​		停止服务：redis-server /myredis/redis7.conf
>
> ​		删除`/usr/local/lib`目录下与redis相关的文件

# Redis 数据类型

## 1、redis键（key）

> keys *：查看当前库所有的key
>
> exists key：判断某个key是否类型
>
> type key：查看你的key是什么类型
>
> del key：删除指定的key数据
>
> unlink key：非阻塞删除，仅仅将keys从keyspace元数据中删除，真正的删除会在后续异步中操作
>
> ttl key：查看还有多少秒过期，-1表示永不过期，-2表示已过期
>
> expire key 秒钟：为给定的key设置过期时间
>
> move key dbindex【0-15】：将当前数据库的key移动到给定的数据库db当中
>
> select dbindex：切换数据库key的数量
>
> <span style="color:#4662d9; font-weight:bold">flushdb：清空当前库</span> 
>
> <span style="color:red; font-weight:bold">flushall：通杀全部库</span> 

## 2、数据类型命令及落地运用

### 命令大全网址

> 英文：https://redis.io/commands/ 
>
> 中文：http://www.redis.cn/commands/ 

### （1）字符串（String）

![1.1](images/1.1.png)
![1.2](images/1.2.png)

> SET key value [NX | XX] [GET] [EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL]
>
> ![1](images/1.png)
>
> 



```
> set k1 v1 nx
OK
> set k1 v1 nx
(nil)
> set k1 v1 xx
OK
> set k1 v1 xx
OK
> get k1    {返回k1的值}
"v1"
> set k1 v2 get    {键k1的返回v1,并将v2赋值给k1}
"v1"
> get k1
"v2"

> set k1 v1 ex 10    {10秒后失效}
OK
> ttl k1    {5秒后}
(integer) 5
> ttl k1    {10秒后}
(integer) -2    {-2表示已过期}
> get k1
(nil)

> set k1 v1 px 10000    {10秒后失效}
OK
> ttl k1    {5秒后}
(integer) 5
> ttl k1    {10秒后}
(integer) -2    {-2表示已过期}
> get k1
(nil)

> set k1 v3 ex 50    {50秒后失效}
OK
> ttl k1
(integer) 45
> set k1 v4 keepttl    {保留上面的设置的失效时间}
OK
> ttl k1
(integer) 25
> get k1
"v4"
> ttl k1
(integer) -2    {-2表示已过期}
> get k1
(nil)

> mset k1 v1 k2 v2 k3 v3    {一次性设置多个参数}
OK
> mget k1 k2 k3    {一次性返回多个值}
1) "v1"
2) "v2"
3) "v3"
> msetnx k1 v2 k4 v4
(integer) 0    {设置失败}
> get k1    {k1有值}
"v1"
> get k4    {k4未设置}
(nil)
{说明 msetnx 要么全部成功，要么全部失败}

> set k1 abcd1234
OK
> GETRANGE k1 0 -1    {截取String}
"abcd1234"
> GETRANGE k1 0 3
"abcd"
> SETRANGE k1 2 xing    {替换String中部分字符}
(integer) 8
> get k1
"abxing34"

> set k1 100
OK
> get k1
"100"
> INCR k1    {加1}
"101"
> INCR k1 3    {加3}
"104"
> DECR k1
"103"
> DECR k1 3
"100"
> set k1 abcd
OK
> get k1
"abcd"
> STRLEN k1
(integer) 4
> APPEND k1 xing
(integer) 8
> get k1
"abcdxing"

> getset k1 wangxing
"abcdxing"
> get k1
"wangxing"
```

### （2）列表（list：单key多value）

![2.1](images/2.1.png)
![2.2](images/2.2.png)

![2](images/2.png)

```
> LPUSH list1 1 2 3 4 5		{从左边插入数据 -> 5,4,3,2,1}
(integer) 5
> LRANGE list1 0 -1		{从左边开始遍历}
1) "5"
2) "4"
3) "3"
4) "2"
5) "1"
> RPUSH list2 11 22 33 44 55	{从右边插入数据 -> 11,22,33,44,55}
(integer) 5
> LRANGE list2 0 -1		{从左边开始遍历}
1) "11"
2) "22"
3) "33"
4) "44"
5) "55"
> type list1	{返回key的类型}
list

> LPOP list1	{从左边弹出}
"5"		{将5弹出}
> LRANGE list1 0 -1
1) "4"
2) "3"
3) "2"
4) "1"
> RPOP list1	{从右边弹出}
"1"		{将1弹出}
> LRANGE list1 0 -1
1) "4"
2) "3"
3) "2"
> LINDEX list1 1	{从左边按下标取值}
"3"
> LLEN list1	{key的元素个数}
(integer) 3

> LPUSH list1 1 2 1 2 1 3 1 4
(integer) 8
> LRANGE list1 0 -1
1) "4"
2) "1"
3) "3"
4) "1"
5) "2"
6) "1"
7) "2"
8) "1"
> LREM list1 2 1	{从左边删除2个元素为1}
(integer) 
> LRANGE list1 0 -1
1) "4"
2) "3"
3) "2"
4) "1"
5) "2"
6) "1"
> LTRIM list1 2 4	{从左边起截取下标为2-4的元素给key为list1}
OK
> LRANGE list1 0 -1
1) "2"
2) "1"
3) "2"
> LRANGE list2 0 -1
1) "11"
2) "22"
3) "33"
4) "44"
5) "55"
> RPOPLPUSH list1 list2		{将list1的最后一个元素push到list2的第一个元素}
"2"
> LRANGE list1 0 -1
1) "2"
2) "1"
> LRANGE list2 0 -1
1) "2"
2) "11"
3) "22"
4) "33"
5) "44"
6) "55"
> LSET list2 0 redis	{将值redis设置给list2中下标为0的元素}
OK
> LRANGE list2 0 -1
1) "redis"
2) "11"
3) "22"
4) "33"
5) "44"
6) "55"
> LINSERT list2 before redis Linux	{在元素redis前插入元素Linux}
(integer) 7
> LRANGE list2 0 -1
1) "Linux"
2) "redis"
3) "11"
4) "22"
5) "33"
6) "44"
7) "55"
```

### （3）哈希（Hash：KV模式不变，但V是一个键值对 [Map<String,Map<Object,Object>>]）

![3.1](images/3.1.png)

![3](images/3.png)

```
> HSET user:001 id 11 name 张三 age 23
(integer) 3
> hget user:001 id
"11"
> hget user:001 name
"\xe5\xbc\xa0\xe4\xb8\x89"
> HMSET user:001 id 12 name lisi age 23		{}
OK
> HMGET user:001 id name age
1) "12"
2) "lisi"
3) "23"
> HGETALL user:001	{遍历user:001}
1) "id"
2) "12"
3) "name"
4) "lisi"
5) "age"
6) "23"
> HDEL user:001 age		{删除user:001的age属性}
(integer) 1
> HGETALL user:001
1) "id"
2) "12"
3) "name"
4) "lisi"

> HEXISTS user:001 name		{判断user:001里是否存在name属性}
(integer) 1
> HEXISTS user:001 sorce
(integer) 0

> HKEYS user:001	{列出user:001中的所有key}
1) "id"
2) "name"
> HVALS user:001	{列出user:001中的所有value}
1) "12"
2) "lisi"

> HSET user:001 id 1001 name lisi age 20 score 97.5
(integer) 2
> HGETALL user:001
1) "id"
2) "1001"
3) "name"
4) "lisi"
5) "age"
6) "20"
7) "score"
8) "97.5"
> HINCRBY user:001 age 2	{将user:001的age属性 加2}
(integer) 22
> HGETALL user:001
1) "id"
2) "1001"
3) "name"
4) "lisi"
5) "age"
6) "22"
7) "score"
8) "97.5"
> HINCRBYFLOAT user:001 score 1.5
"99"
127.0.0.1:6379> HGETALL user:001
 1) "id"
 2) "1001"
 3) "name"
 4) "lisi"
 5) "age"
 6) "22"
 7) "score"
 8) "99"
 
> HSETNX user:001 email lisi@163.com	{不存在赋值}
(integer) 1
> HSETNX user:001 email lisi@163.com	{存在了无效}
(integer) 0
```

### （4）集合（Set：单值多value，且无重复）

![4.1](images/4.1.png)

![4](images/4.png)

```
> SADD set1 1 1 1 2 2 3 4	{向set1集合中添加元素}
(integer) 4
> SMEMBERS set1		{遍历set1集合，没有重复元素}
1) "1"
2) "2"
3) "3"
4) "4"
> SISMEMBER set1 5	{判断该元素是否存在}
(integer) 0		{不存在返回0}
> SISMEMBER set1 3
(integer) 1		{存在返回1}

> SREM set1 5		{删除某个元素}
(integer) 0		{不存在返回0}
> SREM set1 4
(integer) 1		{存在返回1}
> SMEMBERS set1
1) "1"
2) "2"
3) "3"
> SCARD set1	{统计set1有多少元素}
(integer) 3

> SRANDMEMBER set1 2	{随机选出一个数字，但不删除源set集合}
1) "5"
2) "3"

> SMEMBERS set1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "8"
> SPOP set1 3
1) "2"
2) "4"
3) "3"
> SMEMBERS set1
1) "1"
2) "5"
3) "6"
4) "7"
5) "8"

> SMEMBERS set1
1) "1"
2) "5"
3) "6"
4) "7"
5) "8"
127.0.0.1:6379> SADD set2 a b c d
(integer) 4
127.0.0.1:6379> SMEMBERS set2
1) "d"
2) "b"
3) "c"
4) "a"
127.0.0.1:6379> SMOVE set1 set2 6	{将set1中的元素6 push到set2}
(integer) 1
127.0.0.1:6379> SMEMBERS set1
1) "1"
2) "5"
3) "7"
4) "8"
127.0.0.1:6379> SMEMBERS set2
1) "d"
2) "b"
3) "c"
4) "a"
5) "6"		{将随机存储在set2中}

> SADD A 1 2 a b c
(integer) 5
> SADD B a x 1 2 3
(integer) 5

=== 集合差集运算 A-B ===
> SDIFF A B		{属于A但不属于B的元素够成的集合}
1) "b"
2) "c"
> SDIFF B A		{属于B但不属于A的元素够成的集合}
1) "x"
2) "3"

=== 集合并集运算 A ∪ B ===
> SUNION A B
1) "2"
2) "b"
3) "3"
4) "1"
5) "x"
6) "a"
7) "c"

=== 集合并集运算 A ∩ B ===
> SINTER A B
1) "2"
2) "1"
3) "a"
```

### （5）有序集合（Zset）

![5.1](images/5.1.png)
![5.2](images/5.2.png)

![5](images/5.png)

```
> ZADD zset1 60 v1 70 v2 80 v3 90 v4 100 v5
(integer) 5
> ZRANGE zset1 0 -1		{输出键}
1) "v1"
2) "v2"
3) "v3"
4) "v4"
5) "v5"
> ZRANGE zset1 0 -1 withscores		{输出键、值}
 1) "v1"
 2) "60"
 3) "v2"
 4) "70"
 5) "v3"
 6) "80"
 7) "v4"
 8) "90"
 9) "v5"
10) "100"

> ZRANGEBYSCORE zset1 60 90 withscores		{输出 60≤value≤90 }
1) "v1"
2) "60"
3) "v2"
4) "70"
5) "v3"
6) "80"
7) "v4"
8) "90"
> ZRANGEBYSCORE zset1 (60 90 withscores		{输出 60<value≤90 }
1) "v2"
2) "70"
3) "v3"
4) "80"
5) "v4"
6) "90"
> ZRANGEBYSCORE zset1 60 (90 withscores		{输出 60≤value<90 }
1) "v1"
2) "60"
3) "v2"
4) "70"
5) "v3"
6) "80"
> ZRANGEBYSCORE zset1 60 90 withscores limit 1 2	{在 60≤value≤90 下标为1输出2个}
1) "v2"
2) "70"
3) "v3"
4) "80"

> ZSCORE zset1 v5	{获取元素的分数}
"100"
> ZCARD zset1		{获取集合中元素的数量}
(integer) 5
> ZREM zset1 v5		{删除v5元素}
(integer) 1
> ZRANGE zset1 0 -1 withscores
1) "v1"
2) "60"
3) "v2"
4) "70"
5) "v3"
6) "80"
7) "v4"
8) "90"

> ZINCRBY zset1 7 v1	{给v1加7}
"67"
> ZRANGE zset1 0 -1 withscores
1) "v1"
2) "67"
3) "v2"
4) "70"
5) "v3"
6) "80"
7) "v4"
8) "90"

> ZCOUNT zset1 65 70	{统计 65≤value≤70 有几个值}
(integer) 2
> ZMPOP 1 zset1 min count 2
1) "zset1"
2) 1) 1) "v1"
      2) "67"
   2) 1) "v2"
      2) "70"
> ZRANGE zset1 0 -1 withscores
1) "v3"
2) "80"
3) "v4"
4) "90"

> zrank zset1 v4	{返回v4在zset1中的下标值}
(integer) 1
> zrank zset1 v3	{返回v3在zset1中的下标值}
(integer) 0
```

### （6）位图（bitmap）

> 由0和1状态表现的二进制位的bit数组

#### 是什么

![6.1](images/6.1.png)

> 说明：用String类型作为底层数据结构实现的一种统计二值状态的数据类型
>
> 位图本质是数组，它是基于String数据类型的按位的操作。该数组由多个二进制位组成，每个二进制位都对应一个偏移量(我们称之为一个索引)。
>
> Bitmap支持的最大位数是2^32^位，它可以极大的节约存储空间，使用512M内存就可以存储多达42.9亿的字节信息(2^32^ = 4294967296)

#### 基本命令

| 命令                        |                        **作用**                         | 时间复杂度 |
| :-------------------------- | :-----------------------------------------------------: | :--------: |
| setbit key offset val       |          给指定`key`的值的第`offset`赋值`val`           |    O(1)    |
| getbit key offset           |               获取指定`key`的第`offset`位               |    O(1)    |
| bitcount key start end      |        返回指定`key`中`[start, end]`中为1的数量         |    O(n)    |
| bitop operation destkey key | 对不同的二进制存储数据进行位运算`（AND、OR、NOT、XOR）` |    O(n)    |

![6.2](images/6.2.png)

> 保存到`redis`后的数据就是，下面这样
>
> ![6.3](images/6.3.png)

#### 案例

```
> setbit bit1 1 1	{设置数据}
(integer) 0
> setbit bit1 2 1
(integer) 0
> setbit bit1 3 1
(integer) 0
> setbit bit1 4 0
(integer) 0
> type bit1
string
> get bit1
"p"
> getbit bit1 1		{获取数据}
(integer) 1
> getbit bit1 4
(integer) 0
> setbit bit2 day1 1
(error) ERR bit offset is not an integer or out of range	{位移量不是整数或超出范围}
> setbit bit1 9 1
(integer) 0
> get bit1
"p@"
> strlen bit1	{不是字符串长度而是占据几个字节，超过8位后自己按照8位一组一byte再扩容}
(integer) 2
> bitcount bit1	{全部键里面含有1的有多少个？}
(integer) 4


连续2天都签到的用户
> HSET uid:map 0 uid-092iok-lkj
(integer) 1
> HSET uid:map 1 uid-7388c-xxx
(integer) 1
> hgetall uid:map
1) "0"
2) "uid-092iok-lkj"
3) "1"
4) "uid-7388c-xxx"
> setbit 20230101 0 1	{2023年1月1日 用户0:uid-092iok-lkj 签到成功}
(integer) 0
> setbit 20230101 1 1	{2023年1月1日 用户0:uid-7388c-xxx 签到成功}
(integer) 0
> getbit 20230101 0		{获取2023年1月1日 用户0:uid-092iok-lkj 是否签到}
(integer) 1		{1：已签到；0：未签到}
> setbit 20230101 2 1
(integer) 0
> setbit 20230101 3 1
(integer) 0
> setbit 20230102 0 1
(integer) 0
> setbit 20230102 2 1
(integer) 0
> bitcount 20230101		{统计 2023年1月1日 签到人数}
(integer) 4
> bitcount 20230102		{统计 2023年1月2日 签到人数}
(integer) 2
> bitop and k3 20230101 20230102	{统计 2023年1月1日、2023年1月2日 都签到的人数}
(integer) 1		{true}
> bitcount k3	
(integer) 2		{说明有2个人在 2023年1月1日、2023年1月2日 都签到了}
```

### （7） 基数统计（HyperLogLog）

#### 是什么

> 1、去重复统计功能的基数估计算法-就是HyperLogLog
>
> 2、基数：是一种数据集，去重复后的真实个数
>
> 3、基数统计：用于统计一个集合中不重复的元素个数，就是对集合去重复后剩余元素的计算

#### 基本命令

| 命令                                     | 作用                                       |
| ---------------------------------------- | ------------------------------------------ |
| PFADD key element [element...]           | 添加指定元素到`HyperLogLog`中              |
| PFCOUNT key [key...]                     | 返回给定`HyperLogLog`的基数估算值          |
| PFMERGE destkey sourcekey [sourcekey...] | 将多个`HyperLogLog`合并为一个`HyperLogLog` |

#### 案例

```
> pfadd log1 1 1 2 3 5 7 9
(integer) 1
> pfadd log2 0 2 4 6 8 
(integer) 1
> PFCOUNT log1
(integer) 6
> PFCOUNT log2
(integer) 5
> PFMERGE distresult log1 log2
OK
> PFCOUNT distresult
(integer) 10
```

### （8）地理空间（GEO）

#### 常用命令

![8](images/8.png)

> georadius 以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。
>
> GEORADIUS city 116.418017 39.914402 10 km withdist withcoord count 10 withhash desc
>
> GEORADIUS city 116.418017 39.914402 10 km withdist withcoord withhash count 10 desc
>
> WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。 距离的单位和用户给定的范围单位保持一致。
>
> WITHCOORD: 将位置元素的经度和维度也一并返回。
>
> WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大
>
> COUNT: 限定返回的记录数。

#### 案例

```
> GEOADD city 116.403963 39.915119 "天安门" 116.403414 39.924091 "故宫" 116.024067 40.362639 "长城"	{添加经纬度坐标}
3
> type city
zset
> ZRANGE city 0 -1
天安门
故宫
长城
> GEOPOS city 天安门 故宫 长城		{返回经纬度}
116.40396326780319214
39.91511970338637383
116.40341609716415405
39.92409008156928252
116.02406591176986694
40.36263993239462167
> GEOHASH city 天安门 故宫 长城	{返回坐标}
wx4g0f6f2v0
wx4g0gfqsj0
wx4t85y1kt0
> GEODIST city 天安门 故宫 m		{计算两个地点的距离}
998.8332
```

 **当前位置(116.418017 39.914402),阳哥在北京王府井** 

![8.1](images/8.1.png)

### （9）流（Stream）

#### 特殊符号

> `- +`: 最小和最大可能出现的Id
>
> `$`: 表示只消费新的消息，当前流中最大的Id，可用于将要到来信息
>
> `>`: 用于XREADGROUP命令，表示迄今还没有发送给组中使用者的信息，会更新消费者组的最后Id
>
> `*`: 用于XADD命令中，让系统自动生成id

#### 队列相关指令

> XADD: 添加消息到队列末尾
>
> XADD 用于向Stream 队列中添加消息，如果指定的Stream 队列不存在，则该命令执行时会新建一个Stream 队列
>
> `*` 号表示服务器自动生成 MessageID(类似mysql里面主键auto_increment)，后面顺序跟着一堆 业务key/value
>
> ```
> > xadd mystream * id 11 cname z3
> 1684558919211-0
> ```
>
> - 信息条目指的是序列号，在相同的毫秒下序列号从0开始递增，序列号是64位长度，理论上在同一毫秒内生成的数据量无法到达这个级别，因此不用担心序列号会不够用。millisecondsTime指的是Redis节点服务器的本地时间，如果存在当前的毫秒时间戳比以前已经存在的数据的时间戳小的话（本地时间钟后跳），那么系统将会采用以前相同的毫秒创建新的ID，也即redis 在增加信息条目时会检查当前 id 与上一条目的 id， 自动纠正错误的情况，一定要保证后面的 id 比前面大，一个流中信息条目的ID必须是单调增的，这是流的基础。
>
> - 客户端显示传入规则:
>
>     Redis对于ID有强制要求，格式必须是时间戳-自增Id这样的方式，且后续ID不能小于前一个ID
>
> - Stream的消息内容，也就是图中的Message Content它的结构类似Hash结构，以key-value的形式存在。

> XRANGE: 用于获取消息列表（可以指定范围），忽略删除的消息
>
> ​		star: 表示开始值，- 代表最小值
>
> ​		end: 表示结束值，+ 代表最大值
>
> ​		count: 表示最多获取多少个值
>
> XREVRANGE: 反转；获取消息列表元素的方向是相反的
>
> ```
> > XADD mystream * id 11 cname z3
> 1684560052624-0
> > XADD mystream * id 12 cname li4
> 1684560067497-0
> > XADD mystream * k1 v1 k2 v2 k3 v3
> 1684560164922-0
> > type mystream
> stream
> > XRANGE mystream - +	{输出 mystream }
> 1) 1) "1684560052624-0"
>    2) 1) "id"
>       2) "11"
>       3) "cname"
>       4) "z3"
> 2) 1) "1684560067497-0"
>    2) 1) "id"
>       2) "12"
>       3) "cname"
>       4) "li4"
> 3) 1) "1684560164922-0"
>    2) 1) "k1"
>       2) "v1"
>       3) "k2"
>       4) "v2"
>       5) "k3"
>       6) "v3"
> > XRANGE mystream - + count 2	{输出指定条数}
> 1) 1) "1684560052624-0"
>    2) 1) "id"
>       2) "11"
>       3) "cname"
>       4) "z3"
> 2) 1) "1684560067497-0"
>    2) 1) "id"
>       2) "12"
>       3) "cname"
>       4) "li4"
> > XREVRANGE mystream + 1	{反转输出}
> 1) 1) "1684560164922-0"
>    2) 1) "k1"
>       2) "v1"
>       3) "k2"
>       4) "v2"
>       5) "k3"
>       6) "v3"
> 2) 1) "1684560067497-0"
>    2) 1) "id"
>       2) "12"
>       3) "cname"
>       4) "li4"
> 3) 1) "1684560052624-0"
>    2) 1) "id"
>       2) "11"
>       3) "cname"
>       4) "z3"
> ```

> XDEL: 用于根据id删除
>
> XLEN: 用于获取stream队列的消息的长度
>
> ```
> > XDEL mystream 1684560164922-0		{根据id删除}
> (integer) 1
> > XRANGE mystream - +
> 1) 1) "1684560052624-0"
>    2) 1) "id"
>       2) "11"
>       3) "cname"
>       4) "z3"
> 2) 1) "1684560067497-0"
>    2) 1) "id"
>       2) "12"
>       3) "cname"
>       4) "li4"
> > xlen mystream		{用于获取stream队列的消息的长度}
> (integer) 2
> ```

> XTRIM: 用于对Stream的长度行进行截取，如超长会进行截取
> ​		MAXLEN: 允许的最大长度，对流进行修剪限制长度
> ​		MINID: 允许的最小id，从某个id值小的将会抛弃
>
> ```
> > xadd mystream * k1 v1
> "1684562041043-0"
> > xadd mystream * k2 v2
> "1684562045876-0"
> > xadd mystream * k3 v3
> "1684562049694-0"
> > xadd mystream * k4 v4
> "1684562054396-0"
> > xadd mystream * k5 v5
> "1684562058352-0"
> > xadd mystream * k6 v6
> "1684562061854-0"
> > xrange mystream - +
> 1) 1) "1684562041043-0"
>    2) 1) "k1"
>       2) "v1"
> 2) 1) "1684562045876-0"
>    2) 1) "k2"
>       2) "v2"
> 3) 1) "1684562049694-0"
>    2) 1) "k3"
>       2) "v3"
> 4) 1) "1684562054396-0"
>    2) 1) "k4"
>       2) "v4"
> 5) 1) "1684562058352-0"
>    2) 1) "k5"
>       2) "v5"
> 6) 1) "1684562061854-0"
>    2) 1) "k6"
>       2) "v6"
> 
> > xtrim mystream maxlen 5	{截取最大的5个stream}
> (integer) 1		{截取掉了1个}
> > xrange mystream - +
> 1) 1) "1684562045876-0"
>    2) 1) "k2"
>       2) "v2"
> 2) 1) "1684562049694-0"
>    2) 1) "k3"
>       2) "v3"
> 3) 1) "1684562054396-0"
>    2) 1) "k4"
>       2) "v4"
> 4) 1) "1684562058352-0"
>    2) 1) "k5"
>       2) "v5"
> 5) 1) "1684562061854-0"
>    2) 1) "k6"
>       2) "v6"
> > xtrim mystream minid 1684562045876-0		{以截取比id}
> (integer) 0
> > xtrim mystream minid 1684562054396-0
> (integer) 2
> > xrange mystream - +
> 1) 1) "1684562054396-0"
>    2) 1) "k4"
>       2) "v4"
> 2) 1) "1684562058352-0"
>    2) 1) "k5"
>       2) "v5"
> 3) 1) "1684562061854-0"
>    2) 1) "k6"
>       2) "v6"
> ```
>
> XREAD: 用于获取消息（阻塞/非阻塞），只会返回大于指定ID的消息
>
> XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key...] ID [ID...]
> ​		COUNT: 最多读取多少条消息
> ​		BLOCK是否已阻塞的方式读取消息，默认不阻塞，如果milliseconds设置为0，表示永远阻塞
>
> 非阻塞：
>
> ​		$代表特殊ID，表示以当前Stream已经存储的最大的ID作为最后一个ID，当前Stream中不存在大于当前最大ID的消息，因此此时返回nil
> ​		0-0代表从最小的ID开始获取Stream中的消息，当不指定count，将会返回Stream中的所有消息，注意也可以使用0（00/000也都是可以的……）
>
> ```
> > XREAD count 2 streams mystream 000	{指定count=2就返回2个stream}
> 1) 1) "mystream"
>    2) 1) 1) "1684562045876-0"
>          2) 1) "k2"
>             2) "v2"
>       2) 1) "1684562049694-0"
>          2) 1) "k3"
>             2) "v3"
> > XREAD streams mystream 0-0	{不指定count就返回所有stream}
> 1) 1) "mystream"
>    2) 1) 1) "1684562045876-0"
>          2) 1) "k2"
>             2) "v2"
>       2) 1) "1684562049694-0"
>          2) 1) "k3"
>             2) "v3"
>       3) 1) "1684562054396-0"
>          2) 1) "k4"
>             2) "v4"
>       4) 1) "1684562058352-0"
>          2) 1) "k5"
>             2) "v5"
>       5) 1) "1684562061854-0"
>          2) 1) "k6"
>             2) "v6"
> ```
>
> 阻塞：
>
> ```
> > xread count 1 block 0 streams mystream $	{阻塞1个流}
> 1) 1) "mystream"				{,当新建之后，输出刚新建的流}
>    2) 1) 1) "1684806573840-0"
>          2) 1) "k7"
>             2) "v7"
>             3) "k8"
>             4) "v8"
> (50.07s)
> 
> > xadd mystream * k7 v7 k8 v8	{新增流}
> "1684806573840-0"
> ```
>

#### 消费组相关指令

> XGROUP CREATE: 用于创建消费组
>
> XGROUP CREATE key groupname id|$ [MKSTREAM] [ENTRIESREAD entries_read]
>
> $表示从Stream尾部开始消费；0表示从Stream头部开始消费；
>
> 创建消费者组的时候必须指定 ID，ID为0表示从头开始消费，为$表示只消费新的消息，<span style="color:red; font-weight:bold">队尾新来</span> 
>
> ```
> ========准备工作========
> > xadd mystream * k1 v1
> "1684819899589-0"
> > xadd mystream * k2 v2
> "1684819905723-0"
> > xadd mystream * k3 v3
> "1684819911311-0"
> > xadd mystream * k4 v4 k5 v5
> "1684819923515-0"
> > xrange mystream - +
> 1) 1) "1684819899589-0"
>    2) 1) "k1"
>       2) "v1"
> 2) 1) "1684819905723-0"
>    2) 1) "k2"
>       2) "v2"
> 3) 1) "1684819911311-0"
>    2) 1) "k3"
>       2) "v3"
> 4) 1) "1684819923515-0"
>    2) 1) "k4"
>       2) "v4"
>       3) "k5"
>       4) "v5"
> 
> > xgroup create mystream groupA $	{$表示从Stream尾部开始消费}
> OK
> > xgroup create mystream groupB 0	{0表示从Stream头部开始消费}
> OK
> ```
>
> 

> xreadgroup group: ">"，表示从第一条尚未被消息的消息开始读取
>
> ![](images/9.1.png)
>
> ![](images/9.2.png)
>
> 让组内的多个消费者共同分担读取消息，所以，我们通常会让每个消费者读取部分消息，从而实现消息读取负载在多个消费者间是均衡分布的
>
> ![](images/9.3.png)
>
> 

##### 重点问题

| 问题1 | 基于Stream实现的消息队列，如何保证消费者在发生故障或宕机再次重启后，仍然可以读取未处理完的消息？ |
| ----- | ------------------------------------------------------------ |
| 问题2 | Streams 会自动使用内部队列（也称为 PENDING List）留存消费组里每个消费者读取的消息保底措施，直到消费者使用 XACK 命令通知 Streams“消息已经处理完成”。 |
| 问题3 | 消费确认增加了消息的可靠性，一般在业务处理完成之后，需要执行 XACK 命令确认消息已经被消费完成 |

![](images/9.4.png)

> XPENDING: 
>
> ​		查询每个消费组内所有消费组「<span style="color:#4662d9; font-weight:bold">已读取、但尚未确认</span>」的消息
>
> ![](images/9.5.png)
>
> ​		查看某个消费者具体读取了哪些数据
>
> ![](images/9.6.png)
>
> 下面抓图所示：consumer2已读取的消息的 ID是1659430293537-0一旦消息1659430293537-0被consumer2处理了consumer2就可以使用 XACK 命令通知 Streams，然后这条消息就会被删除
>
> ![](images/9.7.png)

> XACK: 向消息队列确认消息处理已完成
>
> ![](images/9.8.png)
>
> ![](images/9.9.png)

#### XINFO: 用于打印Stream\Consumer\Group的详细消息

![](images/9.10.png)

### （10）位域（bitfield）

#### 是什么

![](images/10.png)

#### 能干什么

![](images/10.1.png)

> hello 等价于 0110100001100101011011000110110001101111		{二进制形式}
>
> 将一个Redis字符串看作是<span style="color:#4662d9; font-weight:bold">一个由二进制位组成的数组</span>，并能对变长位宽和任意没有字节对齐的指定整型位域进行寻址和修改

#### 命令基本语法

> BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]
>
> ![](images/10.2.png)

#### 案例

> BITFIELD key [GET type offset]
>
> ​	GET <type> <offset> 返回指定的位域
>
> ```
> > set fieldkey hello
> OK
> > BITFIELD fieldkey get i8 0
> 1) (integer) 104
> > BITFIELD fieldkey get i8 8
> 1) (integer) 101
> > BITFIELD fieldkey get i8 16
> 1) (integer) 108
> > BITFIELD fieldkey get i8 24
> 1) (integer) 108
> > BITFIELD fieldkey get i8 32
> 1) (integer) 111
> ```
>
> | 字母 | 数值 | 二进制（高位<-低位） |
> | :--: | :--: | :------------------: |
> |  h   | 104  |      0110 1000       |
> |  e   | 101  |      0110 0101       |
> |  l   | 108  |      0110 1100       |
> |  l   | 108  |      0110 1100       |
> |  o   | 111  |      0110 1111       |

> BITFIELD key [SET type offset value]
>
> ​	SET <type> <offset> <value> 设置指定位域的值并返回它的原值
>
> ```
> > BITFIELD fieldkey set i8 8 120	{120为x的二进制的编码值}
> 1) (integer) 101
> > get fieldkey
> "hxllo"
> ```
>
> 

> BITFIELD key [INCRBY type offset increment]
>
> ​		INCRBY <type> <offset> <increment> 从第`offset+1`个位开始，对接下来的`4位` `无符号数` `+increment` 
>
> ```
> > BITFIELD fieldkey incrby u4 3 1	{默认overflow为wrap，即循环溢出}
> 1) (integer) 5
> > BITFIELD fieldkey incrby u4 3 1
> 1) (integer) 6
> > BITFIELD fieldkey incrby u4 3 1
> 1) (integer) 7
> ......
> > BITFIELD fieldkey incrby u4 3 1
> 1) (integer) 14
> > BITFIELD fieldkey incrby u4 3 1
> 1) (integer) 15
> > BITFIELD fieldkey incrby u4 3 1
> 1) (integer) 0
> ```
>
> ```
> > set test a
> OK
> 127.0.0.1:6379> BITFIELD test get i8 0
> 1) (integer) 97
> > BITFIELD test set i8 0 126
> 1) (integer) 97
> > BITFIELD test get i8 0
> 1) (integer) 126
> > get test
> "~"
> ```

> BITFIELD key [OVERFLOW WRAP|SAT|FAIL]
>
> ​		WRAR: 使用回绕（wrap around）方法处理有符号整数和无符号整数的溢出情况
>
> ​		SAT: 使用饱和计算方法处理溢出，下溢计算的结果为最下的整数值（-128），而上溢计算的结果为最大的整数值（127）
>
> ​		FALIL: 命令将拒绝执行那些会导致上溢或者下溢情况出现的计算，	并向用户返回空值表示计算未被执行
>
> ```
> > BITFIELD test get i8 0
> 1) (integer) 126
> > BITFIELD test set i8 0 258
> 1) (integer) 126
> > BITFIELD test get i8 0
> 1) (integer) 2
> > BITFIELD test overflow sat set i8 0 128
> 1) (integer) 2
> > BITFIELD test get i8 0
> 1) (integer) 127		{最大为127，最小为-128}
> > BITFIELD test overflow fail set i8 0 128
> 1) (nil)
> ```

# Redis 持久化

## RDB

### 是什么

> 在指定的时间间隔，执行数据集的时间点快照

### 能干啥

> 在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot内存快照，它恢复时再将硬盘快照文件直接读回到内存里

### 案例演示

#### 需求说明：RDB保存到磁盘的文件叫<span style="color:#4662d9; font-weight:bold">dump.rdb</span> 

![持久化-1](images/持久化-1.png)

#### 配置文件

Redis 6.0.16 以下

![持久化-2](images/持久化-2.png)

Redis 6.2 以上Redis-7.0.12

![持久化-3](images/持久化-3.png)

#### 自动触发

##### （1）修改配置

> [root@node2 myredis]# vim redis7.conf
> 		在`行号439`加上`save 5 2` 	{5秒内2次操作}
> 		在`行号487`将`dbfilename dump.rdb`修改为`dbfilename dump6379.rdb` 
> 		在`行号510`将`dir ./`修改为`dir /myredis/dumpfiles` 
>
> 修改完之后
>
> [root@node2 myredis]# ps -ef|grep redis-server		{查看进程`redis-server`}
> [root@node2 myredis]# kill -9 2985		{杀掉进程`redis-server`}
> [root@node2 myredis]# redis-server redis7.conf 		{重启`redis`服务}

#### 手动触发

##### （1）Save

> 在主程序中执行`会阻塞`当前redis服务器，直接持久化工作完成执行save命令期间，Redis不能处理其他命令，线上禁止使用

##### （2）bgsave

> Redis会在后台异步进行快照操作，`不阻塞`快照同时还可以响应客户端请求，该触发方式会fork一个子进程有子进程复制持久化过程
>
> fork: 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，尽量避免膨胀。

> lastsave: 可以通过lastsave命令获取最后一次成功执行快照的时间
>
> ```
> > LASTSAVE
> (integer) 1685061385
> 
> [root@node2 dumpfiles]# date -d @1685061385
> 2023年 05月 26日 星期五 08:36:25 CST
> ```

#### 优点

> 1、适合大规模的数据恢复
>
> 2、按照业务定时备份
>
> 3、对数据完成性和一致性要求不高
>
> 4、RDB文件在内存的加载速度要比AOF快得多

#### 缺点

> 1、在一定间隔时间做一次备份，所以如果redis以外down掉的话，就会丢失从当前至最近一次快照期间的数据，快照之间的数据会丢失
>
> 2、内存数据的全量同步，如果数据量太大会导致I/O严重影响服务器性能
>
> 3、RDB依赖于主进程的fork，在更大的数据集中，这可能会导致服务请求的瞬间延迟。fork的时候内存中的数据被克隆了一份，大致2倍的膨胀性，需要考虑

#### 如何检查修复dump.rdb文件

![如何检查修复dump.rdb文件](images/持久化-5.png)

#### 哪些情况会触发RDB快照

> 1、配置文件中默认的快照配置
>
> 2、手动save/bgsave命令
>
> 3、执行flushall/flushdb命令也会产生dump.rdb文件，但里面是空的，无意义
>
> 4、执行shutdown且没有设置开启AOF持久化
>
> 5、主从复制时，主节点自动触发

#### 如何禁用快照

> 命令：`redis-cli config set save ""` 
>
> 修改配置文件redis7.conf：加上`save ""` 

#### RDB 优化配置项详解

> 配置文件`SNAPSHOTTING`模块：
>
> stop-writes-on-bgsave-error: 默认yes
> 		如果配置成no，表示你不在乎数据不一致或者有其他的手段发现和控制这种不一致，那么在快照写入失败时，也能确保redis继续接受新的写请求
>
> rdbcompression: 默认yes
> 		对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。
> 如果你不想消耗CPU来进行压缩的话，可以设置为关闭次功能
>
> rdbchecksum: 默认yes
> 		在存储快照后，还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
>
> rdb-del-sync-files: 在没有持久性的情况下删除复制中使用的RDB文件启用。默认情况下no，此选项是禁用的。

### 总结

![RDB小结](images/RDB小结.png)

## AOF

### 是什么

> 以日志的形式类记录每个写操作，将Redis执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作
>
> 默认情况下，redis是没有开启AOF的。
>
> 开启AOF功能需要设置配置：appendonly yes

### 工作流程

![AOF-工作流程](images/AOF-工作流程.png)

> 1、Client作为命令的来源，会有多个源头以及源源不断的请求命令。
>
> 2、在这些命令到达Redis Server 以后并不是直接写入AOF文件，会将其这些命令先放入AOF缓存中进行保存。这里的AOF缓冲区实际上是内存中的一片区域，存在的目的是当这些命令达到一定量以后再写入磁盘，避免频繁的磁盘IO操作。
>
> 3、AOF缓冲会根据AOF缓冲区<span style="color:red; font-weight:bold">同步文件的三种写回策略</span>将命令写入磁盘上的AOF文件。
>
> 4、随着写入AOF内容的增加为避免文件膨胀，会根据规则进行命令的合并(又称<span style="color:red; font-weight:bold">AOF重写</span>)，从而起到AOF文件压缩的目的。
>
> 5、当Redis Server 服务器重启的时候会从AOF文件载入数据。

### 写回策略

> Always: 同步写回，每个写命令执行完立刻同步地将日志写回磁盘
>
> everysec: 每秒写回，每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，每隔1秒把缓冲区中的内容写入磁盘
>
> no: 操作系统控制的写回，每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，由操作系统决定何时将缓冲区写回磁盘

|  配置项  |      写回时机      | 优点                     | 缺点                             |
| :------: | :----------------: | ------------------------ | -------------------------------- |
|  Always  |      同步写回      | 可靠性高，数据基本不丢失 | 每个写命令都要落盘，性能影响较大 |
| Everysec |      每秒写回      | 性能适中                 | 宕机时丢失1秒内的数据            |
|    No    | 操作系统控制的写回 | 性能好                   | 宕机时丢失数据较多               |

> 配置文件中`redis7.conf` 
>
> 在`1385行`将`appendonly no`修改为`appendonly yes` 

### 优缺点

> 优点：更好的保护数据不丢失、性能搞、可做紧急恢复
>
> 缺点：①  相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
>
> ​			②  aof运行效率要慢于rdb，每秒同步策略效率较好，不同步效率和rdb相同

### 重写机制

> 自动触发：满足配置文件中的选项后，Redis会记录上次重写时的AOF大小，默认配置是当AOF文件是上次rewrite后大小的一倍且文件大于64M时
>
> 手动触发：客户端向服务器发送bgrewriteaof命令

| 配置指令                                                 |          配置含义          | 配置示例                                                     |
| :------------------------------------------------------- | :------------------------: | ------------------------------------------------------------ |
| appendonly                                               |        是否开启aof         | appendonly yes                                               |
| appendfilename                                           |          文件名称          | appendfilename "appendonly.aof"                              |
| appendfsync                                              |          同步方式          | everysec/always/no                                           |
| no-appendfsync-on-rewrite                                |    aof重写期间是否同步     | no-appendfsync-on-rewrite no                                 |
| auto-aof-rewrite-percentage<br>auto-aof-rewrite-min-size | 重写触发配置、文件重写策略 | auto-aof-rewrite-percentage 100<br/>auto-aof-rewrite-min-size 64mb |

### 总结

![AOF总结](images/AOF总结.png)

# Redis 事务

## 是什么

> 可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不许加塞

## 能干啥

> 一个队列中，一次性、顺序性、排他性的执行一系列命令

## Redis 事务

> 1、单独的隔离操作
>
> ​		redis的事务仅仅是保证事务里的操作会被连续独占的执行，redis命令执行是单线程架构，在执行事务内所有指令前是不可能再去同时执行其他客户端的请求的
>
> 2、没有隔离级别的概念
>
> ​		因为事务提交前任何指令都不会被实际执行，也就不存在“事务的查询要看到事务里的更新，在事务外查询不能看到”这种问题了
>
> 3、不保证原子性
>
> ​		redis的事务不保证原子性，也就是不保证所有指令同时成功或同时失败，只有决定是否执行全部指令的能力，没有执行到一半进行回滚的能力
>
> 4、排它性
>
> ​		redis会保证一个事务内命令依次执行，而不会被其他命令插入

## 常用命令

> MULTI: 标记一个事务块的开始
>
> EXEC: 执行所有事务块的命令
>
> DISCARD: 取消事务，放弃执行事务块内的所有命令
>
> WATCH: 监控一个（或多个）key，如果在事务执行之前key被其他命令所改动，那么事务被打断
>
> UNWATCH: 取消WATCH监控的key
>
> 小结：①  一旦执行了exec之前加的监控锁都会被取消掉了
>
> ​			②  当客户端连接丢失的时候（比如退出链接），所有东西都会被取消监视

## 总结

>  开启：以MUTL开始一个事务
>
> 入队：将多个命令入队到事务中，接到命令并不会立即执行，而是放到等待执行的事务队列里面
>
> 执行：由EXEC命令触发事务

# Redis 管道

## 是什么

> 批处理命令变种优化措施，类似Redis的原生批命令（mget和mset）

![Redis管道](images/Redis管道.png)

## 总结

### `Pipeline`与`原生批量命令`对比

> ①  原生批量命令是原子性（例如：mset，mget），`pipeline`是非原子性
>
> ②  原生批量命令依次只能执行一种命令，`pipeline`支持批量执行不同命令
>
> ③  原生批命令是服务端实现，而`pipeline`需要服务端与客户端共同完成

### `Pipeline`与`事务`对比

> ①  事务具有原子性，管道不具有原子性
>
> ②  管道一次性将多条命令发送到服务器，事务是一条一条的发，事务只有在接收到exec命令后才会执行，管道不会
>
> ③  执行事务时会阻塞其他命令的执行，而执行管道中的命令时不会

### 使用`Pipeline`<span style="color:red; font-weight:bold">注意事项</span> 

> ① ` pipeline`缓冲的指令只是会依次执行，不保证原子性，如果执行中指令发生异常，将会继续执行后续的指令
>
> ②  使用`pipeline`组装的命令个数不能太多，不然数据量过大客户端阻塞的时间可能过久，同时服务器此时也被迫回复一个队列答复，占用很多内存

# Redis 主从复制（replica）

## 是什么

> 就是主从复制，master以写为主，Slave以读为主

















