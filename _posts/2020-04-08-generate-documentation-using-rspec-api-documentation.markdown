---
title:  "用 rspec 写 API 测试的同时生成在线文档"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: rspec testing ruby gem documentation
---

*[**rspec_api_documentation**](https://github.com/zipmark/rspec_api_documentation)*

### 20200408 更新说明

~~跟卡婊学重制~~，本文推出~~二十周年~~一周年重制版

- 适当精简业务 spec 文件例子，着重介绍 `rspec_api_documentation` 的 DSL
- 大幅精简 `apitome` 的配置文件，只讲比较值得关注的点
- 补充一些之前省略了但应有的说明
- 新加入自己踩到的一些坑
- ~~画面大幅提升~~

先来个感性认识

![img](https://user-images.githubusercontent.com/2174219/78761725-8f64a280-79b5-11ea-9a6d-d816b4efc578.png)

以下着重介绍 `rspec_api_documentation` 的用法，至于 rspec 建议参考其[官方文档](https://relishapp.com/rspec)

### 生成文档
Gemfile 加入 `rspec_api_documentation`
```ruby
group :development, :testing do
  gem 'rspec-rails', '~> 3.8'
  gem 'rspec_api_documentation' # 生成本地 API 文档
end
```

新建 `spec/acceptance` 目录，以后需要生成 API 文档的 spec 文件就放这
```shell
mkdir spec/acceptance
```

以下用一个业务 API spec 文件做例子
```ruby
# spec/acceptance/products_controller_spec.rb
require 'rails_helper'
# 只在需要生成文档的 spec 文件引入
# 不建议全局加在 `rails_helper.rb` 或者 `spec_helper.rb`
# 因为文档主要是针对 controller 这种 API 测试
# 其余 spec 文件，例如 model 单元测试，沿用 rspec 的写法
require 'rspec_api_documentation/dsl'

RSpec.describe ProductsController do
  let(:user) { User.where.not(mobile_token: nil).first }
  let(:sku) { Product.first }
  # resource 必须关键词。内容可以随便起任何名字
  # 但由于 rspec_api_documentation 据此生成文档超链接
  # 建议最好用 ASCII 字符集，例如英文、数字、下划线
  resource "Products" do
    # explanation 非必须关键词，描述性内容，用于反映到文档
    # 内容可以是字符串，也可以像这里直接写 html
    explanation "
    关于更新接口
    <ul>
      <li>无论实际情况如何，存储体积三个字段时，</li>
      <li>会自动以最长边作为 length，第二长边作为 width，第三长边作为 height。</li>
      <li>客户端无需处理</li>
      <li>使用 HTTP PATCH 或 PUT 均可</li>
    </ul>
    "
    # header 非必须关键词，描述性内容，用于反映到文档
    # 不影响下面 do_request 实际请求的 headers
    header "Host", "api_endpoint_host"
    header "Content-Type", "application/json"
    # parameter 非必须关键词，描述性内容，用于反映到文档
    # 不影响下面 do_request 实际请求的接口参数
    parameter :token, "手机用户 token", type: :string, require: true
    parameter :id, "产品条形码", type: :string, require: true
    # GET API 例子
    # 必须关键词，影响 do_request 实际发起请求
    # 根据项目路由例如 config/routes 填即可
    get "/v1.0/products/:id/grids" do
      context "返回产品所在货架" do
        # let 非必须关键词，用于 do_request 传参
        let(:token) { user.mobile_token }
        # example 必须关键词，相当于 rspec 的 it 关键词
        # 关系到文档地址，建议英文。原因同 resource
        example "Get product grids, return data" do
          allow_any_instance_of(Product).to receive(:shipping_weight) { 666 }
          # do_request 必须关键词，实际发起请求
          # 可以像下面的写法来传参
          # 或者在 context 和 example 之间用 let 关键词
          do_request(id: BarcodeUtil.barcode(sku))
          # 正如作者所说，需要修改以下 rspec 常用方法的调用
          # You must use response_body, status, response_content_type, etc.
          #   to access data from the last response.
          # You will not be able to use response.body or response.status
          #   as the response object will not be created.
          expect(status).to eq(200)
          j = JSON.parse(response_body)
          expect(j["id"]).to be_present
          # ...
        end
      end
    end
    # POST API 例子
    patch "/v1.0/products/:id" do
      parameter :weight, "货物重量，单位克", type: :integer, require: false
      parameter :length, "货物长度，单位毫米", type: :integer, require: false
      parameter :height, "货物高度，单位毫米", type: :integer, require: false
      parameter :width, "货物宽度，单位毫米", type: :integer, require: false

      context "更新运输重量体积" do
        let(:raw_post) { params.to_json } # POST 参数必须这样来传
        let(:token) { user.mobile_token }
        let(:id) { BarcodeUtil.barcode(sku) }

        example "Update product ok, return success" do
          sku.update!(shipping_weight: 1, shipping_length: 1, shipping_width: 1, shipping_height: 1)
          expect{ do_request(product: { weight: 666, length: 100, width: 200, height: 300 }) }
            .to change { sku.audits.count }.by(1)
          # ...
        end
      end
    end
  end
end

```

如果需要跑测试并生成文档，运行这个命令
```shell
bundle exec rake docs:generate
```
如果只是要跑测试（例如只是内部业务逻辑改了，但对外接口文档不需要改），运行老命令
```shell
bundle exec rspec
```

### 在线文档

生成的文档格式默认是 html 文件，当然 ~~9102~~0202 年了不建议打包将一堆 html 发给同事看。所以强烈建议弄成在线文档，而且 apitome 会加上样式好看很多

Gemfile 加入 `apitome` 相关
```ruby
group :development, :staging do
  # 在线 API 文档
  gem "apitome"
  gem "jquery-rails"
  gem "uglifier"
end
```

执行初始化，生成 `apitome` 的配置文件 `config/initializers/apitome.rb` 以及文档首页 `doc/api.md`
```shell
rails generate apitome:install
```

修改 `rspec_api_documentation` 生成的文档格式

```ruby
# spec/spec_helper.rb
require 'rspec_api_documentation/dsl'
RspecApiDocumentation.configure do |config|
  # 文档改成 json 格式，供 apitome 生成在线文档
  config.format = :json
end
```

进一步定制 `apitome`，我是默认配置了，在线文档入口地址默认是 `项目所在域名/api/doc`

```ruby
# config/initializers/apitome.rb
# 生产环境没有加入 apitome (也不应该有，除非想 API 文档让全世界看到)
# 因此要跳过下面的代码，不然会报错
if !Rails.env.production?
  require 'apitome'

  Apitome.setup do |config|
    # 在线文档入口地址。如果多个项目共用一个域名，建议修改
    config.mount_at = "/api/docs"
    # 例如项目一 config.mount_at = "/apitome/api"
    # 然后项目二 config.mount_at = "/apitome/scanner_api"
  end
end
```

静态文件加载需要 `uglifier` 和 `jquery-rails`，并且如果项目部署时没有执行预编译的话（例如 API 项目一般没有 view 和后台，不会做这件事），要在本地生成编译好的静态文件 `bundle exec rake assets:precompile` 并将其加入版本控制

### Reference

[rspec_api_documentation](https://github.com/zipmark/rspec_api_documentation)

[apitome](https://github.com/jejacks0n/apitome)
