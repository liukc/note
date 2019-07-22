# redis学习笔记

## 一、redis安装

Linux上安装：

```git
$ wget http://download.redis.io/releases/redis-2.8.17.tar.gz
$ tar xzf redis-2.8.17.tar.gz
$ cd redis-2.8.17
$ make
```

启动redis服务：

```git
$ ./redis-server
// 要先cd 到src目录下 
```

启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了：

```git
$ ./redis-cli
127.0.0.1:6379> set boo bar
OK
127.0.0.1:6379> get boo
"bar"
```

注：设置了密码，需要输入密码

```c
127.0.0.1:6379> auth <yourPassword>
```



## 二、redis数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)

### 1. String

string 是 redis 最基本的类型，一个 key 对应一个 value。

string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

```linux
127.0.0.1:6379> set name "Bob"
OK
127.0.0.1:6379> get name
"Bob"
```

**注意：**一个键最大能存储512MB。

### 2. Hash

Redis hash 是一个键值(key=>value)对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

```c
127.0.0.1:6379> HMSET myhash "china" "chinese" "japan" "japanese"
OK
127.0.0.1:6379> HGET myhash china
"chinese"
```

**HMSET** 设置了两个 **field=>value** 对, HGET 获取对应 **field** 对应的 **value**。

每个 hash 可以存储对键值对（约四十亿）
$$ 2^{32} -1$$

### 3. List

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

```c
127.0.0.1:6379> LPUSH myList liu kai chuan
(integer) 3
127.0.0.1:6379> LPUSH myList is a boy
(integer) 6
127.0.0.1:6379> LRANGE myList 0 10
1) "boy"
2) "a"
3) "is"
4) "chuan"
5) "kai"
6) "liu"

```


$$
列表最多可存储2^{32}-1元素 (4294967295, 每个列表可存储40多亿)。
$$

### 4. Set

Redis的Set是string类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

添加一个 string 元素到 key 对应的 set 集合中，成功返回1，如果元素已经在集合中返回 0，如果 key 对应的 set 不存在则返回错误。

```c
127.0.0.1:6379> SADD mySet redis
(integer) 1
127.0.0.1:6379> SADD mySet mysql
(integer) 1
127.0.0.1:6379> SADD mySet mysql
(integer) 0
127.0.0.1:6379> SMEMBERS mySet
1) "mysql"
2) "redis"
```

集合中最大的成员数为 $$2^{32} - 1$$(4294967295, 每个集合可存储40多亿个成员)。

### 5. Zset

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

```c
127.0.0.1:6379> ZADD myZset 0 "liu" 1 "kai" 2 "chuan"
(integer) 3
127.0.0.1:6379> ZRANGEBYSCORE myZset 0 10
1) "liu"
2) "kai"
3) "chuan"
127.0.0.1:6379> ZADD myZset 0 "liuliu" 0 "liukaichuan"
(integer) 2
127.0.0.1:6379> ZRANGEBYSCORE myZset 0 10
1) "liu"
2) "liukaichuan"
3) "liuliu"
4) "kai"
5) "chuan"
```

## 三、redis命令

`redis-cli` 命令：

连接本地的redis服务

```c
[root@zyxy01 src]# ./redis-cli 
127.0.0.1:6379> 
```

在远程服务器上执行命令：

```c
redis-cli  -h host -p port -a password
```

如：

```c
$redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING

PONG
```

## 四、redis键（key）

Redis键命令的语法：

```c
COMMAND KEY_NAME
```

如：

```c
127.0.0.1:6379> set girl "protect"
OK
127.0.0.1:6379> DEL girl
(integer) 1
```

Redis keys命令:

