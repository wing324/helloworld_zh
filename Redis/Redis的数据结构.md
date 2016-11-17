## Redis的数据结构

Redis的数据结构多种多样，千姿百态~



String字符串
------------
1. set  
   设置键值  
```
127.0.0.1:6379> set resource:look "redis demo"
OK
```
2. get  
   获得键值  
```
127.0.0.1:6379> get resource:look
"redis demo"
```
3. expire  
   设置键值过期时间  
4. ttl  
   显示键值是否过期及过期时间（正数：过期秒数，-1：永不过期，-2：已经过期）  
```
127.0.0.1:6379> set resource:look 'redis demo'
OK
127.0.0.1:6379> ttl resource:look
(integer) -1
# 键值永不过期
127.0.0.1:6379> expire resource:look 10
(integer) 1
127.0.0.1:6379> ttl resource:look
(integer) 9
# 键值还有9s过期
127.0.0.1:6379> ttl resource:look
(integer) 7
# 键值还有7s过期
127.0.0.1:6379> ttl resource:look
(integer) -2
# 键值已经过期
```
5. INCR  
   让键值自增  
```
127.0.0.1:6379> set connections 10
OK
127.0.0.1:6379> incr connections
(integer) 11
127.0.0.1:6379> incr connections
(integer) 12
```
6. DEL  
   删除键值  
```
127.0.0.1:6379> set name wing
OK
127.0.0.1:6379> get name 
"wing"
127.0.0.1:6379> del name
(integer) 1
127.0.0.1:6379> get name
(nil)
```


list列表
--------
1. RPUSH  
   在列表的右侧添加值  
```
127.0.0.1:6379> rpush name 'wing'
(integer) 1
127.0.0.1:6379> rpush name 'fianna'
(integer) 2
127.0.0.1:6379> rpush name 'bibi'
(integer) 3

# 列表中可以添加重复值
127.0.0.1:6379> rpush test 'wing'
(integer) 1
127.0.0.1:6379> rpush test 'wing'
(integer) 2
127.0.0.1:6379> rpush test 'wing'
(integer) 3
127.0.0.1:6379> lrange test 0 -1
1) "wing"
2) "wing"
3) "wing"
```
2. LPUSH  
   在列表的左侧添加值  
```
127.0.0.1:6379> lpush name 'zhou'
(integer) 4
```
3. LRANGE  
   从左开始按照对应的范围去取列表值，-1表示一直到最后一个列表值，注意Redis的列表从0开始标记元素  
```
127.0.0.1:6379> lrange name 0 -1
1) "zhou"
2) "wing"
3) "fianna"
4) "bibi"
127.0.0.1:6379> lrange name 0 1
1) "zhou"
2) "wing"
127.0.0.1:6379> lrange name 2 4
1) "fianna"
2) "bibi"
127.0.0.1:6379> lrange name 2 6
1) "fianna"
2) "bibi"
127.0.0.1:6379> lrange name 2 -1
1) "fianna"
2) "bibi"
```
4. LLEN  
   显示列表的长度  
```
127.0.0.1:6379> LLEN name
(integer) 4
```
5. LPOP  
   删除列表左起第一个元素  
```
127.0.0.1:6379> LRANGE name 0 -1
1) "zhou"
2) "wing"
3) "fianna"
4) "bibi"
127.0.0.1:6379> LPOP name 
"zhou"
127.0.0.1:6379> LRANGE name 0 -1
1) "wing"
2) "fianna"
3) "bibi"
```
6. RPOP  
   删除列表右起第一个元素  
```
127.0.0.1:6379> LRANGE name 0 -1
1) "wing"
2) "fianna"
3) "bibi"
127.0.0.1:6379> RPOP name
"bibi"
127.0.0.1:6379> LRANGE name 0 -1
1) "wing"
2) "fianna"
```


SET集合
-------
1. SADD  
   在集合中添加值  
```
127.0.0.1:6379> SADD name 'wing'
(integer) 1
127.0.0.1:6379> SADD name 'fianna'
(integer) 1
127.0.0.1:6379> SADD name 'bibi'
(integer) 1
127.0.0.1:6379> SADD name 'zhou'
(integer) 1
```
2. SREM  
   删除集合中的指定的值  
```
127.0.0.1:6379> SREM name 'zhou'
(integer) 1
```
3. SISMEMBER  
   测试某个值是否在集合中，1表示存在，0表示不存在  
```
127.0.0.1:6379> SISMEMBER name 'fianna'
(integer) 1
127.0.0.1:6379> SISMEMBER name 'zhou'
(integer) 0
```
4. SMEMBERS  
   显示当前集合的所有值  
```
127.0.0.1:6379> SMEMBERS name
1) "fianna"
2) "wing"
3) "bibi"
```
5. SUNION  
   显示所有集合的合并值  
