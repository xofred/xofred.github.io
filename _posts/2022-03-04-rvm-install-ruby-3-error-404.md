---
title:  "RVM install ruby 3 got error 404"
tags: ruby rvm openssl
toc: true

---

*This is stupid. And it's still occurring :(*

### Problem

When I `rvm install ruby-3.0.0`, I got the following errors:

```
ruby-3.0.0 - #removing src/ruby-3.0.0 - please wait
Searching for binary rubies, this might take some time.
Found remote file https://rvm_io.global.ssl.fastly.net/binaries/osx/10.15/x86_64/ruby-3.0.0.tar.bz2
Checking requirements for osx.
Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date.
Requirements installation successful.
ruby-3.0.0 - #configure
ruby-3.0.0 - #download
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (22) The requested URL returned error: 404 Not Found
The requested url does not exist(22): 'https://rvm_io.global.ssl.fastly.net/binaries/osx/10.15/x86_64/ruby-3.0.0.tar.bz2?rvm=1.29.12'
Checking fallback: ftp://rvm_io.global.ssl.fastly.net/binaries/osx/10.15/x86_64/ruby-3.0.0.tar.bz2?rvm=1.29.12
Checking fallback: https://www.mirrorservice.org/sites/rvm_io.global.ssl.fastly.net/binaries/osx/10.15/x86_64/ruby-3.0.0.tar.bz2?rvm=1.29.12
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
curl: (22) The requested URL returned error: 404 Not Found
The requested url does not exist(22): 'https://www.mirrorservice.org/sites/rvm_io.global.ssl.fastly.net/binaries/osx/10.15/x86_64/ruby-3.0.0.tar.bz2?rvm=1.29.12'
Failed download
Downloading https://rvm_io.global.ssl.fastly.net/binaries/osx/10.15/x86_64/ruby-3.0.0.tar.bz2 failed.
Mounting remote ruby failed with status 2, trying to compile.
Checking requirements for osx.
Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date.
Requirements installation successful.
Installing Ruby from source to: /Users/justin/.rvm/rubies/ruby-3.0.0, this may take a while depending on your cpu(s)...
ruby-3.0.0 - #downloading ruby-3.0.0, this may take a while depending on your connection...
ruby-3.0.0 - #extracting ruby-3.0.0 to /Users/justin/.rvm/src/ruby-3.0.0 - please wait
ruby-3.0.0 - #configuring - please wait
Error running ' CFLAGS=-O3 -I/usr/local/opt/libyaml/include -I/usr/local/opt/libksba/include -I/usr/local/opt/readline/include -I/usr/local/opt/zlib/include -I/usr/local/opt/openssl@1.1/include -I/usr/local/opt/libyaml/include -I/usr/local/opt/libksba/include -I/usr/local/opt/readline/include -I/usr/local/opt/zlib/include -I/usr/local/opt/openssl@1.1/include LDFLAGS=-L/usr/local/opt/libyaml/lib -L/usr/local/opt/libksba/lib -L/usr/local/opt/readline/lib -L/usr/local/opt/zlib/lib -L/usr/local/opt/openssl@1.1/lib -L/usr/local/opt/libyaml/lib -L/usr/local/opt/libksba/lib -L/usr/local/opt/readline/lib -L/usr/local/opt/zlib/lib -L/usr/local/opt/openssl@1.1/lib ./configure --prefix=/Users/justin/.rvm/rubies/ruby-3.0.0 --disable-install-doc --enable-shared',
please read /Users/justin/.rvm/log/1646377732_ruby-3.0.0/configure.log
There has been an error while running configure. Halting the installation.
```

I found a [blog post](https://mentalized.net/journal/2021/11/29/ruby-3-0-3-rvm-and-openssl-1-1/) with similar issues, though the errors are not the same. So I try his way:

```shell
PKG_CONFIG_PATH=/opt/local/lib/openssl-1.1/pkgconfig rvm install 3.0.0 --with-openssl-lib=/opt/local/lib/openssl-1.1 --with-openssl-include=/opt/local/include/openssl-1.1
```

And it works. ( the moment I saw `#compiling`, I felt it would work )

```
ruby-3.0.0 - #removing src/ruby-3.0.0 - please wait
Checking requirements for osx.
Certificates bundle '/usr/local/etc/openssl@1.1/cert.pem' is already up to date.
Requirements installation successful.
Installing Ruby from source to: /Users/justin/.rvm/rubies/ruby-3.0.0, this may take a while depending on your cpu(s)...
ruby-3.0.0 - #downloading ruby-3.0.0, this may take a while depending on your connection...
ruby-3.0.0 - #extracting ruby-3.0.0 to /Users/justin/.rvm/src/ruby-3.0.0 - please wait
ruby-3.0.0 - #configuring - please wait
ruby-3.0.0 - #post-configuration - please wait
ruby-3.0.0 - #compiling - please wait
ruby-3.0.0 - #installing - please wait
ruby-3.0.0 - #making binaries executable - please wait
Installed rubygems 3.2.3 is newer than 3.0.9 provided with installed ruby, skipping installation, use --force to force installation.
ruby-3.0.0 - #gemset created /Users/justin/.rvm/gems/ruby-3.0.0@global
ruby-3.0.0 - #importing gemset /Users/justin/.rvm/gemsets/global.gems - please wait
ruby-3.0.0 - #generating global wrappers - please wait
ruby-3.0.0 - #gemset created /Users/justin/.rvm/gems/ruby-3.0.0
ruby-3.0.0 - #importing gemsetfile /Users/justin/.rvm/gemsets/default.gems evaluated to empty gem list
ruby-3.0.0 - #generating default wrappers - please wait
ruby-3.0.0 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-3.0.0 - #complete
Ruby was built without documentation, to build it run: rvm docs generate-ri
```

### Reference

[How to install Ruby 3.0.3 with OpenSSL using MacPorts and RVM](https://mentalized.net/journal/2021/11/29/ruby-3-0-3-rvm-and-openssl-1-1/)
