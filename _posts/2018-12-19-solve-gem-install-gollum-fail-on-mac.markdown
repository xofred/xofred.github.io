---
title:  "解决 Mac 安装 gollum 时依赖的 charlock_holmes 安装时编译失败（好绕）"
toc: true
toc_label: "目录"
toc_icon: "cog"
---

一句话总结：编译时找不到底层库，以及 clang 编译的 ruby 不兼容 c++11 语法

------

## 折腾经过

安装 gollum

```shell
gem install gollum
```

提示 charlock_holmes 编译失败

```shell
ERROR:  Error installing gollum:
	ERROR: Failed to build gem native extension.

    current directory: /Library/Ruby/Gems/2.3.0/gems/charlock_holmes-0.7.6/ext/charlock_holmes
/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/bin/ruby -r ./siteconf20181219-28197-14b38xf.rb extconf.rb
checking for main() in -licui18n... no
checking for main() in -licui18n... yes
checking for unicode/ucnv.h... yes
checking for main() in -lz... yes
checking for main() in -licuuc... yes
checking for main() in -licudata... yes
creating Makefile

To see why this extension failed to compile, please check the mkmf.log which can be found here:

  /Library/Ruby/Gems/2.3.0/extensions/universal-darwin-18/2.3.0/charlock_holmes-0.7.6/mkmf.log
```

查看编译日志

```shell
bat /Library/Ruby/Gems/2.3.0/extensions/universal-darwin-18/2.3.0/charlock_holmes-0.7.6/mkmf.log
```

提示找不到 licui18n 库

```
ld: library not found for -licui18n
```

Google、stack overflow、GitHub 了一圈，各种折腾，最后在安装 charlock_holmes 时指定 icu4 库的位置（不清楚自己 icu4 库位置的话，`brew reinstall icu4 ` 最后会有提示）

```shell
gem install charlock_holmes -- --with-icu-dir=/usr/local/opt/icu4c
```

编译日志找不到错误了，然而发现外面提示某几个地方 c++11 语法出错

```shell
/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/include/ruby-2.3.0/universal-darwin18/ruby/config.h:365:31: error: invalid suffix on literal; C++11 requires a space between literal and identifier [-Wreserved-user-defined-literal]
#define RUBY_ARCH "universal-"RUBY_PLATFORM_OS
                              ^

/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/include/ruby-2.3.0/universal-darwin18/ruby/config.h:366:35: error: invalid suffix on literal; C++11 requires a space between literal and identifier [-Wreserved-user-defined-literal]
#define RUBY_PLATFORM "universal."RUBY_PLATFORM_CPU"-"RUBY_PLATFORM_OS
                                  ^

/Library/Developer/CommandLineTools/SDKs/MacOSX10.14.sdk/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/include/ruby-2.3.0/universal-darwin18/ruby/config.h:366:55: error: invalid suffix on literal; C++11 requires a space between literal and identifier [-Wreserved-user-defined-literal]
#define RUBY_PLATFORM "universal."RUBY_PLATFORM_CPU"-"RUBY_PLATFORM_OS
```

网上说 clang 编译的 ruby 会出现不兼容 c++11 语法的错误

```shell
rvm use 2.3.7                                                                
ruby -rrbconfig -e 'puts RbConfig::MAKEFILE_CONFIG["CC"]'                   
```

```
xcrun clang
```

这时思路是，要么自己用 gcc 编译一个 ruby 2.3.7，要么换一个用 gcc 编译好的 ruby 版本。我决定改用 ruby 2.5.1，设为默认（懒得以后又要搞，公司项目就用专门 ruby 版本的 gemset 好了）

```shell
rvm reinstall 2.5.1
rvm use 2.5.1 --default
ruby -rrbconfig -e 'puts RbConfig::MAKEFILE_CONFIG["CC"]' 
```

```
gcc
```

安装 gollum，按提示恢复其他依赖的 gem

```shell
gem install gollum
```

然后轮到万年报错狂 nokogiri

```shell
ERROR:  While executing gem ... (Gem::Ext::BuildError)
    ERROR: Failed to build gem native extension.

    current directory: /Users/justin/.rvm/gems/ruby-2.5.1/gems/nokogiri-1.7.2/ext/nokogiri
/Users/justin/.rvm/rubies/ruby-2.5.1/bin/ruby -r ./siteconf20181219-45562-9ctedf.rb extconf.rb --use-system-libraries
checking if the C compiler accepts ... yes
checking if the C compiler accepts -Wno-error=unused-command-line-argument-hard-error-in-future... no
Building nokogiri using system libraries.
ERROR: cannot discover where libxml2 is located on your system. please make sure `pkg-config` is installed.
```

指定 libxml2 的 pkg-config 路径（如果没安装先 `brew install libxml2 `）

```shell
export PKG_CONFIG_PATH="/usr/local/opt/libxml2/lib/pkgconfig"
```

继续恢复其他依赖的 gem

```shell
gem pristine pg --version 0.20.0
gem pristine posix-spawn --version 0.3.13
gem pristine puma --version 3.8.2
gem pristine rmagick --version 2.16.0
gem pristine rucaptcha --version 2.2.0
gem pristine unf_ext --version 0.0.7.4
gem pristine websocket-driver --version 0.6.5
```

终于跑起来了

```shell
gollum                                                                       18.12.19    12:14:02 
== Sinatra (v1.4.8) has taken the stage on 4567 for development with backup from Puma
Puma starting in single mode...
* Version 3.8.2 (ruby 2.5.1-p57), codename: Sassy Salamander
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://0.0.0.0:4567
Use Ctrl-C to stop
```

## Reference

ruby 编译器问题 [Install fails with ruby 2.3.0 on OSX 10.11.2](https://github.com/SciRuby/nmatrix/issues/426)

找不到 licui18n 问题 [Installation of mastodon on Centos: "checking for -licui18n... No"](https://github.com/tootsuite/mastodon/issues/5507)