```
# 五重复值时显示所有值
127.0.0.1:6379> SMEMBERS name
1) "fianna"
2) "wing"
3) "bibi"
127.0.0.1:6379> SMEMBERS test
1) "i"
2) "who"
3) "am"
127.0.0.1:6379> SUNION name test
1) "wing"
2) "fianna"
3) "bibi"
4) "i"
5) "who"
6) "am"

# 存在重复值时，去重
127.0.0.1:6379> SMEMBERS test
1) "i"
2) "fianna"
3) "who"
4) "am"
127.0.0.1:6379> SMEMBERS name
1) "fianna"
2) "wing"
3) "bibi"
127.0.0.1:6379> SUNION name test
1) "bibi"
2) "fianna"
3) "i"
4) "wing"
5) "who"
6) "am"

```

6. SINTER  
   显示两个集合中的交集，即共同元素  
```
127.0.0.1:6379> smembers friends:wing
1) "c"
2) "b"
3) "d"
4) "a"
127.0.0.1:6379> smembers friends:fianna
1) "f"
2) "e"
3) "b"
4) "g"
5) "a"
127.0.0.1:6379> sinter friends:wing friends:fianna
1) "b"
2) "a"
```

7. SINTERSTORE  
   将两个集合中的交集，存入到SINTERSTORE后的第一个参数中  
```
127.0.0.1:6379> smembers friends:wing
1) "c"
2) "b"
3) "d"
4) "a"
127.0.0.1:6379> smembers friends:fianna
1) "f"
2) "e"
3) "b"
4) "g"
5) "a"
127.0.0.1:6379> sinter friends:wing friends:fianna
1) "b"
2) "a"
127.0.0.1:6379> sinterstore friends:wing_fianna friends:wing friends:fianna 
(integer) 2
127.0.0.1:6379> smembers friends:wing_fianna
1) "b"
2) "a"
```


SORTED SET有序的集合
-------------------
1. ZADD  
   在有序的集合中添加元素  
```
127.0.0.1:6379> zadd name 10 'wing'
(integer) 1

# 相同的数据是不会被添加到集合中
127.0.0.1:6379> zadd name 11 'wing'
(integer) 0
127.0.0.1:6379> zadd name 9 'fianna'
(integer) 1
```
2. ZRANGE  
   查看有序集合的元素  
```
127.0.0.1:6379> zrange name 0 -1
1) "fianna"
2) "wing"
```

3. ZCOUNT  
   显示有序集合在指定的排名中的数量  
```
127.0.0.1:6379>  zadd friends:wing 100 fianna 90 mama 80 daday 70 sister 60 brother 50 friend 95 who

127.0.0.1:6379> zrange friends:wing 0 -1
1) "friend"
2) "brother"
3) "sister"
4) "daday"
5) "mama"
6) "who"
7) "fianna"

127.0.0.1:6379> zcount friends:wing 90 100
(integer) 3
```

4. ZREVRANK  
   显示指定有序集合元素的对应等级  
```
127.0.0.1:6379>  zadd friends:wing 100 fianna 90 mama 80 daday 70 sister 60 brother 50 friend 95 who

127.0.0.1:6379> zrange friends:wing 0 -1
1) "friend"
2) "brother"
3) "sister"
4) "daday"
5) "mama"
6) "who"
7) "fianna"

127.0.0.1:6379> zrevrank friends:wing who
(integer) 1
```


Hashes哈希
----------
1. HSET  
   在哈希中添加元素  
```
127.0.0.1:6379> HGETALL user:1000
(empty list or set)
127.0.0.1:6379> HSET user:1000 name 'wing'
(integer) 1
127.0.0.1:6379> HSET user:1000 email 'wing324@126.com'
(integer) 1
```
2. HGETALL  
   查看哈希中的所有元素  
```
127.0.0.1:6379> HGETALL user:1000
1) "name"
2) "wing"
3) "email"
4) "wing324@126.com"
```
3. HMSET  
   对于哈希类型，一次设置多个值
```
127.0.0.1:6379> HMSET user:1000 name 'fianna' email 'fianna125@126.com'
OK
127.0.0.1:6379> HGETALL user:1000
1) "name"
2) "fianna"
3) "email"
4) "fianna125@126.com"
```
4. HGET  
   只获取哈希类型的部分指定元素  
```
127.0.0.1:6379> HGET user:1000 name
"fianna"
```
5. HINCRBY  
   对哈希值自增  
```
127.0.0.1:6379>  HSET user:1000 visit 10
(integer) 1
127.0.0.1:6379> HINCRBY user:1000 visit 1
(integer) 11
127.0.0.1:6379> HINCRBY user:1000 visit 1
(integer) 12
127.0.0.1:6379> HINCRBY user:1000 visit 10
(integer) 22
```
6. HDEL  
   删除对应的哈希值  
```
127.0.0.1:6379> HDEL user:1000 visit
(integer) 1
127.0.0.1:6379> HINCRBY user:1000 visit 1
(integer) 1
127.0.0.1:6379> HINCRBY user:1000 visit 10
(integer) 11
```
