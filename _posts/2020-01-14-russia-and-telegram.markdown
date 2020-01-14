---
title:  "转载：俄罗斯和 Telegram 之间的封锁和反封锁"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: censorship

---

*https://www.solidot.org/story?sid=63255*

## 报道
> 在混沌俱乐部年会上，软件开发者 Leonid Evdokimov [回顾了](https://media.ccc.de/v/35c3-9653-russia_vs_telegram_technical_notes_on_the_battle)俄罗斯政府和流行消息应用 Telegram 之间史诗性的封锁和反封锁。2018 年 4 月，因为 Telegram 拒绝遵守**法庭命令交出加密密钥**，俄罗斯**政府下令对其进行屏蔽**，第一天有 400 万 IP 被封，第二天有 1600 万 IP 被封，最多有 1900 万 IP 被封。俄罗斯封掉了云服务商如 Amazon、Google、Microsoft、DigitalOcean 和 Hetzner，涉及了 0.5% 的 IP 地址空间，试图**向云服务商施压，让 Telegram 成为这些企业不受欢迎的人**。俄罗斯还**不小心封掉了本国互联网服务**如 VKontakte 和 Yandex 的 IP 地址。尽管**政府否认，但附带损伤是巨大的**。 到 4 月底封杀力度开始减弱，到 6 月 8 日封杀的 IP 地址减少到 400 万，而在**期间 Telegram 一直能在俄罗斯境内工作**。俄罗斯被观察到使用主动探测去识别代理服务器，主动探测已经日益成为一种常用的[监测和识别代理服务器](https://gfw.report/blog/gfw_shadowsocks/zh.html)的方法。

## 点评
### 【暴力】法庭命令交出加密密钥
类似《xxxx法》，交出加密方法、算法、密钥，从根源上“解决问题”

### 【暴力】向云服务商施压，让 Telegram 成为这些企业不受欢迎的人
类似对付xx系报纸的手段，压缩其生存空间

### 【暴力】不小心封掉了本国互联网服务，但附带损伤是巨大的
流量特征识别是目前“最先进”的防火手段，即便如此，无法百分百识别。另外，如果加中转的话误杀率会更高

### 【谎言】政府否认
几个国家不约而同地选择了墙作为“加强”控制的手段，却否认其存在

## 彩蛋

[Shadowsocks是如何被检测和封锁的](https://gfw.report/blog/gfw_shadowsocks/zh.html)
