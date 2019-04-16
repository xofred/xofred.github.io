---
title:  "HSTS 101 及 Rails 相关配置"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: security rails
---

*tl; dr [HTTP严格传输安全](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8)*

## HSTS 101

复制自[维基百科](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8)

> ### 内容
>
> HSTS的作用是强制客户端（如浏览器）使用HTTPS与服务器创建连接。服务器开启HSTS的方法是，当客户端通过HTTPS发出请求时，在服务器返回的[超文本传输协议](https://www.wikiwand.com/zh-cn/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)（HTTP）响应头中包含`Strict-Transport-Security`字段。非加密传输时设置的HSTS字段无效。[[3\]](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8#citenoterfc3)
>
> 比如，https://example.com/ 的响应头含有`Strict-Transport-Security: max-age=31536000; includeSubDomains`。这意味着两点：
>
> 1. 在接下来的31536000秒（即一年）中，浏览器向example.com或其子[域名](https://www.wikiwand.com/zh-cn/%E5%9F%9F%E5%90%8D)发送HTTP请求时，必须采用HTTPS来发起连接。比如，用户点击[超链接](https://www.wikiwand.com/zh-cn/%E8%B6%85%E9%93%BE%E6%8E%A5)或在地址栏输入 http://www.example.com/ ，浏览器应当自动将 http 转写成 https，然后直接向 https://www.example.com/ 发送请求。
> 2. 在接下来的一年中，如果 example.com 服务器发送的TLS[证书](https://www.wikiwand.com/zh-cn/%E6%95%B0%E5%AD%97%E8%AF%81%E4%B9%A6)无效，用户不能忽略浏览器警告继续访问网站。

> ### 作用
>
> HSTS可以用来抵御SSL剥离攻击。SSL剥离攻击是[中间人攻击](https://www.wikiwand.com/zh-cn/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB)的一种，由Moxie Marlinspike（英语：[Moxie Marlinspike](https://www.wikiwand.com/en/Moxie_Marlinspike)）于2009年发明。他在当年的黑帽大会上发表的题为“New Tricks For Defeating SSL In Practice”的演讲中将这种攻击方式公开。SSL剥离的实施方法是阻止浏览器与服务器创建HTTPS连接。它的前提是用户很少直接在地址栏输入`https://`，用户总是通过点击链接或[3xx重定向](https://www.wikiwand.com/zh-cn/3xx%E9%87%8D%E5%AE%9A%E5%90%91)，从HTTP页面进入HTTPS页面。所以攻击者可以在用户访问HTTP页面时替换所有`https://`开头的链接为`http://`，达到阻止HTTPS的目的。[[9\]](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8#citenote9)
>
> HSTS可以很大程度上解决SSL剥离攻击，因为只要浏览器曾经与服务器创建过一次安全连接，之后浏览器会强制使用HTTPS，即使链接被换成了HTTP[[3\]](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8#citenoterfc3)[[10\]](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8#citenote10)。
>
> 另外，如果中间人使用自己的自签名证书来进行攻击，浏览器会给出警告，但是许多用户会忽略警告。HSTS解决了这一问题，一旦服务器发送了HSTS字段，将不再允许用户忽略警告。

> ### 不足
>
> 用户首次访问某网站是不受HSTS保护的。这是因为首次访问时，浏览器还未收到HSTS，所以仍有可能通过明文HTTP来访问。解决这个不足当前有两种方案，一是浏览器预置HSTS域名列表，[Google Chrome](https://www.wikiwand.com/zh-cn/Google_Chrome)、[Firefox](https://www.wikiwand.com/zh-cn/Firefox)、[Internet Explorer](https://www.wikiwand.com/zh-cn/Internet_Explorer)和[Microsoft Edge](https://www.wikiwand.com/zh-cn/Microsoft_Edge)实现了这一方案[[11\]](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8#citenote11)[[12\]](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8#citenote12)。二是将HSTS信息加入到[域名系统](https://www.wikiwand.com/zh-cn/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F)记录中。但这需要保证DNS的安全性，也就是需要部署[域名系统安全扩展](https://www.wikiwand.com/zh-cn/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F%E5%AE%89%E5%85%A8%E6%89%A9%E5%B1%95)。截至2016年这一方案没有大规模部署。
>
> 由于HSTS会在一定时间后失效（有效期由max-age指定），所以浏览器是否强制HSTS策略取决于当前系统时间。部分操作系统经常通过[网络时间协议](https://www.wikiwand.com/zh-cn/%E7%B6%B2%E7%B5%A1%E6%99%82%E9%96%93%E5%8D%94%E8%AD%B0)更新系统时间，如[Ubuntu](https://www.wikiwand.com/zh-cn/Ubuntu)每次连接网络时、[OS X Lion](https://www.wikiwand.com/zh-cn/OS_X_Lion)每隔9分钟会自动连接时间服务器。攻击者可以通过伪造NTP信息，设置错误时间来绕过HSTS。解决方法是认证NTP信息，或者禁止NTP大幅度增减时间。比如[Windows 8](https://www.wikiwand.com/zh-cn/Windows_8)每7天更新一次时间，并且要求每次NTP设置的时间与当前时间不得超过15小时。

## Rails 关于 https 和 HSTS 的配置

总结自 [BigBinary](https://blog.bigbinary.com/2016/08/24/rails-5-adds-more-control-to-fine-tuning-ssl-usage.html) 和 [Rails 文档](https://api.rubyonrails.org/v5.2.1/classes/ActionDispatch/SSL.html)

```ruby
config.force_ssl = true # 例如在 application.rb 或者环境特定配置
```

这个配置做了三件事

### 将所有 http 请求重定向至 https

   ```ruby
   config.ssl_options = {  redirect: { status: 307, port: 81 } } # 默认 301，80 端口
   ```

### 在 cookies 设置安全标记，告诉浏览器禁用 http 请求。防止中间人攻击

   ```ruby
   config.ssl_options = { secure_cookies: false } # 停用该功能
   ```

### response headers 加上 HSTS

   ```ruby
   # 杜绝首次访问时尚未启用 hsts 的漏洞，前提是在 hstspreload.org 提交了网站域名。默认 false
   config.ssl_options = { hsts: { preload: true } }
   # hsts 有效期，默认 1 年
   config.ssl_options = { hsts: { expires: 10.days } }
   # 告诉浏览器停用 hsts，恢复 http
   config.ssl_options = { hsts: false } # 或者 hsts: { expires: 0 }
   # 子域名是否也启用 hsts，默认 true
   config.ssl_options = { subdomains: false}
   ```

## Reference

[HTTP严格传输安全](https://www.wikiwand.com/zh-cn/HTTP%E4%B8%A5%E6%A0%BC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8)

[Rails 5 adds more control to fine tuning SSL usage](https://blog.bigbinary.com/2016/08/24/rails-5-adds-more-control-to-fine-tuning-ssl-usage.html)

[actionpack/lib/action_dispatch/middleware/ssl.rb](https://api.rubyonrails.org/v5.2.1/files/actionpack/lib/action_dispatch/middleware/ssl_rb.html)
