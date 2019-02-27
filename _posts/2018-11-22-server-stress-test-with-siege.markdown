---
title:  "压测工具 siege 使用笔记"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: tools back-end
---

**ps: 经同事提点，发现 [Loader.io](https://loader.io/) 强大太多了，关键还免费😂**

## 直接上命令

```shell
siege -b --no-follow -t1m -c 255 www.linode.com
```
-b 压测模式，请求之间没有延迟

--no-follow 不重定向

-t【时长】【时 h，分 m，秒 s】

-c【并发数】默认 25，最大 255。单机并发数有限，需要大并发测试用云服务例如 [Loader.io](https://loader.io/) 会比较有参考性

## 报告如下

```shell
Lifting the server siege...
Transactions:		        7940 hits
Availability:		      100.00 %
Elapsed time:		       59.94 secs
Data transferred:	        1.35 MB
Response time:		        1.80 secs
Transaction rate:	      132.47 trans/sec
Throughput:		        0.02 MB/sec
Concurrency:		      238.83
Successful transactions:        7940
Failed transactions:	           0
Longest transaction:	       21.30
Shortest transaction:	        0.36
```

Reference: [Load Testing Web Servers with Siege](https://www.linode.com/docs/tools-reference/tools/load-testing-with-siege/)
