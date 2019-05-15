---
title:  "rails 创建父记录后创建关联记录（一对一关系）"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: rails active-record

---

*tl; dr [has-one-association-reference-when-are-objects-saved](https://guides.rubyonrails.org/association_basics.html#has-one-association-reference-when-are-objects-saved-questionmark)*

## 原理

感谢[大佬的解决方案](https://stackoverflow.com/questions/3808782/rails-best-practice-how-to-create-dependent-has-one-relations)

贴一段 rails guides 的讲解

> ##### [ When are Objects Saved?](https://guides.rubyonrails.org/association_basics.html#has-one-association-reference-when-are-objects-saved-questionmark)
>
> When you assign an object to a `has_one` association, that object is automatically saved (in order to update its foreign key). In addition, any object being replaced is also automatically saved, because its foreign key will change too.
>
> If either of these saves fails due to validation errors, then the assignment statement returns `false`and the assignment itself is cancelled.
>
> If the parent object (the one declaring the `has_one` association) is unsaved (that is, `new_record?`returns `true`) then the child objects are not saved. They will automatically when the parent object is saved.
>
> If you want to assign an object to a `has_one` association without saving the object, use the `build_association` method.

## 例子

```ruby
class User < ApplicationRecord
  has_one :wallet, dependent: :destroy # 一个【用户】对于一个【钱包】

  before_create :create_default_wallet

  def create_default_wallet
    build_wallet # 相当于 new 了一个【钱包】
    true
  end

  # 【用户】创建成功后，对应的【钱包】才 save
end
```

## 参考文献

[has-one-association-reference-when-are-objects-saved](https://guides.rubyonrails.org/association_basics.html#has-one-association-reference-when-are-objects-saved-questionmark)

[Rails - Best-Practice: How to create dependent has_one relations](https://stackoverflow.com/questions/3808782/rails-best-practice-how-to-create-dependent-has-one-relations)
