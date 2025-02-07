## Redis学习笔记

----

### 基础语法

#### 1.数据类型

1. **String**: 最基本的类型，可以存储任何数据，例如文本或数字。示例值为 `hello world`。
2. **Hash**: 用于存储键值对，适合存储对象或结构体。示例值为 `{"name": "Jack", "age": 21}`。
3. **List**: 有序的字符串列表，适用于队列等场景。示例值为 `[A -> B -> C]`。
4. **Set**: 不重复的元素集合，适用于需要唯一性的场景。示例值为 `{A, B, C}`。
5. **SortedSet**: 有序且唯一的元素集合，每个元素有一个对应的分数，用于排序。示例值为 `{A: 1, B: 2, C: 3}`。
6. **GEO**: 用于处理地理数据，比如位置的经纬度。示例值为 `{A: (120.3, 30.5)}`。
7. **Bitmap**: 用于存储位图，可以支持高效的位运算。示例值为 `011011010110101011`。
8. **HyperLog**: 一种用于基数估算的数据结构，节省空间。示例值为 `011011010110101011`。

----

#### 2.通用命令

> 1. **`help @数据类型`**：得到相应数据类型会用到的命令.
> 2. **`KEYS ***???`**：获取符合模糊匹配的所有关键字，"?"代表一个字符，"*"代表任意个字符。
> 3. **`del [keyname] [keyname]....`**：删除key。
> 4. **`EXISTS [keyname]`**：查询key是否存在。
> 5. **`EXPIRE [keyname]`**：设置过期时间。
> 6. **`TTL [keyname]`**：查看过期时间，-1表示永久有效，-2表示不存在这个key。

----

#### 3.String类型

> 1. **`SET [key] [value]`**: 添加或修改一个已有的String类型的键值对。
> 2. **`GET [key]`**: 根据key获取String类型的value。
> 3. **`MSET [key1 value1] [key2 value2] ...`**: 批量添加多个String类型的键值对。
> 4. **`MGET [key1] [key2] ...`**: 根据多个key获取多个String类型的value。
> 5. **`INCR [key]`**: 让一个整型的key自增1。
> 6. **`INCRBY [key] [increment]`**: 让一个整型的key自增并指定步长，例如：`INCRBY num 2`让num值自增2。
> 7. **`INCRBYFLOAT [key] [increment]`**: 让一个浮点类型的数字自增并指定步长。
> 8. **`SETNX [key] [value]`或者`SET [key] [value] NX`**: 添加一个String类型的键值对，前提是这个key不存在，否则不执行。
> 9. **`SETEX [key] [seconds] [value]`或者`SET [key] EX [value]`**: 添加一个String类型的键值对，并且指定有效期（单位：秒）。

----

#### 4.key的层级结构

由于Redis中没有表这一结构，于是我们会需要key按照`项目名:业务名:类型:主键id`的方式命名，但并不固定，比如mysql里面的shopping库中的goods表的id为1的数据的key可以表示为`shopping:goods:1`，而这一个key对应的value可以是结构体(对象)序列化后的json字符串,这里值得一提的是，如果你用的`RDM`的redis图形化界面，这样的命名在图形化界面里面会以`树`的形式出现，显示很清晰，但是`Datagrip`这类软件貌似并不支持这个功能。

#### 5.Hash类型

```sql
key
├── field1: value1
├── field2: value2
└── field3: value3
...
```

哈希类型，它的value是一个无序字典，可以理解为key里面又存储了多个key的键值对，相较于上面json字符串形式存储数据有着一定的优势，那就是对json字符串中的单个数据进行修改很不方便，而hash类型则可以对单个字段进行CRUD。

**常用命令**:

> 1. **`HSET [key] [key] [field1] [value1] [field2] [value2]...`**: 添加或修改 hash 类型 key 的 field 的值。注：hmset也行，不过已经弃用了.
> 2. **`HGET [key] [field]`**: 获取一个 hash 类型 key 的 field 的值。
> 3. **`HMGET [key] [field1] [field2] ...`**: 批量获取多个 hash 类型 key 的 field 的值。
> 4. **`HGETALL [key]`**: 获取一个 hash 类型的 key 中的所有 field 和 value。
> 5. **`HKEYS [key]`**: 获取一个 hash 类型的 key 中的所有 field。
> 6. **`HVALS [key]`**: 获取一个 hash 类型的 key 中的所有 value。
> 7. **`HINCRBY [key] [field] [increment]`**: 让一个 hash 类型 key 的指定 field 值增加并指定步长。
> 8. **`HSETNX [key] [field] [value]`**: 添加一个 hash 类型 key 的 field 的值，前提是这个 field 不存在，否则不执行。

----

#### 6.List类型

可以看作是一个双向队列结构

**特征**：

- 有序
- 元素可以重复
- 支持插入和删除操作
- 查询速度一般

**常用命令**

> 1. **`LPUSH [key] [element] ...`**: 向列表左侧插入一个或多个元素。
> 2. **`LPOP [key]`**: 移除并返回列表左侧的第一个元素，没有则返回 nil。
> 3. **`RPUSH [key] [element] ...`**: 向列表右侧插入一个或多个元素。
> 4. **`RPOP [key]`**: 移除并返回列表右侧的第一个元素。
> 5. **`LRANGE [key] [start] [end]`**: 返回一段角标范围内的所有元素。
> 6. **`BLPOP [key] [timeout]`**: 与 `LPOP` 类似，在没有元素时等待指定时间。
> 7. **`BRPOP [key] [timeout]`**: 与 `RPOP` 类似，在没有元素时等待指定时间。

----

#### 7.Set类型

 相当于C++的 `unordered_set` 或者Java的`HashSet`，可以用于查看共同好友等。

**特征**：

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集、差集等功能

**常用命令**

> 1. **`SADD [key] [member] ...`**: 向 set 中添加一个或多个元素。
> 2. **`SREM [key] [member] ...`**: 移除 set 中的指定元素。
> 3. **`SCARD [key]`**: 返回 set 中元素的个数。
> 4. **`SISMEMBER [key] [member]`**: 判断一个元素是否存在于 set 中。
> 5. **`SMEMBERS [key]`**: 获取 set 中的所有元素。
> 6. **`SINTER [key1] [key2] ...`**: 求 key1 与 key2 的交集。
> 7. **`SDIFF [key1] [key2] ...`**: 求 key1 与 key2 的差集。
> 8. **`SUNION [key1] [key2] ...`**: 求 key1 和 key2 的并集。

---

#### 8.SortedSet类型

可以理解为C++中的Map,可以用于排行榜系统

**特征**：

- 可排序
- 元素不重复
- 查询速度快

**常用命令**

> - **`ZADD [key] [score] [member]`**: 添加一个或多个元素到 sorted set，如果已存在则更新其 score 值。
> - **`ZREM [key] [member]`**: 删除 sorted set 中的指定元素。
> - **`ZSCORE [key] [member]`**: 获取 sorted set 中指定元素的 score 值。
> - **`ZRANK [key] [member]`**: 获取 sorted set 中指定元素的排名。
> - **`ZCARD [key]`**: 获取 sorted set 中的元素个数。
> - **`ZCOUNT [key] [min] [max]`**: 统计 score 值在指定范围内的所有元素的个数。
> - **`ZINCRBY [key] [increment] [member]`**: 让 sorted set 中的指定元素自增，步长为指定的 increment 值。
> - **`ZRANGE [key] [min] [max]`**: 按照 score 升序排序，获取指定排名范围内的元素，在这里，查询的排名的范围为(min, max]
> - **`ZREVRANGE [key] [min] [max]`**: 按照 score 降序排序。
> - **`ZRANGEBYSCORE [key] [min] [max]`**: 按照 score 排序，获取指定 score 范围内的元素。
> - **`ZDIFF`、`ZINTER`、`ZUNION`**: 求差集、交集、并集。

----

