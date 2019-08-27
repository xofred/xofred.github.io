---
title:  "ruby 异常处理的用例两则"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: ruby exception error

---

*tldr 眼见为虚，测试为实*

### 自定义异常类及 `rspec` 测试

```ruby
# 业务 model
class Reseller < ApplicationRecord
  class InsufficientFundError < StandardError
    def message
      '余额不足'
    end
  end

  def recharge(user, coins)
    # 判断余额是否足够充值
    raise InsufficientFundError if coins > ele_coin
    # 充值业务逻辑
  end
end

# model 测试
RSpec.describe Reseller, type: :model do
  let(:reseller) { create(:reseller) }
  let(:user) { create(:user) }

  describe "#recharge" do
    it "余额不足，充值抛异常" do
      # 注意要测 raise_error 的话，expect 跟的是花括号
      expect{reseller.recharge(user, 100)}
        .to raise_error(Reseller::InsufficientFundError)
    end
  end
end
```

### 关于 ensure 和函数返回值

```ruby
def create_order
	begin
    p '可能产生异常的行为'
  ensure
    p '返回订单'
  end
end

create_order
# "可能产生异常的行为"
# "返回订单"
# => "可能产生异常的行为"
```

`ensure`里面的代码一定会执行，但不是最终函数返回值。其实业务想要“返回订单”，但 `create_order` 最终返回的是 `begin` 里面最后的一句，“可能产生异常的行为”

```ruby
def create_order
	begin
    p '可能产生异常的行为'
  end

  p '返回订单'
end

create_order
# "可能产生异常的行为"
# "返回订单"
# => "返回订单"
```

所以，要确保 `create_order` 最终"返回订单”，不能加在 `ensure` 里面执行，而是整个 `begin...end` block 之外

### 总结

很多“眼见功夫”，未必是自己所想的结果。单元测试很重要，尤其是 API、支付、第三方服务相关的。必须从源头确保**自己的**服务端代码是正确的，调试时能减少不必要的排查

### 参考

[Custom Exceptions in Ruby](https://blog.appsignal.com/2018/07/03/custom-exceptions-in-ruby.html)

[RSpec Expectations 3.8 Built in matchers](https://relishapp.com/rspec/rspec-expectations/v/3-8/docs/built-in-matchers)
