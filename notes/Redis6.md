# Redis6

## 相关概念

1. 开源的`key-value`存储系统
2. 为了保证效率，数据都是缓存在`内存`中
3. 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件
4. `redis`的默认端口是 `6379`
5. `redis`是单线程+多路IO复用技术
5. 有16个数据库，默认为第0个数据库
6. 五种数据类型
   1. `String`字符串
   2. `List`链表
   3. `Set`集合
   4. `zset`(sorted set)有序集合
   5. `hash`哈希类型
7. 与 `Memcache`三点不同
   1. 支持多数据类型
   2. 支持持久化
   3. 单线程+多路IO复用
8. docker redis的原始配置文件路径为 `/etc/redis/redis.conf`

## 安装包工具

1. `redis-benchmark`：性能测试工具
2. `redis-check-aof`：修复有问题的 aof文件
3. `redis-check-dump`：修复有问题的 dump.rdb文件
4. `redis-sentinel`：redis集群使用
5. `redis-server`：redis服务器启动命令
6. `redis-cli`：客户端，操作入口

## 操作指令

### 服务管理指令

#### 启动服务

1. 直接开启redis服务（前台启动）

   ```bash
   redis-server
   ```

2. 后台启动操作步骤

   1. 找到安装包中的 `redis.conf`文件复制一份

   2. 将复制后的 `redis.conf`文件中 `daemonize no`设置`yes`

   3. 应用更新后的配置文件启动服务

      ```bash
      redis-server /myredis/redis.conf
      ```

#### 关闭服务

1. 单实例关闭

   ```bash
   redis-cli shutdown
   ```

2. 终端内关闭

   ```bash
   shutdown # 在终端命令行内输入该指令
   ```

3. 多实例关闭，指定某个端口关闭

   ```bash
   redis-cli -p 6379 shutdown
   ```

4. 使用系统进行关闭

   ```bash
   ps -ef | grep redis
   kill -9 PID	# -9 信号，杀死一个进程。PID：进程ID（Process ID)
   ```

   

#### 远程连接

```bash
redis-cli -h 119.3.210.172 -p 6379 -a 123456789
# -h 指定连接地址
# -p 指定连接端口号
# -a 密码
```

### 数据库指令

```bash
select 		# 切换数据库
dbsize 		# 查看当前数据库 key的数量
flushdb 	# 清空当前库
flushall 	# 清除全部库
```

### key指令

```bash
keys * 					# 查看所有key
exists <key>  	# 判断key存在
type <key> 			# 查看key类型
del <key> 			# 直接删除指定key
unlink <key>  	# 根据 value选择非阻塞删除，仅将keys从keyspace元数据中删除，真正的删除在后续异步操作
expire <key> 10 # 为key指定过期时间，10为10s
ttl <key> 			# 查看还有多少秒过期，-1永不过期，-2已过期
```

## 五大数据类型

### String 字符串

#### 简介

1. 二进制安全，可以包含任意数据，如：图片，序列化后的对象
2. 是基本数据类型，一个字符串 value最多是 512MB

#### 常用指令

```bash
set <key> <value>						# 保存一对键值
get <key>										# 查询键对应值
append <key> <value>				# 将给定的 <value>追加到原值的末尾
strlen <key>								# 获取键对应值的长度
setnx <key> <value>					# 只有在键不存在时，设置键的值
incr <key>									# 将 key中存储的数字值加 1，只能对数字值操作，如果为空，则修改值为 1
decr <key>									# 将 key中存储的数字值减 1，只能对数字值操作，如果为空，则修改值为 -1
incrby | decrby <key> <步长> # 自定义步长，进行值的增减

mset <key1> <value1> <key2> <value2>... # 同时设置一个或多个 key-value对
mget <key1> <key2> <key3>...						# 同时获取一个或多个 value
msetnx <key1> <value1> <key2> <value2>	# 同时设置一个或多个 key-value对，仅当所有 key都不存在的才成功

getrange <key> <起始位置> <结束位置> # 获取值的范围，类似于 java中的 substring，前闭后闭
setrange <key> <起始位置> <value>		# 用 value覆写从起始位置（索引从 0计算）的开始的字符串

setex <key> <过期时间> <value> # 设置键值对的同时，设置过期时间，单位为秒
getset <key> <value> 					# 以新换旧，设置新值的同时获取旧值
```

