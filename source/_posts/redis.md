---
title: Redis.1.安装与基础命令
date: 2019-03-20 11:59:05
tags: Redis
categories: 
    - DataBase
---
官方文档：https://redis.io/documentation   
菜鸟教程文档：http://www.runoob.com/redis/redis-tutorial.html   
# Redis 介绍
# 下载并安装redis-3.0.0
``` bash
[root@iZi1qirs2ab583Z ~]#wget http://download.redis.io/releases/redis-3.0.0.tar.gz
[root@iZi1qirs2ab583Z ~]# tar -zxf redis-3.0.0.tar.gz 
[root@iZi1qirs2ab583Z redis-3.0.0]#cd redis-3.0.0
[root@iZi1qirs2ab583Z redis-3.0.0]#make
[root@iZi1qirs2ab583Z redis-3.0.0]#make install PREFIX=/usr/local/redis
cd src && make install
make[1]: Entering directory `/root/redis-3.0.0/src'

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/root/redis-3.0.0/src'

#看到上面界面表示安装成功
```
# Redis启动
## 前端启动（不推荐）
进入redis目录 使用**前端启动命令**`./redis-server` 启动Redis 看到下面画面表示Redis启动成功，可以使用`ctrl+c`快捷键退出
``` bash
[root@iZi1qirs2ab583Z bin]# ./redis-server
25083:C 18 Mar 20:40:15.035 # Warning: no config file specified, using the default config. In order to specify a config file use ./redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.0 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 25083
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

