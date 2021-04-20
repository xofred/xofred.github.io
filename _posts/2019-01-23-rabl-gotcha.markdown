---
title:  "rabl 的一些坑"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: ruby gem rabl gotcha
---

*tldr：jbuilder 劣化版？*

### partial 只能传 `object`

```rabl
node(:data) do
  partial "v3.1/news_feeds/_news_feed", object: @news_feeds
end
```

### 嵌套节点要写成 hash

```rabl
node(:display_style) do |m|
  {
    :type => m.display_type,
    :number => NewsFeed.display_nums[m.display_num]
  }
end
```

### 条件判断要用 `lambda`

```rabl
node(:contents, :if => lambda { |m| m.standard_products.any? } ) do |m|
  partial("v3.1/news_feeds/standard_products", object: m.standard_products)
end
node(:contents, :if => lambda { |m| m.pictures.any? } ) do |m|
  partial("v3.1/news_feeds/pictures", object: m.pictures)
end
```
