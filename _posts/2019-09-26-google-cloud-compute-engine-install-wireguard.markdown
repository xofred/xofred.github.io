---
title:  "Google 云计算实例安装 Wireguard"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: gfw wireguard google-cloud

---

*tldr 去死吧，功夫墙*

### 服务端部分

1. 在 Google 云计算控制台配置防火墙，允许 udp 51820 入站
2. 连上实例，设置 root 密码

```shell
sudo passwd
```

3. 运行一键安装脚本

```shell
wget --no-check-certificate -qO- 'https://moeclub.org/attachment/LinuxShell/wireguard.sh'| bash
```

4. 在 Google 云计算控制台重启实例

5. 再次运行一键安装脚本

### 客户端部分

iOS 安装 Wireguard，扫描二维码

### Reference

[科学上网新方法ShadowSocks继任者WireGuard](https://pincong.rocks/article/5489)

[Debian WireGuard一键安装脚本 新一代快速安全VPN](https://ssr.tools/1079)

[Get root password for Google Cloud Engine VM](https://stackoverflow.com/questions/35016795/get-root-password-for-google-cloud-engine-vm)

[Can't get Wireguard to load the interface wg0 at all. Debian 9.8](https://www.reddit.com/r/WireGuard/comments/b3jp39/cant_get_wireguard_to_load_the_interface_wg0_at/)
