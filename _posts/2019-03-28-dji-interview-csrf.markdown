---
title:  "大疆电面问题之 csrf"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: web security rails interview
---

*tl; dr 那天短路了没答好，现在重新学习下*

维基百科讲得挺到位了，复制粘贴一段

### csrf 攻击

> ## 攻擊的細節
>
> 跨站請求攻擊，簡單地說，是攻擊者通過一些技術手段欺騙用戶的瀏覽器去訪問一個自己曾經認證過的網站並執行一些操作（如發郵件，發消息，甚至財產操作如轉帳和購買商品）。由於瀏覽器曾經認證過，所以被訪問的網站會認為是真正的用戶操作而去執行。這利用了web中用戶身份驗證的一個漏洞：**簡單的身份驗證只能保證請求發自某個用戶的瀏覽器，卻不能保證請求本身是用戶自愿發出的**。
>
> ### 例子
>
> 假如一家銀行用以執行轉帳操作的URL地址如下： <http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName>
>
> 那麼，一個惡意攻擊者可以在另一个網站上放置如下代碼： <img src="<http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman>">
>
> 如果有賬戶名為Alice的用戶訪問了惡意站點，而她之前剛訪問過銀行不久，登錄信息尚未過期，那麼她就會損失1000資金。
>
> 這種惡意的網址可以有很多種形式，藏身於網頁中的許多地方。此外，攻擊者也不需要控制放置惡意網址的網站。例如他可以將這種地址藏在論壇，博客等任何[用戶生成內容](https://www.wikiwand.com/zh/%E7%94%A8%E6%88%B6%E7%94%9F%E6%88%90%E5%85%A7%E5%AE%B9)的網站中。這意味著**如果伺服器端沒有合適的防禦措施的話，用戶即使訪問熟悉的可信網站也有受攻擊的危險**。
>
> 透過例子能夠看出，攻擊者並不能通過CSRF攻擊來直接獲取用戶的帳戶控制權，也不能直接竊取用戶的任何信息。他們能做到的，是**欺騙用戶瀏覽器，讓其以用戶的名義執行操作**。

### csrf 防范

> ## 防禦措施
>
> ### 檢查Referer字段
>
> HTTP頭中有一個Referer字段，這個字段用以標明請求來源於哪個地址。在處理敏感數據請求時，通常來說，Referer字段應和請求的地址位於同一域名下。以上文銀行操作為例，Referer字段地址通常應該是轉帳按鈕所在的網頁地址，應該也位於www.examplebank.com之下。而如果是CSRF攻擊傳來的請求，Referer字段會是包含惡意網址的地址，不會位於www.examplebank.com之下，這時候伺服器就能識別出惡意的訪問。
>
> 這種辦法簡單易行，工作量低，僅需要在關鍵訪問處增加一步校驗。但這種辦法也有其局限性，因其完全依賴瀏覽器發送正確的Referer字段。雖然http協議對此字段的內容有明確的規定，但並無法保證來訪的瀏覽器的具體實現，亦無法保證瀏覽器沒有安全漏洞影響到此字段。并且也存在攻擊者攻擊某些瀏覽器，篡改其Referer字段的可能。
>
> ### 添加校驗token
>
> 由於CSRF的本質在於攻擊者欺騙用戶去訪問自己設置的地址，所以如果要求在訪問敏感數據請求時，要求用戶瀏覽器提供不保存在[cookie](https://www.wikiwand.com/zh/Cookie)中，并且攻擊者無法偽造的數據作為校驗，那麼攻擊者就無法再執行CSRF攻擊。這種數據通常是表單中的一個數據項。伺服器將其生成並附加在表單中，其內容是一個偽亂數。當客戶端通過表單提交請求時，這個偽亂數也一並提交上去以供校驗。正常的訪問時，客戶端瀏覽器能夠正確得到並傳回這個偽亂數，而通過CSRF傳來的欺騙性攻擊中，攻擊者無從事先得知這個偽亂數的值，伺服器端就會因為校驗token的值為空或者錯誤，拒絕這個可疑請求。

### rails 对 csrf 的防范

rails 的措施是**添加校验 token**。怎么用上呢，在 controller（例如直接加在 application_controller.rb，一劳永逸）带上 `protect_from_forgery with: :exception`，在 view（例如直接加在 layouts/application.html.erb，懒人首选）带上 `<%= csrf_meta_tags %>`，就能达到目的。[实现原理猛击这里（我其实还没看完）](https://medium.com/rubyinside/a-deep-dive-into-csrf-protection-in-rails-19fa0a42c0ef)

*ps 记得松本老总的编程世界也有提到*

### Reference

[跨站请求伪造](https://www.wikiwand.com/zh/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)

[A Deep Dive into CSRF Protection in Rails](https://medium.com/rubyinside/a-deep-dive-into-csrf-protection-in-rails-19fa0a42c0ef)