#### 数据结构

​	String的数据结构为简单动态字符串（Simple Dynamic String，缩写为 SDS）,是可以修改的字符串，内部结构实现类似于 java的 Arraylist，采用预分配冗余空间的方式来减少内存的频繁分配。

​	内部实际为字符串分配的空间一般都要高于字符串长度，当字符串长度小于 1MB时，扩容是加倍现有空间，如果超过1MB，每次扩容只会增加 1MB空间，最大长度是 512MB。

### List列表

#### 简介

1. 单键多值
2. List是简单的字符串列表，按照插入顺序排序，列表的两端均可以添加元素
3. 底层是双向链表，对两端的操作性能很高，通过索引访问元素性能相对较差

#### 常用指令

```bash
lpush | rpush <key> <value1> <value2> <value3>... # 从左边或右边插入一个或多个值
lpop | rpop <key>						# 从左边或右边弹出一个值，没有值时，键也会被清理
rpoplpush <key1> <key2>			# 从 key1列表的右边弹出一个值，插到 key2列表的左边
lrange <key> <start> <stop> # 根据索引获取列表部分元素，-1代表末尾，lrange list 0 -1获取所有
lindex <key> <index>				# 按照索引下标获得元素（从左到右）
llen <key> 									# 获得列表长度

linsert <key> before | after <value> <newValue> 		# 在 <value>的前面或后面插入 <newValue>
lset <key> <index> <value>												# 将列表 key中的索引为 index的值替换为 value
lrem <key> <count> <value>
# 根据参数 count的值，移除列表中与参数 value相等的元素
# count > 0: 从表头开始向表尾搜索，移除 count个与 value相等的元素
# count < 0: 从表尾开始向表头搜索，移除 count(绝对值)个与 value相等的元素
# count = 0: 移除表中所有与 value相等的元素
```

#### 数据结构

​	列表的数据结构为快速列表 `QuickList`

​	在列表元素较少的情况下会使用一块连续的内存存储，这个结构是 `zipList`，即压缩列表。它将所有元素紧挨着一起存储，分配一块连续的内存。当数据量比较多的情况下才会改成 `QuickList`。Redis将链表和 `zipList`相结合组成了 `QuickList`。也就是多个 `zipList`使用双向指针穿起来使用。这样既满足了快速的插入删除性能，又不至于出现太大的空间冗余。

### Set集合

#### 简介

 	Set集合与List列表类似，但存储的数据是无序的，并且会自动去重。它是String类型的无序集合，底层是一个 value为 null的 hash表，所以添加、删除、查找的复杂度都是O(1)。

#### 常用指令

```bash
sadd <key> <value1> <value2>...    # 将一个或多个 member元素加入到集合 key中，已经存在的 member元素将会被忽略
smembers <key> 									   # 取出该集合的所有值
sismember <key> <value> 				   # 判断集合中是否为含有该 value值，存在为 1，不存在为 0
scard <key> 										   # 返回该集合的元素个数
srem <key> <value1> <value2>...    # 删除集合中的某个元素
spop <key> 											   # 随机从该集合中吐出一个值
srandmember <key> <n> 						 # 随机从该集合中吐出 n个值，不会从集合中删除
smove <source> <destination> value # 把一个集合中的某个值移动到另一个集合中
sinter <key1> <key2> 						   # 返回两个集合的交集元素
sunion <key1> <key2> 						   # 返回两个集合的并集元素
sdiff <key1> <key2> 						   # 返回两个集合的差集元素
```

#### 数据结构

Set数据结构是 `dict`字典，字典使用哈希实现的。java中 ` HashSet`的内部实现使用的是 `HashMap`，只不过所有的 value都指向同一个对象。Redis的 set结构也一样，内部使用 `hash`结构，所有的 value都指向同一个内部值

### Hash哈希

#### 简介

1. redis hash是一个键值对集合
2. redis hash是一个 string类型的 field和 value映射表，hash特别适合存储对象
3. 类似于 java中 `Map<String, Object>`

#### 常用指令

