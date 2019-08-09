---
title:  "ruby 自定义日志以 json 输出"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: ruby log kibana

---

*tldr 日志收集、汇总、分析用上了 elk 全家桶。kibana 需要 json 格式的日志*

### Gemfile

```ruby
#Gemfile
gem 'log_formatter' # 自定义日志文件时 json 化输出
```

### 构建自定义日志

```ruby
# app/controllers/concerns/log_concern.rb
# 用来自定义日志输出
require 'log_formatter'
require 'log_formatter/ruby_json_formatter'

module LogConcern
  extend ActiveSupport::Concern

  def log(log_path = nil)
    if log_path
      # 非 controller 调用，需指定文件名
      @log ||= Logger.new(Rails.root.join("log", log_path))
    else
      # controller 调用，默认以 controller 作为文件名
      # 例如 PayNotifyController 日志文件名为 pay_notify.json.log
      log_prefix = self.class.to_s.delete_suffix("Controller").underscore
      @log ||= Logger.new(Rails.root.join("log", "#{log_prefix}.json.log"))
    end

    @log.formatter = Ruby::JSONFormatter::Base.new
    @log
  end
end
```

### 写入自定义日志

```ruby
# controller 调用
class SomeController < ApplicationController
  include LogConcern

  def some_method
    log.error(error: "可以直接以 hash 作为日志内容")
    log.info("也可以是字符串")
  end
end
```

```ruby
# 非 controller 调用
module SomeModule
  class << self
    include LogConcern

    def logger
      log("some_module.json.log")
    end

    def some_dummy_client(path, req_params)
    	message = {
        req_path:   path,
        req_params: req_params,
        resp_code:  code.to_i,
        resp_body:  res
      }

      logger.info(message)
    end
  end
end
```

### kibana 效果

![kibana](https://user-images.githubusercontent.com/2174219/62761150-73240500-bab8-11e9-84a5-6697bde1bd17.png)
