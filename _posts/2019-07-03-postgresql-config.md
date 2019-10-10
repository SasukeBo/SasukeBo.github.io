---
title:  "PostgreSQL Config 备忘录"
date:   2019-07-03 20:35:00 +0800
categories: PostgreSQL
permalink: /:categories/:title
header:
  overlay_image: /assets/images/PGBanner.jpg
---

记录Mac OS上配置PostgreSQL数据库。

### 安装路径

通过HomeBrew安装的PostgreSQL在本机的`/usr/local/Cellar/postgresql/11.3/`目录下

### LaunchCtl PostgreSQL

`/usr/local/Cellar/postgresql/11.3/`路径下的`homebrew.mxcl.postgresql.plist`配置文件，
可以设定`DATADIR`，默认为`/usr/local/var/postgres`，可以修改为自定义路径。

### 设置autoboot

使用Mac自带的`launchctl`管理工具。

```sh
# 加载启动配置文件，开机后会自动启动postgresql
launchctl load -w homebrew.mxcl.postgresql.plist
```

### 通过pg_ctl管理postgresql

通过设置系统环境变量
```sh
# add to .zshrc
export PGDATA=/usr/local/var/postgres

# 手动启动
pg_ctl start
# reload config_file
pg_ctl reload
# 关闭
pg_ctl stop
# 重启
pg_ctl restart
```

### 配置postgresql database 认证方式


通过修改`DATADIR`路径下的`pg_hba.conf`文件。
该文件描述为`PostgreSQL Client Authentication Configuration File`。
一般而言修改默认的配置的`METHOD`字段为`password`后连接数据库需要提供密码。
更多认证方式和针对不同应用场景下的认证配置可以阅读`pg_hba.conf`文件中的文档部分。