```bash
hset <key> <field> <value> 				# 给 key集合中的 field域赋值为 value
hget <key> <field> 				 				# 从 key集合中 field取出 value
hmset <key> <field1> <value1> <field2> <value2> # 批量设置 hash的值
hexists <key> <field>			 				# 查看哈希表 key中，给定 field域是否存在
hkeys <key>								 				# 列出该 hash集合的所有 field
hvals <key>								 				# 列出该 hash集合的所有 value
hincrby <key> <field> <increment> # 为哈希表 key中的域 field的加上输入的数值
hsetnx <key> <field> <value>			# 将哈希表 key中的域 field的值设置为 value，当且仅当域 field不存在的情况下
```

#### 数据结构

hash类型有两种数据结构：

1. `ZipList`压缩列表，当 field-value长度较短且个数较少时
2. `HashTable`哈希表，数量较多时

### Zset有序集合

#### 简介

1. zset和set非常相似，是一个没有重复元素的字符串集合
2. 不同之处在于 zset的每个成员都有一个评分 score，用来表示排序集合，score的数值可以重复
3. 由于有序，可以根据 score和 position来获取一个范围的元素
4. 访问有序集合的中间元素是非常快的

#### 常用指令

```bash
zadd <key> <score1> <value1> <score2> <value2>... # 将一个或多个 member元素及其 score值加入到有序集合中
zrange <key> <start> <stop> [WITHSCORES]
# 返回有序集合中，下标在 start和 stop之间的元素，WITHSCORES连同返回分值
zrangebyscore <key> <min> <max> [WITHSCORES] [limit offset count]
# 返回有序集合中，所有 score值介于 min和 max之间的成员，元素按照 score值递增进行排序，-inf最小极限，+inf最大极限
zrevrangebyscore <key> <max> <min> [WITHSCORES] [limit offset count]
# 同上，顺序变为递减
zincrby <key> <increment> <value> # 为元素的 score加上增量
zrem <key> <value>							  # 删除该集合中，指定值的元素
zcount <key> <min> <max> 					# 统计该集合中，分数区间内的元素个数
zrank <key> <value> 							# 返回该值在集合中的排名，起点为 0
```

#### 数据结构

zset底层使用了两个数据结构：

1. hash，hash的作用是关联元素 value和权重 score，保障元素 value的唯一性，可以通过元素 value找到相应的 score值
2. 跳跃表，跳跃表的目的在于给元素 value排序，根据 score的范围获取元素列表

## 新数据类型

### Bitmaps

#### 简介

Redis提供了 Bitmaps这个数据类型实现`位操作`，合理使用`位操作`能够有效提高内存使用率和开发效率

1. Bitmaps本身不是一种数据类型，实质上是字符串（key-value），但可以对字符串的位进行操作
2. Bitmaps提供一套与字符串方法不同的指令，可以把 `Bitmaps`想象成一个以位为单位的数组，数组的每个单元都能存储 0和 1，数组的下标在 Bitmaps中叫做偏移量

#### 常用指令

```bash
setbit <key> <offset> <value> # 设置 Bitmaps中某个偏移量的值（0或 1），offset偏移量（从 0开始）
getbit <key> <offset>					# 获取 Bitmaps中某个偏移量的值，获取键的第 offset位的值（从 0开始）
bitcount <key> [start end]
# 统计字符串被设置为 1的 bit数，一般情况下，给定的整个字符串都会被计数，通过指定额外的 start和 end参数，可以让计数只在特定的位上进行。start和 end参数通常设置为负数值，-1表示最后一位，-2表示倒数第二位，start和 end是指 bit组中字节的下标数
bitop and(or | not | xor) <destkey> <key> [key...]
# bitop是一个复合操作，可以做多个 Bitmaps的 and、or、not、xor(异或)操作并将结果保存在 destkey中
```

#### Bitmaps与Set对比

1. Bitmaps比 Set能节省更多的内存空间，随着时间的推移，节省的内存是非常可观的
2. 不适合存储变动比较少的数据，适合活跃度比较高的数据

### HyperLogLog

#### 简介

​	Redis HyperLogLog是用来做基数统计的算法，优点是在输入元素的数量或体积非常大时，计算基数所需的空间总是固定的，并且是很小的。每个 HyperLogLog键只需要 12kb内存，就可以计算接近 2^64个不同元素的基数，元素越多内存优势体现越明显。它只根据输入元素来计算基数，不会存储输入元素本身，无法像集合一样返回输入的各个元素。