| 序号 | 命令                                 | 作用                                                        |
| ---- | ------------------------------------ | ----------------------------------------------------------- |
| 1    | DEL   key                            | 该命令用于在 key 存在时删除 key                             |
| 2    | DUMP   key                           | 序列化给定 key ，并返回被序列化的值                         |
| 3    | EXISTS key                           | 检查给定 key 是否存在                                       |
| 4    | EXPIRE key second                    | 为给定key 设置时间，以秒计                                  |
| 5    | EXPIREAT key timestamp               | EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| 6    | PEXPIRE key milliseconds             | 设置 key 的过期时间以毫秒计。                               |
| 7    | PEXPIREAT key milliseconds-timestamp | 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计          |
| 8    | KEYS pattern                         | 查找所有符合给定模式( pattern)的 key 。                     |
| 9    | MOVE key db                          | 将当前的key 移动到给定的数据库中                            |
| 10   | PERSIST key                          | 移除key 的过期时间，key将持久保持                           |
| 11   | PTTL key                             | 以毫秒为单位返回key的剩余的过期时间                         |
| 12   | TTL key                              | 以秒为单位，返回给定的key的剩余时间                         |
| 13   | RANDOMKEY                            | 从当前数据库中随机返回一个key                               |
| 14   | RENAME key newkey                    | 修改key的名称                                               |
| 15   | RENAMENX key newkey                  | 仅当newkey 不存在时，将key 的名字设置为newkey               |
|      | TYPE key                             | 返回key 所存储的值的类型                                    |

## 	五、redis字符串

语法：

```c
127.0.0.1:6379> COMMAND KEY_NAME
```

如：

```c
127.0.0.1:6379> set bar foo
OK
127.0.0.1:6379> get bar
"foo"
```

redis 常用字符串命令:

| 序号 | 命令                             | 描述                                                         |
| ---- | -------------------------------- | ------------------------------------------------------------ |
| 1    | SET key value                    | 设置指定key的值                                              |
| 2    | GET key                          | 获取指定key的值                                              |
| 3    | GETRANGE key start end           | 返回 key 中字符串值的子字符                                  |
| 4    | GETSET key value                 | 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。   |
| 5    | GETBIT key offset                | 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。         |
| 6    | MGET key1 [key2..]               | 获取所有(一个或多个)给定 key 的值                            |
| 7    | SETBIT key offset value          | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。   |
| 8    | SETEX key seconds value          | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。 |
| 9    | SETNX key value                  | 只有在 key 不存在时设置 key 的值。                           |
| 10   | SETRANGE key offset value        | 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。 |
| 11   | STRLEN key                       | 返回 key 所储存的字符串值的长度。                            |
| 12   | MSET key value [key value ...]   | 同时设置一个或多个 key-value 对。                            |
| 13   | MSETNX key value [key value ...] | 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。 |
| 14   | PSETEX key milliseconds value    | 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。 |
| 15   | INCR key                         | 将 key 中储存的数字值增一。                                  |
| 16   | INCRBY key increment             | 将 key 所储存的值加上给定的增量值（increment） 。            |
| 17   | INCRBYFLOAT key increment        | 将 key 所储存的值加上给定的浮点增量值（increment） 。        |
| 18   | DECR key                         | 将 key 中储存的数字值减一。                                  |
| 19   | DECRBY key decrement             | key 所储存的值减去给定的减量值（decrement）                  |
| 20   | APPEND key value                 | 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。 |





## 六、redis哈希

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

```c
127.0.0.1:6379> HMSET myhash "china" "chinese" "japan" "japanese"
OK
127.0.0.1:6379> HGET myhash china
"chinese"
```

Redis hash 命令：

下表列出了 redis hash 基本的相关命令：

