---
title:  "Git Using Proxy to Bypass GFW"
tags: censorship git

---

*Fuck GFW*

Usage( [Credit](https://gist.github.com/evantoli/f8c23a37eb3558ab8765) )

```shell
git config --global http.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
```

For example proxy via local SS or V2Ray client

```shell
git config --global http.proxy http://127.0.0.1:1087
```

Or `git config --global --edit`
```gitconfig
[http]
	proxy = http://127.0.0.1:1087
[https]
	proxy = https://127.0.0.1:1087
```