​	什么是基数？比如数据集{1, 3, 5, 7, 5, 7, 8}，那么这个数据集的基数集为{1, 3, 5, 7, 8}，基数（不重复元素）为 5，基数估计就是在误差可接受的范围内，快速计算基数。

​	计数存在一定的误差，误差率整体较低。标准误差为 0.81% 

#### 常用指令

```bash
pfadd <key> <element> [element...] 					 # 添加指定元素到 HyperLogLog中
pfcount <key> [key...] 						 					 # 计算 HLL的近似基数，可以计算多个 HLL
pfmerge <destkey> <sourcekey> [sourcekey...] # 将一个或多个 HLL合并后存储在另一个 HLL中
```

### Geospatial

#### 简介

Geospatial,GEO，地理信息的缩写，该类型是元素的二维坐标，在地图上是经纬度。redis对此类型提供了经纬度设置、查询、范围查询、距离查询、经纬度 Hash等常见操作。

#### 常用指令

```bash
# 添加地理位置（经度、维度、名称），两级无法添加，有效经度：-180到 180，有效维度：-85.05112878到 85.05112878
geoadd <key> <longitude> <latitude> <member> [longitude latitude member...]
# eg:
geoadd china:city 121.47 31.23 shanghai
geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen 116.38 39.90 beijing

# 获取指定地区的坐标值
geopos <key> <member> [member...]

# 获取两个位置之间的直线距离，m米、km千米、ft英尺、mi英里，默认以米做单位
geodist <key> <member1> <member2> [m | km | ft | mi]

# 以给定的经纬度为中心，找出半径内的元素
georadius <key> <longitude> <latitude> radius [m | km | ft | mi]
```

## Jedis操作 Redis6

## 事务_锁机制

### 事务定义

Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化，按照顺序执行。事务在执行过程中，不会被其他客户端发送来的命令请求所打断。事务的主要作用就是串联多个命令防止其他命令插队。

### 事务操作

```bash
Multi 	# 组队过程：从输入 Multi命令开始，输入的命令都会一次进入命令队列中，但不会执行
Exec 		# 执行过程：直到输入 Exec后，Redis会将命令队列中的命令依次执行
Discard # 组队过程中输入 discard表示放弃之前的命令队列
```

### 事务的错误处理

1. 组队中某个命令出现错误报告，执行时整个队列都会被取消
2. 执行阶段某个命令出现错误，只有报错的命令不会被执行，其他命令依旧正常执行

### 事务冲突问题

一个数据被多个操作源同时进行操作，会造成数据不同步，难以得不到预期的操作结果

#### 悲观锁

悲观锁（Pessimistic Lock），每次拿数据进行操作的时候都会上锁，其他操作者只等到锁的释放，才能操作同一份数据。传统的关系型数据库就用到了很多这种锁机制，比如行锁、表锁、读锁、写锁等，都是操作之前先上锁。

#### 乐观锁

乐观锁（Optimistic Lock），每次拿数据会假设别人不会修改，但在更新数据的时候，会判断一下别人是否修改过数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量。Redis就是利用这种 `check-and-set`机制实现事务的。

#### watch操作

```bash
watch key [key...]
# 在执行 multi之前，先执行 watch key，可以监视一个或多个 key，如果在事务执行前 key被其他命令改动过，那么事务将被打断
```

#### unwatch操作

```bash
unwatch
# 取消所有 key的监听
```

### Redis事务三大特性

- 单独的隔离操作

  > 事务中的所有命令都会序列化、按顺序执行，事务在执行过程中，不会被其他客户端发送来的命令所打断

- 没有隔离级别的概念

  > 队列中的命令没有提交之前，实际上都不会执行，因为事务提交前任何指令都不会被实际执行

- 不保证原子性

  > 事务中如果有一条命令执行失败，其他命令仍会被执行，没有回滚操作

## 持久化

### RDB(Redis Database)

#### 简介

在指定的时间间隔内将内存中的数据集快照（snapshot）写入磁盘，恢复是将快照文件直接读到内存中

