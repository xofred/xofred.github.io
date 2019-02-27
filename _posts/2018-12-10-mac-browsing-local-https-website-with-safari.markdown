---
title: Mac 用 Safari 访问本地指定域名端口的 https 网站
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: osx development ssl
---


一句话总结：openssl + nginx + safari

---

### 碰到的问题

公司项目设计成单点登录，所以几个网站的 session 都会跳到同一个地方。我在 hosts 指定了几个域名

```hosts
127.0.0.1 dev.siteA.com
127.0.0.1 dev.siteB.com
127.0.0.1 dev.siteC.com
127.0.0.1 dev.siteCENTRE.com
```

Chrome 就算配置好了以下，还是不能访问任何一个域名，除非是 localhost

Safari 的话在从 ABC 站跳到 CENTRE 的时候会报错，因为 CENTRE 的端口不是 80

### 解决步骤

生成本地证书

```shell
openssl req -x509 -sha256 -nodes -newkey rsa:2048 -days 365 -keyout localhost.key -out localhost.crt
```

重新安装 puma

```shell
gem install puma -v 'YOUR VERSION' -- --with-cppflags=-I/usr/local/opt/openssl/include --with-ldflags=-L/usr/local/opt/openssl/lib
```

配置并重启 nginx

```conf
server {
  listen       dev.siteCENTRE.com:80;
  server_name  dev.siteCENTRE.com;
  location / {
    proxy_pass http://127.0.0.1:3000;
  }
}
```

puma 启动时绑定证书

```shell
rails s -b 'ssl://localhost:3000?key=localhost.key 绝对路径&cert=localhost.crt 绝对路径'
```

最后，在 Safari 强制访问不安全站点

### Reference

[Rails Local Development over HTTPS using a Self-Signed SSL Certificate](https://www.madeintandem.com/blog/rails-local-development-https-using-self-signed-ssl-certificate/)

[Can I map a hostname *and* a port with /etc/hosts? ](https://stackoverflow.com/questions/10729034/can-i-map-a-hostname-and-a-port-with-etc-hosts)

[SSL not available in this build #971](https://github.com/puma/puma/issues/971)

