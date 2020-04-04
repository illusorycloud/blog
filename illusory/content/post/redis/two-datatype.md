---
title: "Redis入门教程(二)---五大基础数据类型与常用命令"
description: "redis五大基础数据类型及相关命令介绍"
date: 2019-02-23 22:00:00
draft: false
tags: ["Redis"]
categories: ["Redis"]
---

本文主要记录了Redis五大基础数据类型与key命令，包括了`String`、`Hash`、`List`、`Set`、`ZSet`。

<!--more-->

> **[Redis系列教程目录](https://www.lixueduan.com/categories/Redis/)**
>
> [Redis入门教程(一)---安装与配置](https://www.lixueduan.com/posts/876962d5.html)
>
> [Redis入门教程(二)---五大基础数据类型与常用命令](https://www.lixueduan.com/posts/8380a4fa.html)
>
> [Redis入门教程(三)---安全性、事务、发布订阅](https://www.lixueduan.com/posts/5df38113.html)
>
> [Redis入门教程(四)---主从复制与持久化](https://www.lixueduan.com/posts/84dd6d72.html)
>
> [Redis入门教程(五)---搭建Redis集群](https://www.lixueduan.com/posts/397ed67.html)
>
> [Redis入门教程(六)---通过JavaApi(Jedis)操作Redis](https://www.lixueduan.com/posts/e456b1e5.html)
>
> [Redis入门教程(七)---通过 Docker 安装 Redis](https://www.lixueduan.com/posts/96375af.html)
>
> .......
>
> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)



Redis一共分5种数据类型：`String`、`Hash`、`List`、`Set`、`ZSet`

## 1. String

### 1.1 set

| 命令                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| **set**  key value                 | 设置指定 key 的值                                            |
| **setnx** key value                | 只有在 key 不存在时设置 key 的值。(not exist)                |
| **setex** key seconds valuel       | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (秒)。expired |
| psetex key milliseconds value]     | 和setex命令相似，但它以毫秒为单位设置 key 的生存时间         |
| **mset** key value [key value ...] | 同时设置一个或多个 key-value 对。                            |
| msetnx key value [key value ...]   | 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。 |

### 1.2 get

| 命令                   | 描述                                |
| ---------------------- | ----------------------------------- |
| **get** key            | 获取指定 key 的值。                 |
| **mget** key1 [key2..] | 获取所有(一个或多个)给定 key 的值。 |
| getrange key start end | 返回 key 中字符串值的子字符         |
| strlen key             | 返回 key 所储存的字符串值的长度。   |

### 1.3 update

| 命令                      | 描述                                                         |
| :------------------------ | ------------------------------------------------------------ |
| getset key value          | 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。   |
| setrange key offset value | 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。 |
| **incr** key              | 将 key 中储存的数字值一。                                    |
| **incrby** key increment  | 将 key 所储存的值加上给定的增量值（increment） 。            |
| incrbyfloat key increment | 将 key 所储存的值加上给定的浮点增量值（increment） 。        |
| **decr** key              | 将 key 中储存的数字值减一。                                  |
| **decrby** key decrement  | key 所储存的值减去给定的减量值（decrement） 。               |
| append key value          | 追加字符串到key末尾                                          |

### 1.4 实例

```java
设置值：set key value-->set myname illusory   //同一个key多次set会覆盖
获取值：get key  ------>get myname
删除值：del key-------->del myname
```

其他set方法：

`setnx(not exist)`: 如果key不存在就设置值，返回1;存在就不设置，返回0；

```shell
#格式：setnx key value
127.0.0.1:6379> set myname illusory
OK
127.0.0.1:6379> setnx myname cloud   #myname 已经存在了 返回0
(integer) 0
127.0.0.1:6379> get myname  # 值也没有发生变化
"illusory"
```

`setex(expired)`: 设置数据过期时间，数据只存在一段时间

```shell
#格式：setnx key seconds value;
setnx vercode 60 123456； 
#设置key-->vercode有效时间60s，60s内获取值为123456,60s后返回nil（Redis中nil表示空）
```

```shell
127.0.0.1:6379> setex vercode 5 123456
OK
127.0.0.1:6379> get vercode #时间没到 还能查询到
"123456"
127.0.0.1:6379> get vercode  #5s到了 数据过期 查询返回nil
(nil)
```

`setrange`：替换字符串

```shell
#格式：setrange key offset value
set email 123456789@gmail.com
setrange email 10 qqqqq # 从第10位开始替换(不包括第10位) 后面跟上用来替换的字符串

```

```shell
127.0.0.1:6379> set email 123456789@gmail.com
OK
127.0.0.1:6379> get email
"123456789@qqail.com"
127.0.0.1:6379> setrange email 10 qqqqq
(integer) 19
127.0.0.1:6379> get email
"123456789@qqqqq.com"
```

mset：一次设置多个值

`mset key1 value1 key2 value2 ...keyn valuen`

mget：一次获取多个值

`mget key1 key2 key3...keyn`

```shell
127.0.0.1:6379> mset k1 111 k2 222 k3 333 
OK
127.0.0.1:6379> mget k1 k2 k3
1) "111"
2) "222"
3) "333"

```

`getset`: 返回旧值并设置新值

```shell
#格式 getset key value
getset name cloud #将name设置为cloud并放回name的旧值
```

```shell
127.0.0.1:6379> set name illusory
OK
127.0.0.1:6379> get name
"illusory"
127.0.0.1:6379> getset name cloud
"illusory"
127.0.0.1:6379> get name
"cloud"
```

`incr/decr`:对一个值进行递增或者递减操作。

```shell
# 格式 incr key/decr key
incr age #age递增1
decr age #age递减1
```

```shell
127.0.0.1:6379> get age
"22"
127.0.0.1:6379> incr age #递增
(integer) 23
127.0.0.1:6379> get age
"23"
127.0.0.1:6379> decr age #递减
(integer) 22
127.0.0.1:6379> get age
"22"
```

`incrby/decrby`:对一个值按照一定`步长`进行递增或者递减操作。

```shell
# 格式 incrby key increment/decrby key increment
incrby age 3 #age递增3
decrby age 3 #age递减3
```

```shell
127.0.0.1:6379> get age
"22"
127.0.0.1:6379> incrby age 3
(integer) 25
127.0.0.1:6379> get age
"25"
127.0.0.1:6379> decrby age 3
(integer) 22
127.0.0.1:6379> get age
"22"

```

`append`:字符串追加

```shell
#格式 append key value
append name cloud #在name后追加cloud
```

```shell
127.0.0.1:6379> get name
"illusory"
127.0.0.1:6379> append name cloud
(integer) 13
127.0.0.1:6379> get name
"illusorycloud"
```

`strlen`：获取字符串长度

```shell
#格式 strlen key 
strlen name #获取name对应的value的长度
```

```shell
127.0.0.1:6379> get name
"illusorycloud"
127.0.0.1:6379> strlen name
(integer) 13
```

## 2. Hash

工作中使用最多的就是Hash类型

将一个对象存储在Hash类型里要比String类型里占用的空间少一些，并方便存取整个对象。

`hset`:类似于set，数据都存为Hash类型，类似于存在map中

```shell
# 格式 hset key filed value
hset me name illusory #me是hash名 name是hash中的key illusory为hash中的value 

#类似于Java中的Map
        Map<Object,Object> me = new HashMap<>();
        me.put("name", "illusory");
```



### 2.1 hset

| 命令                                         | 描述                                                      |
| -------------------------------------------- | --------------------------------------------------------- |
| **hset** key filed value                     | 将哈希表 key 中的字段 field 的值设为 value 。             |
| **hsetnx** key filed value                   | 类似setnx,只有在字段 field 不存在时，设置哈希表字段的值。 |
| **hmset** key filed1 value1[filed2 value2..] | 同时将多个 field-value (域-值)对设置到哈希表 key 中。     |
|                                              |                                                           |



### 2.2 hget

| 命令                                         | 描述                                      |
| -------------------------------------------- | ----------------------------------------- |
| **hget** key filed                           | 获取存储在哈希表中指定字段的值。          |
| **hmget** key filed1 [filed2...]             | 获取所有给定字段的值                      |
| **hgetall** key                              | 获取在哈希表中指定 key 的所有字段和值     |
| **hkeys** key                                | 获取所有哈希表中的字段                    |
| **hvals** key                                | 获取哈希表中所有值                        |
| hscankey cursor [MATCH pattern][COUNT count] | 迭代哈希表中的键值对。                    |
| hexists key filed                            | 查看哈希表 key 中，指定的字段是否存在。   |
| hlen key                                     | 获取哈希表中字段的数量                    |
| hstrlen key filed                            | 返回哈希表 key 中， 给定field的字符串长度 |



### 2.3 update

| 命令                             | 描述                                                     |
| -------------------------------- | -------------------------------------------------------- |
| **hincrby** key field increment  | 为哈希表 key 中的指定字段的整数值加上增量 increment 。   |
| hincrbyfloat key field increment | 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。 |
| **hdel** key                     | 删除哈希表key 中的一个或多个指定域，不存在的域将被忽略。 |



### 2.4 实例



`hset`:类似于set，数据都存为Hash类型，类似于存在map中

`hget`:类似于get

```shell
# 格式 hget hash filed
hget me name #获取hash名为me的hash中的name对应的value
```

```shell
127.0.0.1:6379> hset me name illusory
(integer) 1
127.0.0.1:6379> hset me age 22
(integer) 1
127.0.0.1:6379> hget me name
"illusory"
127.0.0.1:6379> hget me age
"22"

```

同样也有批量操作的`hmset`、`hmget`

```shell
#格式 hmset key filed1 value1 filde2 value2 ....filedn valuen
#格式 hmget key filed1  filde2....filedn 
```

```shell
127.0.0.1:6379> hmset me name illusory age 22
OK
127.0.0.1:6379> hmget me name age
1) "illusory"
2) "22"

```

`hsetnx(not exist)`: 如果key不存在就设置值，返回1;存在就不设置，返回0；

```shell
#格式 hsetnx value filed value
```

`hincrby/hdecrby`:对一个值按照一定`步长`进行递增或者递减操作。

```shell
# 格式 hincrby key filed increment/hdecrby key filed increment
incrby me age 3 #age递增3
decrby me age 3 #age递减3
```

`hstrlen key filed`:回哈希表 `key` 中， 与给定域 `field` 相关联的值的字符串长度（string length）。

如果给定的键或者域不存在， 那么命令返回 `0` 。



`hexists`:判断是否存在

```shell
#格式 hexists value filed
```

`hlen`:查看hash的filed数

```shell
#格式 hlen key
```

`hdel`:删除指定hash中的filed

```shell
#格式 hdel key filed
```

`hkeys`:返回指定hash中所有的filed

```shell
#格式 hkeys key 
```

`hvals`:返回指定hash中所有的value

```shell
#格式 hvals key 
```

`hgetall`:返回指定hash中所有的filed和value

```shell
#格式 hgetall key
```

```shell
127.0.0.1:6379> hgetall me
1) "name"
2) "illusory"
3) "age"
4) "23"

```

## 3. List

可以看做Java中的List，不过更像Queue。

### 3.1 lpush/rpush

| 命令                                  | 描述                                                         |
| :------------------------------------ | :----------------------------------------------------------- |
| **lpush** key value1 [value2..]       | 将一个或多个值插入到列表**头部**                             |
| lpushx key value                      | 将一个值插入到已存在的列表**头部**                           |
| **rpush** key value1 [value2..]       | 将一个或多个值插入到列表**尾部**                             |
| rpushx key value                      | 将一个值插入到已存在的列表**尾部**                           |
| **lset** key index value              | 将列表 `key` 下标为 `index` 的元素的值设置为 `value` 。      |
| linsert key BEFORE\|AFTER pivot value | 将值 `value` 插入到列表 `key` 当中，位于值 `pivot` 之前或之后。 |



### 3.2 lpop/rpop

| 命令                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| **lpop** key                          | 移除并获取列表的第一个元素                                   |
| blpop key [key …] timeout             | lpop的阻塞版本，没有元素可供弹出的时候会阻塞直到有元素或超时。 |
| **rpop** key                          | 移除并获取列表的倒数第一个元素                               |
| brpop key [key …] timeout             | rpop的阻塞版本，没有元素可供弹出的时候会阻塞直到有元素或超时。 |
| rpoplpush source destination          | 移除列表的最后一个元素，并将该元素添加到另一个列表并返回     |
| brpoplpush source destination timeout | rpoplpush的阻塞版，source为空时阻塞直到不为空或者超时。      |
| lrem key count value                  | 移除列表元素                                                 |
| **lrange** key start stop             | 返回列表 `key` 中指定区间内的元素，区间以偏移量 `start` 和 `stop` 指定。 |
| ltrim key start stop                  | 对一个列表进行修剪(trim)，只保留指定区间内的元素。           |
| llen key                              | 返回列表 key的长度。                                         |
| lindex key index                      | 返回列表 key 中，下标为 index的元素。                        |

## 4. Set

`Set`集合是String类型的`无序`集合，通过hashtable实现的，对集合我们可以取交集，并集，差集。

Java中List的升级版。

### 4.1 sadd/spop

| 命令                                | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| **sadd** key member [member …]      | 将一个或多个 `member` 元素加入到集合 `key` 当中，已经存在于集合的 `member` 元素将被忽略 |
| **spop** key                        | 移除并返回集合中的一个随机元素。                             |
| srem key member [member …]          | 移除集合 `key` 中的一个或多个 `member` 元素，不存在的 `member` 元素会被忽略。 |
| **smove** source destination member | 将 `member` 元素从 `source` 集合移动到 `destination` 集合。  |

### 4.2 get

| 命令                                | 描述                                            |
| ----------------------------------- | ----------------------------------------------- |
| **sinter** key [key …]              | 返回给定所有集合的**交集**                      |
| sinterstore destination key [key …] | 返回给定所有集合的交集并存储在 destination 中   |
| **sunion** key [key …]              | 返回所有给定集合的**并集**                      |
| sunionstore destination key [key …] | 返回所有给定集合的并集存储在 destination 集合中 |
| **sdiff**key [key …]                | 返回给定所有集合的**差集**                      |
| sdiffstore destination key [key …]  | 返回给定所有集合的差集并存储在 destination 中   |
| **smembers** key                    | 返回集合 `key` 中的所有成员。                   |
| sismember key member                | 判断 `member` 元素是否集合 `key` 的成员。       |
| **scard** key                       | 返回集合 `key` 的基数(集合中元素的数量)         |
| **srandmember** key [count]         | 返回集合中一个或多个随机数                      |



## 5. ZSet

`ZSet`则是`有序`的。



### 5.1 zadd

| 命令                                        | 描述                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| zadd key score1 member1 [score2 member2...] | 将一个或多个 `member` 元素及其 `score` 值加入到有序集 `key` 当中。 |
| **zincrby** key increment member            | 为有序集 `key` 的成员 `member` 的 `score` 值加上增量 `increment` 。 |



### 5.2 get

| 命令                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| zscore key member                                            | 返回有序集 `key` 中，成员 `member` 的 `score` 值。           |
| zcard key                                                    | 返回有序集 `key` 的基数。(集合中元素的数量)                  |
| zcount key min max                                           | 返回有序集 `key` 中， `score` 值在 `min` 和 `max` 之间(默认包括 `score` 值等于 `min` 或 `max` )的成员的数量。 |
| **zrange** key start stop [withscores]                       | 返回有序集 `key` 中，指定区间内的成员。其中成员的位置按 `score` 值递增(**从小到大**)来排序。 |
| zrevrange key start stop [withscores]                        | 类似zrange,不过成员位置按 `score` 值递增(**从大到小**)来排序。 |
| **zrangebyscore** key min max [withscores][LIMIT offset count] | 返回有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员。有序集成员按 `score` 值递增(**从小到大**)次序排列。 |
| zrevrangebyscore key min max [withscores][LIMIT offset count] | 类似zrangebyscore，不过有序集成员按 `score` 值递增(**从小到大**)次序排列。 |
| **zrank** key member                                         | 返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递增(**从小到大**)顺序排列。 |
| zrevrank key member                                          | 类似zrank，其中有序集成员按 `score` 值递增(**从大到小**)顺序排列。 |
| zrangebylex key min max [LIMIT offset count]                 | 通过字典区间返回有序集合的成员                               |
| zlexcount key min max                                        | 返回该集合中， 成员介于 `min` 和 `max` 范围内的元素数量。    |
| **zuniostore** destination numkeys key [key ...]             | 计算给定的一个或多个有序集的**并集**，并存储在新的 key 中    |
| **zinterstore** destination numkeys key [key ...]            | 计算给定的一个或多个有序集的**交集**，并存储在新的 key 中    |

### 5.3 delete

| 命令                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| zrem key member [member …]     | 移除有序集 `key` 中的一个或多个成员，不存在的成员将被忽略。  |
| zremrangebyrank key start stop | 移除有序集 `key` 中，指定排名(rank)区间内的所有成员。        |
| zremrangebyscore key min max   | 移除有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员 |
| zremrangebylex key min max     | 移除该集合中， 成员介于 `min` 和 `max` 范围内的所有元素。    |

## 6. Key命令

Redis中所有key都可以使用的命令。

| 命令                     | 描述                                                         |
| ------------------------ | :----------------------------------------------------------- |
| **del** key              | 该命令用于在 key 存在时删除 key。                            |
| dump key                 | 序列化给定 key ，并返回被序列化的值。                        |
| **exists** key           | 检查给定 key 是否存在。                                      |
| **expire** key seconds   | 为给定 key 设置过期时间，以秒计。                            |
| expireat key timestamp   | 类似expirre，都用于为 key 设置过期时间。 不同在于 expireat命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
| pexpire key milliseconds | 设置 key 的过期时间以毫秒计。                                |
| pexpireat key timestamp  | 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计           |
| **keys** pattern         | 类似模糊查询，查找所有符合给定模式( pattern)的 key。key l 查出以l开头的 |
| move key db              | 将当前数据库的 key 移动到给定的数据库 db(0~15) 当中。        |

## 7. 其他命令

| 命令                | 描述                                   |
| ------------------- | -------------------------------------- |
| keys *              | 返回满足的所有键 (可以模糊匹配）       |
| exists key[keys...] | 是否存在指定的key                      |
| expire key secondes | 设置某个key的过期时间，                |
| ttl key             | 与expire搭配，查看剩余时间             |
| persist key         | 取消过期时间                           |
| select db(0~15)     | 择数据库数据库,默认进入的是0数据库     |
| move key db(0~15)   | 将当前数据中的key转移到其他数据库中    |
| randomkey           | 随机返回数据库里的一个key              |
| rename key          | 重命名key                              |
| echo                | 打印命令                               |
| dbsize              | 查看数据库的key数量                    |
| info                | 获取数据库信息                         |
| conflg get          | 实时传储收到的请求(返回相关的配置信息} |
| config get *        | 返回所有配置                           |
| flushdb             | 空当前数据库                           |
| flushall            | 清空所有数据库                         |

## 8. 参考

`http://redisdoc.com/internal/index.html`

`http://www.runoob.com/redis/redis-hashes.html`