#### 备份执行机制

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程结束后，再用这个临时文件替换上次持久化好的问题，整个过程中，主进程是不进行任何 IO操作的，确保了极高的性能，如果需要进行大规模数据的恢复，且对数据恢复的完整性不是非常敏感，那 RDB的方式比 AOF更高效。缺点是可能会丢失最后一次持久化后的数据

#### Fork

- Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值和原进程一致，但是一个全新的进程，并作为原进程的子进程
- 在 Linux程序中，`fork()`会产生一个和父进程完全相同的子进程，但子进程在此后多会 exec系统调用，处于效率考虑，linux引入了`写时复制技术`
- 一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段内容要发生变化时，才会将父进程的内容复制一份给子进程

#### dump.rdb文件

在 redis.conf文件中配置文件名称，默认为 dump.rdb

#### 备份恢复

关闭 redis后，将快照文件名称改成配置中的备份文件名后，再放回工作目录，重启即可

#### 优势

- 适合大规模的数据恢复
- 更适合对数据完整性和一致性要求不高的使用
- 节省磁盘空间
- 恢复速度快

#### 劣势

- Fork的时候，内存中的数据被克隆了一份，大致 2倍的膨胀性需要考虑
- 虽然 Redis在 fork时使用了写时复制技术，但如果数据庞大时，也比较消耗性能
- 在备份周期内，一定间隔时间做一次备份，如果期间 Redis意外宕机（down），就会丢失最后一次快照后的所有修改

#### 停止 RDB备份

```bash
redis-cli config set save "" # save后给空值，表示禁用保存策略
```

### AOF(Append Only File)

#### 简介

以日志的形式来记录每个写操作（增量保存），将 Redis执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件，并重新构建数据。换言之，redis重启时就会根据日志文件的内容将写指令从前到后执行一次，以完成数据的恢复工作

#### 执行机制

1. 客户端的请求写指令会被 append追加到 AOF缓冲区内
2. AOF缓冲区根据 AOF持久化策略「always、everysec、no」将操作 sync同步到磁盘的 AOF文件中
3. AOF文件大小超过重写策略或手动重写时，会对 AOF文件 rewrite重写，压缩 AOF文件容量

#### AOF默认不开启

可以在 redis.conf中配置文件名称，默认为 appendonly.aof，AOF文件的保存路径，同 RDB一致

```bash
appendonly yes 	# appendonly no改为 yes
```

#### AOF和RDB同时开启

AOF和 RDB同时开启，系统默认取 AOF的数据（数据不会存在丢失）

#### 备份恢复

同 RDB一样，先关闭 redis后，将 aof文件名称改成配置文件中要求的 aof文件名称后，再放回工作目录，重启即可

#### 异常恢复

- 修改默认的 appendonly no为 yes
- 如遇到 AOF文件损坏，通过 `/usr/local/bin/redis-check-aof --fix appendonly.aof`进行恢复
- 备份被写坏的 AOF文件
- 恢复：重启 Redis，然后重新加载

#### 同步频率

- `appendfsync awalys` 始终同步，每次写操作都会记入日志，性能较差但数据完整性比较好
- `appendfsync everysec` 每秒同步，每秒记入日志一次，如果宕机，本秒数据可能会丢失
- `appendfsync no` 不主动进行同步，将同步时机交给操作系统

#### Rewrite压缩

- AOF采用文件追加方式，为避免文件越来越大，使用了重写机制，当 aof文件超过阈值，redis会压缩 aof文件，只保留可以恢复数据的最小指令集，可以使用指令 `bgrewriteaof`
- aof文件持续增长，会 fork出一条新线程来重写文件（先写临时文件最后 rename），redis4版本后的重写，是将 rdb快照以二进制的形式附在新的 aof头部，作为历史数据，替换掉原来的流水账操作
- 文件大小超过 64MB并且超过 64MB的一倍（128MB）时，才开始重写

#### 优势

- 备份机制更稳健，丢失数据概率更低
- 可读的日志文本，通过操作 aof文件，可以处理误操作

#### 劣势

- 比起 RDB占用更多的磁盘空间
- 恢复备份速度要慢
- 每次读写都同步的话，有一定性能压力

### 总结

- 官方推荐两种机制都开启
- 如果对数据不敏感，可选单独用 RDB
- 不建议单独使用 AOF，可能会出现 bug
- 只是纯内存缓存，可以都不用

