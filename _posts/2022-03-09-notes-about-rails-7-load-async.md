---
title:  "Notes about Rails 7 `load_async`"
tags: rails activerecord 

---

*Use it before time consuming behaviors. Don't try to `load_async` everything*

### Scenarios

- Slow queries
- 3rd party HTTP requests

### Cons

- Extra db connections usage

### Reference

[The In-depth Guide to ActiveRecord load_async in Rails 7](https://pawelurbanek.com/rails-load-async)
