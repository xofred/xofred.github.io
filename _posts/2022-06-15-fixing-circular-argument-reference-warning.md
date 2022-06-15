---
title:  "Fixing circular argument reference warning"
tags: ruby legacy

---

*It's about 'keyword arguments'*

```shell
[8] pry(main)> def m(a:, b:, c: c, d: nil)
  p 'Method :m is called.'
  pp({a: a, b: b, c: c, d: d})
(pry):13: warning: circular argument reference - c
=> :m
```

We can fix it by assigning `c` a default value other than `c`.

```shell
[9] pry(main)> def m(a:, b:, c: 1, d: nil)
  p 'Method :m is called.'
  pp({a: a, b: b, c: c, d: d})
=> :m
```

This definition of arguments specifying hash key is also called 'keyword arguments'.

The order doesn't matter (because it's hash).

However, the :a, :b colons are not followed by anything, which means they are required. If they are not passed when the method is called, an `ArgumentError: missing keywords` will raise.

```shell
[2] pry(main)> m
ArgumentError: missing keywords: a, b
from (pry):1:in `m'
```

As for :c and :d, they are optional, and the default value is after the colon.

```shell
[7] pry(main)> m(a: 'a', b: 'b', d: 'd')
"Method :m is called."
{:a=>"a", :b=>"b", :c=>1, :d=>"d"}
=> {:a=>"a", :b=>"b", :c=>1, :d=>"d"}
```
