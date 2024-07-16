
## 通用操作命令

### 基础命令
删除 KEY：
```
DEL key [key ...]：删除指定的 key，不关心 key 是否存在以及 key 对应的类型。
```

是否存在指定的 KEY：
```
EXISTS key [key ...]：判断指定的 key 是否存在，返回存在 key 的数量。如果指定了多次同样的 key，则返回结果会计数多次。
```

查询 KEY：
```
KEYS pattern：返回匹配模式的所有 key。
```

查询 KEY 的类型：
```
TYPE key：返回 key 的类型。
```

清空所有的 KEY：
```
FLUSHDB：删除当前数据库中所有的 key。

FLUSHALL：删除所有数据库的 key。
```
### 过期时间
Redis 存储的数据，如果有些数据在某个时间点之后就不再使用了，可以使用 `DEL <key_name>`  显示地删除这些无用的数据，也可以通过 Redis 的过期时间特性来让一个键在给定的时限之后自动删除。

对于列表、集合、散列和有序集合这样的容器类型的数据结构来说，键的过期命令只能为整个键设置过期时间，而没有办法为键里的单个元素设置过期时间。**补充：不一定，要看数据类型和版本。**

过期时间相关命令：
```
PERSIST key-name：移除键的过期时间

TTL key-name：查看给定的键距离过期还有多少秒（time-to-live，ttl）

EXPIRE key-name seconds：设置给定的键在指定的秒数之后过期

EXPIREAT key-name timestamp：将给定的键的过期时间设置为给定的 UNIX 时间戳

PTTL key-nname：查看给定的键距离过期还有多少毫秒

PEXPIRE key-name milliseconds：让给定的键在指定的毫秒数之后过期

PEXPIREAT key-name timestamp-millseconds：将一个毫秒级经度的 UNIX 时间戳设置为给定键的过期时间
```

## 字符串

字符串可以存储以下 3 钟类型的值。  

- 字节串 byte string。  
- 整数。  
- 浮点数。

核心 `SET` 命令语法
```
SET key value [NX | XX] [GET] [EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL]
```

- `EX seconds` 设置过期时间，正整数，单位秒。

- `PX milliseconds` 设置过期时间，正整数，单位毫秒。

- `EXAT timestamp-seconds` 设置 key 在某个时间点到期，秒级时间戳。

- `PXAT timestamp-milliseconds` 设置 key 在某个时间点到期，毫秒级时间戳。

- `NX` 仅当键不存在时才执行命令。

- `XX` 仅当键存在时才执行命令。

- `KEEPTTL` 保留键的过期时间属性，常用在当键已经存在时更新键的值，但是不修改原来键的过期时间属性。

- `GET` 返回键原来的值，如果之前不存在则返回 `nil`。

字符串操作命令：
```
GET key-name：获取键的值

MSET key-name value [key-name value ...]：同时设置多个键值

MGET key-name [key-name ...]：同时获取多个键的值

GETRANGE key-name start end：获取值的指定取键，0 表示第一个字符，-1 表示最后一个字符，-2 表示倒数第二个字符

GETDEL key-name：获取值并删除键

APPEND key-name value：在键的值后面追加 value，即使即使键不存在也不影响
```

当用户将一个值存储到 Redis 字符串里面的时候，如果这个值可以被解释为十进制整数或者浮点数，那么 Redis 会察觉到这一点，并允许用户对这个字符串执行各种 `INCT *` 和 `DECR *` 操作。

如果用户对一个不存在的键或者一个保存了空串的键执行自增或自减操作，那么 Redis 在执行操作的时候会将这个键值当作是 0 来处理。如果尝试对一个值无法被解释为整数或者浮点数的字符串执行自增或自减操作，那么 Redis 将返回一个错误。

自增和自减的命令：
```
INCR key-name: 将键存储的值加上 1，只能用于整数

DECR key-name：将键存储的值减去 1，只能用于整数

INCRBY key-name amount：将键存储的值加上整数 amount，只能用于整数

DECRBY key-name amount：将键存储的值减去整数 amount，只能用于整数

INCRBYFLOAT key-name amount：将键存储的值加上 amount，amount 可以是浮点数
```

## 列表
Redis 列表允许用户从序列的两端推入或弹出元素，获取列表元素，以及执行各种常见的列表操作。