| 序号 | 命令                                           | 描述                                                     |
| :--- | :--------------------------------------------- | -------------------------------------------------------- |
| 1    | HDEL key field1 [field2]                       | 删除一个或多个哈希表字段                                 |
| 2    | HEXISTS key field                              | 查看哈希表 key 中，指定的字段是否存在。                  |
| 3    | HGET key field                                 | 获取存储在哈希表中指定字段的值。                         |
| 4    | HGETALL key                                    | 获取在哈希表中指定 key 的所有字段和值                    |
| 5    | HINCRBY key field increment                    | 为哈希表 key 中的指定字段的整数值加上增量 increment 。   |
| 6    | HINCRBYFLOAT key field increment               | 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。 |
| 7    | HKEYS key                                      | 获取所有哈希表中的字段                                   |
| 8    | HLEN key                                       | 获取哈希表中字段的数量                                   |
| 9    | HMGET key field1 [field2]                      | 获取所有给定字段的值                                     |
| 10   | HMSET key field1 value1 [field2 value2 ]       | 同时将多个 field-value (域-值)对设置到哈希表 key 中。    |
| 11   | HSET key field value                           | 将哈希表 key 中的字段 field 的值设为 value 。            |
| 12   | HSETNX key field value                         | 只有在字段 field 不存在时，设置哈希表字段的值。          |
| 13   | HVALS key                                      | 获取哈希表中所有值                                       |
| 14   | HSCAN key cursor [MATCH pattern] [COUNT count] | 迭代哈希表中的键值对。                                   |

## 七、redis列表

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

```c
127.0.0.1:6379> LPUSH myList liu kai chuan
(integer) 3
127.0.0.1:6379> LPUSH myList is a boy
(integer) 6
127.0.0.1:6379> LRANGE myList 0 10
1) "boy"
2) "a"
3) "is"
4) "chuan"
5) "kai"
6) "liu"
```

Redis 列表命令

下表列出了列表相关的基本命令：

| 序号 | 命令                                  | 描述                                                         |
| :--- | :------------------------------------ | ------------------------------------------------------------ |
| 1    | BLPOP key1 [key2 \] timeout           | 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 2    | BRPOP key1 [key2 \] timeout           | 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 3    | BRPOPLPUSH LIST1 ANOTHER_LIST TIMEOUT | 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| 4    | LINDEX key index                      | 通过索引获取列表中的元素                                     |
| 5    | LINSERT key BEFORE\|AFTER pivot value | 在列表的元素前或者后插入元素                                 |
| 6    | LLEN key                              | 获取列表长度                                                 |
| 7    | LPOP key                              | 移出并获取列表的第一个元素                                   |
| 8    | LPUSH key value1 [value2\]            | 将一个或多个值插入到列表头部                                 |
| 9    | LPUSHX key value                      | 将一个值插入到已存在的列表头部                               |
| 10   | LRANGE key start stop                 | 获取列表指定范围内的元素                                     |
| 11   | LREM key count value                  | 移除列表元素                                                 |
| 12   | LSET key index value                  | 通过索引设置列表元素的值                                     |
| 13   | LTRIM key start stop                  | 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。 |
| 14   | RPOP key                              | 移除列表的最后一个元素，返回值为移除的元素。                 |
| 15   | RPOPLPUSH source destination          | 移除列表的最后一个元素，并将该元素添加到另一个列表并返回     |
| 16   | RPUSH key value1 [value2\]            | 在列表中添加一个或多个值                                     |
| 17   | RPUSHX key value                      | 为已存在的列表添加值                                         |

## 八、redis集合

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

```c
127.0.0.1:6379> SADD mySet redis
(integer) 1
127.0.0.1:6379> SADD mySet mysql
(integer) 1
127.0.0.1:6379> SADD mySet mysql
(integer) 0
127.0.0.1:6379> SMEMBERS mySet
1) "mysql"
2) "redis"
```

redis集合命令

表列出了 Redis 集合基本命令：

