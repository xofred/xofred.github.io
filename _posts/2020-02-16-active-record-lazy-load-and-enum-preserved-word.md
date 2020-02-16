---
title:  "active-record 的懒加载行为和 enum 保留字"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: rails active-record

---

*当局者迷，旁观者清*

### active-record 的懒加载行为

```
[1] pry(main)> def a
[1] pry(main)*   sp = StandardProduct.limit(10)
[1] pry(main)*   sp = sp.limit(5)
[1] pry(main)* end
=> :a
[2] pry(main)> a
  StandardProduct Load (1.1ms)  SELECT  `standard_products`.* FROM `standard_products` WHERE `standard_products`.`active` = 1 LIMIT 5
```

可以看出，第一条查前 10 条记录的并没立即执行 SQL 查询，而是等到第二条查前 5 条记录才真正执行。有什么具体用途呢？例如很多时候我们做后台，需要根据运营人员需要添加一些过滤条件。这时我们写起来的时候，可以直接写查全部的语句，不必担心性能问题

```ruby
@hmfs = HelpMeFind.all # 就是这句，并不立即执行 SQL 查询。这个实例变量接下来很有用
@hmfs = @hmfs.where(id: params[:id]) if params[:id].present?
@hmfs = @hmfs.send(params[:state]) if (params[:state].present? && params[:state] != 'all' )
@hmfs = @hmfs.where('model LIKE ?', "%#{params[:model]}%") if params[:model].present?
@hmfs = @hmfs.where('email LIKE ?', "%#{params[:email]}%") if params[:email].present?
@hmfs = @hmfs.where("created_at >= ?", Time.zone.parse(params[:requestedStartAt])) if params[:requestedStartAt].present?
@hmfs = @hmfs.where("created_at < ?", Time.zone.parse(params[:requestedEndAt]) + 1.day) if params[:requestedEndAt].present?
```

### enum 保留字

enum 里面不能有叫 'new' 的，不然会与 active-record 冲突。因为 `new` 方法是每个继承了 active-record 的 model 都有的。然后如果我们 enum 有个叫 `new` 的 key，本意是用来过滤状态 `new` 的，并不是新建一条 model 记录。但由于方法重名了，类没法区分这两个方法到底哪个是哪个

> You tried to define an enum named "state" on the model "HelpMeFind", but this will generate a class method "new", which is already defined by Active Record.

### 彩蛋

这次写的东西可能很浅，然而有些道理本来就很浅，只是我们当局者迷
