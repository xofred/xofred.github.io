---
title:  "rspec 奇淫技巧"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: rspec unit-test

---

*情况很烂，但可以尽力朝好的方向努力*

### 用例跳过数据库删创

```ruby
# spec/rails_helper.rb
RSpec.configure do |config|
  # If you're not using ActiveRecord, or you'd prefer not to run each of your
  # examples within a transaction, remove the following line or assign false
  # instead of true.
  # 公司项目需要事先有表和数据
  # 所以本地和测试环境都不能随便 drop
  config.use_transactional_fixtures = false
end
```

### 命名路由如何测试
```ruby
# routes
RouteTarget.routes.draw do
  scope '(:locale)', locale: /fr|en/ do
    match "/looking_forward" => "wish_lists#looking_forward", via: [:get, :post]
  end
end
```

```shell
rails routes|grep looking_forward
```

> looking_forward GET|POST
> (/:locale)/looking_forward(.:format)
> wish_lists#looking_forward {:locale=>/fr|en/}

```ruby
# spec
routes.draw { get "en/looking_forward" => "wish_lists#looking_forward" }
```

### controller 的 response 是 js
```ruby
# controller
respond_to do |format|
  if @messages.blank?
    flash[:notice] = I18n.t('search.receive_tip')
    format.js { render js: "window.location='#{new_arrival_path}'" }
  else
    format.js { render 'search_plus/looking_forward' }
  end
end
```

```ruby
# spec
expect { get(:looking_forward, xhr: true, format: :js) }.to change { HelpMeFind.count }.by(1)
```

### before all 和 before each 的差异

`all` 只跑一次，`each` 每个用例都会跑。例如下面的例子应放在 `each` 执行

```ruby
before(:each) do
  allow_any_instance_of(WishListsController).to receive(:check_session) { true }
  HelpMeFind.delete_all
end
```

### change 语法(期望新增/不要新增数据库记录)

```ruby
# 期望新增数据库记录
expect { get(:looking_forward) }.to change { HelpMeFind.count }.by(1)

# 期望不要新增数据库记录
expect { get(:looking_forward) }.to change { HelpMeFind.count }.by(0)
```

### 完整例子
```ruby
require 'rails_helper'

RSpec.describe WishListsController, type: :controller do
  before(:each) do
    allow_any_instance_of(WishListsController).to receive(:check_session) { true }
    HelpMeFind.delete_all
  end

  describe "#looking_forward" do
    let(:brand) { Catalog::INK_TONER_BRAND_IDS.to_a.sample }
    let(:brand_name) { brand[0] }
    let(:brand_id) { brand[1].join('-') } # 例如 "4989-8844"
    let(:email) { 'mr@shithole.cn' }
    let(:model_name) { 'i m model_name' }
    let(:search_value) { 'i m search_value' }

    it "首次产品请求，发通知邮件，创建 HelpMeFind 记录" do
      routes.draw { get "en/looking_forward" => "wish_lists#looking_forward" }
      begin
        expect { get(:looking_forward, xhr: true, format: :js, params: { brand_id: brand_id, model_name: model_name, email: email, search_value: search_value }) }.to change { HelpMeFind.count }.by(1)
      rescue
      end
    end

    it "再次产品请求，发通知邮件，不创建 HelpMeFind 记录" do
      HelpMeFind.create(brand_name: brand_name, model: model_name, email: email)

      routes.draw { get "en/looking_forward" => "wish_lists#looking_forward" }
      begin
        expect { get(:looking_forward, xhr: true, format: :js, params: { brand_id: brand_id, model_name: model_name, email: email, search_value: search_value }) }.to change { HelpMeFind.count }.by(0)
      rescue
      end
    end
  end
end
```

### Reference

[Draw custom routes for defined controllers](https://relishapp.com/rspec/rspec-rails/v/3-9/docs/controller-specs/anonymous-controller#refer-to-application-routes-in-the-controller-under-test)

[`change` matcher](https://relishapp.com/rspec/rspec-expectations/v/3-9/docs/built-in-matchers/change-matcher)

[`before` and `after` hooks](https://relishapp.com/rspec/rspec-core/v/3-9/docs/hooks/before-and-after-hooks)

[`format: :js` causes fails on Rails 4.1.0.rc1 #950](https://github.com/rspec/rspec-rails/issues/950)
