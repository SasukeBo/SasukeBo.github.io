---
title: 'Redis数据类型'
date: 2020-01-15 09:35:00 +0800
categories: redis
header:
  overlay_image: /assets/images/banners/backend.png
  teaser: /assets/images/default-teaser.png
---

Redis 不仅仅是一个`key-value`存储系统，也是一个数据结构服务器，redis 的值类型不只是 string 类型。

- Binary-safe strings
- Lists
- Sets
- Sorted sets, 与 Sets 相似但是 sorted sets 每一个 string 元素都关联了一个浮点数值，称为 score。元素按照 score 值进行排序。
- Hashes, both the field and value are strings.
- Bit arrays
- HyperLogLogs
- Streams

## Redis keys

Redis keys 是二进制安全的字符串，可以使用任何字符串作为 key，甚至是一个图片文件的二进制值。
关于 key 需要知道一些规则：

- key 不适合过长，否则会浪费存储空间，增加查询开销。
- key 不适合过短，否则可读性不高，例如：`u1000flw`推荐写成`user:1000:flowers`，不会很长且可读性很高。
- 命名规则应该参考一定的格式，例如：`object-type:id-field`或者`object-type:id.field`
- key 最大长度不超过 512MB

### Redis String

string 数据类型是 redis 中最简单的类型。
通过`Set key value`可以设置 string 值。

```redis
> set mykey somevalue
OK
> get mykey
somevalue
```

如果想为已存在的 key 设置值，可以在语句末尾加上 XX，表示仅当 key 存在时更新 value，
而 NX 则表示仅当 key 不存在时创建新的 key，设置 value。

```redis
> set mykey newvalue NX
(nil)
> set mykey newvalue XX
OK
```

虽然 string 是 redis 最基础的类型，但是也可以对它做一些有趣的操作。
比如，数值**原子性**增长。

```redis
> set counter 100
OK
> incr counter
(integer 101)
> incr counter
(integer 102)
> incrby counter 50
(integer 152)
```

`INCR`指令将 string 解析为 integer 然后增加 1 在设置为新的值。相似的指令还有`INCRBY`、`DECR`和`DECRBY`。
原子性：即使有多个客户端同时对同一个 key 进行`INCR`也不会造成资源竞争的情况。

`GETSET`指令在设置新的 value 的同时也会返回旧的值。

设想有一个应用场景，你需要统计网站用户访问数量，一小时获取一次，每次用户访问网站就可以使用`INCR`指令增加 counter，
然后一小时后`GETSET`counter 为 0，返回值便是这一小时内统计到的访问次数。

`MSET`和`MGET`指令可以同时设置和获取多个值。

```redis
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

### 对 key 做修改/查询

- `EXISTS` 查询 key 的存在性
- `DEL`删除 key
- `TYPE`查询 key 存储的 value 类型

### key 存活时间

_Redis expires_

- 过期时间长度单位可以是秒和毫秒
- Redis 服务器停止不会影响过期时间的计时。

```redis
> set key some-value
OK
> expire key 5
(integer 1)
> get key (immediately)
"some-value"
> get key (after some time)
(nil)
```

通过命令`EX` 可以在设置 key 值时同时设置有效时间。

```redis
> set key 100 ex 10
OK
> ttl key
(integer) 7
```

## Redis Lists

Redis Lists 是用链表实现的，对于因为数据库系统而言，快速的在很长的列表中插入一个元素是
非常重要的。

### 初步认识 redis lists

- `LPUSH` 指令可以在 list 左侧插入数据
- `RPUSH` 指令可以在 list 右侧插入数据
- `LRANGE` 指令可以按照区间范围获取列表元素

```redis
> RPUSH list A
(integer 1)
```
