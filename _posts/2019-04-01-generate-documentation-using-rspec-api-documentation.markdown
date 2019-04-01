---
title:  "用 rspec 写 API 测试的同时生成在线文档"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: rspec testing ruby gem documentation
---

*tl; dr [**rspec_api_documentation**](https://github.com/zipmark/rspec_api_documentation)*

### 生成文档

Gemfile 加入测试相关

```ruby
group :development, :testing do
  gem "factory_bot_rails", "~> 4.0" # 生成测试 model
  gem 'rspec-rails', '~> 3.8'
  gem 'rspec_api_documentation' # 生成本地 API 文档
  gem 'pry' # 断点调试
end
```

业务 spec 文件例子，[rspec 的功能](https://relishapp.com/rspec) 就不多讲了

```ruby
# 先新建 spec/acceptance 目录，以后 spec 文件就放这
# 业务 spec 文件 spec/acceptance/recently_vieweds_controller_spec.rb
require 'rails_helper'

RSpec.describe V3::RecentlyViewedsController, type: :controller do
  render_views

  let(:member) { Member.where("mobile_token IS NOT NULL AND mobile_token != ''").first }
  let(:generate_data) {
    spu = create(:standard_product, name_fr: 'i m spu name in french')
    create(:viewed_product, customer_id: member.customer.id, standard_product_id: spu.id)
  }
  let(:generate_more_data) {
    11.times do |i|
      viewed_at = Time.current - i.days
      spus = []
      25.times do |i|
        spu = create(:standard_product, name_fr: "i m spu name in french#{i}")
        spus.push(spu)
      end
      spus.each do |spu|
        vp = create(:viewed_product, customer_id: member.customer.id, standard_product_id: spu.id)
        vp.update_columns(updated_at: viewed_at)
      end
    end
  }
  let(:clean_up) {
    StandardProduct.delete_all
    ViewedProduct.delete_all
  }

  before(:each) {
    clean_up
  }

  resource "Recently Viewed Products" do
    # API 描述，可以是字符串，或者像下面用 html 标签格式化输出
    explanation "
    <h3>通用说明</h3>
    <ul>
      <li>API 版本：header 必传 SALEAPP_VERSION_NAME 例如 3.3</li>
      <li>法语支持：header 传 LANG=fr，默认英语</li>
      <li>返回格式：json utf-8</li>
    </ul>
    "

    # header 描述
    header "Host", "test.shopperplus.com"
    header "Content-Type", "application/json"
    header "SALEAPP_VERSION_NAME", "3.3"

    # 参数描述
    parameter :token, "手机用户 token", type: :string, require: true

    # 获取用户最近查看商品列表
    get "/api/v3.3/recently_vieweds" do
      parameter :per, "指定返回商品数量", type: :integer, default: 6
      parameter :per, "指定返回第几页", type: :integer, default: 1

      context "获取首页用户最近查看商品" do
        # 因为 apitome 对中文支持不太好（就像 gollum，这类文件管理系统的通病了😓）
        # example 人肉加上前缀弄成唯一的文件名（因为文件名太长会被截掉，所以加最开头）
        example "1_1 GET /api/v3.3/recently_vieweds 获取首页用户最近查看商品(没传 token 或找不到用户)" do
          do_request # 发起 API 请求

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 401
          expect(JSON.parse(response_body)["meta"]["message"]).to eq "member auth failed!"
        end
      end

      context "获取首页用户最近查看商品" do
        let(:token) { member.mobile_token }# 传参数

        example "1_2 GET /api/v3.3/recently_vieweds 获取首页用户最近查看商品(没有数据)" do
          do_request

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 200
          expect(JSON.parse(response_body)["meta"]["pagination"]["total"]).to eq 0
          expect(JSON.parse(response_body)["meta"]["pagination"]["total_pages"]).to eq 0
          expect(JSON.parse(response_body)["meta"]["pagination"]["current_page"]).to eq 1
          expect(JSON.parse(response_body)["data"].size).to eq 0
        end
      end

      context "获取首页用户最近查看商品" do
        let(:token) { member.mobile_token }

        example "1_3 GET /api/v3.3/recently_vieweds 获取首页用户最近查看商品(有数据)" do
          generate_data
          do_request

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 200
          expect(JSON.parse(response_body)["meta"]["pagination"]["total"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["total_pages"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["current_page"]).to eq 1
          data = JSON.parse(response_body)["data"]
          expect(data.size > 0).to be_truthy
          expect(data[0]["name"].include?("french")).to be_falsy
          expect(data[0]["viewed_product"]).to be_present
        end
      end

      context "获取首页用户最近查看商品" do
        let(:token) { member.mobile_token }

        example "1_4 GET /api/v3.3/recently_vieweds 获取首页用户最近查看商品(指定 page)" do
          generate_more_data
          do_request(page: 2) # 参数也可以这样传

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 200
          expect(JSON.parse(response_body)["meta"]["pagination"]["total"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["total_pages"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["current_page"]).to eq 2
          data = JSON.parse(response_body)["data"]
          expect(data.size > 0).to be_truthy
          expect(data[0]["name"].include?("french")).to be_falsy
          expect(data[0]["viewed_product"]).to be_present
        end
      end

      context "获取首页用户最近查看商品" do
        let(:token) { member.mobile_token }
        let(:per) { 12 } # 多个参数

        example "1_4 GET /api/v3.3/recently_vieweds 获取首页用户最近查看商品(指定 per)" do
          generate_more_data
          do_request

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 200
          expect(JSON.parse(response_body)["meta"]["pagination"]["total"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["total_pages"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["current_page"]).to eq 1
          data = JSON.parse(response_body)["data"]
          expect(data.size).to eq 12
          expect(data[0]["name"].include?("french")).to be_falsy
          expect(data[0]["viewed_product"]).to be_present
        end
      end

      context "获取首页用户最近查看商品" do
        let(:token) { member.mobile_token }

        example "1_5 GET /api/v3.3/recently_vieweds 获取首页用户最近查看商品(有数据，法语支持)" do
          header "lang", 'fr' # 传 header

          generate_data
          do_request

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 200
          expect(JSON.parse(response_body)["meta"]["pagination"]["total"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["total_pages"] > 0).to be_truthy
          expect(JSON.parse(response_body)["meta"]["pagination"]["current_page"]).to eq 1
          data = JSON.parse(response_body)["data"]
          expect(data.size > 0).to be_truthy
          expect(data[0]["name"].include?("french")).to be_truthy
          expect(data[0]["viewed_product"]).to be_present
        end
      end
    end

    # 移除用户查看商品记录
    post "/api/v3.3/recently_vieweds/:id/remove" do
      parameter :id, "查看商品记录 ID", type: :integer, require: true

      context "获取首页用户最近查看商品" do
        example "2_1 POST /api/v3.3/recently_vieweds/:id/remove 移除用户查看商品记录(没传 id 找不到记录)" do
          do_request

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 404
          expect(JSON.parse(response_body)["meta"]["message"]).to include("Can't find viewed product")
        end
      end

      context "获取首页用户最近查看商品" do
        let(:id) { ViewedProduct.first.id }

        example "2_2 POST /api/v3.3/recently_vieweds/:id/remove 移除用户查看商品记录(成功)" do
          generate_data
          do_request

          expect(response_headers["Content-Type"]).to eq "application/json; charset=utf-8"
          expect(status).to eq 200
          expect(JSON.parse(response_body)["meta"]["message"]).to eq("success")
          expect(JSON.parse(response_body)["data"]).to be_present
          pp JSON.parse(response_body)["data"]
        end
      end
    end

  end
end

```

跑测试，我习惯顺便加上格式 `--format d`

`bundle exec rspec --format d ./spec/acceptance/` 如果测试跑通了会生成文档文件

### 在线文档

生成的文档格式默认是 html 文件，当然 9102 年了不建议打包将一堆 html 发给同事看。所以强烈建议弄成在线文档，而且 apitome 会加上样式好看很多

配置 RspecApiDocumentation

```ruby
# spec/spec_helper.rb
require 'rspec_api_documentation/dsl'
RspecApiDocumentation.configure do |config|
  # 文档改成 json 格式，供 apitome 生成在线文档
  config.format = :json
end
```

Gemfile 加入 apitome 相关

```ruby
group :development, :staging do
  # 在线 API 文档
  gem "apitome"
  gem "jquery-rails"
end
```

配置 apitome，我是默认配置了，文档首页路由默认是 `/api/doc`

```ruby
# 生产环境的 gem 组没有 apitome，要去掉这个文件，不然会报错
# config/initializers/apitome.rb
unless Rails.env.production?
  require 'apitome'

  Apitome.setup do |config|
    # This determines where the Apitome routes will be mounted. Changing this to '/api/documentation' for instance would
    # allow you to browse to http://localhost:3000/api/documentation to see your api documentation. Set to nil and mount
    # it yourself if you need to.
    config.mount_at = "/api/docs" # 在线文档路由

    # This defaults to Rails.root if left nil. If you're providing documentation for an engine using a dummy application
    # it can be useful to set this to your engines root.. E.g. Application::Engine.root
    config.root = nil

    # This is where rspec_api_documentation outputs the JSON files. This is configurable within RAD, and so is
    # configurable here.
    config.doc_path = "doc/api" # rspec_api_documentation 生成的文档文件位置

    # Set a parent controller that Apitome::DocsController inherits from. Useful if you want to use a custom
    # `before_action`.
    config.parent_controller = "ActionController::Base"

    # The title of the documentation -- If your project has a name, you'll want to put it here.
    config.title = "Apitome Documentation"

    # The main layout view for all documentation pages. By default this is pretty basic, but you may want to use your own
    # application layout.
    config.layout = "apitome/application"

    # We're using highlight.js (https://github.com/isagalaev/highlight.js) for code highlighting, and it comes with some
    # great themes. You can check http://softwaremaniacs.org/media/soft/highlight/test.html for themes, and enter the
    # theme as lowercase/underscore.
    config.code_theme = "default"

    # This allows you to override the css manually. You typically want to require `apitome/application` within the
    # override, but if you want to override it entirely you can do so.
    config.css_override = nil

    # This allows you to override the javascript manually. You typically want to require `apitome/application` within the
    # override, but if you want to override it entirely you can do so.
    config.js_override = nil

    # You can provide a 'README' style markdown file for the documentation, which is a useful place to include general
    # information. This path is relative to your doc_path configuration.
    config.readme = "../api.md"

    # Apitome can render the documentation into a single page that uses scrollspy, or it can render the documentation on
    # individual pages on demand. This allows you to specify which one you want, as a single page may impact performance.
    config.single_page = true

    # You can specify how urls are formatted using a Proc or other callable object.
    # Your proc will be called with a resource name or link, giving you the opportunity to modify it as necessary for in the documentation url.
    config.url_formatter = -> (str) { str.gsub(/\.json$/, '').underscore.gsub(/[^0-9a-z\:]+/i, '-') }

    # You can setup the docs to be loaded from a remote URL if they are
    # not available in the application environment. This defaults to
    # false.
    config.remote_docs = false

    # If the remote_docs is set to true, this URL is used as the base for
    # the doc location. This should be the root of the doc location, where
    # the readme is located. It uses the doc_path setting to build the
    # URLs for the API documentation. This defaults to nil.
    config.remote_url = nil

    # If you would like to precompile your own assets, you can disable auto-compilation.
    # This defaults to true
    config.precompile_assets = true
  end
end

```

如果项目部署时没有执行预编译的话（例如 API 项目一般没有 view 和后台，不会做这件事），要在本地生成编译好的静态文件 `bundle exec rake assets:precompile` 并将其加入版本控制

### Reference

[**rspec_api_documentation** 生成测试文档](https://github.com/zipmark/rspec_api_documentation)

[**apitome** 配合 **rspec_api_documentation** 使用的在线文档](https://github.com/jejacks0n/apitome)
