---
title:  "ruby 元编程用例"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: ruby metaprogramming
---

*`method_missing` 很神奇，但也不少坑*

### 打开类

与其死记硬背 `strftime` 的格式化参数，或者每次都复制粘贴

```ruby
Time.now.strftime("%Y-%m-%d %H:%M:%S")
# => "2019-09-10 16:37:11"
```

不如打开 `Time` 这个类，加些语法糖进去

```ruby
### lib/time.rb
[:to_js_timestamp, :to_s_datetime, :to_s_date].each do |m|
  # 避免复写已有方法
  #if Time.instance_methods(:false).grep(m).empty?
  if !Time.instance_methods(:false).include?(m)
    class Time
      # 时间转 javascript 时间戳
      def to_js_timestamp
        (self.to_f * 1000.0).to_i
      end

      # 时间格式化成"年-月-日 时-分-秒"
      def to_s_datetime
        strftime("%Y-%m-%d %H:%M:%S")
      end

      # 时间格式化成"年-月-日"字符串
      def to_s_date
        strftime("%Y-%m-%d")
      end
    end
  end
end

#if Time.methods.grep(/from_js_timestamp/).empty?
if !Time.methods.include?(:from_js_timestamp)
  class Time
    # javascript 时间戳转时间
    def self.from_js_timestamp(ms)
      Time.at((ms/1000).to_i)
    end
  end
end
```

```bash
pry
[1] pry(main)> Time.methods.include?(:from_js_timestamp)
=> false
[2] pry(main)> load 'lib/time.rb'
=> true
[3] pry(main)> Time.methods.include?(:from_js_timestamp)
=> true
```

```ruby
[4] pry(main)> Time.from_js_timestamp(Time.now.to_i * 1000)
=> 2019-10-30 10:42:57 +0800
[5] pry(main)> [:to_js_timestamp, :to_s_datetime, :to_s_date].each do |m|
[5] pry(main)*   p Time.now.public_send(m)
[5] pry(main)* end
1572405151510
"2019-10-30 11:12:31"
"2019-10-30"
```

### 祖先链：父类继承和模块包含

![page46image57902096](https://user-images.githubusercontent.com/2174219/65105852-eb131280-da08-11e9-8f5c-3059647569b7.jpg)

##### 父类继承：Rails 的 ApplicationController

```ruby
class ApplicationController < ActionController::Base
  def current_user
    @current_user = User.find_by(id: params[:id])
  end

  def check_login!
    return render status: 403 if !current_user
  end
end
```

```ruby
class SomeController < ApplicationController
  before_action :check_login!, except: [:actions_do_not_require_login]

  def most_actions
  end

  def actions_do_not_require_login
  end
end
```

##### 模块包含：Rails 的 Concern

```ruby
# app/models/concerns/uuid_helper.rb
require 'snowflake-rb'

module UuidHelper
  def self.included(base)
    base.before_create :gen_outid
  end

  private

  def gen_outid
    sf = Snowflake::Rb.snowflake(0, 0)
    self.out_id = sf.next.to_s if out_id.blank?
  end
end
```

```ruby
# User 表已有 out_id 字段
class User < ApplicationRecord
  include UuidHelper
end
```

```ruby
# Order 表也已有 out_id 字段
class Order < ApplicationRecord
  include UuidHelper
end
```

### 动态方法

##### （任意）方法调用：（send）public_send

```ruby
# 根据时间段(day/ week/ month)
# 动态调用 Rails 的 beginning_of_xxx 和 end_of_xxx
# 返回时间段开始、结束时间字符串
# 例如当日 2019092520190925
module RangeHelper
  def gen_range_no(range)
    begined_at = Time.now.public_send("beginning_of_#{range}")
    ended_at = Time.now.public_send("end_of_#{range}")

    begined_at.strftime("%Y%m%d") + ended_at.strftime("%Y%m%d")
  end
end
```

```ruby
class Ranking
  include RangeHelper
  # 根据时间段(day/ week/ month)
  # 返回唯一字符串，用作排行榜缓存的键值
  # 例如日榜 ranking/day/2019092520190925
  def range_lb_name(range)
    date_str = gen_range_no(range)
    "ranking/#{range}/#{date_str}"
  end
end
```

##### 运行时定义：define_method

```ruby
# 根据日期和时间段(day/ week/ month)
# 批量生成排行榜名称，用作排行榜缓存的键值
# 例如日榜 day_lb_name 返回 ranking/day/2019092520190925
class Ranking
  include RangeHelper
	['day', 'week', 'month'].each do |range|
  	define_method "#{range}_lb_name" do
      date_str = gen_range_no(range)
      "ranking/#{range}/#{date_str}"
    end
  end
end
```

```ruby
Ranking.instance_methods.grep(/_lb_name$/)
#=> [:day_lb_name, :range_lb_name, :week_lb_name, :month_lb_name]
```

##### 幽灵方法：method_missing

下面的例子其实不是很好。`method_missing` 比较适用的场景应该是 `active-record` 或者 `builder` 这种**针对大量未知属性，自动生成读写方法**的库，目的是为了**自适应**。而可能性已知或有限的情况下，只是为了**消除重复代码**，用 `define_method` 更合适。鉴于 `method_missing` 可能会带来副作用，记住这条原则：*use Dynamic Methods if you can and Ghost Methods if you have to*.

```ruby
# 消除 model 中相似的 scope
def self.method_missing(name, day)
  m = name.to_s.gsub('by_', '')
  super if !day.respond_to?("beginning_of_#{m}") || !day.respond_to?("end_of_#{m}")
  where(paid_at: day.send("beginning_of_#{m}")..day.send("end_of_#{m}"))
end
```

```ruby
# 用 define_method 更合适
class << self
  ['day', 'week', 'month'].each do |name|
    define_method "by_#{name}" do |day|
      where(paid_at: day.send("beginning_of_#{name}")..day.send("end_of_#{name}"))
    end
  end
end
```

与改造前的 scope 功能一致

```bash
[1] pry(main)> PromoOrder.by_week(Time.now)
  PromoOrder Load (22.0ms)  SELECT "promo_orders".* FROM "promo_orders" WHERE ("promo_orders"."paid_at" BETWEEN $1 AND $2)  [["paid_at", "2019-11-03 16:00:00"], ["paid_at", "2019-11-10 15:59:59.999999"]]
=> []
```

确认不会带来副作用。没有定义的方法依然返回 `NoMethodError`

```bash
[2] pry(main)> PromoOrder.by_weak(Time.now)
NoMethodError: undefined method `by_weak' for #<Class:0x00007ff827ea83d8>
from /Users/justin/.rvm/gems/ruby-2.6.3/gems/activerecord-5.1.3/lib/active_record/dynamic_matchers.rb:22:in `method_missing'
```

### Reference
Programming Ruby - Metaprogramming Ruby 2. Program Like the Ruby Pros (2014)