| 序号 | 命令                                            | 描述                                                |
| :--- | :---------------------------------------------- | --------------------------------------------------- |
| 1    | SADD key member1 [member2\]                     | 向集合添加一个或多个成员                            |
| 2    | SCARD key                                       | 获取集合的成员数                                    |
| 3    | SDIFF key1 [key2\]                              | 返回给定所有集合的差集                              |
| 4    | SDIFFSTORE destination key1 [key2\]             | 返回给定所有集合的差集并存储在 destination 中       |
| 5    | SINTER key1 [key2\]                             | 返回给定所有集合的交集                              |
| 6    | SINTERSTORE destination key1 [key2\]            | 返回给定所有集合的交集并存储在 destination 中       |
| 7    | SISMEMBER key member                            | 判断 member 元素是否是集合 key 的成员               |
| 8    | SMEMBERS key                                    | 返回集合中的所有成员                                |
| 9    | SMOVE source destination member                 | 将 member 元素从 source 集合移动到 destination 集合 |
| 10   | SPOP key                                        | 移除并返回集合中的一个随机元素                      |
| 11   | SRANDMEMBER key [count\]                        | 返回集合中一个或多个随机数                          |
| 12   | SREM key member1 [member2\]                     | 移除集合中一个或多个成员                            |
| 13   | SUNION key1 [key2\]                             | 返回所有给定集合的并集                              |
| 14   | SUNIONSTORE destination key1 [key2\]            | 所有给定集合的并集存储在 destination 集合中         |
| 15   | SSCAN key cursor [MATCH pattern\] [COUNT count] | 迭代集合中的元素                                    |

## 九、redis有序集合

Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

```c
127.0.0.1:6379> ZADD myZset 0 "liu" 1 "kai" 2 "chuan"
(integer) 3
127.0.0.1:6379> ZRANGEBYSCORE myZset 0 10
1) "liu"
2) "kai"
3) "chuan"
127.0.0.1:6379> ZADD myZset 0 "liuliu" 0 "liukaichuan"
(integer) 2
127.0.0.1:6379> ZRANGEBYSCORE myZset 0 10
1) "liu"
2) "liukaichuan"
3) "liuliu"
4) "kai"
5) "chuan"
```

Redis 有序集合命令

下表列出了 redis 有序集合的基本命令:

| 序号 | 命令及描述                                      |                                                              |
| :--- | :---------------------------------------------- | ------------------------------------------------------------ |
| 1    | ZADD key score1 member1 [score2 member2\]       | 向有序集合添加一个或多个成员，或者更新已存在成员的分数       |
| 2    | ZCARD key                                       | 获取有序集合的成员数                                         |
| 3    | ZCOUNT key min max                              | 计算在有序集合中指定区间分数的成员数                         |
| 4    | ZINCRBY key increment member                    | 有序集合中对指定成员的分数加上增量 increment                 |
| 5    | ZINTERSTORE destination numkeys key [key ...\]  | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中 |
| 6    | ZLEXCOUNT key min max                           | 在有序集合中计算指定字典区间内成员数量                       |
| 7    | ZRANGE key start stop [WITHSCORES\]             | 通过索引区间返回有序集合成指定区间内的成员                   |
| 8    | ZRANGEBYLEX key min max [LIMIT offset count\]   | 通过字典区间返回有序集合的成员                               |
| 9    | ZRANGEBYSCORE key min max [WITHSCORES\] [LIMIT] | 通过分数返回有序集合指定区间内的成员                         |
| 10   | ZRANK key member                                | 返回有序集合中指定成员的索引                                 |
| 11   | ZREM key member [member ...\]                   | 移除有序集合中的一个或多个成员                               |
| 12   | ZREMRANGEBYLEX key min max                      | 移除有序集合中给定的字典区间的所有成员                       |
| 13   | ZREMRANGEBYRANK key start stop                  | 移除有序集合中给定的排名区间的所有成员                       |
| 14   | ZREMRANGEBYSCORE key min max                    | 移除有序集合中给定的分数区间的所有成员                       |
| 15   | ZREVRANGE key start stop [WITHSCORES\]          | 返回有序集中指定区间内的成员，通过索引，分数从高到底         |
| 16   | ZREVRANGEBYSCORE key max min [WITHSCORES\]      | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
| 17   | ZREVRANK key member                             | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| 18   | ZSCORE key member                               | 返回有序集中，成员的分数值                                   |
| 19   | ZUNIONSTORE destination numkeys key [key ...\]  | 计算给定的一个或多个有序集的并集，并存储在新的 key 中        |
| 20   | ZSCAN key cursor [MATCH pattern\] [COUNT count] | 迭代有序集合中的元素（包括元素成员和元素分值）               |

