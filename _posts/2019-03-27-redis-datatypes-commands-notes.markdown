---
title:  "Redis 原生数据类型和命令笔记"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: redis cache ruby rails
---

*tl; dr 以往项目一般只用到 `Rails.cache` 的一些方法，这次第一次大量用到 redis 的原生命令*

*ps: rails 5.2 新增了 ActiveSupport::Cache::RedisCacheStore*

### 原生命令

删除 key

 `del(key)`

找出符合条件的 keys，可用星号匹配所有字符串

`keys("NewsFeed:device_id:*")`

例如知道 key 的前缀，要清除缓存的时候

```ruby
  def remove_users_top
    user_keys = $REDIS.keys("NewsFeed:device_id:*")
    return if user_keys.empty?

    user_keys.each do |key|
      # 假设 key 长这样
      # NewsFeed:device_id:123456:top
      device_id = key.split(":")[-2]
      key = "NewsFeed:device_id:#{device_id}:top"
      $REDIS.del(key)
    end
  end
```

### 有序数组（有排序的需求时用）

添加元素，必须指定分数

 `zadd(key, score, value)`

复制数组

 `zunionstore(new_key, [origin_key])`

查询指定分数范围的元素个数

 `zcount(key, start_score, end_score)`，例如 `zcount(key, 1, "+inf")`返回分数大于等于 1 的元素个数

返回指定分数范围的元素集，按分数倒序，可指定返回 n 个元素

`zrevrangebyscore(key, end_score, start_score, limit: [start_index, end_index])`，例如 `zrevrangebyscore(key, "+inf", 1, limit: [0, 9])`返回前 10 个分数大于等于 1 的元素

删除（多个）元素

`zrem(key, [value1, value2...])`

### 无序数组（自带随机方法，类似 ruby 的 `Array#sample`）

添加元素

 `sadd(key, value)`

复制数组

 `sunionstore(new_key, [origin_key])`

查询整个数组的元素个数

`scard(key)`

随机返回 n 个元素

`srandmember(key, size)`

删除（多个）元素

`srem(key, [value1, value2...])`

### Reference

[Redis 命令手册（中文）](http://www.redis.net.cn/order/)

[Redis 命令手册（官网）](https://redis.io/commands)
