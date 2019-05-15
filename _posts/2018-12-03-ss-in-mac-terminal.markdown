---
title:  "Mac OSX 终端走 shadowsocks 代理"
tags: osx terminal shadowsocks
---

*tl; dr [这位大佬的教程更简洁有力](https://github.com/mrdulin/blog/issues/18)*

原理：用本地 SS 客户端的 Socks5 作为终端 HTTP(S) 的代理

------

安装所需软件

```shell
brew install proxychains-ng
```

测试

```shell
sudo proxychains4 curl ip.gs
```

因为端口和本地 SS 客户端的 Socks5 端口不一致，所以失败

```
[proxychains] config file found: /usr/local/etc/proxychains.conf
[proxychains] preloading /usr/local/Cellar/proxychains-ng/4.13/lib/libproxychains4.dylib
[proxychains] DLL init: proxychains-ng 4.13
[proxychains] Strict chain  ...  127.0.0.1:9050  ...  timeout
curl: (7) Couldn't connect to server
```

编辑配置文件 `/usr/local/etc/proxychains.conf`

```conf
#socks4 	127.0.0.1 9050
socks5 	127.0.0.1 1086 # 本地 SS 客户端的 Socks5 端口
```

再次测试

```shell
sudo proxychains4 curl ip.gs
```

成功使用上 SS 的运营商

```
[proxychains] config file found: /usr/local/etc/proxychains.conf
[proxychains] preloading /usr/local/Cellar/proxychains-ng/4.13/lib/libproxychains4.dylib
[proxychains] DLL init: proxychains-ng 4.13
[proxychains] Strict chain  ...  127.0.0.1:1086  ...  ip.gs:80  ...  OK
Current IP / 当前 IP: 172.105.194.73
ISP / 运营商:  linode.com
City / 城市: Tokyo Tokyo
Country / 国家: Japan
```

再需要安装国外源（例如 `bundle` `brew install`）的时候，命令前加`proxychains4`，即可感受速度与激情

```shell
proxychains4 bundle
```

```
[proxychains] Strict chain  ...  127.0.0.1:1086  ...  rubygems.org:443  ...  OK
[proxychains] DLL init: proxychains-ng 4.13
```



Reference: [Mac OSX终端走shadowsocks代理](https://github.com/mrdulin/blog/issues/18)