## 主从复制

### 简介

- 主机数据更新后根据配置和策略，自动同步到备机的 master/slaver机制，master以写为主，slaver以读为主
- 一主一从，一主多从
- 读写分离，性能拓展
- 容灾快速恢复

### 操作步骤

> 演示一台服务器中启动多个 redis服务

1. 先创建 `/myredis`文件夹

2. 复制 `redis.conf`配置文件到文件夹中

3. 配置一主两从，创建三个配置文件并写入配置

   - redis6379.conf

     ```bash
     include /myredis/redis.conf
     pidfile /var/run/redis_6379.pid
     port 6379
     dbfilename dump6379.rdb
     ```
   
   - redis6380.conf
   
   - redis6381.conf
   
4. 启动三台服务器
  
  ```bash
  redis-server redis6379.conf
  redis-server redis6380.conf
  redis-server redis6381.conf
  ```
  
5. 查看三台主机运行情况

   ```bash
   # 分别连接三个 redis服务，分别使用下面指令打印主从复制的相关信息
   info replication
   ```

6. 配从（库）不配主（库）

   ```bash
   slaveof <ip> <port>
   # 称为某个实例的从服务器
   # eg: salveof 127.0.0.1 6379
   ```

### 常见复制策略

#### 一主二仆

主服务器下面有一层从机，从机宕机，仍需要手动设置为从机，但会全部读取主服务器的备份数据

#### 薪火相传

主服务器下面有少量从机，从机下面还有自己的从机，只需要从机 `slaveof 从机`就可以了

#### 反客为主

主服务器宕机，一台从机切换为新的主服务器，需要手动输入指令 `slaveof no one`进行切换

#### 哨兵模式

##### 简介

自动化的反客为主，能够后台监控主机是否故障，如果故障了根据投票数自动将从机转换为主机。Java代码中新建 redis连接池时，不直接指定 host和 port，而是指定主机的别名

##### 操作步骤

1. 在 /myredis文件夹中，创建 sentinel.conf文件
2. 在文件中加入内容 `sentinel monitor mymaster 127.0.0.1 6379 1`，其中 `mymaster`为监控对象起的服务器名称，`1`为至少有多少个哨兵同意迁移的数量
3. 启动哨兵模式 `redis-sentinel /myredis/sentinel.conf`

##### 复制延时

主服务每次写操作都需要复制给从机，这个工作的延时性跟从机数量，网络性能有直接关系

##### 故障恢复

主服务宕机，从机升级为主服务是需要投票的

1. 首先看配置文件中的优先级，在 redis.conf中`replica-priority:100`，值越小优先级越高
2. 优先级相同，比较从机的偏移量，选择偏移量最大的，偏移量是指获得原主机数据最全的
3. 偏移量相同，选择 runid最小的从机，每个 redis实例启动后都会随机生成一个 40位的 runid

### 复制原理

1. 当从服务器连接上主服务器后，会向主服务器发送进行数据同步的消息
2. 主服务器收到数据同步的消息后，会进行数据持久化，保存在 rdb文件中，再把 rdb文件发送给从服务器，从服务器拿到 rdb进行读取
3. 每次主服务器进行写操作后，会和从服务器进行数据同步

## Redis集群

无中心化集群

### 制作实例

#### 配置基本信息

- 开启 `daemonize yes`
- `Pid`文件名字
- 指定端口
- `Log`文件名字
- `Dump.rdb `名字
- `Appendonly`关掉或换名字

#### redis cluster配置修改

- `cluster-enabled yes`打开集群模式
- `cluster-config-file nodes-6379.conf`设定节点配置文件名
- `cluster-node-timeout 15000`设定节点失联时间，超过该时间（毫秒），集群自动进行主从切换

```bash
include /home/bigdata/redis.conf
port 6379
pidfile /var/run/redis_6379.pid
dbfilename dump6379.rdb
dir /home/bigdata/redis_cluster
logfile /home/bigdata/redis_cluster/redis_err_6379.log
cluster-enable yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
```

### 多个节点合成集群

1. 确保各节点实例正常运行，nodes-xxx.conf文件都生成正常

