

# redis

## Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API

[TOC]



## 1、linux系统redis运行服务

------

安装linux中Redis

redis默认安装路径：/usr/local/bin

```
mv redis-7.0.5.tar.gz  /opt #移动文件
tar zxvf redis-7.0.5.tar.gz 
cd redis-7.0.5/
ls -l


yum install gcc-c++ #Redis是基于c语言编写的需要安装依赖，需要安装gcc
gcc -v #查看gcc版本


make #输入make命令配置Gcc需要的文件(输入两次)
make install #确认所有都安装完成


mkdir redisconfig #我们要将配置文件复制一份，我们以后就是用这个配置文件来启动
cp /opt/redis-7.0.5/redis.conf redisconfig


```



```
cd /usr/local/bin/ #切换目录
redis-server redisconfig/redis.conf #开启reids服务
redis-cli -p 6379  #使用redis-cli连接测试
```

## 2、Redis 是单线程的

------

官方表示，**Redis 是基于内存操作的**，CPU 不是 Redis 的性能瓶颈。Redis 的瓶颈是根据机器的内存和网络带宽，既然可以使用单线程，就使用单线程了

核心：**redis 是将全部数据放在内存中**，所以使用单线程去操作效率就是最高的，对于内存系统来说，如果**没有上下文切换效率就是最高的**，多次读写都是在一个 CPU 上。

```
select 1  #切换1数据库
DBSIZE  #查看DB大小
keys *  #查看所有key
flushall #清空所有数据库
flushdb #清空当前数据库
exists key #判断当前key是否存在
move key 1 #移除当前的key
expire key second #设置过期时间
ttl key  #查看key剩余过期时间
type key #查看key的类型
```

