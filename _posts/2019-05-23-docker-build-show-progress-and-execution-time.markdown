---
title:  "docker build 镜像时在终端输出进度和用时"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: docker docker-build tool
---

*tl; dr [How to measure Docker build steps duration?](https://stackoverflow.com/questions/46166293/how-to-measure-docker-build-steps-duration)*

### 部署脚本

有时 build 镜像要跑很久（以 ruby 或 rails 项目为例，主要是 bundle 拉 GitHub gems 用时很久），想知道进度和最终用时，可以加上 `export DOCKER_BUILDKIT=1`

```ruby
# 部署脚本片段
def build_ruby(pwd, hub_url, version)
  path = File.join(pwd, "build", "ruby")
  run("export DOCKER_BUILDKIT=1 && docker build -t #{hub_url}:#{version} #{path}")
end

def run(command)
  ap "准备执行的命令: #{command}"
  system(command) # system 跑系统命令可以打印信息
  if $?.to_i != 0 # 非正常退出
    raise $?.to_s
  end
end
```

### 输出例子

docker build 命令本身就不解释了，可以 `docker build —help`或者看 [docker 文档](https://docs.docker.com/engine/reference/commandline/build/)

```shell
"准备执行的命令: export DOCKER_BUILDKIT=1 && docker build -t registry-intl.cn-hongkong.aliyuncs.com/18back/sex-ruby-staging:0.0.19 /Users/justin/projects/18build/build/ruby"
[+] Building 0.5s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                                                                                      0.1s
 => => transferring dockerfile: 298B                                                                                                      0.0s
 => [internal] load .dockerignore                                                                                                         0.1s
 => => transferring context: 80B                                                                                                          0.0s
 => [internal] load metadata for registry-intl.cn-hongkong.aliyuncs.com/18utils/ruby:2.6                                                  0.0s
 => [internal] load build context                                                                                                         0.3s
 => => transferring context: 1.72MB                                                                                                       0.3s
 => [internal] helper image for file operations                                                                                           0.0s
 => [1/4] FROM registry-intl.cn-hongkong.aliyuncs.com/18utils/ruby:2.6                                                                    0.0s
 => CACHED [2/4] COPY ./ /ruby                                                                                                            0.0s
 => CACHED [3/4] RUN bundle config build.nokogiri --use-system-libraries                                                                  0.0s
 => CACHED [4/4] RUN bundle install --path vendor/bundle                                                                                  0.0s
 => exporting to image                                                                                                                    0.0s
 => => exporting layers                                                                                                                   0.0s
 => => writing image sha256:caa0ccd8abd48adf830a6c2f6537a4f73cefa2d3fc2dff234161bfa004aeaca6                                              0.0s
 => => naming to registry-intl.cn-hongkong.aliyuncs.com/18back/sex-ruby-staging:0.0.19                                                    0.0s
```

### 参考

[How to measure Docker build steps duration?](https://stackoverflow.com/questions/46166293/how-to-measure-docker-build-steps-duration)

[Running a command from Ruby displaying and capturing the output](https://stackoverflow.com/questions/10224481/running-a-command-from-ruby-displaying-and-capturing-the-output)