2. 合成集群

   ```bash
   cd /opt/redis-6.2.1/src
   # 此处不要用 127.0.0.1，要用真实 IP地址
   # --replicas 1采用最简单的方式配置集群，一台主机，一台从机，6个实例正好三组
   redis-cli --cluster create --cluster-replicas 1 192.168.11.101:6379 192.168.11.101:6380 192.168.11.101:6381 192.168.11.101:6389 192.168.11.101:6390 192.168.11.101:6391
   ```

### 客户端连接

```bash
redis-cli -c -p 6379
# -c采用集群策略连接，设置数据会自动切换到响应的写主机
```

### 查看集群信息

```bash
cluster nodes
# 客户端连接上之后，在命令行中输入该指令查看集群信息
```

### Slots

- 一个 redis集群包含 16384个插槽（hash slot），数据库中的每个键都属于这些插槽的其中一个
- 集群使用公式 `CRC16(key) % 16384`来计算键  key属于哪个槽，其中 `CRC16(key)`语句用于计算键 key的 `CRC16`校验和
- 集群中的每个节点负责处理一部分插槽

### 录入值

- 在 redis-cli每次录入、查询键值，redis都会计算该 key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应前往的 redis实例地址和端口

- redis-cli客户端提供了 `-c`参数实现自动重定向

  > `redis-cli -c -p 6379`登入后，再录入，查询键值对可以重定向