25083:M 18 Mar 20:40:15.037 # Server started, Redis version 3.0.0
25083:M 18 Mar 20:40:15.037 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
25083:M 18 Mar 20:40:15.037 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
25083:M 18 Mar 20:40:15.038 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
25083:M 18 Mar 20:40:15.038 * The server is now ready to accept connections on port 6379
```


## 后端启动
### 准备工作
1. Redis源码包中的`redis.conf` 复制到安装目录下（`/usr/local/redis/bin/`）
2. 修改安装目录下redis.config 将`daemonize`改为`yes`

### 启动与关闭
1. 后端启动
``` sh
[root@iZi1qirs2ab583Z bin]# ./redis-server redis.conf
```
2. 正常关闭
``` sh
[root@iZi1qirs2ab583Z bin]# ./redis-cli shutdown
```
3. 非正常关闭（不推荐）
直接kill进程id 不推荐 

### 端口占用问题
1. 查找被占用的端口
    > ps -ef|grep redis
    
2. 杀掉占用端口的进程
    > kill -9 进程id

## 如何登陆
### 命令行登陆
1. 本地登陆（redis/bin目录下）
    > ./redis-cli
2. 远程登陆（redis/bin目录下）
    > ./redis-cli -h 127.0.0.1 -p 6379
3. 退出
    > exit

### Redis客户端【了解】
1. 安装客户端（redis-desktop-manager-0.8.0.3841.exe）
2. 使用客户端登陆
![1.png](1.png)

远程登陆错误请参考下面linux防火墙设置

### 解决登陆超时问题 设置防火墙
contos6.5 使用iptable防火墙策略
1. 方法一:使用命令添加端口号  
    ``` bash
    /sbin/iptables -I INPUT -p tcp --dport 8081 -j ACCEPT
    /etc/rc.d/init.d/iptables save
    ```
2. 方法一:修改`iptables`文件 复制一行修改为要开启的端口号
    >vi /etc/sysconfig/iptables

3. 修改iptable 需要重启iptable
    >service iptables restart

## Redis 数据类型介绍
Redis中存储数据是通过key-value存储的，对于value的类型有以下几种：   
- 字符串   
- Hash类型   
- List   
- Set   
- SortedSet（zset）   

PS：
在redis中的命令语句中，命令是忽略大小写的，而key是不忽略大小写的。

## 使用数据库
在使用客户端软件登陆后我们发现有redis默认有15个库,使用`select <db number>`切换数据库,当我们登陆后指定使用哪个库时候默认用第一个库（db0）,通过使用发现redis会为我们自动管理库的个数
``` bash
[root@iZi1qirs2ab583Z bin]# ./redis-cli
127.0.0.1:6379> select 1
OK
```

## String类型
自己理解：相当于普通的key value 键值对   
### String类型相关命令
1. 存入 `SET <key> <value>`
    > set name xiaoli   

2. 取出 `GET <key>`   
    > get name
3. 存入多个值 `MSET <key> <value> [key value …]`   
    > mset k1 v1 k2 v2 k3 v3 k4 v4    

4. 取出多个值 `MGET <key> [key …]` 
    > mget k1 k2 k3 k4
5. 取出值并重新赋值 `GETSET <key> <value>`
    >getset name zhangsan   
6. 删除 `del <key>`
    > del name
7. 递增数值 `INCR <key>`
（如没有自动创建key,如果存在key value必须为数字）
    > incr num
8. 递减数值 `DECR <key>`
    > decr num
9. 增加指定的数值 `INCRBY <key> <increment>`
    > incrby num 2
10. 减少指定的数值 `RECRBY <key> <decrement>`
    > decrby num 2
11. 追加value值(value后) `APPEND <key> <value>`
    > append name world
12. 获取字符串长度 `STRLEN <key>` 返回0 key存在
    > strlen name

## Hash类型
自己理解：相当于map存入map，一般用于存入json对象，当我们将json数据(jiso格式：`User “{“username”:”gyf”,”age”:”80”}”`)用redis String方式存储时候若只修改字符串中某一个值，需要重新赋值整个字符串操作，redis hash解决了这个问题，使用hash方式存储可以我们只对字符串中某个值操作，提高效率。   

hash叫散列类型，它提供了字段和字段值的映射。字段值只能是字符串类型，不支持散列类型、集合类型等其它类型。如下：   
![2.png](2.png)

### Hash类型相关命令
HSET命令不区分插入和更新操作，当执行插入操作时HSET命令返回1，当执行更新操作时返回0。   
1. 插入/更新 `HSET <key> <field> <value>`
    > hset user username xiaoli

2. 插入多个/更新 `HMSET <key> <field> <value> [field value ...]`
    > hmset user1 username xiaoli age 20

3. 当不key不存在时插入 `HSETNX <key> <field> <value>`
    > hsetnx user username zhangsan

4. 取值 `HGET <key> <field>`
    > hget user username

5. 取多个值 `HMGET <key> <field> [field ...]`
    > hmget user1 username age

6. 取所有值 `HGETALL <key>`
    > hgetall user1

7. 删除一个或多个字段 `HDEL <key> <field> [field ...]`
    > hdel user1 age

8. 增加数字（字段为数字）`HINCRBY <key> <field> increment`
    > hincrby user1 age 2

9. 判断字段是否存在 `HEXISTS <key> <field>`
    > hexists user1 age

10. 只获取字段名或字段值 `HKEYS <key>` `HVALS <key>`
    > hkeys user

11. 获取字段数量 `HLEN <key>`
    > hlen user1

## List 类型
### 回顾ArrayList 与LinkedList 的区别
比较|ArrayList|LinkedList
---|---|---
实现原理|数组（通过下标存储）|双向链表（每个元素都记录前后元素指针）
查询速度|快(根据索引查询)|根据要查询的元素位置决定（需要按顺序查询元素，若元素考前或靠后快）
新增删除|慢（数组需要移位操作）|**快**（插入或删除时候更改相邻元素的指针）

### Redis list 介绍
列表类型（list）可以存储一个有序的字符串列表，常用的操作是向**列表两端添加元素**，或者**获得列表的某一个片段**。   
**列表类型内部是使用双向链表（double linked list）实现的**，所以向列表两端添加元素的时间复杂度为0(1)，获取越接近两端的元素速度就越快。这意味着即使是一个有几千万个元素的列表，获取头部或尾部的10条记录也是极快的。   

### list类型常用命令
1. 向列表左边添加元素 `LPUSH <key> <value> [value ...]`
    > lpush list:1 1 2 3 4 5 6 7 8 9
2. 向列表右边增加元素 `RPUSH <key> <value> [value ...]`
    > rpush list:1 1 2 3 4 5 6 7 8 9
3. 查看列表 `LRANGE <key> <start> <stop>` stop参数为-1 查询到最后的那个元素 
    > lrange list:1 0 -1
4. 从列表左边/右边 弹出1个元素 `LPOP <key>` / `RPOP <key>`
    > lpop list:1
5. 获取列表中元素的个数 `LLEN <key>`
    >llen list:1 
6. 删除列表中指定的值 `LREM <key> <count> <value>` count `>0`从左侧删除`<0`从右侧删除`=0`删除全部值为value的元素(例子从右侧删除2个元素值为7的元素)
    > lrem list:1 -2 7
7. 通过索引获取元素 `LINDEX <key> <index>`
    >lindex list:1 10
8. 通过索引设置元素值 `LSET <key> <index> <value>`
    > lset list:1 10 5
9. 只保留列表指定片段 `LTRIM <key> <start> <stop>`
    > ltrim list:1 1 6
10. 查找元素值并再元素左|右插入元素 `LINSERT <key> <BEFORE|AFTER> <pivot> <value>`
    > linsert list:1 before 4 10
11. 将元素从一个列表转移到另一个列表中`RPOPLPUSH <source> <destination>` 只移动一个
    > rpoplpush list:1 list:2


## Set 类型
**集合中的数据是不重复且没有顺序**   
集合类型的常用操作是**向集合中加入或删除元素、判断某个元素是否存在**等，由于集合类型的Redis内部是使用值为空的散列表实现，所有这些操作的时间复杂度都为0(1)。    
Redis还提供了多个集合之间的**交集、并集、差集**的运算。   

## Set类型常用命令
1. 增加一个或多个元素 `SADD <key> <member> [member ...]`
    > sadd set:1 1 2 3 4 5 6

2. 删除一个或多个元素 `SREM <key> <member> [member ...]`
    > srem set:1 5

3. 获得集合中的所有元素 `SMEMBERS <key>`
    > smembers set:1

4. 判断元素是否在集合中 `SISMEMBER <key> <member>`
    > sismember set:1 7

5. 集合差集运算 `SDIFF <key1> <key2> `
    > sdiff set:1 set:2

6. 集合交集运算 `SINTER <key1> <key2>`
    > sinter set1 set2

7. 集合并集运算 `SUNION <key1> <key2>`
    > sunion user newuser

8. 获得集合中元素的个数 `SCARD <key>`
    > scard set:1

9. 从集合中弹出一个元素 `SPOP <key>`
    > spop set:1 

## SortedSet类型zset 
**zset是有序集合**
在集合类型的基础上，有序集合类型为集合中的每个元素都关联一个分数，这使得我们不仅可以完成插入、删除和判断元素是否存在在集合中，还能够获得分数最高或最低的前N个元素、获取指定分数范围内的元素等与分数有关的操作。    

在某些方面有序集合和列表类型有些相似。   
1、二者都是有序的。   
2、二者都可以获得某一范围的元素。   
但是，二者有着很大区别：   
1、列表类型是通过链表实现的，获取靠近两端的数据速度极快，而当元素增多后，访问中间数据的速度会变慢。   
2、有序集合类型使用散列表实现，所有即使读取位于中间部分的数据也很快。   
3、列表中不能简单的调整某个元素的位置，但是有序集合可以（通过更改分数实现） 
4、有序集合要比列表类型更耗内存。   
  
自己理解：set中存入map,元素（value）中加入了对应的score，通过scored对set进行排序
### zset常用命令
1. 添加元素 `ZADD <key> <score> <member> [score member ...]`
    > zadd scoreboard 80 zhangsan 89 lisi 94 wangwu 
2. 获取元素的score `ZSCORE <key> <member>`
    > zscore scoreboard zhangsan
3. 删除元素 `ZREM key <member> [member ...]`
    > zrem scoreboard zhangsan
4. 取范围内的元素列表从小到大 `ZRANGE <key> <start> <stop> [WITHSCORES]` *WITHSCORES 表示显示 score值*
    > zrange scoreboard 0 2 WITHSCORES
5. 取范围内的元素列表从大到小 `ZREVRANGE <key> <start> <stop> [WITHSCORES]`
    >zrevrange scoreboard 0 2
6. 指定score范围的元素 `ZRANGEBYSCORE <key> <min> <max> [WITHSCORES] [LIMIT offset count]`  *LIMIT offset count 参数指分页*
    > ZRANGEBYSCORE scoreboard 70 100 limit 1 2
7. 增加某个元素的score值 `ZINCRBY <key> <increment> <member>`
    > zincrby scoreboard 10 chener
8. 获得集合中元素的数量 `ZCARD <key>` 
    > zcard scoreboard
9. 获得指定score范围内的元素个数 `ZCOUNT <key> <min> <max>`
10. 按照排名范围删除元素 `ZREMRANGEBYRANK <key> <start> <stop>`
11. 按照score范围删除元素 `ZREMRANGEBYSCORE <key> <min> <max>`
12. 获取元素的排名从小到大 `ZRANK <key> <member>`
13. 获取元素的排名从大到小 `ZREVRANK <key> <member>`


## Keys命令【了解】
### 关于key生存时间
1. 设置key的生存时间 `EXPIRE <key> <seconds>`
2. 查看key剩余生存时间 `TTL <key>`
3. 清除生存时间 `PERSIST <key>`
4. 设置kay生存时间-毫秒 `PEXPIRE <key> <milliseconds>`

### 关于key操作
1. 模糊查询数据库中的key `KEYS *key* `
2. 查询指定的key是否存在 `KEYS <key>`
3. 删除一个key `DEL <key>`
4. 对key进行重命名 `RENAME <key> <new name>`
5. 判断key的类型 `TYPE <key>`