列表常用的命令：
```
RPUSH key val [val ...]：将一个或多个值推入列表的右端。

LPUSH key val [val ...]：将一个或多个值推入列表的左端。

RPOP key：移除并返回列表最右端的元素。

LPOP key：移除并返回列表最左端的元素。

LINDEX key offset：返回列表中偏移量为 offset 的元素，0 表示最左边的元素，-1 表示最右边的元素。

LRANGE key start end：返回列表中从 start 偏移量到 end 偏移量范围内的元素，包括 start 和 end，0 表示最左边的元素，-1 表示最右边的元素。

LTRIM key start end：对列表进行修剪，只保留从 start 偏移量到 end 偏移量范围内的元素，包括 start 和 end，0 表示最左边的元素，-1 表示最右边的元素。
```

阻塞命令：
```
BLPOP/BRPOP key [key ...] timeout

- 从第一个非空列表中弹出位于最左/右端的元素。

- 是 LPOP 的阻塞版本，当没有元素可以从给定的列表中弹出时，会阻塞连接。

- timeout 为阻塞等待的时间，0 表示一直等待，正整数表示等待的秒数。
```

## 集合
Redis 集合以无须的方式来存储多个各不相同的元素，支持快速的对集合执行添加元素、移除元素及检查元素是否在集合中。

集合常用操作命令：
```
SADD key item [item ...]：将一个或多个元素添加到集合中，并返回被添加元素中原本不存在于集合里面的数量。

SREM key item [item ...]：从集合里面移除一个或多个元素，并返回被移除元素的数量。

SISMEMBER key item：检查元素 item 是否存在于集合里面。

SCARD key：返回集合包含的元素数量

SMEMBERS key：返回集合包含的所有元素

SPOP key：随机的移除集合中的元素，并返回被移除的元素。

SMOVE src-key dst-key item：如果集合 sre-key 包含元素 item，那么从集合 src-key 里面移除 item，并将元素 item 添加到 dst-key 中；如果 item 被成功移除，那么命令返回 1，否则返回 0。 
```

集合运算命令：
```
SDIFF key [key ...]：返回存在于第一个集合但不存在于其它集合的元素。

SDIFFSTORE dst-key key [key ...]：将存在于第一个集合但是不存在于其它集合的元素，存储到集合 dst-key。 

SINTER key [key ...]：返回同时存在于所有集合中的元素。

SINTERSTORE dst-key key [key ...]：将同时存在于所有集合中的元素存储到 dst-key。

SUNION key [key ...]：返回所有集合中的元素。

SUNIONSTORE dst-key key [key ...]：将所有集合中的元素存储到 dst-key。
```

## 散列
Redis 的散列可以将多个键值对存储到一个 Redis 键里面。

散列常用命令：
```
HSET key field valie [field valie ...]：设置散列里面指定的键值对，如果键已经在散列里面存在则覆盖值。

HDEL key field [field ...]：删除散列里指定的键。

HEXISTS key field：判断散列里面是否包含指定的键。

HGET key field：获取散列里面指定的键对应的值。

HMGET key field [field ...]：从散列里面获取一个或多个键对应的值.

HGETALL key：获取散列里面所有的键值对。

HKEYS key：获取散列里面所有的键。

KVALS key：获取散列俩面所有的值。

HLEN key：获取散列里面包含键的数量。
```


散列更高级的操作命令：
```
HINCRBY key field increment：将散列里面指定的键对应的值加上一个整数，如果键不存在则创建键，值默认为 0，然后相加。

HINCRBYFLOAT key field increment：将散列里面指的键对应的值加上一个浮点数。
```

## 有序集合
和散列存储着键值之间的映射类似，有序集合也存储着成员与分值之间的映射，并提供了分值处理命令，以及根据分值大小有序地获取或扫描成员和分值的命令。

有序集合 `ZADD` 用命令：
```
ZADD key [NX | XX] score member [score member ...]
```

- 添加指定的一个或多个 member 和 score 到有序集合中。不加选项，如果 member 已经存在与集合中，则更新 score 并重新插入 member 使排序正确。

- `NX` 仅当 member 不存在时新增。如果 member 已经存在无操作。

- `XX` 仅当 member 存在时更新 score。如果 member 不存在则无操作。


有序集合常用命令：
```
ZREM key member [member ...]：从有序集合中删除给定的成员，并返回被移除成员的数量。

ZCARD key：返回有序集合包含的成员数量。

ZINCRBY key increment member：将有序集合中 member 成员的分值加上 increment。

ZCOUNT key min max：返回有序集合中分值介于 min 和 max 之间的成员数量，-inf 表示负无穷，+inf 表示正无穷。是否包含边界值？

ZRANK key member：返回成员 member 在有序集合中的排名。

ZSOCRE key member：返回成员 member 的分值。

ZRANGE key start stop [WITHSCORES]：返回有序集合中排名介于 start 和 stop 之间的成员，如果指定了 WITHSCORES 选项，那么成员的分值也一起返回。是否包含边界值？
```
操作命令


适用业务