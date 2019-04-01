---
title:  "大疆电面问题之 xss"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: web security xss
---

*tl; dr 发觉 xss 这个话题其实挺大的，面试官你究竟想聊啥*

*个人经历过的 xss，是新浪微博的密码被反复修改、自动关注、自动发帖。感觉就是传说中的存储型/反射型 xss，攻击代码已深深植入了新浪的数据库/页面😓*

*[google 出品的"xss 闯关游戏"](https://xss-game.appspot.com/)， 我通关了，可以加深 xss 的了解 :)*

### 如何防范 xss

贴一段[美团技术团队](https://tech.meituan.com/2018/09/27/fe-security.html) 的总结

> 虽然很难通过技术手段完全避免 XSS，但我们可以总结以下原则减少漏洞的产生：
>
> - **利用模板引擎** 开启模板引擎自带的 HTML 转义功能。例如： 在 ejs 中，尽量使用 `<%= data %>` 而不是 `<%- data %>`。
> - **避免内联事件** 尽量不要使用 `onLoad="onload('{{data}}')"`、`onClick="go('{{action}}')"` 这种拼接内联事件的写法。在 JavaScript 中通过 `.addEventlistener()` 事件绑定会更安全。
> - **避免拼接 HTML** 前端采用拼接 HTML 的方法比较危险，如果框架允许，使用 `createElement`、`setAttribute` 之类的方法实现。或者采用比较成熟的渲染框架，如 Vue/React 等。
> - **时刻保持警惕** 在插入位置为 DOM 属性、链接等位置时，要打起精神，严加防范。
> - **增加攻击难度，降低攻击后果** 通过 CSP、输入长度配置、接口安全措施等方法，增加攻击的难度，降低攻击的后果。
> - **主动检测和发现** 可使用 XSS 攻击字符串和自动扫描工具寻找潜在的 XSS 漏洞。

### Reference

[netsparker 在开发Ruby on Rails Web应用程序时防止跨站点脚本漏洞](https://www.netsparker.com/blog/web-security/preventing-xss-ruby-on-rails-web-applications/#htmlescaping)

[netsparker 跨站点脚本（XSS）漏洞：定义和预防](https://www.netsparker.com/blog/web-security/cross-site-scripting-xss/)

[netsparker 基于DOM的跨站点脚本编制漏洞](https://www.netsparker.com/blog/web-security/dom-based-cross-site-scripting-vulnerability/)

[rails guide 安全指南](https://guides.rubyonrails.org/security.html#cross-site-scripting-xss)

[brakeman——检测 rails 项目风险的 gem](https://brakemanscanner.org/docs/quickstart/)

[某 xss 大佬的经验](https://github.com/s0md3v/AwesomeXSS)

[google 关于 xss 的总结](https://www.google.com/about/appsecurity/learning/xss/index.html#Learn)

[美团技术团队关于 xss 的总结](https://tech.meituan.com/2018/09/27/fe-security.html)
