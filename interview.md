
## 消息中间件

### post email

1. 丢失消息
2. 重复发送

## Redis

### 雪崩
   当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，也会给后端系统(比如DB)带来很大压力。

### 击穿

  key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求
  可能会瞬间把后端DB压垮。

### 穿透
  key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，
  不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

解决方案: https://developpaper.com/redis-cache-avalanche-cache-breakdown-cache-penetration/

---
双删操作, 执行写操作之前,删除缓存,写操作之后,隔一段时间,在删除缓存.
解决的场景是: before write done, when customer request, the value in redis is null,
            read the database, return the old data, meanwhile redis record the value;
            when writing operation is done, the value in redis is old data, so we need
            delete the value of redis, after a while, the value of redis sync the database.

### 一致性如何保正


## 幂等性

### 代码逻辑保正

### token 存储redis 并校验

## 全文检索
