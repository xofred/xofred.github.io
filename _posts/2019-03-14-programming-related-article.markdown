---
title:  "技术好文分享 190314"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: misc
---

**RT**

---

## [Deconstructing the Monolith: Designing Software that Maximizes Developer Productivity](https://engineering.shopify.com/blogs/engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity?utm_source=wanqu.co&utm_campaign=Wanqu+Daily&utm_medium=website)

by Kirsten Westeinde

应用庞大后，应按业务（例如订单、物流）拆分，而不是按代码功能目录（例如 Model、Controller、View）拆分

> 总之，在系统早期，没有任何架构通常是最好的架构。这并不是说不实施良好的软件实践，而是花费数周和数月的时间来尝试构建一个您还不知道的复杂系统。Martin Fowler的Design Stamina Hypothesis  通过解释在大多数应用程序的早期阶段，只需极少的设计即可快速移动，从而很好地说明了这一想法。将设计质量与上市时间进行权衡是切合实际的。一旦您可以添加特性和功能的速度开始减慢，那就是投资良好设计的时候了。

> 重构和重新构建的最佳时间是尽可能晚，因为您在构建时不断了解有关系统和业务领域的知识。在拥有领域专业知识之前设计一个复杂的微服务系统是一个冒险的举措，太多的软件项目都属于这个问题。根据Martin Fowler的说法，“几乎所有我听说过从头开始构建为微服务系统的系统，它已经结束了严重的麻烦......你不应该开始一个带微服务的新项目，即使你'确保你的应用程序足够大，以使其值得“。


## [MySQL 事务隔离级别解析和实战](https://juejin.im/post/5c88d74af265da2d8d6a1c58?utm_source=gold_browser_extension#heading-13)

by [学霸的一天](https://juejin.im/user/5a9d5797f265da23945efa29)

之前项目碰过并发创建的问题，后来将隔离级别设为 `serializable` 解决了（当然，性能也就最差了。管他呢，大佬说的，性能问题等项目未死再优化不迟）

```ruby
Post.transaction(isolation: :serializable) do
  # ...
end
```

> 每种数据库隔离级别都解决了一个问题。数据库隔离级别依次增强，性能也依次变差。大部分环境中使用 READ-COMMITTED 是可行的。
