---
title: Ruby 关于时区的坑
toc: true
toc_label: "目录"
toc_icon: "cog"
---

## [A summary of do’s and don'ts with time zones](https://robots.thoughtbot.com/its-about-time-zones#a-summary-of-do39s-and-don39ts-with-time-zones)

## DON’T USE（固定使用本机的时区）

```ruby
Time.now
Date.today
Date.today.to_time
Time.parse("2015-07-04 17:05:37")
Time.strptime(string, "%Y-%m-%dT%H:%M:%S%z")
```

## DO USE（使用全局或临时定义的时区）

```ruby
Time.current
2.hours.ago
Time.zone.today
Date.current
1.day.from_now
Time.zone.parse("2015-07-04 17:05:37")
Time.strptime(string, "%Y-%m-%dT%H:%M:%S%z").in_time_zone
```

## 是否处于夏令时
```ruby
Time.zone = 'Eastern Time (US & Canada)'    # => 'Eastern Time (US & Canada)'
Time.zone.parse("2012-5-30").dst?           # => true
Time.zone.parse("2012-11-30").dst?          # => false
```

## 参考

[Elle Meredith - It's About Time (Zones)](https://robots.thoughtbot.com/its-about-time-zones)

[时区大全 - 加拿大时间时区及夏令时简介](http://www.timeofdate.com/country/Canada)

[apidock - dst?](https://apidock.com/rails/ActiveSupport/TimeWithZone/dst%3F)
