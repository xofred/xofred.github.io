---
layout: single
title:  "Rails 引入静态文件"
toc: true
toc_label: "目录"
toc_icon: "cog"
---


### 图片存放位置

`app/assets/images/`，支持子文件夹，例如抽奖活动页图片存放在 `app/assets/images/activity/wheel/`



### js 和 css 存放位置

在 `app/assets/javascripts/` 和 `app/assets/stylesheets/` 下新增 js 和 css 两个文件，里面包含页面所需其他 js 和 css 文件名，然后在需要的 view 里引入这两个文件

在 config/initializers/assets.rb 里添加文件

```ruby
# 例如 activity/wheel.js 和 activity/wheel.css 是抽奖页的
Rails.application.config.assets.precompile += %w( clipboard.min.js activity/wheel.js activity/wheel.css )
```

例如 `activity/wheel.js` 引入了3个文件

```js
//= require activity/wheel/vue.min
//= require activity/wheel/jquery-1.10.2
//= require activity/wheel/awardRotate
```



### view 引入静态文件

```erb
<%= javascript_include_tag 'activity/wheel' %> 
<%= stylesheet_link_tag 'activity/wheel' %>
```

js 获得图片地址

```erb
$("#tab2 img").attr("src", "<%= asset_path 'activity/wheel/NG_07.png' %>");
```

生成 img 标签

```erb
<%= image_tag "activity/wheel/1.png", id: "diy1-img", style: "display:none;" %>

```

[使用绝对路径](https://stackoverflow.com/questions/2582950/how-to-display-image-pointed-by-url-in-rails?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

```erb
<img src="http://power.itp.ac.cn/~jmyang/funny/fun4.jpg">
```