## 十、Redis HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

### 什么是基数

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

如：

```c
127.0.0.1:6379> PFADD person "l"
(integer) 1
127.0.0.1:6379> PFADD person "i"
(integer) 1
127.0.0.1:6379> PFADD person "u"
(integer) 1
127.0.0.1:6379> PFADD person "k"
(integer) 1
127.0.0.1:6379> PFADD person "a"
(integer) 1
127.0.0.1:6379> PFCOUNT person
(integer) 5
127.0.0.1:6379> PFADD person "i"
(integer) 0
127.0.0.1:6379> PFADD person "c"
(integer) 1
127.0.0.1:6379> PFADD person "h"
(integer) 1
127.0.0.1:6379> PFADD person "u"
(integer) 0
127.0.0.1:6379> PFADD person "a"
(integer) 0
127.0.0.1:6379> PFADD person "n"
(integer) 1
127.0.0.1:6379> PFCOUNT person
(integer) 8
```

### Redis HyperLogLog 命令

下表列出了 redis HyperLogLog 的基本命令：

| 序号 | 命令                                       | 描述                                      |
| :--- | :----------------------------------------- | ----------------------------------------- |
| 1    | PFADD key element [element ...\]           | 添加指定元素到 HyperLogLog 中。           |
| 2    | PFCOUNT key [key ...\]                     | 返回给定 HyperLogLog 的基数估算值。       |
| 3    | PFMERGE destkey sourcekey [sourcekey ...\] | 将多个 HyperLogLog 合并为一个 HyperLogLog |

## 十一、redis发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 客户端可以订阅任意数量的频道。

![img](redis学习笔记.assets/pubsub1.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](redis学习笔记.assets/pubsub2.png)

```c
127.0.0.1:6379> SUBSCRIBE FM
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "FM"
3) (integer) 1
```

```c
127.0.0.1:6379> PUBLISH FM "this is a test"
(integer) 1
127.0.0.1:6379> 

// 订阅者收到消息如下
1) "message"
2) "FM"
3) "this is a test"
127.0.0.1:6379> PUBLISH FM "this is a test"
(integer) 1
127.0.0.1:6379> 
```

Redis 发布订阅命令

下表列出了 redis 发布订阅常用命令：

| 序号 | 命令                                         | 描述                               |
| :--- | :------------------------------------------- | ---------------------------------- |
| 1    | PSUBSCRIBE pattern [pattern ...\]            | 订阅一个或多个符合给定模式的频道。 |
| 2    | PUBSUB subcommand [argument [argument ...\]] | 查看订阅与发布系统状态。           |
| 3    | PUBLISH channel message                      | 将信息发送到指定的频道。           |
| 4    | PUNSUBSCRIBE [pattern [pattern ...\]]        | 退订所有给定模式的频道。           |
| 5    | SUBSCRIBE channel [channel ...\]             | 订阅给定的一个或多个频道的信息。   |
| 6    | UNSUBSCRIBE [channel [channel ...\]]         | 指退订给定的频道。                 |

## 十二、redis事物

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

```c
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET book_name "Thinking in Java"
QUEUED
127.0.0.1:6379> GET book_name
QUEUED
127.0.0.1:6379> LPUSH readers "xiaoming" "xiaohong" "xiaojun"
QUEUED
127.0.0.1:6379> LRANGE readers 0 10
QUEUED
127.0.0.1:6379> EXEC 
1) OK
2) "Thinking in Java"
3) (integer) 3
4) 1) "xiaojun"
   2) "xiaohong"
   3) "xiaoming"
```

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。

事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

Redis 事务命令

下表列出了 redis 事务的相关命令：

