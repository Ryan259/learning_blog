
## 消息中间件

### post email

1. 丢失消息
2. 重复发送

## Redis

### 雪崩

### 熔断

### 穿透

解决方案: https://developpaper.com/redis-cache-avalanche-cache-breakdown-cache-penetration/

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
