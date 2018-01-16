---
layout: default
title:  "定制 facebook 分享页"
---

# 定制 facebook 分享页

> ## # 开放图谱标记
>
> 大部分内容都以网址的形式分享到 Facebook，因此，使用开放图谱标签标记网站很重要，可帮助您控制内容在 Facebook 的显示方式。
>
> 在没有这些标签的情况下，[Facebook 爬虫](https://developers.facebook.com/docs/sharing/webmasters/crawler)会使用内部试探法对内容的标题、说明和预览图作出最合理猜测。使用开放图谱标签显式指定此信息，可确保在 Facebook 上发布高质量的内容。
>
> 以下示例展示了使用开放图谱标签生成内容，以便在 Facebook 呈现最优显示效果：
>
> ![img](https://scontent-nrt1-1.xx.fbcdn.net/v/t39.2178-6/10956906_396737803821010_168799778_n.png?oh=0346dbe6e82042b7945162f7169a2daf&oe=5AE15BC0)
>
> ### 标记示例
>
> 例如，以下介绍了如何使用 `og:type="article"` 和多个附加属性标记文章、动态消息或博文：
>
> ```html
> <meta property="og:url"                content="http://www.nytimes.com/2015/02/19/arts/international/when-great-minds-dont-think-alike.html" />
> <meta property="og:type"               content="article" />
> <meta property="og:title"              content="When Great Minds Don’t Think Alike" />
> <meta property="og:description"        content="How much does culture influence creative thinking?" />
> <meta property="og:image"              content="http://static01.nyt.com/images/2015/02/19/arts/international/19iht-btnumbers19A/19iht-btnumbers19A-facebookJumbo-v2.jpg" />
> ```
>
> 
>
> 这些属性包括我们在用户分享文章时，具体想要呈现的、与文章有关的描述性元数据。
>
> ### 基本标签
>
> 这些是您应该用于所有内容类型的最基本的元标签：
>
> | 标签               | 说明                                       |
> | ---------------- | ---------------------------------------- |
> | `og:url`         | 页面的权威链接。此标签应该是未加修饰的网址，没有会话变量、用户识别参数或计数器。此网址的“赞”和“分享”将在此网址中汇总。例如，移动域网址应将桌面版网址指定为权威链接，用于跨不同页面版本汇总“赞”和“分享”。 |
> | `og:title`       | 文章的标题，不包括任何品牌，例如您的网站名称。                  |
> | `og:description` | 内容的简略说明，通常为 2-4 个句子。此标签会显示在 Facebook 帖子的标题下方。 |
> | `og:image`       | 用户将内容分享到 Facebook 时显示的图片的网址。请参阅[下文](https://developers.facebook.com/docs/sharing/webmasters#images)了解详情，并查看[最佳实践指南](https://developers.facebook.com/docs/sharing/best-practices#images)了解如何指定高质量的预览图。 |
> | `fb:app_id`      | 要使用 [Facebook 成效分析](https://developers.facebook.com/docs/sharing/referral-insights)，您必须将应用编号添加到您的主页中。您可以通过成效分析查看来自 Facebook 的网站访问量。在[应用面板](https://developers.facebook.com/apps/redirect/dashboard)中查找应用编号。 |
>
> 您还可以添加两个附加标签，推动内容的传播并提高参与度：
>
> | 标签          | 说明                                       |
> | ----------- | ---------------------------------------- |
> | `og:type`   | 内容的媒体类型。此标签会影响您的内容在动态消息中的显示方式。 如果不指定类型，类型会默认为 `website`。每个网址都应该是单一对象，因此不可能存在多个 `og:type` 值。在[对象类型参考文档](https://developers.facebook.com/docs/reference/opengraph#object-type)中查找对象类型的完整列表 |
> | `og:locale` | 资源的区域设置。 默认为 `en_US`。如果提供其他可用语言的翻译，您还可使用 `og:locale:alternate`。请阅读[本地化文档](https://developers.facebook.com/docs/internationalization#locales)，了解我们支持的区域设置。 |



> #### # 图片尺寸
>
> 为了在高分辨率设备上完美展示，建议使用分辨率至少达到 1200 x 630 像素的图片。作为最低要求，您应使用 600 x 315 像素的图片，以便显示带较大图片的链接页面帖子。 文件大小不得超过 8MB



> ### # 预缓存图片
>
> 首次分享内容时，Facebook 爬虫将抓取并缓存所分享网址的元数据。爬虫必须“看到”图片至少一次，然后方能加以呈现。这表示第一个分享某内容的用户无法看到呈现的图片：
>
> 
>
> 我们可以通过两种方法来避免这种情况，让图片在首次获得点赞或分享时即呈现：
>
> ##### 1.通过[分享调试器](https://developers.facebook.com/tools/debug)预缓存图片
>
> 通过网址调试器运行网址，预先获取页面的元数据。更新某内容的图片时，也应执行此操作。
>
> ##### 2.使用 `og:image:width` 和 `og:image:height` 开放图谱标签
>
> 使用这些标签可为爬虫指定图片尺寸，让它立即呈现图片，无需异步下载和处理。



Reference:

[网站管理员分享指南](https://developers.facebook.com/docs/sharing/webmasters#markup)

[网站和移动应用的分享最佳实践](https://developers.facebook.com/docs/sharing/best-practices#images)

[分享调试器](https://developers.facebook.com/tools/debug)
