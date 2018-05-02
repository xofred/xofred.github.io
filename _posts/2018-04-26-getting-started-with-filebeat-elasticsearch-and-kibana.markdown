---
layout: single
title:  "Filebeat、Elasticsearch、Kibana 安装配置笔记"
toc: true
toc_label: "目录"
toc_icon: "cog"

---

## change log

- 2018-04-27 添加 X-Pack 卸载步骤，添加部署真实环境流程

## 部署真实环境流程

- 找一台应用服务器安装 Elasticsearch、Kibana，作为【日志服务器】
- [【日志服务器】上的 Nginx 代理 Elasticsearch、Kibana 的地址，访问需密码](https://coder.tw/?p=7235)
- 所有应用服务器安装 Filebeat，日志文件写向【日志服务器】
- 本地登【日志服务器】 Kibana 管理后台，查看日志

## 安装

以下均以 mac 为例，其他操作系统的见官方相关文档

[Elasticsearch、Kibana 安装](https://www.elastic.co/guide/en/beats/libbeat/6.2/getting-started.html)

[X-Pack 安装步骤](https://www.elastic.co/downloads/x-pack)

没想到最基本的安全模块——用户密码登录——都要收费。。。服

[X-Pack 卸载步骤](https://www.elastic.co/guide/en/x-pack/current/xpack-uninstalling.html)



Elasticsearch 安装 X-Pack 后初始化内置账号密码

```shell
bin/x-pack/setup-passwords auto
```

> Initiating the setup of passwords for reserved users elastic,kibana,logstash_system.
> The passwords will be randomly generated and printed to the console.
> Please confirm that you would like to continue [y/N]y
>
> Changed password for user kibana
> PASSWORD kibana = Y7YKxbphxRGmByIlAb24
>
> Changed password for user logstash_system
> PASSWORD logstash_system = PTNPg4pKUsQGESjD8224
>
> Changed password for user elastic
> PASSWORD elastic = X8XP5tYgMxJUU01ZJAzU



[Filebeat 快速入门](https://www.elastic.co/guide/en/beats/filebeat/6.2/filebeat-getting-started.html)

由于 [lograge](https://github.com/roidrage/lograge) 已对 Rails 日志做了筛选和优化，暂不打算使用 Logstash

## 配置

Filebeat 配置

```yaml
filebeat.prospectors:
- type: log
  enabled: true
  paths: # 一定要绝对路径，~/prjects/ 这种的会找不到
    - /Users/MD212/projects/elephant-server/log/development.log
  
setup.template.fields: "./fields.yml"

output.elasticsearch:
  hosts: ["localhost:9200"]
  username: "elastic"
  password: "X8XP5tYgMxJUU01ZJAzU"

output.elasticsearch.index: "sb-%{[beat.version]}-%{+yyyy.MM.dd}"
setup.template.name: "sb"
setup.template.pattern: "sb-*"
setup.dashboards.index: "sb-*"
setup.dashboards.enabled: true

setup.kibana:
  host: "localhost:5601"
  username: "kibana"
  password: "Y7YKxbphxRGmByIlAb24"

# 用 logstash 的话 index 必须手动加载到 es
#output.logstash:
#  hosts: ["localhost:5044"]
```



[输出 Logstash 的话，Elasticsearch 需要手动加载 Filebeat 索引](https://www.elastic.co/guide/en/beats/filebeat/6.2/filebeat-template.html#load-template-manually-alternate)

```shell
./filebeat setup --template -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
```





[Kibana 手动添加 Elasticsearch 索引](https://stackoverflow.com/questions/41722972/why-are-there-no-logstash-indexes-in-kibana)

**Management > Index Patterns > Add New**