| 序号 | 命令                 | 描述                                                         |
| :--- | :------------------- | ------------------------------------------------------------ |
| 1    | DISCARD              | 取消事务，放弃执行事务块内的所有命令。                       |
| 2    | EXEC                 | 执行所有事务块内的命令。                                     |
| 3    | MULTI                | 标记一个事务块的开始。                                       |
| 4    | UNWATCH              | 取消 WATCH 命令对所有 key 的监视。                           |
| 5    | WATCH key [key ...\] | 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 |

## 十三、脚本

## 十四、redis连接

Redis 连接命令主要是用于连接 redis 服务。

```c
127.0.0.1:6379> AUTH P@ssw0rd
OK
127.0.0.1:6379> ping
PONG
```

Redis 连接命令

下表列出了 redis 连接的基本命令：

| 序号 | 命令          | 描述               |
| :--- | :------------ | ------------------ |
| 1    | AUTH password | 验证密码是否正确   |
| 2    | ECHO message  | 打印字符串         |
| 3    | PING          | 查看服务是否运行   |
| 4    | QUIT          | 关闭当前连接       |
| 5    | SELECT index  | 切换到指定的数据库 |

## 十五、服务器

Redis 服务器命令主要是用于管理 redis 服务。

Redis 服务器命令：

下表列出了 redis 服务器的相关命令:

| 序号 | 命令及描述                                                   |                                                              |
| :--- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 1    | BGREWRITEAOF                                                 | 异步执行一个 AOF（AppendOnly File） 文件重写操作             |
| 2    | BGSAVE                                                       | 在后台异步保存当前数据库的数据到磁盘                         |
| 3    | CLIENT KILL [ip:port\] [ID client-id]                        | 关闭客户端连接                                               |
| 4    | CLIENT LIST                                                  | 获取连接到服务器的客户端连接列表                             |
| 5    | CLIENT GETNAME                                               | 获取连接的名称                                               |
| 6    | [CLIENT PAUSE timeout](https://www.runoob.com/redis/server-client-pause.html) | 在指定时间内终止运行来自客户端的命令                         |
| 7    | [CLIENT SETNAME connection-name](https://www.runoob.com/redis/server-client-setname.html) | 设置当前连接的名称                                           |
| 8    | [CLUSTER SLOTS](https://www.runoob.com/redis/server-cluster-slots.html) | 获取集群节点的映射数组                                       |
| 9    | [COMMAND](https://www.runoob.com/redis/server-command.html)  | 获取 Redis 命令详情数组                                      |
| 10   | [COMMAND COUNT](https://www.runoob.com/redis/server-command-count.html) | 获取 Redis 命令总数                                          |
| 11   | [COMMAND GETKEYS](https://www.runoob.com/redis/server-command-getkeys.html) | 获取给定命令的所有键                                         |
| 12   | [TIME](https://www.runoob.com/redis/server-time.html)        | 返回当前服务器时间                                           |
| 13   | [COMMAND INFO command-name [command-name ...\]](https://www.runoob.com/redis/server-command-info.html) | 获取指定 Redis 命令描述的数组                                |
| 14   | [CONFIG GET parameter](https://www.runoob.com/redis/server-config-get.html) | 获取指定配置参数的值                                         |
| 15   | [CONFIG REWRITE](https://www.runoob.com/redis/server-config-rewrite.html) | 对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写    |
| 16   | [CONFIG SET parameter value](https://www.runoob.com/redis/server-config-set.html) | 修改 redis 配置参数，无需重启                                |
| 17   | [CONFIG RESETSTAT](https://www.runoob.com/redis/server-config-resetstat.html) | 重置 INFO 命令中的某些统计数据                               |
| 18   | [DBSIZE](https://www.runoob.com/redis/server-dbsize.html)    | 返回当前数据库的 key 的数量                                  |
| 19   | [DEBUG OBJECT key](https://www.runoob.com/redis/server-debug-object.html) | 获取 key 的调试信息                                          |
| 20   | [DEBUG SEGFAULT](https://www.runoob.com/redis/server-debug-segfault.html) | 让 Redis 服务崩溃                                            |
| 21   | [FLUSHALL](https://www.runoob.com/redis/server-flushall.html) | 删除所有数据库的所有key                                      |
| 22   | [FLUSHDB](https://www.runoob.com/redis/server-flushdb.html)  | 删除当前数据库的所有key                                      |
| 23   | [INFO [section\]](https://www.runoob.com/redis/server-info.html) | 获取 Redis 服务器的各种信息和统计数值                        |
| 24   | [LASTSAVE](https://www.runoob.com/redis/server-lastsave.html) | 返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示 |
| 25   | [MONITOR](https://www.runoob.com/redis/server-monitor.html)  | 实时打印出 Redis 服务器接收到的命令，调试用                  |
| 26   | [ROLE](https://www.runoob.com/redis/server-role.html)        | 返回主从实例所属的角色                                       |
| 27   | [SAVE](https://www.runoob.com/redis/server-save.html)        | 同步保存数据到硬盘                                           |
| 28   | [SHUTDOWN [NOSAVE\] [SAVE]](https://www.runoob.com/redis/server-shutdown.html) | 异步保存数据到硬盘，并关闭服务器                             |
| 29   | [SLAVEOF host port](https://www.runoob.com/redis/server-slaveof.html) | 将当前服务器转变为指定服务器的从属服务器(slave server)       |
| 30   | [SLOWLOG subcommand [argument\]](https://www.runoob.com/redis/server-showlog.html) | 管理 redis 的慢日志                                          |
| 31   | [SYNC](https://www.runoob.com/redis/server-sync.html)        | 用于复制功能(replication)的内部命令                          |

## 十六、redis数据备份与恢复

Redis **SAVE** 命令用于创建当前数据库的备份。

redis Save 命令基本语法如下：

```c
127.0.0.1:6379> SAVE 
```

该命令将在 redis 安装目录中创建dump.rdb文件。

恢复数据

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 **CONFIG**

```c
127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
```

创建 redis 备份文件也可以使用命令 **BGSAVE**，该命令在后台执行。

```
127.0.0.1:6379> BGSAVE

Background saving started
```

## 十七、redis安全

我们可以通过 redis 的配置文件设置密码参数，这样客户端连接到 redis 服务就需要密码验证，这样可以让你的 redis 服务更安全。

通过以下命令查看是否设置了密码验证：

```c
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) "P@ssw0rd"
```

默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。

你可以通过以下命令来修改该参数：

```c
127.0.0.1:6379> CONFIG set requirepass "runoob"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "runoob"
```

## 十八、redis性能测试



Redis 性能测试是通过同时执行多个命令实现的。

redis 性能测试的基本命令如下：

```c
redis-benchmark [option] [option value]
```

**注意**：该命令是在 redis 的目录下执行的，而不是 redis 客户端的内部指令。

redis 性能测试工具可选参数如下所示：

| 序号 | 选项      | 描述                                       | 默认值    |
| :--- | :-------- | :----------------------------------------- | :-------- |
| 1    | **-h**    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**    | 指定服务器端口                             | 6379      |
| 3    | **-s**    | 指定服务器 socket                          |           |
| 4    | **-c**    | 指定并发连接数                             | 50        |
| 5    | **-n**    | 指定请求数                                 | 10000     |
| 6    | **-d**    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**    | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **--csv** | 以 CSV 格式输出                            |           |
| 12   | **-l**    | 生成循环，永久执行测试                     |           |
| 13   | **-t**    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | **-I**    | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

## 十九、redis客户端连接

Redis 通过监听一个 TCP 端口或者 Unix socket 的方式来接收来自客户端的连接，当一个连接建立后，Redis 内部会进行以下一些操作：

- 首先，客户端 socket 会被设置为非阻塞模式，因为 Redis 在网络事件处理上采用的是非阻塞多路复用模型。
- 然后为这个 socket 设置 TCP_NODELAY 属性，禁用 Nagle 算法
- 然后创建一个可读的文件事件用于监听这个客户端 socket 的数据发送

最大连接数：

在 Redis2.4 中，最大连接数是被直接硬编码在代码里面的，而在2.6版本中这个值变成可配置的。

maxclients 的默认值是 10000，你也可以在 redis.conf 中对这个值进行修改。

客户端命令

| S.N. | 命令               | 描述                                       |
| :--- | :----------------- | :----------------------------------------- |
| 1    | **CLIENT LIST**    | 返回连接到 redis 服务的客户端列表          |
| 2    | **CLIENT SETNAME** | 设置当前连接的名称                         |
| 3    | **CLIENT GETNAME** | 获取通过 CLIENT SETNAME 命令设置的服务名称 |
| 4    | **CLIENT PAUSE**   | 挂起客户端连接，指定挂起的时间以毫秒计     |
| 5    | **CLIENT KILL**    | 关闭客户端连接                             |

## 二十、redis管道技术

Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。这意味着通常情况下一个请求会遵循以下步骤：

- 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
- 服务端处理命令，并将结果返回给客户端。

Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

## 二十一、redis分区

分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存key的一个子集。

分区的优势：

- 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
- 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

分区的不足：

redis的一些特性在分区方面表现的不是很好：

- 涉及多个key的操作通常是不被支持的。举例来说，当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作。
- 涉及多个key的redis事务不能使用。
- 当使用分区时，数据处理较为复杂，比如你需要处理多个rdb/aof文件，并且从多个实例和主机备份持久化文件。
- 增加或删除容量也比较复杂。redis集群大多数支持在运行时增加、删除节点的透明数据平衡的能力，但是类似于客户端分区、代理等其他系统则不支持这项特性。然而，一种叫做presharding的技术对此是有帮助的。

#### 分区类型

Redis 有两种类型分区。 假设有4个Redis实例 R0，R1，R2，R3，和类似user:1，user:2这样的表示用户的多个key，对既定的key有多种不同方式来选择这个key存放在哪个实例中。也就是说，有不同的系统来映射某个key到某个Redis服务。

#### 范围分区

最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。

比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。

这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。

#### 哈希分区

另外一种分区方法是hash分区。这对任何key都适用，也无需是object_name:这种形式，像下面描述的一样简单：

- 用一个hash函数将key转换为一个数字，比如使用crc32 hash函数。对key foobar执行crc32(foobar)会输出类似93024922的整数。
- 对这个整数取模，将其转化为0-3之间的数字，就可以将这个整数映射到4个Redis实例中的一个了。93024922 % 4 = 2，就是说key foobar应该被存到R2实例中。注意：取模操作是取除的余数，通常在多种编程语言中用%操作符实现。

## 二十二、Java使用redis

需要下载驱动包 [**下载 jedis.jar**](https://mvnrepository.com/artifact/redis.clients/jedis)，确保下载最新驱动包。在classpath 中包含该驱动包。

```java
import redis.clients.jedis.Jedis;
 
public class RedisJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
    }
}
```

Redis Java List(列表) 实例

```java
import java.util.List;
import redis.clients.jedis.Jedis;
 
public class RedisListJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //存储数据到列表中
        jedis.lpush("site-list", "Runoob");
        jedis.lpush("site-list", "Google");
        jedis.lpush("site-list", "Taobao");
        // 获取存储的数据并输出
        List<String> list = jedis.lrange("site-list", 0 ,2);
        for(int i=0; i<list.size(); i++) {
            System.out.println("列表项为: "+list.get(i));
        }
    }
}
```



编译以上程序。

```
连接成功
列表项为: Taobao
列表项为: Google
列表项为: Runoob
```

------

Redis Java Keys 实例

```java
import java.util.Iterator;
import java.util.Set;
import redis.clients.jedis.Jedis;
 
public class RedisKeyJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
 
        // 获取数据并输出
        Set<String> keys = jedis.keys("*"); 
        Iterator<String> it=keys.iterator() ;   
        while(it.hasNext()){   
            String key = it.next();   
            System.out.println(key);   
        }
    }
}
```



编译以上程序。

```
连接成功
runoobkey
site-list
```