![](https://pic1.imgdb.cn/item/67c971ab066befcec6df1950.png)

## 3、Redis 五大数据类型

### 1、String

```
set key value #设置key-value
get key  #获取key的value
exists key  #key是否存在
append key value #追加字符串，若key不存，相当于set key value
strlen key #获取字符串长度
incr key #当前key的value加1
decr key #当前key的value减一
incrby key 10 #当前key加10
decrby key 10 #当前key减10
getrange key 0 3 #字符串范围 （getrange key 0 -1 获取全部字符串）
setrange key 1 xx #替换指定位置开始的字符串
setex key second value    #（set with expire）设置过期时间
setnx key value   #（set if not with exists ）不存在再设置 （分布式锁中常使用）
mset key1 v1 key2 v2  #批量设置
mget key1 key2 key3  #批量获取
msetnx key1 v1 key2 v2  #不存在再设置(批量 原子性操作  一起成功 一起失败)
getset key value #先获取原值再设置新值
```

```
127.0.0.1:6379> FLUSHALL
OK
127.0.0.1:6379> set name lqh 
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> set age 1 
OK
127.0.0.1:6379> keys *  #查看所有的key
1) "age"
2) "name"
127.0.0.1:6379> EXISTS key
(integer) 0
127.0.0.1:6379> EXISTS name #判断当前key是否存在
(integer) 1
127.0.0.1:6379> move name 1 #将key移动到1号数据库中
(integer) 1
127.0.0.1:6379> keys *
1) "age"
127.0.0.1:6379> set name 123
OK
127.0.0.1:6379> keys *
1) "name"
2) "age"
127.0.0.1:6379> EXPIRE name 10 #设置key过期的时间，单位是秒
(integer) 1
127.0.0.1:6379> ttl name #查看当前key的剩余时间
(integer) 6
127.0.0.1:6379> set name lqh
OK
127.0.0.1:6379> keys *
1) "name"
2) "age"
127.0.0.1:6379> TYPE name #查看当前key对应value的类型
string
127.0.0.1:6379> TYPE age
string
```

```
127.0.0.1:6379> set key1 v1  #设置值
OK
127.0.0.1:6379> get key1  #获得值
"v1"
127.0.0.1:6379> keys *  #获得所有key
1) "key1"
2) "name"
3) "age"
127.0.0.1:6379> EXISTS key1  #判断某一个key是否存在
(integer) 1
127.0.0.1:6379> APPEND key1 "hello"  #追加字符串，如果key不存在就相当于set key
(integer) 7
127.0.0.1:6379> get key1
"v1hello"
127.0.0.1:6379> APPEND key1 ",nihao"  
(integer) 13
127.0.0.1:6379> STRLEN key1  #获取字符串长度
(integer) 13
127.0.0.1:6379> get key1
"v1hello,nihao"
```

```
#步长
127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> get views
"0"
127.0.0.1:6379> incr views  #自增一
(integer) 1
127.0.0.1:6379> incr views
(integer) 2
127.0.0.1:6379> get views
"2"
127.0.0.1:6379> decr views  #自减一
(integer) 1
127.0.0.1:6379> decr views
(integer) 0
127.0.0.1:6379> decr views
(integer) -1
127.0.0.1:6379> get views
"-1"
127.0.0.1:6379> INCRBY views 10  #设置步长，指定增量
(integer) 9
127.0.0.1:6379> DECRBY views 5   #设置步长，指定减量
(integer) 4
127.0.0.1:6379> get views
"4"
#字符串范围
127.0.0.1:6379> set key1 "hello,lqh"  #设置key的值
OK
127.0.0.1:6379> get key1
"hello,lqh"
127.0.0.1:6379> GETRANGE key1 0 3  #截取字符串  [0,3]
"hell" 
127.0.0.1:6379> GETRANGE key1 0 -1  #获取全部的字符串 和get key是一样的
"hello,lqh"
#替换
127.0.0.1:6379> set key2 abcdefg
OK
127.0.0.1:6379> get key2
"abcdefg"
127.0.0.1:6379> SETRANGE key2 1 xx  #替换指定位置开始的字符串
(integer) 7
127.0.0.1:6379> get key2
"axxdefg"
#setex key #（set with expire）设置过期时间
#setnx key #（set if not with exists ）不存在再设置 在分布式锁中会常使用
127.0.0.1:6379> setex key3 30 "hello"  #设置一个key3的值为hello，30秒后过期
OK
127.0.0.1:6379> ttl key3
(integer) 19
127.0.0.1:6379> setnx mykey "redis"  #如果mykey不存在，创建mykey
(integer) 1
127.0.0.1:6379> keys *
1) "mykey"
2) "key1"
3) "key2"
127.0.0.1:6379> ttl key3
(integer) -2
127.0.0.1:6379> setnx mykey "hello"  #如果mykey存在，创建失败
(integer) 0
127.0.0.1:6379> get mykey
"redis"
mset key1 v1 key2 v2  #批量设置
mget key1 key2 key3  #批量获取
 
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3  #同时设置多个值
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> mget k1 k2 k3  #同时获取多个值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v1 k4 v4  #msetnx是一个原子性的操作，要么一起成功要么一起失败
(integer) 0
127.0.0.1:6379> get k4
(nil)
 
#对象
set user:1 {name:zhangsan,age:2} #设置一个user:1对象，值为json字符来保存一个对象
 
#这里的key是一个巧妙的设计：user:{id}:{filed}，如此设计在redis中是完全ok的
127.0.0.1:6379> mset user:1:name zhangsan user:1:age 2
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "2"

getset #先get再set
 
127.0.0.1:6379> getset db redis  #如果不存在值则返回nil
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> getset db Mongodb #如果存在值，获取原来的值并设置新的值
"redis"
127.0.0.1:6379> get db
"Mongodb"
```

### 2、List

Redis 中，可以将 list 用作栈、队列、阻塞队列的数据结构
所有 list 命令都是以 l 开头

```
lpush key v1 v2 ...  #将一个值或多个值插入列表的头部(左)
rpush key v1 v2 ...  #将一个值或多个值插入列表的尾部(右)
lrange key start end  #用过区间获取具体的值  （0 -1 区间获取全部值）
lpop key  #移除列表头部第一个值（左）
rpop key  #移除列表尾部第一个值（右）
lindex key index #通过索引获取值
llen key   #获取列表长度
lrem key count value  #移除list集合中指定个数的value  精确匹配
ltrim key start stop   #通过下标截取指定长度，list已经改变，只剩下截取后的元素
rpoplpush key otherkey  #移除列表中最后一个元素，并将它插入另一个列表头部
lset key index value  #将列表中指定下标的值替换为另外一个值，更新操作 （如果列表或索引不存在  会报错）
linsert key before v1 v2  #在v1前插入v2
linsert key after v1 v2  #在v1后插入v2
127.0.0.1:6379> LPUSH list one  #将一个或者多个值，插入到列表的头部（左边）
(integer) 1
127.0.0.1:6379> LPUSH list twe
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1  #获取list中的值
1) "three"
2) "twe"
3) "one"
127.0.0.1:6379> LRANGE list 0 1  #通过区间获取list中具体的值
1) "three"
2) "twe"
127.0.0.1:6379> RPUSH list right  #将一个或者多个值，插入到列表的尾部（右边）
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "twe"
3) "one"
4) "right"
#LPOP
#RPOP
127.0.0.1:6379> LRANGE list 0 -1 
1) "three"
2) "twe"
3) "one"
4) "right"
127.0.0.1:6379> LPOP list #移除list的第一个元素
"three"
127.0.0.1:6379> RPOP list #移除list的最后一个元素
"right"
127.0.0.1:6379> LRANGE list 0 -1
1) "twe"
2) "one"
#lindex
127.0.0.1:6379> LRANGE list 0 -1
1) "twe"
2) "one"
127.0.0.1:6379> LINDEX list 1  #通过下标获得list中的某一个值
"one"
127.0.0.1:6379> LINDEX list 0
"twe"
 
127.0.0.1:6379> Lpush list one
(integer) 1
127.0.0.1:6379> Lpush list twe
(integer) 2
127.0.0.1:6379> Lpush list three
(integer) 3
127.0.0.1:6379> Llen list  #返回列表长度
(integer) 3
#移除指定的值
127.0.0.1:6379> Lpush list one
(integer) 1
127.0.0.1:6379> Lpush list twe
(integer) 2
127.0.0.1:6379> Lpush list three
(integer) 3
127.0.0.1:6379> Llen list
(integer) 3
127.0.0.1:6379> Lpush list three
(integer) 4
127.0.0.1:6379> lrem list 1 one #移除list集合中指定个数的valus，精确匹配
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "twe"
127.0.0.1:6379> lrem list 1 three
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "twe"
127.0.0.1:6379> Lpush list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "twe"
127.0.0.1:6379> lrem list 2 three
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "twe"
#ltrim 修剪
 
127.0.0.1:6379> Lpush mylist "hello"
(integer) 1
127.0.0.1:6379> Lpush mylist "hello1"
(integer) 2
127.0.0.1:6379> Lpush mylist "hello2"
(integer) 3
127.0.0.1:6379> Lpush mylist "hello3"
(integer) 4
127.0.0.1:6379> ltrim mylist 1 2  #只截取区间内的，剩余的将全部被删除
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello2"
2) "hello1"
rpoplpush  #移除列表中最后一个元素，并将它移动到新的列表中
 
127.0.0.1:6379> lpush list "hello"
(integer) 1
127.0.0.1:6379> lpush list "hello1"
(integer) 2
127.0.0.1:6379> lpush list "hello2"
(integer) 3
127.0.0.1:6379> rpoplpush list mylist  #移除列表中最后一个元素，并将它移动到新的列表中
"hello"
127.0.0.1:6379> lrange list 0 -1  #查看原来的列表
1) "hello2"
2) "hello1"
127.0.0.1:6379> lrange mylist 0 -1  #查看目标列表中却是存在该值
1) "hello"
lset #将列表中指定下标的值替换为另外一个值
 
127.0.0.1:6379> EXISTS list  #判断这个列表是否存在
(integer) 0
127.0.0.1:6379> lset list 0 item  #如果不存在我们去更新就会报错
(error) ERR no such key
127.0.0.1:6379> lpush list value1
(integer) 1
127.0.0.1:6379> lrange list 0 0
1) "value1"
127.0.0.1:6379> lset list 0 item  #如果存在则更新下标的值
OK
127.0.0.1:6379> lrange list 0 0
1) "item"
127.0.0.1:6379> lset list 1 other  #如果不存在就会报错
(error) ERR index out of range
linsert #将某个具体的value插入到列表中某个元素的前面或者后面
 
127.0.0.1:6379> rpush list hello
(integer) 1
127.0.0.1:6379> rpush list world
(integer) 2
127.0.0.1:6379> LINSERT list before "world" "other" #在world前插入other
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "hello"
2) "other"
3) "world"
127.0.0.1:6379> lpush mylist hello
(integer) 1
127.0.0.1:6379> lpush mylist world
(integer) 2
127.0.0.1:6379> LINSERT mylist before "world" "other"
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "hello"
2) "other"
3) "world"
127.0.0.1:6379> LRANGE mylist 0 -1
1) "other"
2) "world"
3) "hello"
127.0.0.1:6379> LINSERT list after "world" "123" #在world后插入123
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "hello"
2) "other"
3) "world"
4) "123"
```

### 3、Set

set 中的值不能重复

```
sadd key value  #添加元素
smembers key   #查看指定set中所用元素
sismember key value  #判断某一个值在指定set中是否存在
scard key  #获取set中的内容元素个数
srem key value   #移除set中指定元素
srandmember key count  #随机选出指定个数的成员
spop key  #随机移除元素
smove oldkey  newkey member  #将一个指定的值，从一个set移动到另一个set
sdiff key...   #获取多个set差集
sinter key...  #获取多个set交集 （共同好友、共同关注）
sunion key...  #获取多个set并集
127.0.0.1:6379> sadd myset hello  #添加元素
(integer) 1
127.0.0.1:6379> sadd myset world
(integer) 1
127.0.0.1:6379> sadd myset 123
(integer) 1
127.0.0.1:6379> SMEMBERS myset  #查看指定set中所用元素
1) "123"
2) "world"
3) "hello"
127.0.0.1:6379> SISMEMBER myset hello #判断某一个值在指定set中是否存在
(integer) 1 
127.0.0.1:6379> SISMEMBER myset lqh
(integer) 0
127.0.0.1:6379> SCARD myset  #获取set集合里元素的个数
(integer) 3
127.0.0.1:6379> srem myset 123  #移除set中的指定
(integer) 1
127.0.0.1:6379> SCARD myset
(integer) 2
127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "hello"
127.0.0.1:6379> SRANDMEMBER myset  #随机选出指定个数的成员
"world"
127.0.0.1:6379> SRANDMEMBER myset
"hello"
127.0.0.1:6379> SRANDMEMBER myset
"hello"
127.0.0.1:6379> SRANDMEMBER myset
"world"
 
127.0.0.1:6379> SMEMBERS myset
1) "def"
2) "abc"
3) "world"
4) "hello"
127.0.0.1:6379> spop myset  #随机移除元素
"def"
127.0.0.1:6379> spop myset
"hello"
#将一个指定的值，从一个set移动到另一个set
127.0.0.1:6379> sadd myset hello
(integer) 1
127.0.0.1:6379> sadd myset world
(integer) 1
127.0.0.1:6379> sadd myset lqh
(integer) 1
127.0.0.1:6379> sadd myset2 set2
(integer) 1
127.0.0.1:6379> smove myset myset2 lqh #将一个指定的值，从一个set移动到另一个set集合中
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "hello"
127.0.0.1:6379> SMEMBERS myset2
1) "set2"
2) "lqh"
127.0.0.1:6379> sadd key1 1
(integer) 1
127.0.0.1:6379> sadd key1 2
(integer) 1
127.0.0.1:6379> sadd key1 3
(integer) 1
127.0.0.1:6379> sadd key2 3
(integer) 1
127.0.0.1:6379> sadd key2 4
(integer) 1
127.0.0.1:6379> sadd key2 5
(integer) 1
127.0.0.1:6379> sadd key2 6
(integer) 1
127.0.0.1:6379> SDIFF key1 key2  #差集
1) "1"
2) "2"
127.0.0.1:6379> SINTER key1 key2  #交集 共同好友可以这么实现
1) "3"
127.0.0.1:6379> SUNION key1 key2  #并集
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
```

### 4、Hash

Map 集合 key-(key-value)

```
hset key field value  #存入一个具体键值对
hget key field   #获取一个字段值
hmset key field value field1 value1 ...  #存入多个具体键值对
hmget key field field1 ...  #获取多个字段值
hgetall key    #获取全部数据
hdel key field  #删除hash指定的key字段，对应value也就没有了
hlen key     #获取hash中字段数量
hexists key field   #判断hash中某个字段是否存在
hkeys key    #获取hash中全部key
hvals key    #获取hash中全部value
hincrby key field 1  #hash中指定key的value加1
hincrby key field -1  #hash中指定key的value减1
hsetnx key field value   #如果hash中指定key不存在则创建，存在则创建失败
127.0.0.1:6379> hset myhash field1 lqh #set一个具体的key value
(integer) 1
127.0.0.1:6379> hget myhash field1  #获取一个字段值
"lqh"
127.0.0.1:6379> hmset myhash field1 hello field2 world #set多个key value
OK
127.0.0.1:6379> hmget myhash field1 field2 #获取多个字段值
1) "hello"
2) "world"
127.0.0.1:6379> hgetall myhash  #获取全部字段值
1) "field1"
2) "hello"
3) "field2"
4) "world"
127.0.0.1:6379> HDEL myhash field1  #删除哈希指定的字段，对应的value值也就消失了
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "field2"
2) "world"
#获取hash中字段数量
127.0.0.1:6379> hmset myhash field1 hello field2 world
OK
127.0.0.1:6379> hgetall myhash
1) "field2"
2) "world"
3) "field1"
4) "hello"
127.0.0.1:6379> hlen myhash  #获取hash中字段数量
(integer) 2
127.0.0.1:6379> HEXISTS myhash field1  #判断hash中某个字段是否存在
(integer) 1
127.0.0.1:6379> HEXISTS myhash field3
(integer) 0
#获取hash中全部key
#获取hash中全部value
127.0.0.1:6379> hkeys myhash  #获取hash中全部key
1) "field2"
2) "field1"
127.0.0.1:6379> hvals myhash  #获取hash中全部value
1) "world"
2) "hello"
#hash中没有HDECRBY 所以减用负数增代替
127.0.0.1:6379> hset myhash field3 5   
(integer) 1
127.0.0.1:6379> HINCRBY myhash field3 1  #hash中指定key的value加1
(integer) 6
127.0.0.1:6379> HINCRBY myhash field3 -2  #hash中指定key的value减二
(integer) 4
#如果hash中指定key不存在则创建，存在则创建失败
127.0.0.1:6379> hsetnx myhash field4 hello  #hash中指定key不存在则创建
(integer) 1
127.0.0.1:6379> hsetnx myhash field4 world  #存在则创建失败
(integer) 0
```

hash 应用场景： 变更的数据，比如用户信息，以及其他经常变动的信息；**hash 更加适合对象的存储，String 更适合字符串的存储。**





### 5、[Zset](https://so.csdn.net/so/search?q=Zset&spm=1001.2101.3001.7020)

有序集合，在 set 的基础上增加了一个排序的值



```
zadd key score value  #添加元素
zrange key 0 1   #通过索引区间返回有序集合指定区间内的成员   （0 -1）返回全部
zrangebyscore key min max   #排序并返回 从小到大  例如：zrangebyscore key1 -inf +inf （-inf：负无穷   +inf：正无穷 ）
zrevrangebyscore myset +inf -inf  #逆序 
zrevrange key 0 -1     #排序并返回 从大到小
zrem key value   #移除指定元素
zcard key        #获取有序集合中的数量
zcount key start stop   #获取指定区间中的成员数量
127.0.0.1:6379> zadd myset 1 one  #添加一个值
(integer) 1
127.0.0.1:6379> zadd myset 2 twe 3 three  #添加多个值
(integer) 2
127.0.0.1:6379> zrange myset 0 -1
1) "one"
2) "twe"
3) "three"
#排序如何实现
127.0.0.1:6379> zadd salary 2500 xiaoming #添加三个用户
(integer) 1
127.0.0.1:6379> zadd salary 1500 xiaohong
(integer) 1
127.0.0.1:6379> zadd salary 500 lihua
(integer) 1
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf  #显示全部用户 从小到大排序
1) "lihua"
2) "xiaohong"
3) "xiaoming"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf withscores #显示全部用户和分数 从小到大排序
1) "lihua"
2) "500"
3) "xiaohong"
4) "1500"
5) "xiaoming"
6) "2500"
127.0.0.1:6379> zrevrangebyscore salary +inf -inf  #显示全部用户 从大到小排序
1) "xiaoming"
2) "xiaohong"
3) "lihua"
127.0.0.1:6379> zrevrangebyscore salary +inf -inf withscores #显示全部用户和分数 从大到小排序
1) "xiaoming"
2) "2500"
3) "xiaohong"
4) "1500"
5) "lihua"
6) "500"
127.0.0.1:6379> ZRANGEBYSCORE salary -inf 1500  #显示小于1500的降序排列
1) "lihua"
2) "xiaohong"
#移除元素
127.0.0.1:6379> zrange salary 0 -1
1) "lihua"
2) "xiaohong"
3) "xiaoming"
127.0.0.1:6379> zrem salary xiaohong  #移除元素
(integer) 1
127.0.0.1:6379> zrange salary 0 -1
1) "lihua"
2) "xiaoming"
127.0.0.1:6379> zcard salary  #获取有序集合中的个数
(integer) 2
#获取指定区间中的成员数量
127.0.0.1:6379> zadd myset 1 hello
(integer) 1
127.0.0.1:6379> zadd myset 2 world 3 lqh
(integer) 2
127.0.0.1:6379> zcount myset 1 3  #获取指定区间中的成员数量
(integer) 3
127.0.0.1:6379> zcount myset 1 2
(integer) 2
```





## 4、Redis 三种特殊数据类型

------

### 1、geospatial

地理位置 （定位、附近的人、打车距离……）
GEO 底层就是 Zset 可以用 Zset 命令操作 Geo

```
#geoadd 添加地理位置
#规则：两极无法直接加入，通常通过java一次性导入  有效经度：-180到180 有效纬度：-85.05112878到85.05112878
127.0.0.1:6379> geoadd china:city 116.39 39.91 beijin
(integer) 1
127.0.0.1:6379> geoadd china:city 121.48 31.40 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 114.02 30.58 wuhan
(integer) 1
127.0.0.1:6379> geoadd china:city 113.88 22.55 shenzhen
(integer) 1
127.0.0.1:6379> geoadd china:city 120.21 30.20 hangzhou
(integer) 1
 
#geopop 获取指定成员的经度和纬度
#获取当前定位，一定是一个坐标值
127.0.0.1:6379> GEOPOS china:city beijin
1) 1) "116.38999968767166138"
   2) "39.90999956664450821"
127.0.0.1:6379> GEOPOS china:city beijin hangzhou
1) 1) "116.38999968767166138"
   2) "39.90999956664450821"
2) 1) "120.21000176668167114"
   2) "30.19999988833350102"
 
#geodist 查看成员间的的直线距离 
127.0.0.1:6379> GEODIST china:city beijin shanghai
"1051905.1484"
127.0.0.1:6379> GEODIST china:city beijin shanghai km
"1051.9051"
 
#georadius 以给定经纬度为中心，找出某一半径内的元素
#
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km #以100,30这个经纬度为中心，寻找方圆1000KM内的城市
1) "shenzhen"
2) "wuhan"
3) "hangzhou"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km
1) "wuhan"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withdist #显示到中心距离的位置
1) 1) "wuhan"
   2) "391.4185"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withcoord #显示他人的经纬度定位信息
1) 1) "wuhan" 
   2) 1) "114.01999980211257935"
      2) "30.58000021509926825"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withdist withcoord count 1 #筛选出指定的结果
1) 1) "wuhan"
   2) "391.4185"
   3) 1) "114.01999980211257935"
      2) "30.58000021509926825"
 
#georadiusbymember 以给定成员为中心，找出某一半径内的元素
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijin 1000 km
1) "beijin"
127.0.0.1:6379> GEORADIUSBYMEMBER china:city shanghai 400 km
1) "hangzhou"
2) "shanghai"
 
#geohash 返回一个或多个位置元素的geohash表示  将二维的经纬度转换成一维的11位字符串 如果两个字符串越接近，则距离越近。
127.0.0.1:6379> geohash china:city beijin hangzhou
1) "wx4g092see0"
2) "wtm7z3wrb00"
```

**GEO 底层实现原理其实就是 Zset，我们可以用 Zset 来操作 GEO**

```
127.0.0.1:6379> zrange china:city 0 -1 #查看地图中的全部元素
1) "shenzhen"
2) "wuhan"
3) "hangzhou"
4) "shanghai"
5) "beijin"
127.0.0.1:6379> zrem china:city beijin #移除指定元素
(integer) 1
127.0.0.1:6379> zrange china:city 0 -1
1) "shenzhen"
2) "wuhan"
3) "hangzhou"
4) "shanghai"
```

### 2、hyperloglog

**什么是基数？**

A={1,3,5,7,9,7}

B={1,3,5,7,9}

基数（不重复的元素）=5 **可以接受误差 大概有 0.81% 的错误率**

A 中有 6 个元素，但是 7 是重复的，所以不重复的有 5 个，所以基数是 5，而 B 中 5 个数都不重复，所以基数也是 5

**Redis hyperloglog 基数统计算法：**

**优点**： 占用内存固定，存放 2^64 不同的元素的技术，只需要占用 12KB 内存
**网页的 UV （一个人访问一个网站多次，统计出还是一个人）**
传统的方式：set 集合保存用户 id，统计 set 中元素数量作为标准判断。这个方式保存大量的用户 id 就会比较麻烦，相对消耗更多内存，我们的目的并不是保存用户 id，目的只是计数。

```
#测试使用
 
127.0.0.1:6379> pfadd mykey a b c d e f g h i j #创建第一组元素
(integer) 1
127.0.0.1:6379> pfcount mykey  #统计key中元素基数数量
(integer) 10
127.0.0.1:6379> pfadd mykey2 i j z a s g f i h a q #创建第二组元素
(integer) 1 
127.0.0.1:6379> pfcount mykey2
(integer) 9
127.0.0.1:6379> PFMERGE mykey3 mykey mykey2 #合并两组mykey mykey2 -> mykey3 并集
OK 
127.0.0.1:6379> pfcount mykey3 #查看并集数量
(integer) 13
```

### 3、Bitmaps

位存储
统计用户信息 活跃 不活跃 登录 未登录 打卡
两个状态的 都可以使用 bitmaps
bitmaps 位图数据结构，都是操作二进制位来进行记录的，非 0 即 1

```
setbit key offset value  #设置位图
getbit key offset        #获取指定位图的值
bitcount key     #统计数量
#例如 一周打卡   0为打卡 1打卡
127.0.0.1:6379> setbit sign 0 1 #设置打卡天数和打卡情况
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 1
(integer) 0
127.0.0.1:6379> setbit sign 5 0
(integer) 0
127.0.0.1:6379> setbit sign 6 0
(integer) 0
127.0.0.1:6379> getbit sign 3   #查看某天是否打卡
(integer) 1
127.0.0.1:6379> getbit sign 5
(integer) 0
127.0.0.1:6379> BITCOUNT sign   #查看一周打卡次数
(integer) 3
```

### 5、[Redis 事务](https://so.csdn.net/so/search?q=Redis事务&spm=1001.2101.3001.7020)

**Redis 单条命令时保证原子性的，但是事务不保证原子性**

Redis 事务的本质：一组命令的集合！一个事务中所有命令都会被序列化，在事务执行的过程中，会按照顺序执行。**一次性、顺序性、排他性**

**Redis 事务没有隔离级别的概念！**
所有命令在事务中，并没有直接执行！只有在发起执行命令的时候才会执行！

Redis 的事务：

- 开启事务（multi）
- 命令入队（…）
- 执行事务（exec）\ 放弃事务（discard）

```
127.0.0.1:6379> multi  #开启事务
OK
#命令入队
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> get k2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> exec #执行事务
1) OK
2) OK
3) "v2"
4) OK
127.0.0.1:6379> multi  #开启事务
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> DISCARD  #取消事务
OK
127.0.0.1:6379> get k4  事务队列中命令都不会执行
(nil)
#编译型异常（代码有问题，命令有错），事务中所有的命令都不会被执行
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> getset k3  #错误的命令
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> set k5 v5
QUEUED
127.0.0.1:6379(TX)> exec  #执行事务报错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k5  #所有的命令都不会被执行
(nil)
#运行时异常（1/0），如果事务中存在语法型错误，那么执行命令的时候，其他命令是可以正常执行的
#错误命令抛出异常
 
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> incr k1
QUEUED
127.0.0.1:6379(TX)> set k1 "v1"
QUEUED
127.0.0.1:6379(TX)> incr k1  #会执行的时候失败
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> get k3
QUEUED
127.0.0.1:6379(TX)> exec
1) (integer) 1
2) OK
3) (error) ERR value is not an integer or out of range #虽然命令报错了，但是依旧正常执行成功了
4) OK
5) OK
6) "v3"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k3
"v3"
```

### 6、Redis 实现[乐观锁](https://so.csdn.net/so/search?q=乐观锁&spm=1001.2101.3001.7020)

监控！Watch/unwatch(解锁，如果事务执行失败，先解锁，然后再次手动去监视)

- 悲观锁：

  - 很悲观，什么时候都会出问题，无论做什么都会加锁！

- 乐观锁

  - 很乐观，认为什么时候都不会出问题，不会加锁！更新数据的时候判断，在此期间是否有人修改过这个数据。
  - 获取 version
  - 更新时比较 version

  ```
  #正常执行成功的情况
  127.0.0.1:6379> set money 100
  OK
  127.0.0.1:6379> set out 0
  OK
  127.0.0.1:6379> watch money  #监视money对象
  OK
  127.0.0.1:6379> multi  #事务正常结束数据期间没有发生变动，这个时候就正常执行成功
  OK
  127.0.0.1:6379(TX)> DECRBY money 20
  QUEUED
  127.0.0.1:6379(TX)> INCRBY out 20
  QUEUED
  127.0.0.1:6379(TX)> exec
  1) (integer) 80
  2) (integer) 20
  #测试多线程修改值，使用watch可以当做redis的乐观锁操作
  127.0.0.1:6379> watch money  #监视 money
  OK
  127.0.0.1:6379> multi
  OK
  127.0.0.1:6379(TX)> DECRBY money 10
  QUEUED
  127.0.0.1:6379(TX)> INCRBY out 10
  QUEUED
  127.0.0.1:6379(TX)> exec  #执行之前，另外一个线程修改了我们的值，这个时候就会导致事务执行失败
  (nil)
   
  127.0.0.1:6379> unwatch  #如果事务执行失败，就先解锁
  OK
  127.0.0.1:6379> watch money  #获取最新的值，再次监视，select version
  OK
  127.0.0.1:6379> multi
  OK
  127.0.0.1:6379(TX)> decrby money 10
  QUEUED
  127.0.0.1:6379(TX)> incrby out 10
  QUEUED
  127.0.0.1:6379(TX)> exec  #比对监事的值是否发生变化，如果没有变化就执行成功，有变化则失败
  1) (integer) 990
  2) (integer) 30
  ```

  ### 7、Jedis

  什么是 Jedis： Redis 官方推荐的 java 连接开发工具！

  1、导入 jar 包
  2、连接数据库
  3、操作命令
  4、断开连接

  1、导入对应的依赖





```
 <!--导入jedis的包-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.7.0</version>
        </dependency>
        <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.79</version>
        </dependency>
```

2、编码测试：

- 连接数据库
- 操作命令
- 断开连接

```
public class TestPing {
    public static void main(String[] args) {
        //1、new jedis对象即可
        Jedis jedis = new Jedis("IP地址",6379);
        jedis.auth("password");
        //jedis命令就是我们之前学的所有指令
        System.out.println(jedis.ping());
 
    }
}
```

```
jedis 理解事务

public class TestTX {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("IP", 6379);
        jedis.auth("password");
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello","world");
        jsonObject.put("name","lqh");
        //开启事务
        Transaction multi = jedis.multi();
        String s = jsonObject.toJSONString();
        try {
            multi.set("user1",s);
            multi.set("user2",s);
            multi.exec();//关闭事务
        } catch (Exception e) {
            multi.discard(); //放弃事务
            e.printStackTrace();
        }finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();//关闭连接
        }
    }

```

```
事务正常完成。当加入了异常，并且加锁以后

        jedis.flushDB();
 
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello","world");
        jsonObject.put("name","lqh");
        jedis.watch("user1");//加锁 监视
        //开启事务
        Transaction multi = jedis.multi();
        String s = jsonObject.toJSONString();
        try {
            multi.set("user1",s);
            multi.set("user2",s);
            int i = 1/0; //代码抛出异常事务，执行失败
            multi.exec();//关闭事务
        } catch (Exception e) {
            multi.discard(); //放弃事务
            e.printStackTrace();
        }finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();//关闭连接
        }
```

### 8、SpringBoot 集成 Redis

------



在 springboot2.X 之后，jedis 被替换为了 lettuce



jedis 采用直连，多个线程操作不安全，如果想避免安全问题，使用 jedis pool 连接池！ 更像 BIO 模式



lettuce：采用 netty，实例可以在多个线程中进行共享，不存在线程不安全的问题，可以减少线程数量。 更像 NIO 模式



源码分析



```
@Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    ) //我们可以自己定义一个redisTemplate来替换这个默认的
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        //默认的RedisTemplate没有过多的设置，redis中对象都是需要序列化的
        //两个泛型都是Object, Object的类型，我们之后需要强制转换<String, Object>
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
 
    @Bean
    @ConditionalOnMissingBean //由于Spring是redis中最常使用的类型，所以说单独提出来了一个bean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        return new StringRedisTemplate(redisConnectionFactory);
    }
整合测试
1、导入依赖

        <!--操作Redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
2、配置连接

#配置redis
spring.redis.host=IP
spring.redis.port=6379
spring.redis.password=passowrd
3、测试

    @Autowired
    private RedisTemplate redisTemplate;
 
    @Test
    void contextLoads() {
        //redisTemplate  操作不同的数据类型，api和我们的指令是一样的
        //opsForValue()  操作字符串 类似String
        //opsForList()   操作List  类似List
        //opsForSet()
        //opsForHash()
        //opsForZSet()
        //opsForGeo()
        //opsForHyperLogLog()
 
        //除了基本的操作，我们常用的方法都可以直接通过redisTemplate操作，比如事务和基本的CRUD
 
        //获取redis的连接对象
        //RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
        //connection.flushDb();
        //connection.flushAll();
 
        redisTemplate.opsForValue().set("mykey","hello");
        System.out.println(redisTemplate.opsForValue().get("mykey"));
    }
```

### 自定义 RedisTemplate

```
@Configuration
public class redisConfig {
    //可直接使用
    //编写我们自己的redisTemplate
    @Bean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
 
        // 序列化配置
        // 使用Jackson2JsonRedisSerialize替换默认序列化方式
        Jackson2JsonRedisSerializer<?> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        //String的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        //启用默认的类型
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,ObjectMapper.DefaultTyping.NON_FINAL);
        //序列化类，对象映射设置
        jackson2JsonRedisSerializer.setObjectMapper(om);
        //key使用String序列化
        redisTemplate.setKeySerializer(stringRedisSerializer);
        //value使用jackson序列化
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        //hash的key使用String序列化
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        //hash的value使用jackson序列化
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
 
}
```

### 9、Redis.conf 详解



1、单位 units



配置文件 unit 单位对大小写不敏感

![](https://pic1.imgdb.cn/item/67c9754d066befcec6df1ab4.png)

2、包含 includes 

![](https://pic1.imgdb.cn/item/67c9754d066befcec6df1ab3.png)

3、网络 network



```
bind 127.0.0.1 #绑定访问ip
protected-mode yes #保护模式 yes开启 no关闭
port 6379   #端口
```



 4、通用配置



```
daemonize yes  #以守护进程的方式进行,默认是no
pidfile /var/run/redis_6379.pid  #如果以后台（守护进程）的方式运行，我们就需要指定一个pid文件！
```



5、日志



```
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)   （生产环境）
# warning (only very important / critical messages are logged)
loglevel notice  #日志级别
logfile ""     #日志文件名
databases 16   #数据库个数   默认16
always-show-logo yes   #是否总是展示logo
```



6、快照（snapshotting）
持久化，在规定时间内，执行了多少次操作，则会持久化到文件 .rdb .aof
redis 是内存数据库，如果没有持久化，断电数据丢失



```
save 3600 1   #如果3600秒内，至少一个key进行了修改，那么就进行持久化操作
save 300 100   #如果300秒内，至少有100个key进行了修改，那么就进行持久化操作
save 60 10000  #如果60秒内，至少有10000个key进行了修改，那么就进行持久化操作
stop-writes-on-bgsave-error yes  #持久化如果出错，是否还需要继续工作
rdbcompression yes  #是否压缩.rdb文件（会消耗一定的cpu资源）
rdbchecksum yes    #保存.rdb文件的时候，进行错误检查校验
dir ./     #.rdb文件保存路径
```



7、安全（security）



```
requirepass root    #设置密码为root  默认是没有密码的
```



8、客户端限制（clients）



```
maxclients 10000   #最大连接数
```



9、内存设置（memory management）



```
maxmemory <bytes>  #最大内存容量
maxmemory-policy noeviction   #内存处理策略  内存达到上限之后的处理策略
    1、volatile-lru：只对设置了过期时间的key进行LRU（默认值）
    2、allkeys-lru ： 删除lru算法的key
    3、volatile-random：随机删除即将过期key
    4、allkeys-random：随机删除
    5、volatile-ttl ： 删除即将过期的
    6、noeviction ： 永不过期，返回错误
```



10、aof 配置（append only mode）



```
appendonly no   #aof模式默认不开启,默认是使用rdb方式持久化的，在大部分情况下,rdb完全够用
appendfilename "appendonly.aof"  #.aof文件名
# appendfsync always  #每次修改都会执行同步，消耗性能
appendfsync everysec  #每秒执行一次同步，但是可能会丢失这1s的数据
# appendfsync no      #不执行同步，这个时候操作系统自己同步数据，速度最快
####重写规则####
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb  #如果aof文件大于64mb，太大了，fork一个新的进程来讲我们的文件进行重写。
```



### 10、Redis 的持久化

#### 1、RDB（Redis DataBase）



在指定时间间隔将内存中的数据集体快照写入磁盘，也就是行话讲的 snapshot 快照，它恢复时是将快照文件直接读到内存中。



Redis 会单独创建（fork）一个子进程来进行持久化，会先将数据写入一个临时文件中，待持久化过程结束后，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何 I/O 操作的。这就确保了极高的性能。如果需要大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那 RDB 方式要比 AOF 方式更加高效。



优点：
1、适合大规模数据恢复。
2、对数据完整性要求不高。



缺点：
1、需要一定时间间隔进程操作，如果在时间间隔内 redis 宕机，最后一次持久化后的数据丢失。
2、fork 子进程的时候，会占用一定的内存空间。



默认的持久化方式就是 RDB 方式，一般情况下不需要修改这个配置。默认保存的 rdb 文件为 **dump.rdb**



触发规则：
1、save 的规则满足的情况下，自动触发 rdb 规则
2、flushall 命令执行后，自定触发 rdb 规则
3、退出 redis 时，自动触发 rdb 规则



**恢复 rdb 文件**：
1、只需要将 rdb 文件放在 redis 的启动目录就可以，redis 启动的时候会自动检查 dump.rdb 文件，自动恢复 rdb 文件中的数据

![image-20250303111413944](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303111413944.png)

#### 2、AOF（Append Only File）



aof 默认是不开启的，需要手动开启（appendonly yes）。aof 默认保存的是 appendonly.aof 文件。
重启，redis 就可以生效了

![image-20250303111507392](https://pic1.imgdb.cn/item/67c972d1066befcec6df19e4.png)



*将所有执行过的写命令都记录下来，history，在恢复的时候将记录的命令全部执行一遍。*



以日志的形式来记录每个写操作，将 Redis 执行过的的所有指令记录下来（读操作不记录），只许追加文件不许改写文件，redis 启动时，会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容，将写指令从前到后执行一次以完成数据的恢复工作。



 **如果 aof 文件损坏或有错误，redis 无法启动。我们需要修复 aof 文件。**aof 文件损坏修复: redis-check-aof —fix appendonly.aof





![](https://pic1.imgdb.cn/item/67c975d0066befcec6df1ad4.png)

如果文件正常，重启就可以启动恢复了

![](https://pic1.imgdb.cn/item/67c975d0066befcec6df1ad3.png)





**重写规则说明**



**aof 默认的是文件的无限追加，文件会越来越大

![](
https://pic1.imgdb.cn/item/67c975d1066befcec6df1ad5.png)

 如果 aof 文件大于 64m，太大了。fork 一个新的进程来将我们的文件进行重写



**优点和缺点**



```
appendonly no #默认是不开启aof模式的，默认是使用rdb方法持久化，在大部分情况下rdb够用了
appendfilename "appendonly.aof"  #持久化文件的名字
 
# appendfsync always  #每次修改都会执行同步，消耗性能
appendfsync everysec  #每秒执行一次同步，但是可能会丢失这1s的数据
# appendfsync no      #不执行同步，这个时候操作系统自己同步数据，速度最快
```



优点：



1、每次修改都会执行同步，消耗性能, 文件数据完整性更好。
2、每秒执行一次同步，但是可能会丢失这 1s 的数据
3、不执行同步，这个时候操作系统自己同步数据，速度最快



缺点：
1、相对于数据文件来说，aof 文件远远大于 rdb 文件，修复的速度也较慢。
2、aof 运行效率也要比 rdb 慢，所以我们 redis 默认的配置就是 rdb 持久化。



**扩展：**
1、RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2、AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF 命令以 Redis 协议追加保存每次写的操作到文件末尾，Redis 还能对 AOF 文件进行后台重写，使得 AOF 文件的体积不至于过大。
**3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化**



4、同时开启两种持久化方式



- 在这种情况下，当 redis 重启的时候会优先载入 AOF 文件来恢复原始的数据，因为在通常情况下 AOF 文件保存的数据集要比 RDB 文件保存的数据集要完整。
- RDB 的数据不实时，同时使用两者时服务器重启也只会找 AOF 文件，那要不要只使用 AOF 呢？作者建议不要，因为 RDB 更适合用于备份数据库（AOF 在不断变化不好备份），快速重启，而且不会有 AOF 可能潜在的 Bug，留着作为一个万一的手段。



5、性能建议



- 因为 RDB 文件只用作后备用途，建议只在 Slave 上持久化 RDB 文件，而且只要 15 分钟备份一次就够了，只保留 save 900 1 这条规则。
- 如果 Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只 load 自己的 AOF 文件就可以了，代价一是带来了持续的 IO，二是 AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少 AOF rewrite 的频率，AOF 重写的基础大小默认值 64M 太小了，可以设到 5G 以上，默认超过原大小 100% 大小重写可以改到适当的数值。
- 如果不 Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔 IO，也减少了 rewrite 时带来的系统波动。代价是如果 Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB 文件，载入较新的那个，微博就是这种架构。

## 11、Redis 发布订阅



Redis 发布订阅（pub/sub）是一种**消息通信模式**：发送者（pub）发送信息，订阅者（sub）接收信息。



Redis 客户端可以订阅任意数量的频道。



订阅 / 发布消息图：
第一个：消息发送者 第二个：频道 第三个：消息订阅者

![image-20250303163540353](https://pic1.imgdb.cn/item/67c972cc066befcec6df19dd.png)



![image-20250303163553324](https://pic1.imgdb.cn/item/67c972cd066befcec6df19de.png)

**命令**



| 命令                                   | 描述                               |
| -------------------------------------- | ---------------------------------- |
| PSUBSCRIBE pattern [pattern..]         | 订阅一个或多个符合给定模式的频道。 |
| PUNSUBSCRIBE pattern [pattern..]       | 退订一个或多个符合给定模式的频道。 |
| PUBSUB subcommand [argument[argument]] | 查看订阅与发布系统状态。           |
| PUBLISH channel message                | 向指定频道发布消息                 |
| SUBSCRIBE channel [channel..]          | 订阅给定的一个或多个频道。         |
| UNSUBSCRIBE channel [channel..]        | 退订一个或多个频道                 |



**测试**



订阅端



```
[root@iZbp180bogui4iuorjl7ruZ bin]# redis-cli -p 6379
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> SUBSCRIBE javastudy  #订阅频道
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "javastudy"
3) (integer) 1
#等待读取推送的信息
1) "message"  #消息
2) "javastudy" #接收频道
3) "hello"    #接收消息
1) "message"
2) "javastudy"
3) "world"
```



发送端



```
[root@iZbp180bogui4iuorjl7ruZ bin]# redis-cli -p 6379
127.0.0.1:6379> PUBLISH javastudy "hello"  #向指定频道部分消息
(integer) 1 
```



**原理**

Redis 是使用 C 实现的，通过分析 Redis 源码里的 pubsub.c 文件，了解发布和订阅机制的底层实现，籍此加深对 Redis 的理解。



Redis 通过 PUBLISH 、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。



每个 Redis 服务器进程都维持着一个表示服务器状态的 redis.h/redisServer 结构， 结构的 pubsub_channels 属性是一个字典， 这个字典就用于保存订阅频道的信息，其中，字典的键为正在被订阅的频道， 而字典的值则是一个链表， 链表中保存了所有订阅这个频道的客户端。

![image-20250303163633297](https://pic1.imgdb.cn/item/67c972cd066befcec6df19e0.png)



## 12、Redis 主从复制



### 概念



主从复制，是指将一台 Redis 服务器的数据，复制到其他 Redis 的服务器。前者称为主节点（master/leader），后者称为从节点（slave/follower）；数据的复制是单向的，只能从主节点到从节点。Master 以写为主，Slave 以读为主。



主从复制，读写分离！80% 的情况下都是进行读的操作！减缓服务器压力！



**默认情况下，每台 Redis 服务器都是主节点；且一个主节点可以有多个从节点（或没有从节点），但是一个从节点有且只有一个主节点。**



**主从复制的作用主要包括**



- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
- 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点），分担服务器负载；尤其是写少读多的场景下，通过多个节点分担负载，可以大大提高 Redis 服务器的并发量。
- 高可用（集群）基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是 Redis 高可用的基础。



一般来说呀，要将 redis 运用于工程项目之中，只使用一台 Redis 是万万不能的（宕机），原因如下：
1、从结构上，单个 redis 服务器会发生单点故障，并且一台服务器需要处理所有的请求负载呀，压力较大。
2、从容量上，单个 redis 服务器内存容量有限，就算一台 redis 服务器内存容量为 256GB，也不能将所有内存用作 Redis 存储内存，**一般来说，单台 Redis 最大使用内存不应该超过 20GB。**



### 环境配置



只需要配置从库，不需要配置主库，因为默认情况下，每台 Redis 服务器都是主节点。



```
127.0.0.1:6379> info replication  #查看当前库的信息
# Replication
role:master   #角色  master
connected_slaves:0   #从库个数
master_failover_state:no-failover
master_replid:ada8f9fad2a4f4215bc6ddc844a4def15e96ee38
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```



复制 3 个配置文件，然后修改对应信息：
1、端口
2、pid
3、log 文件名字
4、rdb 备份文件名字

![image-20250303163700981](https://pic1.imgdb.cn/item/67c972cd066befcec6df19e1.png)

### 一主二从 



默认情况下，每台 redis 服务器都是主节点；我们只需要配置从节点就好了（认老大）。



一主（79）二从（80,81）



#### **方式一：命令配置 （临时）**



```
############ 从节点1 ###########
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379  #设置为主节点的从节点（认老大）
OK
127.0.0.1:6380> info replication
# Replication
role:slave   #当前角色为从节点
master_host:127.0.0.1   #主节点信息
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_read_repl_offset:14
slave_repl_offset:14
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:88094280f7464b766cb067bcac5db93fffb5d16d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14
 
############ 从节点2 ###########
127.0.0.1:6381> SLAVEOF 127.0.0.1 6379   #设置为主节点的从节点（认老大）
OK
127.0.0.1:6381> info replication
# Replication 
role:slave   #当前角色为从节点
master_host:127.0.0.1   #当前角色为从节点
master_port:6379
master_link_status:up
master_last_io_seconds_ago:3
master_sync_in_progress:0
slave_read_repl_offset:630
slave_repl_offset:630
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:88094280f7464b766cb067bcac5db93fffb5d16d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:630
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:631
repl_backlog_histlen:0
 
############ 主节点 ###########
127.0.0.1:6379> info replication
# Replication
role:master   #当前角色为主节点
connected_slaves:2   #从节点个数
slave0:ip=127.0.0.1,port=6380,state=online,offset=770,lag=1   #从节点信息
slave1:ip=127.0.0.1,port=6381,state=online,offset=770,lag=0   #从节点信息
master_failover_state:no-failover
master_replid:88094280f7464b766cb067bcac5db93fffb5d16d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:770
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:770
 
```



真实的主从配置应该在配置文件中配置，这样的话是永久的，我们这里使用的是命令，是暂时的



#### **方式二：配置文件 （永久）**



修改配置文件，设置主节点信息

![image-20250303163725897](https://pic1.imgdb.cn/item/67c972cc066befcec6df19dc.png)

**细节**



**主节点可以写，从节点不能写只能读！主节点中所有信息和数据都会自动被从节点保存！** 



```
############### 主节点（可以写） ###############
127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> keys *
1) "k1"
127.0.0.1:6379> get k1
"v1"
 
############### 从节点（不可写，只能读） ###############
127.0.0.1:6380> keys *
(empty array)
127.0.0.1:6380> keys *
1) "k1"
127.0.0.1:6380> get k1
"v1"
127.0.0.1:6380> set k2 v2
(error) READONLY You can't write against a read only replica.
127.0.0.1:6381> keys *
(empty array)
127.0.0.1:6381> keys *
1) "k1"
127.0.0.1:6381> get k1
"v1"
127.0.0.1:6381> set k2 v2
(error) READONLY You can't write against a read only replica.
```



**注意：
1、主节点宕机，从节点依旧以从节点的角色连接主节点，没有写的权限；当主节点重新连接，从节点依旧可以直接获取到主节点写的信息！
2、如果是用命令行配置的从节点，从节点重启，主从配置失效！但是当重新配置成为从节点，立刻就会从主节点中获取值！
3、如果是配置文件配置的从节点，从节点宕机，主节点进行写操作，当从节点重新连接时，会从主节点中获取值，之前主节点写操作的信息依旧在此从节点存在！**



#### 复制原理



Slave 启动成功连接到 Master 后会发送一个 sync 同步命令！
Master 接到同步命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕后，**Master 将传送整个数据文件到 Slave，并完成一次同步！**



全量复制：Slave 服务在接收到数据库文件数据后，将其存盘并加载到内存中。



增量复制：Master 继续将新的所有收集到的修改命令依次传给 Slave，完成同步。



但是只要重新连接 Master，一次完全同步（全量复制）将自动执行。数据一定会在 Slave 中看到。



#### 宕机后手动配置主节点



链路模型：M-S(M)-S , 当第一个主节点存活时，S(M) 节点为从节点，无法写操作!



当第一个主节点宕机后，可以手动将节点设置为主节点:



```
slaveof no one #使自己变成主节点
```



如果第一个主节点恢复，需要重新配置。



## 13、哨兵模式（自动选举老大的模式）



**概述**





主从切换技术的方法是：当主节点宕机后，需要手动把一台从节点切换为主节点，这就需要人工干预，费时费力，还会造成一段时间内的服务不可用。这不是一种推荐的方式，更多的时候，我们优先考虑哨兵模式。Redis 从 2.8 开始，正式提供了 Sentinel（哨兵）架构来解决这个问题。



能够后台监控主节点是否故障，如果故障了根据投票数自动将从节点转换为主节点。



哨兵模式是一种特殊的模式，首先 Redis 提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它能独立运行。其原理是**哨兵通过发送命令，等待 Redis 服务器响应，从而监控运行的多个 Redis 实例**

![image-20250303163751454](https://pic1.imgdb.cn/item/67c972c7066befcec6df19d6.png)



![image-20250303163801688](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163801688.png)



假设主服务器宕机，哨兵 1 先检测到这个结果，系统并不会马上进行 failover（故障转移）过程，仅仅是哨兵 1 主观认为主服务器不可用，这个现象称为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行 failover（故障转移）操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。



**测试**



1、配置哨兵配置文件 sentinel.conf



```
# sentinel monitor 被监控的名称 host port 1
sentinel monitor headredis 127.0.0.1 6379 1
```



后面这个数字 1，代表主机挂了，slave 投票看让谁接替成为主机，票数最多的，就会成为主机。



“1” 这个参数值得是 < quorum>，即多少个主节点检测到主节点有问题就进行故障转移。其实就是有多少个哨兵（sentinel）认为这个主节点主观下线才算真正的下线，然后进行故障转移，让其客观下线



2、启动哨兵



```
[root@iZbp180bogui4iuorjl7ruZ bin]# redis-sentinel lqhconfig/sentinel.conf 
24222:X 17 Feb 2022 20:08:18.995 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
24222:X 17 Feb 2022 20:08:18.995 # Redis version=6.2.6, bits=64, commit=00000000, modified=0, pid=24222, just started
24222:X 17 Feb 2022 20:08:18.995 # Configuration loaded
24222:X 17 Feb 2022 20:08:18.996 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.2.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 24222
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               
 
24222:X 17 Feb 2022 20:08:19.003 # Sentinel ID is c01306dba4facd34ffb19a0758ffc524f60cb4dd
24222:X 17 Feb 2022 20:08:19.003 # +monitor master myredis 127.0.0.1 6379 quorum 1 #主节点信息
24222:X 17 Feb 2022 20:08:19.004 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379  #从节点1信息
24222:X 17 Feb 2022 20:08:19.011 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379  #从节点2信息
```



如果 master 节点断开了，这个时候就会从从机中随机选择一个服务器作为主机！（投票算法）



```
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> SHUTDOWN
not connected> exit
```

![image-20250303163831774](https://pic1.imgdb.cn/item/67c972c7066befcec6df19d7.png)



![image-20250303163843447](https://pic1.imgdb.cn/item/67c972c8066befcec6df19d8.png)

![](https://pic1.imgdb.cn/item/67c97735066befcec6df1b04.png)





![](https://pic1.imgdb.cn/item/67c97736066befcec6df1b05.png)

 当之前宕机的主节点重新连接，之前宕机的主节点会自动被哨兵转换成为新选举的主节点的从节点，这就是哨兵模式的规则





**哨兵模式**



优点：
1、哨兵集群，基于主从复制模式，所有的主从配置的优点，它都有。
2、主从可以切换，故障可以转移，系统的可用性更好。
3、哨兵模式就是主从模式的升级，手动到自动，更加健壮！



缺点：
1、Redis 不好在线扩容，集群容量一旦达到上限，在线扩容就十分麻烦！
2、实现哨兵模式的配置其实是很麻烦的，里面有很多选择！



### 哨兵模式的全部配置



完整的哨兵模式配置文件 sentinel.conf



```
# Example sentinel.conf
# 哨兵sentinel实例运行的端口 默认26379
# 如果有多个哨兵，还需要配置每个哨兵的端口
port 26379
# 哨兵sentinel的工作目录
dir /tmp
# 哨兵sentinel监控的redis主节点的 ip port 
# master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 1
# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
这个数字越小，完成failover所需的时间就越长，
但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。  
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
# SCRIPTS EXECUTION
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则：
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
#这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，
#一个是事件的类型，
#一个是事件的描述。
#如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
#通知脚本
# sentinel notification-script <master-name> <script-path>
  sentinel notification-script mymaster /var/redis/notify.sh
# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。 
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```













## 14、Redis 的缓存穿透与雪崩（面试高频，工作常用~）



**服务的高可用问题**



Redis 缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。其中，最要害的问题，就是数据一致性问题。从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。



另外的一些典型问题就是，缓存穿透、缓存雪崩和缓存击穿。目前，业界也都有比较流行的解决方案。



### 缓存穿透



#### **概念**



缓存穿透的概念很简单，用户想要查询一个数据，发现 redis 内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就出现了缓存穿透。



#### 解决方案



**1、布隆过滤器**
布隆过滤器是一种数据结构，对所有可能查询的参数以 hash 形式存储，在控制层先校验，不符合则丢弃，从而避免了对底层存储系统的查询压力；



![image-20250303163055419](https://pic1.imgdb.cn/item/67c972d1066befcec6df19e5.png)



**2、缓存空对象**
当存储层不命中后，即使返回的空对象也将其缓存起来，同时设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源；

![image-20250303163111987](https://pic1.imgdb.cn/item/67c972d1066befcec6df19e6.png)





但是这种方法存在两个问题：



- 如果空值能被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键；
- 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对需要保持数据一致性的业务会有影响。



### 缓存击穿



#### 概念



这里需要注意和缓存穿透的区别。缓存击穿，是指一个 key 非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个 key 在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在屏障上凿开了一个洞。



当某个 key 在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导致数据库压力瞬间变大。



#### 解决方案



**1、设置热点数据不过期**
从缓存层面来看，没有设置过期时间，所以不会出现热点 key 过期后产生的问题。



**2、加互斥锁**
分布式锁：使用分布式锁，保证对于每个 key 同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁考验很大。



### 缓存雪崩



#### 概念



缓存雪崩，是指在某一时间段，缓存集中过期失效。Redis 宕机！



产生雪崩的原因之一，比如零点抢购，商品时间集中放入缓存，假设缓存一小时。那么 1 点的时候，大量缓存集体过期，对于这批商品的访问查询，都落到了数据库。对于数据库而言，就会产生周期性的压力波峰，于是所有请求都会到达存储层，存储层调用会暴增，造成存储层也会挂掉的情况。

![](https://pic1.imgdb.cn/item/67c972d2066befcec6df19e8.png)



其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因此自然形成的缓存雪崩，一定是在某一时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。



#### 解决方案



**1、Redis 高可用**
这个思想的含义是，既然 Redis 也有可能挂掉，那多增加几台 redis 服务器，这样一台挂了还有其他的可以继续工作，其实就是搭建集群。



**2、限流降级**
这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读取数据库写缓存的线程数量。比如对某个 key 只允许一个线程查询数据和写缓存，其他线程等待。



**3、数据预热**
数据加热的含义就是在正式部署之前，先把可能的数据先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的 key，设置不同的过期时间，让缓存失效的时间尽量均匀。