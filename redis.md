



# Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API

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

![image-20250303090621743](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303090621743.png)

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





![img](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAhsAAAC8CAYAAADRoWzCAAAgAElEQVR4AezBP2hcB6Iv4E+JOElsxmGCJptkQBoQAaMYw3bDNIYERJotxoUrVyqmMQuu5jVTurlTHVjcqFCVysVOkeYi8EIaMd2CSETADMxd8O7GEhGROFp7cPg9/4k3zu5N7E3i9zb3zvct3L59O81m09zc3Nzc3Nzci/CSubm5ubm5ubkX6CVzc3Nzc3Nzcy/QS+bm5ubm5ubmXqCXzM3Nzc3Nzc29QC+Zm5ubm5ubm3uBXjI3Nzc3Nzc39wK9ZG5ubm5ubm7uBXrJ3Nzc3Nzc3NwL9JK5ubm5ubm5uRfoJXNzc3Nzc3NzL9BL5ubm5ubm5uZeoJfM/T/WVh9UGh3/nlpDzcFQYe7n1VYfVBod/4+01QeVRscvQ2dkZTBU+Bd1RlYGQ4WfQ1t9UGl0/Iza6oNKo+MXrK0+qDQ6vtUaag6GCv+s6O5aLivLZWVlMFSY+662+qDS6PhvFd1dy2VlpdxVb/mf4/bt20EQBEEQBEEQBEEQBEEQRGeU5bJKoyMIojPKSm8jCIIgCIIgSNHdTbPbDoIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCFJ0d9PstoMgCIIgCIIgCIIgCIIgCIIgCIIgCIIgiNYwzcEwBUEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQRGeUlcEwBUEQBEEQBEEQRGeUlcEwBUEQBEEQBEEQBEEQBEEQBEEQpOjuptltB0EQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEG0hmkOhikIgiAIguiMsjIYpiAIgiAIgiAIgiAIgiAIUnR30+y2gyAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiBspFGOUiMIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAveZbWUHMwVKDW21VveS6n3h8qzM3Nzc3NzT2X1lmF/5kWPcs77/LZf5jZ8PraxN1Nz7RwsO3EFa93+vZ3fFdrqHH5iteW+Hrvuv3NvpkHOiPLl9Y9dtPyBRZs+/Jq17GH2uq9TbW1VQsmTj7p2R+NPVNrqHH5iteW+Hrvuv3NvpkHWkPNy5zc+VBtbdWCiaPyvMOpH9Yaal5m/1rfzAOdkZX3b/nLtb5Za6h5mZM7H6qtrVowcVSedzj1SNHd9daFVQ/97cZp+zse64wsX1r32E3LF1iw7curXccouiONC+texsLBtqOPug6nfqS2+mCTj847nHrKhkZ51f0b/+mh071db62tWjBxVJ53OPWNtnpvU21t1YKJk0969kdjWkPNy5zc+VBtbdWCiaPyvMOp79caal7+EKsWbTu6s6q2turrTz5wezT2WFu9t6m2tmrBxMknPft/vKh5+UOsWrTt6M6q2tqqrz/5wO3R2COtocblK15b4uu96/Y3+2aeaKsPNi1+NlFcWPcyFg6u+8u1z71eXnW/PO9w6rHOyMr7t/zlWt/M9yu6I40L617GwsG2o4+6DqceKbq73rqw6qG/3Thtf8c3NjTK33nNtxYOrvvLtb6Zh9rqvU21tVULJk4+6dkfjT1L0d311oVVD/3txmn7O/6u6I40Lqx7GQsH244+6jqc+mGtoeZlTu58qLa2asHEUXne4dQ32uq9TbW1VQsmTj7p2R+NPdZW722qra1aMHHySc/+aOyR1lDz6hWLJk72Jr6rrd7bVFtbtWDi5JOe/dHYI62h5tUrFk2c7E38HIrurrcurHrobzdO29/xWGdk+dK6x25avsCCbV9e7Tr2w4rurrcurHrobzdO29/xd0V3pHFh3ctYONh29FHX4dQPaw01L3Ny50O1tVVf7123v9k3843WUOPyFa8t8fXedfubfTOPFd2RxoV1L2PhYNvRR12HU48U3ZHGhXUvY+Fg29FHXYdTjxTdXW9dWPXQ326ctr/jO073dr21turrvev2N/tmnkdbvbeptrZqwcTJJz37o7Ef1BlZvrTusZuWL7Bg25dXu4490BpqXL7itSW+3rtuf7Nv5tmK7kjjwrqXsXCw7eijrsMpWkPNy5zc+VBtbdXXe9ftb/bNPNFW722qra1aMHHySc/+aOyxtnpvU21t1YKJkxs9+ztjDxXdXW9dWPXQ326ctr/jG231wU21JY/Uy8obJo7K8w6nGxrlVffL8w6nHuuMrLx/y39d6/vFuH37dhAEQRCdUZbLKstlleWyynJZZbmsslLupt4SBEEQBNEZZWUwTNEZZWUwTEF0RlnpbYR26oMqjU47tFPrVWl220EQpOjuptltB0EQpNar0uxupCBaG2kMqjQ6giAIgiAI7dQHVRqddmin1qvS7LaDaA3TLKs0Ou0gRXc3K72NIAiCIAiCaA3THAxTEERnlJXBMAXRGqZZVml02kGK7m5WehtBEIR26oMqjY4gCIIU3d00u+0gCKI1TLMcpd4SRGsj9d4wBUEQrWGaZZVGRxAEQRAEQZBar0qjIwiCaA3TLEeptYZpllUanXaQorubld5GEKTWq9LsbqQgWhtpDKo0OqI1TLOs0ui0gxTd3az0NoIgCIIgiNYwzXKUmnbqgyrNbjs6o6wMhikIUutVaXY3UhCtjTQGVRrdYZrlKDXt1AdVmt12dEZZGQxTENqpD6o0Ou3QTq1XpdltB0Fopz6osjIYpiAIgtR6VZrddhCk1qvS6AiCIAiCIFrDNMtR6i1BtDZS7w1TEAShnfqgSqMjCIIgCO3UB1Wa3XYQpNar0uxupCBaG2kMqjQ6giAIgiAIgtBOfVCl0REE0RqmWY5SbwmitZF6b5iCIAiCIAiiNUyzrNLotIMU3d2s9DaCILVelWZ3IwXR2khjUKXREaTWq9LsbqQgWhtpDKo0OkI79UGVRqcd2qn1qqwMhikIUutVaXY3UhCtjTQGVRodoZ36oEqj0w7t1HpVVgbDFARBEARBEARBEARBEIR26oMqjY4gCIIU3d00u+0gCIIgCIIgCILQTn1QpdERBNEaplmOUm8JorWRem+YgiAIgiAIojVMs6zS6LRDO7VelWa3HYR26oMqjU47tFPrVWl220G0hmmWo9RbgmhtpN4bpiBawzTLUeotQbQ2Uu8NUxAEoZ36oEqjIwiiNUyzrNLotEM7tV6VZrcdBEF0RlkZDFMQBEFqvSrN7kYKorWRxqBKoyMIgiAIgiBI0d1Ns9sOgiC0Ux9UaXTaoZ1ar0qz2w6CIAiCIIjWMM1ylHpLEK2N1HvDFERrmGZZpdFph3ZqvSrNbjsIUutVaXY3UhCtjTQGVRodQWq9Ks3uRgpCO7XuRgqCILRTH1RpdARBEERrmGY5So0gCFLrVWl220GQWq9KoyMIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgrzk++x0/enqB44Pth1ePe2vn0x8/ckH/uvqeYdTz7bzH4586HTLt1oXnVradndnjLHjT7ctNtY8U2vozNq2o9GWmQemW776bKL4VdsPal10amnb3Z0xxo4/3bbYWPOtbXd3xh6afTHhzbMKP9W2uztjD82+mPDmWYWfapV32h6Zbjnc7Jv58e7tTxS/amNDo9xVb+Gddy3ufezYQ9vu7ow9NPtiwptnFR5oDZ1Z23Y02jLzwHTLV59NFL9qe2zb3Z2xh2ZfTHjzrMIzHNxyz9j9O8y+GPPnW+77RmvozNq2o9GWmQemW776bKJovMvBLfeM3b/D7Isxf77lvm+0Ljq1tO3uzhhjx59uW2ys+Ucnf+ib+a7jT7ctvndR4aENr65tu7vjOazyTtsj0y2Hm30zz6/objpz57duj8YeaQ2dWdt2NNoy88B0y1efTRS/avtpVnmn7ZHplsPNvpnnse3uzthDsy8mvHlW4YHW0Jm1bUejLTMPTLd89dlE8as2raEza9uORltmHphu+eqzieJXbVoXnVradndnjLHjT7f9XWvozNq2o9GWmQemW776bKL4VZvWRaeWtt3dGWPs+NNtv0yrvNP2yHTL4WbfzPPYdndnjLHjT7ctNtY80rro1NK2uztjjB1/um2xseZbq7zT9sh0y+Fm38wTq7zT9sh0y+Fm38zz2HZ3Z4yx40+3LTbWPFNr6MzatqPRlpkHplu++myi+FXbj9a66NTStrs7Y4wdf7ptsbHm+azyTtsj0y2Hm30zT2y7uzPG2PGn2xYbax5pDZ1Z23Y02jLzwHTLV59NFL9q0xo6s7btaLRl5qGx49GWmZ/m+NNti+9dVHhow6tr2+7u+EVZ9ENaF51yy7620+9x8tHY8xs7/MPEyvqGw0996+CWe57y5lkFZp5lXb2s1H1rYW8NYz/o4JZ7nvLmWQVmHji45Z6f2cEt9/yMpn37N3j9/U3NS6sWTRyV5x1OfWvad/tq3/Oa/fE/uXxR8UcccOrXbZVV9/f3cJaDW+75PuvqZaXuWwt7ax45uOWen9u6elmp+9bCAffd8oMObrnnKW+eVWDmiYn7f/bPdj52cumq060+v77q1F5p3zNM+/Zv8Pr7m5qXVi2aOCrPO5x6Pp2Rty9MfHl1y3etq5eVum8t7K1h7EeZ9u3f4PX3NzUvrVo0cVSedzj1bAe33PN91tXLSt23FvbWPLauXlbqvrWwt+aRg1vu+T7r6mWl7lsLe2seObjlnl+wad/+DV5/f1Pz0qpFE0fleYdTz3Zwyz1PefOsAjMPHNxyz1PePKvAbNq3f4PX39/UvLRq0cRRed7hFNO+/Ru8/v6m5qVViyaOyvMOp57t4JZ7nvLmWQVmnmVdvazUfWthbw1jP9rBLfc85c2zCsz8gGnf/g1ef39T89KqRRNH5XmHU48d3HLPU948q8DMQ+vqZaXuWwt7ax45uOWen9nOx04uXXW61efXV53aK+37ZVn0PRpl5TWPvVVe8cjVyqlPPnB7NPZcdj52cumq+v7E3y296xXMfOPO52aew8F1f73WN/MvWnrXK5j5xp3PzfyyzHb69nf6Hiq6I29fHqqu9c38SNPPzZZ+4/Svuf9RyfpFp62afTrGRT/o4Lq/Xuub+QetoRfi4Lq/XuubeUprqHnZD1t61yuY+cadz808jy1ffXJV49cbvLfq5A9bnsdsp29/p++hojvy9uWh6lrfzDO0hpqXVh2V5x37BwfX/fVa38zPZ7bTt7/T91DRHXn78lB1rW/mJzi47q/X+mb+QWvIwXV/vdY38w9aQ5be9Qpm/hsH1/31Wt/MP2gNWXrXK5j55Zrt9O3v9D1UdEfevjxUXeubeYald72CmW/c+dzMN5be9QpmvnHnczOPzXb69nf6Hiq6I29fHqqu9c0w2+nb3+l7qOiOvH15qLrWN/MMS+96BTPfuPO5medwcN1fr/XN/IyW3vUKZr5x53Mzzzbb6dvf6Xuo6I68fXmoutY388DSu17BzDfufG7mGwfX/fVa38w/aA1ZetcrmPk5bfnqk6sav97gvVUnf9jyS/OS77F/9bTDvYm/3TjtTze2Lez91p+unnZ7NPb8tnz1CbUL6x6Z/t7JwbpXO2201c6tu7+/5x8tNi4qPGXad3Tnika37YmiM1Lv+GHT3zs5WPdqp4222rl19/f3/CTTz82WPnS65YG22rl1P6fFxkWFp7SGGt0NhSdWufO5mae0hpplpdHxnPbcP1h35j3uT7fcdcWZtW13d/ywad/RnSsa3bYnis5IvePFmPYd3bmi0W17ouiM1H/th01/7+Rg3audNtpq59bd39/zvGZ//E8u/M6ZpW13dzxba6jR3VB4YpU7n5t5lrb65StmN847nPquad/RnSsa3bYnis5IvePHaw01uhsKT6xy53MzP8G07+jOFY1u2xNFZ6TewbTv6M4VjW7bE0VnpN7B9PdODta92mmjrXZu3d9N+47uXNHotj1RdEbqHUx/7+Rg3audNtpq59b9k9ZQs6w0On5Wi42LCj+D1lCju6HwxCp3PjfzPNa92mmjrXZu3f39PY9Mf+/kYN2rnTbaaufW3d/f80hrqNHdUHhilTufm3mgNdTobig8scqdz808j3Wvdtpoq51bd39/z3f8+Zb7S+96xVOmfUd3rmh0254oOiP1jue22Lio8JTp750crHu100Zb7dy6+/t7nqk11OhuKDyxyp3PzTyx7tVOG221c+vu7+95ZNp3dOeKRrftiaIzUu9g2ne0t+5Md0PhobZad0Php5v98T+58Dtnlrbd3fGL85LvteHVtYm7OxS/WnXy6ZYfYzYq/c0TY4cfXef9m5bLm864bn809rTZqHTy5hVvlZWVcqTmsePND5w0Ni2XleWy0jj3sWrHM4wdfnSd929aLm8647r90dhPs+WrT6hdrayUmxb3tz2fDY2yslzeVFvitUuV5bLS6Pi72ah08uYVb5WVlXKk5oFp31d+o1FWlsvK2+9NfLm95acZqz6bcOdjx7i3P+Hglnue7XjzAyeNTctlZbmsNM59rNrxwhxvfuCksWm5rCyXlca5j1V/9Axjhx9d5/2blsubzrhufzT23Ka/d3KAvY8dew7Tvq/8RqOsLJeVt9+b+HJ7y2MbGmVlubyptsRrlyrLZaXRQeuiU0u8dqmyXFaWy8rKYKjw2PHmB04am5bLynJZaZz7WLXjGTY0yspyeVNtidcuVZbLSqODad9XfqNRVpbLytvvTXy5veWnOt78wElj03JZWS4rjXMfq3Y8crz5gZPGpuWyslxWGuc+Vu14YOzwo+uKSzetlJtete1px5sfOGlsWi4ry2Wlce5j1Y4Hxg4/uq64dNNKuelV2/7JO+9atO3ujue0oVFWlsubaku8dqmyXFYaHX83G5VO3rzirbKyUo7UPMuGRllZLm+qLfHapcpyWWl0MO37ym80yspyWXn7vYkvt7c8l4Pr7p/btFzedMZ1+6Oxx8YOP7rO+zctlzedcd3+aOyRad9XfqNRVpbLytvvTXy5veWRad9XfqNRVpbLytvvTXy5veWxDY2yslzeVFvitUuV5bLS6Hjs4Lr75zYtlzedcd3+aOw7pn1He6vqZWWlHKl57HjzAyeNTctlZbmsNM59rNrxXGaj0smbV7xVVlbKkZqHxg4/us77Ny2XN51x3f5o7JmmfV/5jUZZWS4rb7838eX2lr87uO7+uU3L5U1nXLc/GnviePMDJ41Ny2Vluaw0zn2s2vHI8eYHThpXvVVWVspNr36xZ+ahDY2yslzeVFvitUuV5bLS6Hg+0987OcDex479At2+fTsIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiC0Ux9UaXQEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEqfWqNLvtIAiCIAiCIAiCIAiCIAiCIAiCIAiCIAiCIAiCIAiCIAiCIAiCIAiCaA3THAxTEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEK1hmoNhCoIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgtFMfVGl0BEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBFk0Nzf3Xa2LTi1tO9ox9wt0vHnasbm5/2FaF51a2na04xdp0dzc3N/VepU31iZObpx3bG5ubu7/v1qv8sbaxMmN8479Mi3cvn07zWbT3Nzc3Nzc3NyL8JK5ubm5ubm5uRfoJS9MW31QaXT8jNrqg0qj48VqDTUHQ4VfrqK7a7msLJeVlcFQ4V9XdHctl5WVcle95d9W0d21XFZWyl31lh+vNdQcDBXmfrTOyMpgqPD/V9HdtVxWVspd9ZYXozXUHAwV5ubmnmXRCzN2eO20/07R3dXQc3s09q8ZO7x22s+p6O5q6Lk9GvufZDY6708jdEZW3vcjbHj9wsTh1fOO/Tvb8PqFicOr5x17PkV311sXVj309ScfuD0a+2na6oObakv+bsHEUXne4dQDbbXupjMXVr2Mr/eu29/sm3VGli+te2LBxMmNnv2dsee3oVH+TvHJB26PxnRGli+t+9uN0/Z3PLChUf7Oqb3f+q/NLf87bHj9wsTh1fOO/TyK7q6Gntujsbm5uX/domdpDTUvs3+t75XersXt8w6n5v6na51V4K5/c62zCtz1/Gaj8/40oujuavi5TByX5x1O/ZNa76Y3/NZfrm6ZoegMne4ww8LBdX+51jfzQGtD4+qm+p/PO5x6LrXeVfYm/tGpcxvsbNH5jeJg4n+V1lkF7pqbm/t3sehZ3nmXz/7DzIbX1ybubnqmorvrrQurHvrbjdP2dzzWGVm+tO6xm5YvsGDbl1e7jv2worvrrQurHvrbjdP2dzzWGmpe5uTOh2prqxZMHJXnHU79sM7I8qV1j920fIEF27682nXssdO9XW+trVowcVSedzj1jbZ6b1NtbdWCiZNPevZHYz+oNdS8zP61vpkHOiMr79/yl2t9MxTdkcaFdS9j4WDb0Uddh1O0hpqX2b/WN/NAZ2Tl/Vv+cq1vhlqvUl/zyMLBdX+51jfzU7TVBzfVljxSLytvmDgqzzucojXUuHzFa0t8vXfd/mbfzBNt9cGmxc8migvrXvbYwsHEfasWbTu6s6q2turrTz5wezT2TK2hxuUrXlvi673r9jf7Zh5qqw9uqi15pF5W3jBxVJ53OPUzaKsPbjr12Qduj8Z+vA2vrm378uqWmcdmO30zD3R813TL3b3fefUdTD1bZ+SM0v7+VQ3fWjjYdvLmb9RscW7VyWcTZxqeqdar1Nc8snBw3V+u9c080BpqXubkzodqa6sWTByV5x1OPdYaal69YtHEyd7E8yq6I40L617GwsG2o4+6Dqceaw01Ll/x2hJf7123v9k38yxt9cFNtSWP1MvKGyaOyvMOp2gNNS5f8doSX+9dt7/ZN/NEW32wafGzieLCupexcHDdX/7wrrcurXvspuULLNj25dWuY4+d7u16a23Vgomj8rzDqbm5uX90+/btIAiCIDqjLJdVlssqy2WV5bLKclllpdxNvSUIgiAIgiC0Ux9UaXQEQRCk6O6m2W0HQRAEQRAEQRCEduqDKo2OIIjWMM2ySqPTDlJ0d7PS2wiCIAiCIAiCFN3dNLvtIAiiNUyzrNLotIMU3d2s9DaCILVelWZ3IwXR2khjUKXREQRBEARBtIZpDoYpCKIzyspgmIJoDdMsR6m3BNHaSL03TEG0hmkOhikIojPKymCYgiAIghTd3az0NoIgiM4oK4NhCoIgCIIgCIIgiNYwzXKUGkEQ2qkPqjQ67dBOrVel2W0HQWinPqiyMhimIIjWMM1ylJp26oMqzW47OqOsDIYpCIIgCIIgtFMfVGl02qGdWq9Ks9sOgiBawzTLUWoEQRAEQRAEQRCk6O6m2W0HQbSGaQ6GKbRTH+ym2W0HQRAEQRAEoZ36oMpyWWW5rLJS7qbRaQfRGWVlMExBEARBdEZZGQxTEERrI41yN/WWIAiCIAiC0E59MEqNFN3dNLvtIDqjrAyGqXV30+hspDEYpuiMstLbCIIgCIIgCIIgRXc3K72NIFrDNMsqjU47SNHdzUpvIwjt1AdVGp12aKfWq7IyGKYgCIIgCIJoDdMsR6m3BNHaSL03TEFopz6o0ui0Qzu1XpVmtx0EQRAEQRAE0RqmWY5SIwhCO/VBlUanHdqp9ao0u+0gCO3UB1VWBsMUBEEQpOjuptltB0EQrWGaZZVGpx2k6O5mpbcRBEG0hmmWVRodQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEWfR9drr+tNNWH/wf96913evuaui5PRr797Pt7s7YQ7MvJrx3VoGZn2Lb3Z2xh2ZfTHjvrAKz1tCZtW1HV7fMPDDd8tVnVzV+1cbYj7fKO22mY6ZbDjf9y2ZfTHjvrAIzL0DrolNL2452xh46/nTbG+fWMPa0kz/0zTzl4JZ7xtxh9sWYP99y33NoXXRqadvRzthDx59ue+PcGsZenHe9PripuPNbt0djz2fs8Npph77R2tC4elPDafu+q9ar1NdY2Put//qULF3xVnnFQwsm7n9S2p96pqK76dRnPbdR+Gf3/vifzly+ymc9M//Hv2r2xYT3ziow89C2uztjD82+mPDeWQVmrYtOLW072hl76PjTbW+87zmt8k6b6ZjplsNNj7UuOrW07Whn7KHjT7e9cW4NYz9a66JTS9uOdsYeOv502xvn1jD2tJM/9M38K7bd3Rl7aPbFhPfOKjAzNzf3tEU/pHXRKbfsazv9Hicfjf1bOrjlnp/ZwS33fJ919bJS962FvTWM/SjTvv0bvP7+pualVYsmjsrzDqeeqeiONC6se9k3Dm55oQ5uuecpb55VYOaJift/9vM5uOWep7x5VoGZF2RpXbE3sbj2GzVbjtguw9AAACAASURBVP0I0y37N35j5dwGn/qO483TjjsjK+c8snBw3V+u9c081FYf3NS05/Zo7PtteP29iaNrY99r+nsnrlj8Yux5Fd2RxoV1L/vGwS1/d3DLPd/j4JZ7/kXTvv0bvP7+pualVYsmjsrzDqceO7jlnqe8eVaBmZ/g4JZ7nvLmWQVmnpi4/2f/moNb7vkB077bV/vm5v63W/Q9GmXlNY+9VV7xyNXKqU8+cHs09r/awXV/vdY38/OZ7fTt7/Q9VHRH3r48VF3rm/kBraHGBU7K0w6n6IysvO/FWnrXK5j5xp3PzbxAS+96BTPfuPO5mRfo4Lr9zb5XepU3ehv/lz34CY37vhMG/IyrThwNcpjUk7gZGA0Il7yyMRj2MFFxBBXo3Yv7MjkYFnrSC3Nxkp1DUS/iPeVS0cNv168vc9ApJx8yZH3JChRQTJW5BYZEb4gRyAK1bkaJscSozsTh8/pPXNtJHTtx3CTN93nstBY8lJWzdk8c9xgG7qfj0puL9h0eR8c9TRz3+P5pj2d9RbcsGT102p/f9LmOS68U3DDh/qrzSpPsZgWX1jHRNvorD2b/QY9h4OsZrMzqrcy6Ll9v+/lv5vVfmTVwzf6DHsPA5z5838BD2n/QYxj43IfvG0iS5B9hj3voNQsura7565mCjTOLcqsv2WgWbLY7vg1DpRfkfT8MlV6Q94DWZ21/eFKpXnNLfqKtOOGrrb9vsP9fFaquqRk5PO1vqvNK9Rl5t4zx4fsGrll/32D/vypUXVMzcnjaF111Xc3I4Wlf8qfzru4/6DHfgvXX7G5N2ztRQ83I4WlXe6semfXX7G5N2ztRQ83I4WlXe6v+EXZaL9kdP6U04f4m5pXqM/I+V51ROjFt990FLLiyOu3Jxoy8m/JPu4ea4uFpV3urvtJK3UazYKNZsNEsuLi85rPlKRdemTXwcK66rmbk8LQHsv6a3a1peydqqBk5PO2BVOeV6jPybhnjw/cNXLP+mt2taXsnaqgZOTztam/VQ1l/ze7WtL0TNdSMHJ52tbfq6xgqvSDva6rOK2d9pQlJ8qO2xz3N2Du+5soK+afH7L674MHMKGV9lWzJyH4eP9FXyfpKE/5m0M7sPnXSgaxvNGsbcT8zSllfJVsysp/HT/RVsr7ShIc2aGd2nzrpQNY3mrWNuL+d1pTdUksl66tkfaXDZ/VX3MeCy8uMNPtGs5ah3qK/WZ912XGlrK+S9f380JqPFxfctODyMiPNvtGsZai36G/WZ22vjik2+0azlqHeoi9Zn7W9OqaY9Y1mbSMeRselV0/zqyWVbMk+p/XaHY9Ox6VXT/OrJZVsyT6n9dodD22irZL1HZgc85PJJZWsrzThCxb0ziwaPtFVrPpqK6+5oqmU9VWyvtFmkzNTeitu2GlN+bjXVMr6Klnfzw+xvbjguth/0oGsr5L1VbIlw17Sa3f8w63P2l4dU2z2jWYtQ71FD6bj0qun5U8sGc1a9lr0QNZnXXZcKeurZH0/P7Tm48UFN3VcevU0v1pSyZbsc1qv3fFwOi69eppfLalkS/Y5rdfueFCDdmb3qZMOZH2jWduIJEm+jtzm5maUy2VJkiRJkiSPwh5JkiRJkiSP0B5JkiRJkiSP0B5JkiRJkiSP0B5JkiRJkiSP0B5JkiRJkiSP0B5JkiRJkiSP0B73U51XnpuXx0ijq1iVJEmSJEnywPa4n2cO8t5rBmbsHV9zdV2SJEmSJMkDy21ubka5XPYlE22VE9O+KGfNheYRSZIkSZIkD2KPe1mp22hO2dladKlZcHF5zWfLUy40j0iSJEmSJHlQe3yV6guGnfeJmsIhdt/pSJIkSZIk+Tpym5ubUS6XfVEp63vcl322PGWz3ZEkSZIkSfIghtxDr1kw0uja++4RPW2jh8+60FqQJEmSJEnydexxTzP2jq+5skL+6TG77y5IkiRJkiT5unKbm5tRLpclSZIkSZI8CnskSZIkSZI8QkMe0qe1miRJHt5POx1JkiT/jPZIkiRJkiR5hPZIkiRJkiR5hIZ8i37a6fhK1Xnl39B7ZdZA8jfVeeXf0Htl1kDyY/JprSZJkuSf3R6PQL7eVcn6KllfuV7zfZWvd1WyvkrWV67X3K2m2OiqZH2jWVdpoua2mmKjq5L1jWZdpYma22qKja5K1jeadZUmah61fL2rXK9JkiRJku+jPe6nOq88Ny+PkUZXseq+Bu0jNpoFF5fXfJ8N2kdsNAsuLq/5opHGkuFe5mKz4EL2BidailU3jDSWDPcyF5sFF7I3ONFSrLphpLFkuJe52Cy4kL3BiZZiVZIkSZL8aO1xP88c5L3XDMzYO77m6rpvSU1xrq9cr3kY+XpbuTEjX/UtmrF3fNF2e8HANeuzLi8zfLSGGXvHF223Fwxcsz7r8jLDR2uYsXd80XZ7wcA167MuLzN8tOZBFBpdlayv3JiXd8uMUtZVrLptom10bl5+oq2S9R2YHPOTySWVrG80axtxS02x0VXJ+kazrlK95pZ8va2c9VWyvtG5tmJVkiRJkjwSe9zLRFsl66ucmPaTySWV7JTHTXsy6ypWPaSa4lzL8HtTNtsdD2PQ/r3t3nGlZt/oXFtxouahVZ+V3zrvE7cN/rJmqDRO9Vn5rfM+cdvgL2uGSuNUn5XfOu8Ttw3+smaoNO6+9p809G7DRnPKtpNK9ZqbFlxZHTN8tOaWkcPTdt+cNVip22gWXFxe89nylI1mwYVm3Y6bRhpLhnuZi82CC1nGoSWlCVTnlSbZzQo2mgUXXj3L9Ly8JEmSJPn27XEvK3UbzSk7W4suNQsuLq/5bHnKheYRl9Y9hIOemFsy/GFms93xIPL1rkrWV8n6KllfJeurZH2jjRl07LTrNpsFF149y+GW0ayrXK95aNV55ayrWPVl1XnlrKtY9WXVeeWsq1j1NSy6stJBx867i4ZK427ZeXfR0KEX5F03Y+/4oisrvlp13r7xRdvtBQPXrC+4/N6a/NM1N43xTM0N6wsutWYNJEmSJMm3b8hXqb5g2Hk9NYVD7L7a8dD2T8uvrhkaP27Egh33N2gfsdH2j7c+a7M564Zn3G191mZz1g3PuNv6rM3mrBue8WC2zvvEHZ56Vh4D16yctXuiqVCd5WjT8Gqm50FMK2Z9RbflVsdZn9U7wxO/aimfGDNkzXZ2xKV1SZIkSfKtG3IPpazvcTcdyE66odk3vDxls93xjW2d1mvNeqzR92Rjxk5rwf3k610HJsd8UW71JRdaq0bqv7NvctrQ1qLtNxsutDoeyvr7BvuPewwDN+WfHnO1t8o6g/3HPYaBm/JPj7naW2Wdwf7jHsPATfmnx1ztrbqv/Qc9hoHPffi+gVsWXF5uKh2d4dCY3TcXPJCt0y6+MmvgywYrs3ors67L19t+/pt5/VdmDSRJkiTJt2uPe+g1Cy6trvnrmYKNM4tyqy/ZaBZstju+DTutl+yOn1KacF+D9hEbzYKNZsFGs2CjWbDRLLjQWpCv/86+0lm9rODCK3WXVjoe3oIrq9P21WfkXVOd98Qku+90sODK6rR99Rl511TnPTHJ7jsdLLiyOm1ffUbeNdV5T0yy+07H/U3bO1FDzcjhaVd7q+40eOcNJk/Zt3/RlRVfMlR6Qd4d1mdtf3hSqV5zS36irTiB6rxSfUbeLWN8+L6BJEmSJPn27XFPM/aOr7myQv7pMbvvLnhgE22VrO/A5JifTC6pZH2lCV+woHdm0fCJrmLVNzZo1222FgzWfX0TbZWs78DkmJ9MLqlkfaUJN+y0puyWmg5kfaPNf+VMw6V1N+y0puyWmg5kfaPNf+VMw6V1N+y0puyWmg5kfaPNf+VMw6V197d12tXDLZVsyT6n9dodd1l/ze4WVs/acbdBO7P71EkHsr7RrG3ETTutKbullkrWV8n6SofP6q9gfdZlx5WyvkrW9/NDaz5eXJAkSZIkj0Juc3MzyuWyb+rTWs0tP+10JI9KTXFuydCbBb0VyT+JT2s1t/y005EkSfLPaI/kh6H6guH9i66sSJIkSZIflD2S772RRt9o818NztTtSJLvp4jwwQcfeFAvv/yyt99+27188MEHXn75Zffy+uuv29ra8kUR4eWXX5YkyffHHsn33k6r4ELziN6KJPlBiQivv/66v+edd95Rq9W8/vrrvuj111/35JNPeuedd9zL//gf/8Mf//hHdzp27Jj/+q//8h//8R+OHTsmSZLvic3NzUAgEAgEAoFAIBAIBALxaa0Wn9Zq8WmtFggEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKB+LRWi09rtfi0VgsEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoE4duxYXPf6668HAoFAIBCnTp2K615++eVAIE6dOhXXHTt2LBAIBAKBOHbsWFx37NixQCAQCMQHH3wQH3zwQSAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEIg9/pGq88pz8/L+juq88ty8vOQu1XnluXl5SfLP4dixYyJCRHjxxRdd9x//8R8iQkR48cUXXffWW2+JCBHh2LFj7vTb3/7W+fPnHT16VESICBEhIkSEgwcPOnjwoIgQESLC22+/LUmS78Yej0C+3lXJ+ipZX7le80OUr3dVsr5K1leu19ytptjoqmR9o1lXaaLmlny9rZz1VbK+0bm2kaq/Y0Yp6yvXax61fL2rXK9Jki/a2trywQcf+KKI8Pbbb7vlgw8+sLW15eWXXxYRIkJEOHXqlDttbW354IMPXPf222976623XPfrX/9aRHj77bfd6fnnn5fL5eRyOblcTqfT0el05HI5uVxOLpfz/PPP+3t++ctf+uijj/znf/6nXC4nl8vJ5XJyuZxcLieXy8nlcnK5nFwuJ5fLee655yRJ8t3Y436q88pz8/IYaXQVq+5r0D5io1lwcXnND9WgfcRGs+Di8povGmksGe5lLjYLLmRvcKKlWEV13hOls3rNgo1mwZ/fG/Pkb+bl3W2k0WR1TZL8UPzsZz/z4osvyuVycrmcTqfjxRdf9PLLL/t7nnvuOc8//7zr/uu//ksul/Pcc8+501tvvSUiRISIUKvV1Go1ESEiRIS33nrLF506dcrPfvYzt7z88ssiwrFjx/w9EeH111+XJMl3Z4/7eeYg771mYMbe8TVX131LaopzfeV6zZ0Kja5K1lduzMt7OPl6W7kxI1/1LZqxd3zRdnvBwDXrsy4vM3y0xvqsXmvBwE2DdmZ3/0GPucNE2z6Zyz1fS6HRVcn6yo15ebfMKGVdxarbJtpG5+blJ9oqWd+ByTE/mVxSyfpGs7YRt9QUG12VrG806yrVa27J19vKWV8l6xudaytWJYlf/OIXbpmdnXXd1NSUb+r555+Xy+Xkcjm5XE6n09HpdORyOblcTi6X8/zzz/uif/u3f3On//zP//TRRx/57W9/64tOnTrluj/84Q+SJPnu7HEvE22VrK9yYtpPJpdUslMeN+3JrKtY9ZBqinMtw+9N2Wx3/M3+k4bebdhoTtl2Uqle8zAG7d/b7h1XavaNzrUVJ2oeWvVZ+a3zPnHb4C9rhkrjvmTiuOGt8z5xS03xV2y3Fnwt+08aerdhozll20mles1NC66sjhk+WnPLyOFpu2/OGqzUbTQLLi6v+Wx5ykaz4EKzbsdNI40lw73MxWbBhSzj0JLSBKrzSpPsZgUbzYILr55lel5e8mP20UcfudO5c+dc99RTT/lHev311/3sZz/T6XTc6Y9//KNf/vKXvuhf/uVfnD9/3rlz5yRJ8t3Z415W6jaaU3a2Fl1qFlxcXvPZ8pQLzSMurXsIBz0xt2T4w8xmu+Nui66sdNCx8+6iodK4+8nXuypZXyXrq2R9layvkvWNNmbQsdOu22wWXHj1LIdbRrOucr3moVXnlbOuYtU9zCidmLb75qyBm/L1luH3fm/H17XoykoHHTvvLhoqjbtl591FQ4dekHfdjL3ji66s+GrVefvGF223Fwxcs77g8ntr8k/X3DTGMzU3rC+41Jo1kCTfrrfeektEiAgRoVarqdVqIkJEiAhvvfWWOz311FP+/d//3Rf94Q9/8LOf/cyxY8fcqVar+e///m9Jkny3hnyV6guGnddTUzjE7qsdD23/tPzqmqHx40Ys2HGHrfM+cYennpXHwL0N2kdstP3jrc/abM664RlfUFOcOyW/PGVzxedmPHFozfYrHV/b1nmfuMNTz8pj4JqVs3ZPNBWqsxxtGl7N9DyIacWsr+i23Oo467N6Z3jiVy3lE2OGrNnOjri0Lvkn8/HHH/suPf/8886dO+eWt99+23XPPfecW44dO+att95yy3PPPee6f/u3f3Onc+fOOX/+vN/+9rfOnTvnulOnTvnoo4+89NJLkiT5bg25h1LW97ibDmQn3dDsG16estnu+Ma2Tuu1Zj3W6HuyMWOnteBv9h/0GAY+9+H7Br5avt51YHLMF+VWX3KhtWqk/jv7JqcNbS3afrPhQqvjoay/b7D/uMcwcFP+6TFXe6tuqinOLRl+b8pmu+NvJo57fP+0x7O+oluWjB467c+vzBr4CvsPegwDn/vwfQO3LLi83FQ6OsOhMbtvLnggW6ddfGXWwJcNVmb1VmZdl6+3/fw38/qvzBpI/tk8+eST7vTyyy971M6dOyeXy3kQ586dk8vlPIj/9//+n1/+8pdu+Z//83/64x//KEmS796Qe+g1C0YaXXvfPaKnbfTwWRdaC74tO62X7M1OKU0s6K343LS9EzU7K4wcnna193v3M2gfsdH2d+XrbftKZ/WyusG6b8mCK6un7KvP+KS9YFCd98Qku1kHNcW5luH3pmy2O+6yUrex4m/y9a6Shs12x/1N2ztRs7PCyOFpV3u/d6fBO2/QPGWfRR+v+JKh0gvyOgY+tz5r+8O+Uv01m+2O6/ITbQV1l/40r3T0fZfbCwauG+PDswaSfzb//d//7cUXX3Tq1CkvvfSS6/7P//k/vi3nzp1z3VNPPeWWDz74wMGDB32ViPD3nD9/3i9+8Qv38oc//MGvf/1rx44dc93Bgwf97//9vyVJ8t0bck8z9o6vudIiXx+z++6CBzbRVjkx7aYllUn+eqag9yd3WNA7c9zoia7in4645Jqt064ebqmcGPPZ6mm9dsfDGLTrNn1DE22VE9NuWlKZ5K9nCnor7LSmDDVaDmSn5KzZPdNwaR0TvzOyf4zJJZVJN+Qs+rhZt+MhbJ129XBL5cSYz1ZP67U77rL+mt2tk/Z9eNaOuw3amd25Uw5kJ+Us+rhZt4Od1pShRkslG3PdZ6sv6bVcM+vy0bZSdspPkNta9PGrC5J/Pi+99JJ/+Zd/8eKLL3rxxRdd9/zzz3vrrbd8W/7v//2/XnzxRRGh0+n4xS9+4V7efvtt1z333HO+iXPnzsnlcq57/fXXnT9/3rlz5yRJ8t3LbW5uRrlc9k19Wqu55aedjuS7UFOcWzL0ZkFvRfID8mmt5pafdjp+zN5++23XPffcc+7n7bffdt2HH37o17/+tQfx0Ucf2b9/vyRJ/vGGJD981RcM71+0vSJJflT+1//6X5Ik+f4bkvygjTT6nhxfs3vmiB1J8sP13HPPeVDPPfecJEl+OIYkP2g7rYIdSZIkSfL9tUeSJEmSJMkjtEfyz6k6rzw3Ly9JkiRJvlt7PAL5elcl66tkfeV6zfdVvt5VyfoqWV+5XnO3mmKjq5L1jWZdpYma22qKja5K1jeadZUmam6rKTa6KlnfaNZVmqi5rabY6KpkfaNZV2mi5raaYqOrkvWNZl2liZrbaoqNrkrWN5p1lSZqHrV8vatcr0mSJEmSh7XH/VTnlefm5THS6CpW3degfcRGs+Di8prvs0H7iI1mwcXlNV800lgy3MtcbBZcyN7gREux6oaRxpLhXuZis+BC9gYnWopVN4w0lgz3MhebBReyNzjRUqy6YaSxZLiXudgsuJC9wYmWYtUNI40lw73MxWbBhewNTrQUq24YaSwZ7mUuNgsuZG9woqVYlSRJkiQ/CHvczzMHee81AzP2jq+5uu6bqc4rz3WV5/pG59qKja5K1leu1zyMfL2t3JiRr/oWzdg7vmi7vWDgmvVZl5cZPlrDjL3ji7bbCwauWZ91eZnhozXM2Du+aLu9YOCa9VmXlxk+WsOMveOLttsLBq5Zn3V5meGjNczYO75ou71g4Jr1WZeXGT5aw4y944u22wsGrlmfdXmZ4aM1D6LQ6KpkfeXGvLxbZpSyrmLVbRNto3Pz8hNtlazvwOSYn0wuqWR9o1nbiFtqio2uStY3mnWV6jW35Ott5ayvkvWNzrUVq5IkSZLEHvcy0VbJ+ionpv1kckklO+Vx057MuopV38z+NduvTNk2bbjXsHFm0dChF+R9c4P27233jis1+0bn2ooTNQ+t+qz81nmfuG3wlzVDpXGqz8pvnfeJ2wZ/WTNUGqf6rPzWeZ+4bfCXNUOlcarPym+d94nbBn9ZM1Qap/qs/NZ5n7ht8Jc1Q6Vxqs/Kb533idsGf1kzVBp3X/tPGnq3YaM5ZdtJpXrNTQuurI4ZPlpzy8jhabtvzhqs1G00Cy4ur/lsecpGs+BCs27HTSONJcO9zMVmwYUs49CS0gSq80qT7GYFG82CC6+eZXpeXpIkSfJjt8e9rNRtNKfsbC261Cy4uLzms+UpF5pHXFr3zWyd94mOqx8y+EuHP5131f3l612VrK+S9VWyvkrWV8n6Rhsz6Nhp1202Cy68epbDLaNZV7le89Cq88pZV7Hqy6rzyllXserLqvPKWVex6suq88pZV7Hqy6rzyllXserLqvPKWVex6mtYdGWlg46ddxcNlcbdsvPuoqFDL8i7bsbe8UVXVny16rx944u22wsGrllfcPm9Nfmna24a45maG9YXXGrNGkiSJEl+7IZ8leoLhp3XU1M4xO6rHd+FQfuIjbZ/vPVZm81ZNzzjbuuzNpuzbnjG3dZnbTZn3fCMu63P2mzOuuEZd1uftdmcdcMz7rY+a7M564ZnPJit8z5xh6eelcfANStn7Z5oKlRnOdo0vJrpeRDTillf0W251XHWZ/XO8MSvWsonxgxZs50dcWldkiRJ8iM35B5KWd/jbjqQnXRDs294ecpmu+MfKV/vOjA55otyqy+50Fo1Uv+dfZPThrYWbb/ZcKHV8VDW3zfYf9xjGLgp//SYq71V1hnsP+4xDNyUf3rM1d4q6wz2H/cYBm7KPz3mam+VdQb7j3sMAzflnx5ztbfKOoP9xz2GgZvyT4+52ltlncH+4x7DwE35p8dc7a26r/0HPYaBz334voFbFlxebiodneHQmN03FzyQrdMuvjJr4MsGK7N6K7Ouy9fbfv6bef1XZg0kSZIkP2Z73EOvWXBpdc1fzxRsnFmUW33JRrNgs93xjzZoH7HRLNhoFmw0CzaaBRvNggutBfn67+wrndXLCi68UndppePhLbiyOm1ffUbeNdV5T0yy+04HC66sTttXn5F3TXXeE5PsvtPBgiur0/bVZ+RdU533xCS773Sw4MrqtH31GXnXVOc9McnuOx0suLI6bV99Rt411XlPTLL7TgcLrqxO21efkXdNdd4Tk+y+03F/0/ZO1FAzcnja1d6qOw3eeYPJU/btX3RlxZcMlV6Qd4f1WdsfnlSq19ySn2grTqA6r1SfkXfLGB++byBJkiT5sRtyTzP2jq+50iJfH7P77oIHNtFWOTHtpiWVSf66fNqjMGjXbfqGJtoqJ6bdtKQyyV/PFPRW2GlNGWq0HMhOyVmze6bh0robdlpThhotB7JTctbsnmm4tO6GndaUoUbLgeyUnDW7ZxourbthpzVlqNFyIDslZ83umYZL627YaU0ZarQcyE7JWbN7puHSuht2WlOGGi0HslNy1uyeabi07v62Trt6uKVyYsxnq6f12h13WX/N7tZJ+z48a8fdBu3M7twpB7KTchZ93KzbwU5rylCjpZKNue6z1Zf0Wq6ZdfloWyk75SfIbS36+NUFSZIkSZLb3NyMcrnsm/q0VnPLTzsdyQ9JTXFuydCbBb0VyXfg01rNLT/tdCRJkvwz2iP58aq+YHj/oisrkiRJkuSRGZL8KI00+p4cX7N75ogdSZIkSfLoDEl+lHZaBTuSJEmS5NHbI0mSJEmS5BHa4/uiOq88Ny8v+d6qzivPzctLkiRJkge3xyOQr3dVsr5K1leu1/wQ5etdlayvkvWV6zV3qyk2uipZ32jWVZqouSVfbytnfZWsb3SubaTq75hRyvrK9ZovGmm0jbhbvt5VyfoqWV+5XnO3mmKjq5L1jWZdpYmaW/L1tnLWV8n6RufaRqr+jhmlrK9cr3nU8vWucr0mSZIk+XHZ436q88pz8/IYaXQVq+5r0D5io1lwcXnND9WgfcRGs+Di8povGmksGe5lLjYLLmRvcKKlWEV13hOls3rNgo1mwZ/fG/Pkb+bl3W2k0WR1zW01xUbbSNXnakYabcWqGwbtIzaaBReXYPnq6wAAIABJREFU13zRSGPJcC9zsVlwIXuDEy3FKqrzniid1WsWbDQL/vzemCd/My/vbiONJqtrkiRJkuRR2eN+njnIe68ZmLF3fM3Vdd9MdV55rqs81zc611ZsdFWyvnK95k6FRlcl6ys35uU9nHy9rdyYka/6Fs3YO75ou71g4Jr1WZeXGT5aY31Wr7Vg4KZBO7O7/6DH3GGibZ/M5Z47dFxq/Z7ptn3j0/bN/Y7Fukvr7mPG3vFF2+0FA9esz7q8zPDRGuuzeq0FAzcN2pnd/Qc95g4TbftkLvd8LYVGVyXrKzfm5d0yo5R1Fatum2gbnZuXn2irZH0HJsf8ZHJJJesbzdpG3FJTbHRVsr7RrKtUr7klX28rZ32VrG90rq1YlSRJkvzA7HEvE22VrK9yYtpPJpdUslMeN+3JrKtY9c3sX7P9ypRt04Z7DRtnFg0dekHe5/afNPRuw0ZzyraTSvWahzFo/95277hSs290rq04UfPQqs/Kb533idsGf1kzVBr3JRPHDW+d94lbaoq/Yru14FtRfVZ+67xP3Db4y5qh0rgvmThueOu8T9xSU/wV260FX8v+k4bebdhoTtl2Uqlec9OCK6tjho/W3DJyeNrum7MGK3UbzYKLy2s+W56y0Sy40KzbcdNIY8lwL3OxWXAhyzi0pDSB6rzSJLtZwUaz4MKrZ5mel5ckSZL8kOxxLyt1G80pO1uLLjULLi6v+Wx5yoXmEZfWfTNb532i4+qHDP7S4U/nXXWnRVdWOujYeXfRUGnc/eTrXZWsr5L1VbK+StZXyfpGGzPo2GnXbTYLLrx6lsMto1lXuV7z0KrzyllXseoeZpROTNt9c9bATfl6y/B7v7fji2qKjd+xWLe9umj7ld8z3VasejDVeeWsq1h1DzNKJ6btvjlr4KZ8vWX4vd/b8XUturLSQcfOu4uGSuNu2Xl30dChF+RdN2Pv+KIrK75add6+8UXb7QUD16wvuPzemvzTNTeN8UzNDesLLrVmDSRJkiQ/JEO+SvUFw87rqSkcYvfVjkdq67xP3OGpZ+UxcG+D9hEbbf9467M2m7NueMYX1BTnTskvT9lc8bkZTxxas/1Kx5d1XGrVXTfiuo6dVt0DW5+12Zx1wzO+oKY4d0p+ecrmis/NeOLQmu1XOr62rfM+cYennpXHwDUrZ+2eaCpUZznaNLya6XkQ04pZX9FtudVx1mf1zvDEr1rKJ8YMWbOdHXFpXZIkSfIDMuQeSlnf4246kJ10Q7NveHnKZrvjkdh/0GMY+NyH7xv4avl614HJMV+UW33Jhdaqkfrv7JucNrS1aPvNhgutjoey/r7B/uMew8BN+afHXO2tuqmmOLdk+L0pm+2Ov5k47vH90x7P+opuWTJ66LQ/vzJr4KadVt0DW3/fYP9xj2HgpvzTY672Vt1UU5xbMvzelM12x99MHPf4/mmPZ31FtywZPXTan1+ZNfAV9h/0GAY+9+H7Bm5ZcHm5qXR0hkNjdt9c8EC2Trv4yqyBLxuszOqtzLouX2/7+W/m9V+ZNZAkSZL8UAy5h16zYKTRtffdI3raRg+fdaG14NGatneiZmeFkcPTrvZ+734G7SM22v6ufL1tX+msXlY3WPctWXBl9ZR99RmftBcMqvOemGQ366CmONcy/N6UzXbHXVbqNlb8Tb7eVdKw2e745hZcWT1lX33GJ+0Fg+q8JybZzTqoKc61DL83ZbPdcZeVuo0Vf5Ovd5U0bLY77m/a3omanRVGDk+72vu9Ow3eeYPmKfss+njFlwyVXpDXMfC59VnbH/aV6q/ZbHdcl59oK6i79Kd5paPvu9xeMHDdGB+eNZAkSZL8kAy5pxl7x9dcaZGvj9l9d8EDm2irnJh205LKJH9dPu2+tk67erilcmLMZ6un9dodD2PQrtv0DU20VU5Mu2lJZZK/ninorbDTmjLUaDmQnZKzZvdMw6V1TPzOyP4xJpdUJt2Qs+jjZt2OhzDRVjkx7aYllUn+eqagt8JOa8pQo+VAdkrOmt0zDZfWMfE7I/vHmFxSmXRDzqKPm3U7HsLWaVcPt1ROjPls9bReu+Mu66/Z3Tpp34dn7bjboJ3ZnTvlQHZSzqKPm3U72GlNGWq0VLIx1322+pJeyzWzLh9tK2Wn/AS5rUUfv7ogSZIk+WHJbW5uRrlc9k19Wqu55aedjuTHrqY4t2TozYLeiuQ+Pq3V3PLTTkeSJMk/oz2S5NtUfcHw/kVXViRJkiTJDUOS5Fsy0uh7cnzN7pkjdiRJkiTJTUOS5Fuy0yrYkSRJkiR32yNJkiRJkuQR2iNJHlR1XnluXl6SJEmSPLg9HoF8vauS9VWyvnK95vsqX++qZH2VrK9cr7lbTbHRVcn6RrOu0kTNbTXFRlcl6xvNukoTNbfVFBtdlaxvNOsqTdTcVlNsdFWyvtGsqzRRc1tNsdFVyfpGs67SRM1tNcVGVyXrG826ShM1t9UUG12VrG806ypN1NxWU2x0VbK+0ayrNFFzW02x0VXJ+kazrtJEzaOWr3eV6zVJkiTJj8zm5mYgEAgEAoFQnY/y3HzkiZFGN4pVgUAgPq3V4tNaLT6t1QKBQCDy9W6U67VAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgcjXu1Gu1wKBQIw0+lGuz0SeUJ2PUtaNYlUgRhr9KNdnIk+ozkcp60axKhAjjX6U6zORJ1Tno5R1o1gViJFGP8r1mcgTqvNRyrpRrArESKMf5fpM5AnV+Shl3ShWBWKk0Y9yfSbyhOp8lLJuFKsCMdLoR7k+E3lCdT5KWTeKVYEYafSjXJ+JPKE6H6WsG8WqQIw0+lGuz0SeUJ2PUtaNYlUgEAgEAoFQnY/y3HzkCQQCgUAgEAgEAoFA5OvdKNdrgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCA+rdXi01otPq3VAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEAoFAIBAIBAKBQCAQCAQCgUAgEAgEYo/7eeYg771mYMbe8TVX1z2c6rxy1jeadZUabaNz8/K+uXy9rdyYka/6Fs3YO75ou71g4Jr1WZeXGT5aw4y944u22wsGrlmfdXmZ4aM1zPj/7MFBaNyHgS/gb7JmnFrIQY6U9FUgDRibrBMMhj38rWViqEH7Lq6ZHgKBnnTQpfagw6JezB5KLhF7+MdyLz7otFDoocMklyJwwDV4dCgERKOWEQbZoNekmtTYZvyccZbfs+J6ndhx4ibdku7T9z17aNnN1pKBezbm3bjIniMFZjx7aNnN1pKBezbm3bjIniMFZjx7aNnN1pKBezbm3bjIniMFZjx7aNnN1pKBezbm3bjIniMFZjx7aNnN1pKBezbm3bjIniMFZjx7aNnN1pKBezbm3bjIniMFZjx7aNnN1pKBezbm3bjIniOFpzE0u2qi7BufXVD1wIyxctVIzUNTLZNnFlSnWibKvu8e2+8fjl0wUfZNli3DHiiMzK6aKPsmy1VjjcID1UbLeNk3UfZNnmkZqdmxY8eOHX9nnvEkUy0TZd/Ea9P+4dgFE+Wi75i2r1w1UvM1FUZ+9GODXxx3dW7WHdO+qUHrTTe3Thib65s80zIyVfjGai+p9tZ97KHBh1fsGjtE7SXV3rqPPTT48IpdY4eovaTaW/exhwYfXrFr7BC1l1R76z720ODDK3aNHaL2kmpv3cceGnx4xa6xQ9ReUu2t+9hDgw+v2DV2iNpLqr11H3to8OEVu8YOUXtJtbfuYw8NPrxi19ghai+p9tZ97KHBh1fsGjvkK43+2K7fzro2d9xNPzbWKNy35M7afnuOFB4YfmXa7XfnDS43XJsb8sHFK/7z4nHX5oZcnWu45b7h2Qv2bJU+mBtytSx5+YKxKdQWjB3jdjnk2tyQq//xDtMLqnbs2LHj70en09HpdDzJ4uKiXq/nf7JnPMnlhmtzx93qLbs+N+SDi1f858Xjrs4ddn3D11P7oT2jy+5cXsGKW79d9jSqjVUTZd9E2TdR9k2UfRNl3+TsDFbcajVszg25+h/v8Mp5k+Wq8UbhG6stGC9XjdQ8rrZgvFw1UvO42oLxctVIzeNqC8bLVSM1j6stGC9XjdQ8rrZgvFw1UvO42oLxctVIzeNqC8bLVSM1j6stGC9XjdT8BZbdubyCFbd+u2zX2CEP3Prtsl0v/1DVthnPHlp257IvV1uw99Cym60lA/dsLLnx/hXVFwv37ed7hU9tLLl+ft7Ajh07/t51u12dTsfX1W639Xo9fw+KovCb3/zGk/zTP/2T9fV1X0e9XpfEtm63a3Fx0bfRM75M7Yf2WPexwtDL3H5vxTfWW/exv8ygddi1uSHX5oZcmxtybW7ItbkhV88v+W+1MW9z7rDrGx63MW9z7rDrGx63MW9z7rDrGx63MW9z7rDrGx63MW9z7rDrGx63MW9z7rDrGx63MW9z7rDrGx63MW9z7rDrGx63MW9z7rDrG55eb93HPuOFl1T92eV33B7934ZqVBtz9qy945anMW2k7Jso+ybKvu8e22/X2CE25m394ld2ff+88bJvslw1UrNjx44dT6Ver0siiSSSSCKJJJJIIokkkkgiiW3tdlsSSSSRRBJJJJFEEkkkkUQSSXS7Xdva7bZtp06dkkQSSSSRxLYDBw74zW9+45s6d+6cU6dOqdfrvm12eYKxsu877vtu+WOfmuvbc/G4zdaKr230gN0YeHrVxqrvHtvvUZW1066eXzPc+Im9x6bt6i27+e6sq+dXfCMbvzcYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trbDAYPWE3Bu6rvrjfJ1trvtLoAbsx8Gd//L2BB5bcuDhn7MgML+93+90lT6X3Mx+8MW/gcYPL87Yuz9tWbbT8rx8t6L8xb2DHjh07ns6rr77q0qVLnkaz2fTWW2/ZdvLkSV8mibffftvJkyd9kXq97gc/+IFz5845ffq0L9JsNj3//PNOnTrl1KlTHlhfX3fw4EH1et2vf/1rXyWJB3796197++23nTx50rfFM55ga27I9bUr/u8vhlz7xbLK2mnX5oZstlZ8bRu/dLs37dmpAoXhV6Y9jUHrsGtzQ67NDbk2N+Ta3JBrc0Ounl9SbfzE3rF3bJVDrr7RcP3yim9uyZ21aXsbM6ruqS147hi331vBkjtr0/Y2ZlTdU1vw3DFuv7eCJXfWpu1tzKi6p7bguWPcfm8FS+6sTdvbmFF1T23Bc8e4/d4KltxZm7a3MaPqntqC545x+70VLLmzNm1vY0bVPbUFzx3j9nsrWHJnbdrexoyqe2oLnjvG7fdWsOTO2rS9jRlV99QWPHeM2++tYMmdtWl7GzOq7qkteO4Yt99b8dWmPTtVoDD8yrRPttZ81uC9X3Fs0d7RZXcue8yusR+q+oyNeTf/+GNjjcID1amWkSnUFow1ZlQ9sJ8//t7Ajh07/n/Tbrd1u11fpF6vS6LZbHqSdrut1+v5a+l0Os6dO+fkyZOeZGFhwfr6ulOnTkkiiSSSSKJer3v99detr6+rVCoqlYpz585ZX1938OBBn/Xqq6+qVCoqlYpKpaJSqahUKiqVim2vvvqqSqWiUqmoVCpOnjzpW2VzczMIgiAIMxkrWxkm1cZqxqYEQRAEuVsUuVsUuVsUQRBTrUyU/UyU/UyU/UyU/YxNCaK2kPGyn8lyNWOzrUyeWUiVIAiCIAiCIAiCIAiCIAiCIAiCIAiCIKZamSj7mSj7mSj7mSj7GZsShCIjs6uZKPuZLFczNlUEQSgyMruaibKfyXI1Y1NFEIQiI7OrmSj7mSxXMzZVBEEoMjK7momyn8lyNWNTRRCEIiOzq5ko+5ksVzM2VQRBKDIyu5qJsp/JcjVjU0UQhCIjs6uZKPuZLFczNlUEQSgyMruaibKfyXI1Y1NFEIQiI7OrmSj7mSxXMzZVBEEQBEEQRG0h42cWMjK7momyn/HZhVQJgiAUGTnTz+TsTBAEQZjJ2Jl+Jsp+JstWhglCkZHZ1UyU/UyU/YzPzqRKkGqjlfGyn4myn8kzrQzXBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEGQu0WRu0WRu0URBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQBEEQpN1u5+vqdDpBEGTb4uJiEKTdbqfX6wVBut1uut1uEARBEKTT6eRpIQiCIAiCLC4uZlun0wmCIAiCIEiz2cy2er2ebUi9Xs+2ZrOZBxCk3W6n0+kEQer1era12+18kXa7nV6vl2azGaTT6aTdbgdBEARBEARBEARBEARBEARBEARBEARBEARBEMTm5mYQBEEQBEEQBEEQBLlbFLlbFLlbFEEQBEEQBEEQBDHVyuTsTBAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEocjImX7GpgRBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEAS5WxS5WxS5WxRBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEARBEATp9XrpdDpBEARBEARBFhcXsw1BkHa7nV6vFwRpNpvZtri4GARBEKTX66XdbgdBEARBEARBEARBEKTZbKbT6aTb7abT6QRBEARBEGRxcTG9Xi9Is9lMt9tNvV7PtsXFxXQ6nfR6vdTr9SDdbjftdjsIUq/Xs61erwdBEARBut1u2u12ms1mtiEIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiDP+BurNlpGau4pDL8y7ZOtNTv+B6n90J7RZXcu27Fjx44nev755x04cMDT+Jd/+Rdvv/22L3P27Fnr6+tef/11j6rX655//nkXLlzwTTSbTadOnXL06FEPNJtNSdTrdU/ypz/9ybazZ886ePCgB06fPu3o0aP+9Kc/ee2112zbt2+fCxcueODIkSO2Xbp0Sa/Xk0QSSSRRr9f97ne/88///M/eeust586d8620ubkZBEEQBEEQBEEQBEHuFkXuFkXuFkUQBEEQBEEQioyc6Wei7Gd8diFVgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiAIgiDDs/1MlqsZmxIEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAE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)



0

2、包含 includes 





![img](data:image/png;base64,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28bMae2ss/x0KUcqO+U8Zr415oTkL7Jw2ajHt41hzP57RnX1FP24slPOY+ZbY34kf5GFy0Y9vm0MY/bfM6qrp+jEjHp825g5rZ11lp8u5Qn5msXFUftGtmh5QmOLf56oS7205Egzt23UcoLyNYuLo/aNbNEyZ8z+kS1anpC/yMJlox7fNoYx++8Z1dVT1DHq8W1j5rR21ll+upREIpFIvBh0OZ78RRbaYUrJolcyc8MYK0qel+kdDjiO6R0OeIFN73DA8fTLDjZldcybLGLMs+p7s4WTg+5v+HHTOxxwhOWnS6GF1MCInvP6LfC06R1OyPQOBzyDvjdbODno/oYTN73DAUdYfroUWp7W92YLJwfd3/Djpnc44AjLT5dCy7OY3uGA4+mXHWzK6pg3WcSYp9S1f+C5md7hgOOY3uGAIyw/XQotT5je4YBEIpFIvBh1OYaewaaTPWXF4BWetKHJ367zM7OsoBstL6BlBd1oOYbpzR7etFHLc1WSvWC1fTds8ROWFXSj5Wm7vqflCfmanvOYGVxkTwN9I1Zd4AVQkr1gtX03bPGcLCvoRsvTdn1Py2El2QtW23fDFj9hWUE3Wp6263tanqfpzR7etFHLC2hZQTdajmFZQTdanrbre1oSicQviyVLlnjLW97izDPPtGTJEnv37rV9+3Z/93d/Z//+/Y62du1a69ats2rVKieddJJdu3bZtm2br371q9rttsMKhYJLL73UkWZnZ01PT/vOd77j61//uoMHDzrSGWec4aKLLjI7O+vTn/60AwcOONo73/lOXV1d/vt//++ei3POOce5557r5S9/uUwmY//+/e677z533HGHu+++29HOOuss//bf/ltzbrzxRjt27HA873nPe+RyObt373bNNdd4rs4880xvectbfPazn7Vz507P1fnnn++8885z9dVXm52ddTxvetObnHnmmTZt2uSXwXzHMLVhkT2TdY/dtMgDN42aN/kBD2xY5GemsdG+yX6LB8pS5pRkBspSjvCDHdrLCrodofE9rWUXWpT3hJLMmn4/0hg2M90v3VdCSWZNvx9pbLRv1xV6BkoOS/WNyPZ5dn1/arGbNRuOoV+6r4SSzJp+7alJR2qbU5JZ0+9YunoukvIc9P2pxW7WbDiurp6LpByhMWxmul+6r4SSzJp+7alJP9L3pxa7WbPhxzWGzUz3S/eV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9FGvfe1r3X777Y5UKBScdtppbr/9dueee66jnXvuud7//vebs2vXLitWrPDYY4/58z//c/fee69nc+6553r/+99vzq5du6xYscJjjz3mz//8z917772eSXd3t6uvvtpJJ53kwx/+sCOtWrXKJz7xCXPS6bTu7m6f+9znfPnLX/bzNF8ikUgkEr9A3d3dTjvtNDt37rR7927Hct999/n0pz/t//2//2fON77xDY8//rhLL73UFVdcoVgsWrBggefjG9/4hlQq5Td+4zccbWRkxMMPP+w973mP+fPney7mz59v3bp1/u7v/k6z2XTSSSf56Ec/aufOnd797ne76qqrfPazn/X617/etm3bzLn77rvNnz/fqlWrHGl2dtbXvvY1a9assXz5ckdat26der3u+9//vqO98pWv9Md//Mfuvvtul112mQ9+8IMuv/xy9XpdtVqVyWQ8k1e+8pX++I//2N133+2yyy7zwQ9+0OWXX65er6tWqzKZjOOZP3++K6+8Ujab9bGPfcxjjz3msPnz53vf+97nr/7qr1x22WUuvfRS3/72t/3hH/6hTCbj52m+xAujsdFDmzZqSSQSicRzsXjxYvPmzTM9Pe1E7d2719VXX2337t1e//rX++QnP+l//I//4eqrr/bWt77V0qVLPVc//OEPzVm2bJmjtdttf/M3f2PVqlUuvPBCz8XLXvYyCxcudPfdd5tz/vnn6+3t9dnPftbs7Kw5U1NT5nzta18z5+DBg2ZnZ0WEo912223mzZvnggsucFhXV5ff+Z3fcfvttzuWyy+/3KOPPuraa6/VarXMaTabrrnmGplMxute9zrP5PLLL/foo4+69tprtVotc5rNpmuuuUYmk/G6173O8fzRH/2RV7ziFT72sY/Zs2ePIy1YsMDdd99t+/bt5szOzvr7v/97qVTKr//6r/t5mi+RSCQSiV+ggwcPmnPw4EHPxbe//W3vfve7ffzjH/elL33Jgw8+aM2aNS677DKf+9znrFu3znMxf/58c7q6uhzL17/+df/3//5fl156qZe85CVO1JIlS8yZnp4255WvfKUf/OAHfvCDHzjsN37jN0xNTdmxY4c5v/7rv27BggUajYaj3XvvvR566CHr1q1z2Ktf/WqLFi1y5513KLhG2gAADv9JREFUOtqSJUu84hWv8N3vftdLXvISp556qlNPPdWpp55qwYIFpqenvepVr3I8S5Ys8YpXvMJ3v/tdL3nJS5x66qlOPfVUp556qgULFpienvaqV73KsVx++eXOP/98X/jCF3z/+993LHfeeacj7dq1y5xly5b5eeqSSCQSicQv0J49exw6dMjixYs9VwcPHrR9+3bbt283Z8mSJS688EJvf/vbvf/973fvvfd68MEHnYjly5ebs3fvXsfzuc99zjXXXOPSSy/1X//rf3UiZmZmzFm6dKmHHnpIRDjppJMcNn/+fP39/e6++26HveMd73DHHXeYmZlxLLfddpt3vetd1qxZ45577rFu3Trf+ta37N2719GWL19uztq1a61du9axHDx40PEsX77cnLVr11q7dq1jOXjwoKO95S1v8drXvtbk5KTXv/71vvKVrzh06JAjRYSpqSlHmp2dNWfBggV+nrokEolEIvELFBEefPBBp512mu7ubgcOHHC0JUuWeMMb3uDOO+/00EMPOZ69e/f627/9W4cOHbJ+/Xq//du/7cEHH3Qifuu3fsucf/qnf3I83//+933lK1/xpje9yc033+xE3H///WZnZ/32b/+2kZERd911lwsuuMDFF1/sa1/7mre//e1+7dd+zUMPPSSfz/v93/99q1ev9h/+w39wPLfffrv169d77Wtf6/777/fqV7/aZz7zGcfSarXM+fKXv+zzn/+8Y4kIx9Nqtcz58pe/7POf/7xjiQhH+4M/+AP/8T/+R81m0zXXXGPdunVuueUWR2q32w4dOuSXwXyJRCKRSPyC3XHHHbq6urzmNa9xLOeff753vOMdXvKSl+jq6vKZz3zGBz/4Qcdz3333mbNgwQIn4mUve5kzzjjDPffcY/fu3Z7JF77wBfv37/ee97zHiZiZmfF//s//8da3vtWSJUt885vfdN1113nTm97k4x//uG9/+9uGhob85m/+pk9+8pNOPvlkGzdutHfvXsczPT1tfHxcX1+f17/+9Vqtlq9//euOZefOnQ4dOmT58uVmZ2fNzs6anZ01OztrdnbWoUOHtNttx7Nz506HDh2yfPlys7OzZmdnzc7Omp2dNTs769ChQ9rttqNde+21Jicn3X///e68807vfOc7dXd3+2U1XyKRSCQSv2D/8A//YGZmxvr16y1evNiRli1b5q1vfatGo+Hee+/VbrdNTk5at26dtWvXOpbf/d3fNefee+/1bHp6enzoQx8SEa6//nrPptlsuv76651xxhmKxaIT8fnPf96BAwdcffXVXvrSlxoeHnb55Zd773vf684773TzzTd717ve5ZJLLvGpT33K9PS0Z3PbbbdJp9Pe+c532rZtmwMHDjiWxx57zDe/+U2vetWrrFixwpGWLl3qxhtvdMkllziexx57zDe/+U2vetWrrFixwpGWLl3qxhtvdMkllzja2NiYw2688UZLlizx+7//+35ZdUkkEolE4hdsz549/vqv/9qHPvQhf/mXf+mLX/yiXbt2WbVqlYsuukgqlfJXf/VXDrvhhhu8/OUv95GPfMRdd91l+/bt9u7da8mSJX73d3/XWWed5R//8R+Nj4870plnnunkk082p7u727/6V//Kq1/9aieddJL/8l/+i+9+97tOxC233OINb3iD1atXOxF79+511VVXqVarrr32Wl/96lfdddddvv/97zt48KBTTz3VGWecYfv27R555BEnYtu2bd773vdKp9Nuv/12z+T666935pln+tjHPua6665z//336+3tdfnll5udnXXLLbd4Jtdff70zzzzTxz72Mdddd537779fb2+vyy+/3OzsrFtuucUzefjhh/3DP/yDiy++2P/+3//bvn37/LLp8lOaLZUkEolEIvFCufPOOz366KP+8A//0Pvf/35zIsLExIShoSGNRsNhjz32mD/7sz9z8cUX+zf/5t8499xzHbZr1y7XXXedL33pS462du1aa9euNSci7Nu3z/bt2/3P//k/7dixw4mKCENDQz71qU85UT/84Q/9+3//773xjW/U39/voosuclhEuO+++zz88MMeeeQRJ+LAgQPuuusuZ555pu985zueyQMPPOCqq67yvve9z0c+8hFzDh06ZPv27T7xiU945JFHPJMHHnjAVVdd5X3ve5+PfOQj5hw6dMj27dt94hOf8Mgjj3g2X/jCF5x//vkuueQSQ0NDftnMQzhBs6WSRCLxwjtpbEwikehYsmSJU045xZ49e+zbt8+zWbJkiVNOOcX+/fvt3r3bvwSnnHKKJUuWaLVadu/e7cCBA34eli5dKpPJmJ6e1mw2PVdLly6VyWRMT09rNpt+VcxDOEGzpZJEIvHCO2lsTCKRSCR+0jve8Q4n4gtf+ILno8tzcNLYmEQikUgkEolfNfMQEolEIpFIJF7E5kskEolEIpF4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)





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

![image-20250303111507392](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303111507392.png)



*将所有执行过的写命令都记录下来，history，在恢复的时候将记录的命令全部执行一遍。*



以日志的形式来记录每个写操作，将 Redis 执行过的的所有指令记录下来（读操作不记录），只许追加文件不许改写文件，redis 启动时，会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容，将写指令从前到后执行一次以完成数据的恢复工作。



 **如果 aof 文件损坏或有错误，redis 无法启动。我们需要修复 aof 文件。**aof 文件损坏修复: redis-check-aof —fix appendonly.aof





![img](data:image/png;base64,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QRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQZAnPelJufHGG/OhD30oz3zmM4MgyN/6W38rH/nIR/LP/tk/C4L84A/+YG666ab84i/+Yra2toIgyGWXXZYPf/jD+fmf//kgyGWXXZabbropP/VTPxUEQZAf/dEfzb/+1/86N954Y570pCcFQX78x388N910U9761rfmQx/6UJ761KcGQRDkzW9+c9761rcGQRAEQRAEueSSS/LBD34w119/ffbu3RsEQRDk+7//+/Pud787wzDk3HPPDYL85E/+ZG666aa87W1vy2/+5m9mz549QRAEQV7+8pfnxhtvzHve85782q/9WhAEQRAEQRAEQZArr7wyN910U57xjGcEQRAEQRAEQRAEec1rXpObbropj3/844MgCIIgyM/93M/l3/7bfxsEQRAEQRAEQRAEQRAEQRAEQRAEQRAEQRAE2bLxyCnb2jJajzY2Nja+K2xvb7vwwgt96EMf8rnPfc6Jfv/3f9/v/M7v+OEf/mFN0zhi3759jvjYxz7m0KFDTvTpT3/ab/3Wb7njjjuciU9+8pN+5Vd+xYUXXujqq692sve9730OHTrkH/yDf+DhevKTn+y6667zP/7H//DmN7/Z7bffbs+ePX78x3/cS1/6UhdffLHdu3e7/fbb/dIv/ZJzzjnHtdde62T/8T/+R094whNcccUVTmWxWPhP/+k/SeJUnvjEJ7riiitcddVVLrvsMrt373Y2nvjEJ7riiitcddVVLrvsMrt373am9uzZ46KLLrJnzx4ne8ITnuAFL3iBF73oRS6++GKPht02HhHtUA3tbFwujDY2Nja+O/zYj/2YI/79v//3TuWjH/2oPXv2eNKTnuSIgwcPOuKHfuiH/Pf//t+d7P3vf7+z8clPftKf/MmfuPLKK914442+/vWvO+b//J//49/8m3/jta99rec85zn+8A//0Nl6xSte4ZxzzvGOd7zDvffe63u/93u95S1vsWfPHrfeequXvvSlPvnJT3ruc5/r2muv9eu//uve+MY3+pf/8l/62te+5phPfepTvva1r3nhC19omiYnuuSSS1x88cWmafKCF7zAyV7wghd4/etf74gvf/nLnva0p7nrrru87W1vc+DAAQ/lBS94gde//vWO+PKXv+xpT3uau+66y9ve9jYHDhzwYPbs2eP666/3uMc9zhvf+EYnevrTn+6tb32rIx7/+Mfbs2eP97znPcZx9J20ZeMRMS4bTbOwHG1sbGx8V9izZ4+LL77Yl770JV/96ledyp/+6Z96xzveYZ5nR3zqU5/yl3/5l1796lf7h//wH3rmM59p165dvhWf+tSnfM/3fI99+/Y52Yc//GH/+3//b6973etsbW05G1tbW6688kr/7t/9O3fccYfHPe5x3vzmN/vSl77k2muvdd111/nVX/1VL37xi/3e7/2eI2655RZbW1ue/vSnO9HBgwd94hOf8OxnP9tFF13kRFdeeaXPf/7zvvjFLzrZs571LL/wC7/glltucfXVV3vDG97gmmuu8fnPf96b3vQm5513ngfzrGc9yy/8wi+45ZZbXH311d7whje45pprfP7zn/emN73Jeeed53S2trb80i/9kr/21/6a/fv3u+uuuxyztbXl537u5/zKr/yKq6++2qtf/Wqf/exn/czP/IzzzjvPd9KWjY2NjY3/Lz35yU92zjnnuO2225yp22+/3fXXX++rX/2qF7/4xd7+9rf7zd/8Tddff72Xvexlzj//fGfrf/2v/+WICy+80Mnuuecev/7rv+7pT3+6q666ytl4xjOe4YlPfKJbbrnFEYvFwvd93/f51V/9VQcPHnTEV77yFUd84hOfcMS9997r4MGDkjjZxz/+ceecc46f+ImfcMzu3bv92I/9mGmanMo111zja1/7mne9613uvvtuR9xxxx1uuOEG5513nhe96EUezDXXXONrX/uad73rXe6++25H3HHHHW644QbnnXeeF73oRU7nH/2jf6SUYv/+/f7iL/7CiXbt2uWWW27xmc98xhEHDx7027/9277ne77HD/zAD/hO2nI67aDWqtZqaD2GFd1UDa0zUzrT1CkeA0pnmjrFd1rRTdXQOktFN1VD6zumdJOpK44p3aTWqtaqTp3iMap0pqlTPJiim6qhdRZaQ61qraau2HgEtYM6dYqHr3STWqtaqzp1iseG0k1qrWqddMUD7r33Xkfce++9zsZnP/tZ1157rbe85S0+8pGPuPXWWz372c929dVXe897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o1KK+5RtbRmtxxmzcT16JJRu0I690SOkbGvLaD3OmI3rUSnFyca+N3uElW1tGa1H32Tse7Oz1RqGYmfZm52p0Xr8f+3BT6jlZd0A8M8rFi+BKbpJDi177lZwoYsWHQQX9RyDdNGljasOSAu7P4jQRaarFr8DElg/V4FKm0DiPCtNjgTVInRRhN1vFI3woISOZg35D7/vHefOH0eduaNTvcX5fAKhrZtSiqNp1i2cFPtB2VFcRFuazeZW0SxnM/NViNXcbDY3hg9XBkNtxrEJJ4U2NuH9Yj8oO4rLJ/aDsqM4rVm3QGjrppTirGbdAqGtm1KKd5XBUJtxbMKBaMYWyk5xSrNu4aTYD8qO4kBZqKVZt0Bo6+aMslBLs26B0NZNKcVZzbqFk2I/KDuKi2vrptSF4qRqUZt1c0liPyg7igNloZZm3QKhrZtSikvxyiuveOihh1x33XXGcXTrrbe64YYbfPnLXzaOo09+8pMefPBBpz366KN+//vfu+eee3zrW99yyy23uPHGG91yyy2++93v+sIXvuBXv/qV3/zmN851ww03uOOOO9xxxx2+9rWv+fa3v+0HP/iBz372s374wx967rnnHMWTTz7pj3/8o0996lOO4tVXX3Xvvfc66fvf/74777zT5z73Of/7v//rE5/4hM985jNuvfVW1113naP65S9/6fXXX3fllVfabDYu5JFHHvH222+777773Hzzza6//no33nij+++/31tvveXJJ590IY888oi3337bfffd5+abb3b99de78cYb3X///d566y1PPvmkC3nxxRf97Gc/c/vtt/v0pz/t/6MrfSQhwvtFCOcoO4omHIgQzlF2FE34cGWniAhUw17R5jOjwVSLo6jTpKzmluG9IoRzlB1FEw5ECOcoO4omXEg1bQaxnJu1QDFMG5thbr52Sozms9G7irMihHOUHUUTDkQI5yg7iiYciBAup2qoYZyHyypCOEfZUTThtBDhsqvDHqu55nwhwiWr06Ss5pbh6CKEc5QdRRMuIkL4CMpCFdaKRaUtw5FECB+sDJNpryoOxcrHVYbJtFcVh2LljAjhHGVH0YQDEcI5yo6iCSdVU+/eoxXvihA+RITwISKEc5QdRRMORAgfQVtr02BRRuvFoLbR0sWVYTLtVcWhWDkjQjhH2VE04eh+/vOf+/vf/+7OO+/0jW98w0mZ6Xe/+52HH37Yn//8Z6f94x//8J3vfMftt9/ui1/8os9//vNO+8tf/uJHP/qRn/70p8530003uemmm5yUmV577TXPPPOMn/zkJ/7whz84qsz08MMP+973vueoXnjhBd/85jd96Utfcuutt/rKV77itMz0pz/9yYsvvujll192FG+88YZf/OIXbrjhBr/97W9dyPPPP+/ee+911113ueeee5z0zjvveOaZZzzwwANefvllF/L888+799573XXXXe655x4nvfPOO5555hkPPPCAl19+2cX8+Mc/Np/PffWrX/Xwww/7/+ZKl1MpCsKh2BcOlaIgHIp94Z+oTqayMl+G9ylFQTgU+8KhUhSEQ7EvXETZUaIZWzgljOtmb1GIfVEWCsIpZaeICOxQioJwKPaFQ6UoCIdiXzhUioJwmdSFWqrau7M2el2Zz0fhIypFQTgU+8I/WRkMtRln4bKok6mszJfhkpSiIByKfeGfoZr6pDpl0/e8a9PV1dx8DBdUioJwnjKY9mjzmTFQJ33w8ZTBtEebz4yBOumDs0pREA7FvnCoFAXhUOwLh2JlPh+F85QdF1SKgvABSlEQDsW+8HE142owLSq1aGNzUWUw7dHmM2OgTvrgrFIUhEOxL1y6Z5991rPPPuuaa65x9dVXe+WVV7z22ms+yOuvv+6xxx7z2GOPueaaa1x99dX+9re/OX78uPP9+te/dtttt7kUTz/9tKefftqHee6559x2220uxZtvvunxxx/3+OOPu/rqq11zzTXefPNNx48f98Ybb/gwTzzxhCeeeML5HnzwQR/k61//uvNFhLvvvtu1117rqquu8tJLLzlx4oTzPfXUU5566inniwh33323a6+91lVXXeWll15y4sQJ53v00Uc9+uijznf8+HF33HGH0x566CEPPfSQ8z3//PNuu+02/2pXuFxirUW1qAVFXVQR4V2x1qJa1IKiLqqIcL5SFoqzYj+UUtCMq1A33WYoiouppqlq4yh8kGpRC4q6qCLCWdWiFhR1UUWEIynVohanFMOiOqVZt2oYquJAGQx7tHU4pVrUgqIuqojwrlhrUS1qQVEXVUR4V6y1qBa1oKiL6ozYF6VaFAeKuqiOpC3NZjOz2cxsNjNfhVjNzeajcAGxL0q1KA4UdVGdEWstqkUtKOqiigiXRYQoRfF+ZVGVttYcUeyLUi2KA0Vd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)





如果文件正常，重启就可以启动恢复了





![img](data:image/png;base64,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ne98x4tf/GK33HKL7T3vec/zjW98w9e+9jW78qAHPcjCwoJDDjnEF77wBd///vcdfvjhTjzxRLfccot3vOMd/v3f/902Rx99tGc84xk2bdrk1FNP9V//9V929JCHPMSrX/1qW7zlLW+xYcMGd8b8/LwjjzzS5z//eXfGM57xDAcccIAvfelLbs/znvc8119/vUsvvdT/psV222A6v6D0vbrSrQazhQXTAcWPGaZTs5WdxlazhXkLbaerneJWw8yqhZnRaDT6v+o+97mPM8880/e+9z2vec1rrF+/3hZPecpTtG1rxYoV3vnOd9riN37jNzzrWc/y93//9z784Q9LYounPvWp/viP/9jVV1/tn/7pn2zv9NNPd+2119pi//33Nz8/79RTT3Xf+97XmWeeKYltvvOd79h///2ddNJJ3v/+9/tpHHnkkc444wxr1qzxZ3/2Z26++WbbLF682NOf/nRnnHGGs846y9lnn22bb33rWw4++GC/+Zu/6aMf/agdHX300b75zW869NBDjXZuse3NFkxm7sDMwvzET5gtmMxsZ2ZhMvEjg9l03mxqNBqNRrf67d/+bXvvvbdXv/rV1q9fb5tPf/rTfvVXf9UJJ5zgve99rxtvvNEjHvEIN9xwgw996EO29y//8i/uf//722effdyRa6+91kc/+lFXX321V7ziFY4//nhnn322ba655hrnnHOO5zznOc4991xXXXWVO+Pwww93xhlneMc73uHss8+2xUMf+lCHH364iy66yPr16/V971vf+pZXvepVLrroIrVWW1x99dXWr1/vmGOO8dGPftT29thjD095ylN85CMfsWLFCjtzv/vdTynFzTff7OKLL3bttdfaHfe73/2UUtx8880uvvhi1157rV1ZtGiRAw880HXXXeemm26yvQc84AEOP/xw69evt3btWnelxUaj0Wh0l5qbm3PUUUe59NJLXX755Xa0Zs0amzdvtv/++7vxxhtdd9119tprL4cffrjLL7/c9t7znvfYXZ/5zGc8+9nP1jSNs88+2/Y+/OEPO/bYY5166qne/OY3uzNOO+00n/3sZ5199tkWLVrkj/7ojzzmMY9x8cUXe/azn23dunU2bdrkr//6r33uc59z/PHHe9e73mWbc8891yte8QoPfvCDrVu3zjaPe9zjJHHBBRdYsWKF7c3NzXnRi17k2GOPtXbtWnvttZdDDz3U29/+duecc47bMzc350UvepFjjz3W2rVr7bXXXg499FBvf/vbnXPOOe7IwsKCX/u1X/PKV77STTfdZJsVK1b4rd/6LVdddZUHPOABzjvvPG95y1vcVRYbjUaj0V1qMpm45z3vae3atXbmyiuv1HWdbT71qU858cQTvfGNb/TJT37ShRde6Ctf+Yof/vCH7qwLLrjAc57zHPvss4/rr7/eNrfccot3vvOdXvOa1/j4xz/ukksusTvuc5/7eMQjHuElL3mJLZ797Gd75CMf6WUve5lrr73WIx/5SK9//eu98Y1vtMUXv/hFT37yk23v/PPPd8MNNzjmmGOsW7fONkcffbRPf/rTNm3aZEennHKK+fl5r3zlK1122WW2OP74473kJS/x1a9+1de+9jU7c8opp5ifn/fKV77SZZddZovjjz/eS17yEl/96ld97WtfszPPf/7zPfGJT3TGGWe49tprbVNKcfPNN1uxYoWNGzc69thjvfzlL/exj33M17/+dXeFOaPRaDS6S+2zzz62+P73v293fOtb3/La177WN77xDSeeeKLXve513ve+9/mrv/orxx13nLm5Obvre9/7ni0OOOAAOzr//PP953/+p9NOO82iRYvsjgc/+MGuueYaV155pUWLFnnmM5/pwx/+sGuvvdYWmzZtctNNN7ngggtssccee7jxxhtt75ZbbvGv//qvnvrUp1q8eLEt9t13X49//OP1fW9He+65pxNPPNHZZ5/tsssus80nP/lJtVYnnHCCndlzzz2deOKJzj77bJdddpltPvnJT6q1OuGEE+zMcccd56STTvK6173Ohg0bbG/ffffVdZ2NGzfa4tOf/rQtHvSgB7mrLDb65RG3b5HRaHQ3cf3119tijz32sLsuueQSr3rVq9z73vf2yEc+0qMe9Si//uu/7mUve5lHPepR3vSmN9kd++67ry1uueUWO3PWWWd5y1ve4thjj3XOOefYlXve855uvvlmW+y3337uda97WbdunW0e+9jHuuCCC9x00022OPLII/3Hf/yHHZ133nmOO+44T3jCE3z2s5/1lKc8xfr16331q1912GGH2d5DHvIQe+65p6VLlzrllFNsb25uzhFHHGFnHvKQh9hzzz0tXbrUKaecYntzc3OOOOIIOzryyCO9+MUvtn79euvWrbOjr3/9666//nrb3HTTTW6++Wb77befu8qc0Wg0Gt2lrrrqKlssXbrUzszNzdlrr73szDXXXOPTn/60t771rf7gD/7AF7/4RU9+8pMdccQRdschhxxi48aNvve979mZK664wic+8Qm/+7u/a++997Yr3/3udx100EH22Wcf1113nR/84Ace9rCH2eI+97mPE044wVe/+lVbHHPMMSaTiXPPPdeOLrnkEuvXr3fMMcfY4uijj9b3vZ3Ze++9bbFx40ZLliyxZMkSS5YssWTJEueff77PfOYzdmbvvfe2xcaNGy1ZssSSJUssWbLEkiVLnH/++T7zmc/Y3iGHHOIP//AP/fmf/7l73/vejjvuODu65ppr7Gjz5s0WLVrkrjJnB6Xt1VrVWvVtsb3Sdvpa1VrVvtMUWzWdWqtaq1qrWqtaq9o1dq1ou16tVa29ril2T9F2vVqrWntdU+yO0vboGCR4AAAMOklEQVRqrWqt+rbYfUXT9vpa1Vr1XWOb0rS6vqq1qrXXtcVtmk6tVa1VrVWtVa1V7Rq3Ka2ur2qtau11bTEajX753XjjjS655BKPfvSjLVq0yI4e/ehHe9/73ueBD3ygvffe2/Of/3yHHXaYHf3whz/0kY98xBYHH3ywXdl7770dddRRLrroIhs3bnR73vOe95ibm3PKKafYlUsvvdR1113n6U9/us2bN3vXu97lBS94gTPPPNPLX/5yX/7yl5100kne/OY3O/nkk5155pluuukmO3Peeed5zGMe4+EPf7gjjjjCpz71KTtz9dVX2+Lzn/+81atXW716tdWrV1u9erXVq1f74Ac/aGeuvvpqW3z+85+3evVqq1evtnr1aqtXr7Z69Wof/OAHbe+AAw7wt3/7t77whS/40Ic+5JRTTrHnnnvaXhL/2+bsYJjOm0wm5lcNfkxptWWNhcnEZDIxPyu6rlXcarZgMpmYTCYmk4nJZN6qYbBqOrMrTddrhqn5ycRkfkbXaYtdarpeM0zNTyYm8zO6Tlvs0jCdN5lMzK8a3Bml7XTNzMJkYjKZWFizTFvcqrF8OWsWJiaTicn81ND0ugazBZPJxGQyMZlMTCbzVg2DVdMZGl2/kum8yWRiMj81NJ22GI1G/wd88pOfdN/73tfxxx9ve4sWLXLyySfbsGGDr3/962644QZPetKT/P7v/765uTk7OvTQQ22xfv16u/J7v/d79txzTx/4wAfckeuvv94//MM/aJrGwQcf7I5s3rzZu971Ls9//vM94QlPcO655zr99NO9+93v9trXvtYb3vAGr3/965111lle+tKXuvLKK92evu/Nzc1p29bFF1/su9/9rp25/PLLff/73/f4xz/ejk4//XTHHHOMnbn88st9//vf9/jHP96OTj/9dMccc4ztXXrppT772c/a4mMf+5hFixZ51rOe5e5mzu4aphYWZgZbDdOpWSmKnWhaK82sGexCY3kzM53ODG41TE1X0Swv7lhjeTMznc4MbjVMTVfRLC9+PhrtysHC/NRgq2E2NR3cama6MDUbbDXMrJkNyrLiJzStlWbWDCjLlGGV6Wxwm2FmOh00y4vSdvquUYrRaPRLqu97X/jCF5x22mme9axnOeSQQzz0oQ/1mte8xsMe9jBve9vbbPPWt77VQx/6UG94wxs86UlP8oAHPMCDH/xgz33uc61YscKFF17osssus73HPvaxjjzySEcddZRnPOMZ3vzmNzvuuOP84z/+oy9/+ct25ZxzznHFFVdYunSpXen73rvf/W5/+qd/6qUvfal99tnHZZddZvPmzebm5ixevNi6dets3rzZHdmwYYNLLrnEQQcdpO97t2fTpk3e//73e+ITn+jkk0+2//7722+//axYscLTnvY03/72t+3Mpk2bvP/97/fEJz7RySefbP/997fffvtZsWKFpz3tab797W/b3ubNm21z0003+cAHPuCkk05yr3vdy93JYj+tZrlmGEztqGjbxmw6MdiFskwZBoMfGdYOyvKCwe0qy5RhMPiRYe2gLC8Y/ExKq+9XGhYmFma2KsuUYbCs63VNscUwW7CwMDPYQWm0TTFMBz+uaNvGbDoxuH2lFMPC1LRtdX2nDDOrplPT2WA0Gv1yef3rX+/UU091yimneOELX2iLb37zm84880wXXnihbdauXevVr3613/md3/GKV7zCokWLbHHttdeazWbe+9732tGf/Mmf2OaHP/yhdevW+Yu/+Auf+9zn7I7Nmzc766yz/OVf/qXd8aEPfcjFF1/sec97nlWrVtm0aZObbrrJvvvu6/LLL/c3f/M3rrjiCrty3nnnOeKII/zbv/2bO/Lxj3/cPe5xD8997nOtWLHCFt/85jedeeaZLrnkErfn4x//uHvc4x6e+9znWrFihS2++c1vOvPMM11y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)

 





**重写规则说明**



**aof 默认的是文件的无限追加，文件会越来越大**





![img](data:image/png;base64,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9ltffPGFf1LvlevWP+t99eLGYrFYLBaLxW/1mU9xvvXVixvX4eDy4pk3x958vPHVixuLxWKxWCwWv8dnPlXRSr02q6Q107m3WCwWi8Vi8Xv97e3bt+HRN9984/PPP/fk22+/9cUXX/i5vBul/tl8vPHm2FssFovFYrH4rf7uE1xePJOuT9LzjYu9sn7lq4eDxWKxWCwWi9/rM5+klda96UxSVKbzwWKxWCwWi8Uf4W9v374Nj7755huff/65J99++60vvvjCYrFYLBaLxZ/lM4vFYrFYLBZ/AX//7//+b0/+67/+y2KxWCwWi8X/lL+9ffs2PPrmm298/vnnnnz77be++OILi8VisVgsFn+WzywWi8VisVj8BXzmP1ol24zy2i9KmpOyG5XdSVZY/AdJmpOyG5XdSVb4X6iSbUZ57a+j3is3W4n/xeq9crOV+LFKthnltd+v2FptthL/LGlOym5UdqNys5X4Nyu2VputxOKXVbLNKK8t/kL+7s9SbOXrl9LCd+bznetxZxr8G/Wu98/8slbW9C4vbkwWHyNpTnK33hx7f22trOldXtyY/G/Vu94/839R0pzkbr059v4cvev9M/9u8/HGV0fUe2XjD5U0J7lbb469v7KkOcndenPs/RmS5iR3682x98961/tn/litvNtLhztv7ndm77XyzV5aeNSbHm5dzj0q2eYkK/zEfLzx5tj7v+jvfotia7Xmcr+TrE+S443r4Fe08s1z88ONr869J0m9ldWt6XjwP6KoJBb/kYpKYrFYLP5k9XPp0JuLLyWYPalkm73kfOOrY0+xlW/2svsb18F3podnLmeLR3/3WxRfcr4za2V1b3rw6+rn0uGVN+fee/N55+JRsbVaMw3PZXVlPt+5POzM3qtk672srtCbjrcux947lWy9l9UVetPDrcu59yRpTlZN5cn08Mzl7HuVbHOSFb6Tdy161/sb16GVdy/N9zeug3fqvbJ57c39zuxfS9ejvPbOcOfN/c7sA4qt1ZppeC6rK/P5zuVhZ/ZeJVvvZXWF3nS8dTn23qlkm73k3EuaVuLRcOfN/c6skq33srpCb3q4dTn33qlk672srtCbjrcux55ia7VmGp7L6gq96/2N64B6r1y33jkpG48OLi9uTUjXo7z2znDnzf3O7HvF1mrzUqI3nXtp8dqb+53Zk0q23svqCr3peOty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)





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

![image-20250303163540353](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163540353.png)



![image-20250303163553324](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163553324.png)

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

![image-20250303163633297](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163633297.png)



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

![image-20250303163700981](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163700981.png)

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

![image-20250303163725897](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163725897.png)

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

![image-20250303163751454](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163751454.png)



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

![image-20250303163831774](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163831774.png)



![image-20250303163843447](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163843447.png)

![image-20250303163853514](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163853514.png)



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



![image-20250303163055419](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163055419.png)



**2、缓存空对象**
当存储层不命中后，即使返回的空对象也将其缓存起来，同时设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源；

![image-20250303163111987](C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163111987.png)





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

<img src="C:\Users\苏传辉\AppData\Roaming\Typora\typora-user-images\image-20250303163218209.png" alt="image-20250303163218209" style="zoom:100%;" />



其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因此自然形成的缓存雪崩，一定是在某一时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。



#### 解决方案



**1、Redis 高可用**
这个思想的含义是，既然 Redis 也有可能挂掉，那多增加几台 redis 服务器，这样一台挂了还有其他的可以继续工作，其实就是搭建集群。



**2、限流降级**
这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读取数据库写缓存的线程数量。比如对某个 key 只允许一个线程查询数据和写缓存，其他线程等待。



**3、数据预热**
数据加热的含义就是在正式部署之前，先把可能的数据先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的 key，设置不同的过期时间，让缓存失效的时间尽量均匀。