- 不在一个 slot下的键值，是不能使用 mget,mset等多键操作

  ```bash
  mset k1 v1 k2 v3 k3 v3
  # (error) CROSSSLOT keys in request dont`t hash to the same slot
  ```

- 可以通过 `{}`来定义组的概念，从而使 key中 `{}`内相同内容的键值对放到一个 slot中

  ```bash
  mset k1{cust} v1 k2{cust} v2 k3{cust} v3
  ```

### 查看集群中的值

```bash
cluster getkeysinslot <slot> <count>
# 返回 count个 slot槽中的键
cluster keyslot cust
# 返回某个键的 slot值
cluster getkeysinslot 4847
# 返回 slot中的键数
cluster getkeysinslot 4847 10
# 返回 4847这个槽中 10个键
```

### 故障处理

#### 主从切换

- 根据 `cluster-node-timeout `设置的失联时间，主机失联超时，从机升主机
- 失联主机恢复正常，会变成当前主机的从机

#### 某段插槽的主从服务都挂掉

- `cluster-require-full-coverage yes`整个集群都挂掉
- `cluster-require-full-coverage no`只是当前段插槽无法使用，无法存储

### 集群优势

- 实现扩容
- 分摊压力
- 无中心化配置简单

### 集群劣势

- 多键操作是不被支持的
- 多键的 redis事务是不被支持的，lua脚本也不支持
- 由于集群方案出现较晚，公司可能有其他方案，迁移到 redis cluster，需要整体迁移，复杂度高

## Redis应用问题解决

### 缓存穿透

##### 问题描述

键对应的数据在缓存中不存在，因此每次对此键的请求从缓存中获取不到，只能从数据源中获取，从而压垮数据源。比如一直用一个不存在的用户 id查询用户信息，缓存和数据源中都不存在该数据，频繁请求可能会压垮数据源

##### 现象

- 应用服务器压力变大
- redis正常运行，但命中率下降
- 频繁直查数据库
- 出现大量非正常的数据访问请求

##### 解决方案

- 对空值进行缓存：对空结果做缓存，设置较短的过期时间，通常为五分钟
- 设置可访问的名单（白名单）：使用 bitmap类型定义一个名单，每次请求在 bitmap中做匹配，匹配不到则拦截请求
- 采用布隆过滤器
- 进行实时监控：当 redis命中率急速下降，需要排查访问对象和访问的数据，设置黑名单限制服务

### 缓存击穿

##### 问题描述

某个 key对应的数据存在，但在 redis中已经过期，此时有大量并发请求过来，这些请求发现缓存过期，一般都会直接从数据库加载数据并回设到缓存中，这个时候大并发的请求可能会瞬间压垮后端数据库

##### 现象

- 数据库访问压力瞬间增加
- redis中没有出现大规模的 key过期
- redis运行正常

##### 解决方案

- 预设热门数据：在 redis高峰访问之前，把热门 key加入缓存，并延长过期时间
- 实时调整：监控热门数据，实时调整 key的过期时长
- 使用锁：
  1. 在缓存失效的时候，不去立即 load db
  2. 先使用缓存工具的某些带成功操作返回值的操作（比如 redis的 `SETNX`）去 set一个 mutex key
  3. 当操作返回成功后，再进行 load db，并回设缓存，删除 mutex key
  4. 当操作返回失败后，证明有线程在 load db，当前线程先睡眠一段时间后再重试整个 get缓存方法

### 缓存雪崩

##### 问题描述

缓存雪崩同缓存击穿一样，都是由于缓存过期，并发量上来直接去数据库拿数据，但区别是雪崩为批量 key过期，击穿则是某个 key

##### 现象

- 数据库压力变大，服务器崩溃
- 在极少时间段内，查询中有大量 key过期的情况

##### 解决方案

- 构建多级缓存结构：nginx + redis + 其他缓存（ehcache等）
- 使用锁或队列：不适合高并发情况
- 设置过期标志更新缓存：记录缓存数据是否过期（提前量），过期则使用其他线程在后台更新缓存时间
- 设置缓存失效时间的时候，错开时间

### 分布式锁

##### 问题描述

随着业务发展的需要，单体单机部署演化为分布式集群系统。由于分布式系统多线程、多进程并且分部在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，单纯的 Java API并不能提供你那个分布式锁的能力。为了解决这个问题就需要一种跨 JVM的互斥机制来控制共享资源的访问

##### 解决方案

- 基于数据库实现分布式锁
- 基于缓存（redis），性能最高

  - 通过设置一个带有过期时间的 key，通过 check key是不是存在来判断是否可以执行
  - 使用 UUID来为每个线程设置锁的 value，从而解决不同线程之间释放错锁的问题
  - 使用 lua脚本来保证获取释放锁的原子操作，确保一个线程获取锁到释放锁的过程中，不会有其他线程来影响
- 基于 Zookeeper，可靠性最高

## Redis6新功能

### ACL

Redis ACL是 Access Control List(访问控制列表)的缩写，该功能允许根据可以执行的命令和可以访问的键来限制某些连接。

在 Redis5之前，Redis安全规则只有密码控制，还有通过 rename来调整高危命令比如 flushdb，keys*，shutdown等。Redis6则提供 ACL功能对用户进行更细粒度的权限控制

- 接入权限：用户名和密码
- 可以执行的命令
- 可以操作的 KEY

```bash
acl list   # 展现用户权限列表
acl cat  	 # 查看添加权限指令类别
acl whoami # 查看当前用户
acl setuser # 创建和编辑用户 ACL
```

### IO多线程

IO多线程是指客户端交互部分的网路 IO交互处理模块多线程，而非执行命令多线程。redis6执行命令依旧是单线程。

多线程部分只是用来处理网络数据的读写和协议解析，之所以这么设计是不想因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP等等的并发问题。

多线程 IO默认不开启，需要在配置文件中配置

```bash
io-threads-do-reads yes
io-threads 4
```

### 工具支持 Cluster

之前老版本想要搭建集群需要单独安装 ruby环境，Redis5之后将 redis-trib.rb的功能集成到 redis-cli。另外官方 redis-benchmark工具开始支持 cluster模式，通过多线程的方式对多个分片进行压测。

### 其他新功能

- RESP3新的 Redis通信协议：优化服务端与客户端之间的通信
- Client side caching客户端缓存：基于 RESP3协议实现的客户端缓存功能。为了进一步提升缓存的性能，将客户端经常访问的数据 cache到客户端。减少 TCP网络交互
- Proxy集群代理模式：Proxy功能，让 Cluster拥有像单实例一样的接入方式，降低大家使用 cluster的门槛，不过需要注意的事代理不改变 Cluster的功能限制，不支持的命令还是不会支持，比如跨 slot的多 key操作
- Modules API：Redis6中模块 API开发进展非常大，因为 Redis Labs为了开发复杂的功能，从一开始就用上了 Redis模块。Redis可以变成一个框架，利用 Modules来构建不同系统，而不需要从头开始写，并且还需要 BSD许可。Redis一开始就是一个向编写各种系统开发的平台f
