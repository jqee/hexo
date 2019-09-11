---
title: CentOS7安装Yapi
top: false
cover: false
toc: true
mathjax: true
date: 2019-09-11 15:46:04
tags: centos yapi
password:
summary: 记录在公司安装yapi服务
categories:
author: weibing
---

### 1.安装nodejs

1.1 安装 nvm,官网: https://github.com/nvm-sh/nvm

1.2  脚本

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
```

1.3 安装nodejs, 需要重新连接

```shell
nvm insatll stable
```

1.4 查看版本

```shell
[root@izbp19l85l2pc4aycovp8fz ~]# node -v
v12.10.0
[root@izbp19l85l2pc4aycovp8fz ~]# npm -v
6.10.3
```



### 2.安装mongoDB

2.1 官网: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/

2.2 创建repo

```shell
vi /etc/yum.repos.d/mongodb-org-4.2.repo
```

2.3 插入以下内容

```
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

2.4 安装:

```shell
sudo yum install -y mongodb-org
```

2.5 开启服务 mongod

```shell
systemctl start mongod
```

2.6 加入开机启动

```shell
systemctl enable mongod
```

2.7 查看版本

```shell
[root@izbp19l85l2pc4aycovp8fz ~]# mongo --version
MongoDB shell version v4.2.0
git version: a4b751dcf51dd249c5865812b390cfd1c0129c30
OpenSSL version: OpenSSL 1.0.1e-fips 11 Feb 2013
allocator: tcmalloc
modules: none
build environment:
    distmod: rhel70
    distarch: x86_64
    target_arch: x86_64


```



### 3 安装Yapi

3.1 下载yapi安装包

GitHub网站: https://github.com/YMFE/yapi

下载最新安装包: https://github.com/YMFE/yapi/releases

目前最新 v1.8.3

```shell
mkdir /opt/yapi
cd /opt/yapi
wget https://github.com/YMFE/yapi/archive/v1.8.3.tar.gz
tar -zxvf v1.8.3.tar.gz

```

3.2

复制配置文件并修改

```
cd /opt/yapi
pwd
cp yapi-1.8.3/config_example.json ./config.json
vi config.json

```

原内容

```json
{
  "port": "3000",
  "adminAccount": "admin@admin.com",
  "db": {
    "servername": "127.0.0.1",
    "DATABASE": "yapi",
    "port": 27017,
    "user": "test1",
    "pass": "test1",
    "authSource": ""
  },
  "mail": {
    "enable": true,
    "host": "smtp.163.com",
    "port": 465,
    "from": "***@163.com",
    "auth": {
      "user": "***@163.com",
      "pass": "*****"
    }
  }
}

```

3.3

更改说明:

1. port : yapi 向外暴露的端口,默认3000
2. adminAccount: 系统管理员账号,密码默认: ymfe.org
3. closeRegister：禁止注册
4. user/pass：刚才mongodb设置的yapi的用户名和密码
5. mail：暂时关闭，设置为false

更改后:

```json
{
  "port": "12000",
  "adminAccount": "weibing@primeton.com",
  "db": {
    "servername": "127.0.0.1",
    "DATABASE": "yapi",
    "port": 27017,
    "user": "",
    "pass": "",
    "authSource": ""
  },
  "mail": {
    "enable": false,
    "host": "smtp.163.com",
    "port": 465,
    "from": "***@163.com",
    "auth": {
      "user": "***@163.com",
      "pass": "*****"
    }
  }
}

```

3.4 后面可以根据需求添加LADP配置

3.5 配置yapi依赖

```shell
cd /opt/yapi/yapi-1.8.3
npm install --production --registry https://registry.npm.taobao.org
npm run install-server //安装程序会初始化数据库索引和管理员账号，管理员账号名可在 config.json 配置
node server/app.js //启动服务器后，请访问 127.0.0.1:{config.json配置的端口}，初次运行会有个编译的过程，请耐心等候

```

3.6 安装后/opt/yapi目录结构如下

```
|-- config.json
|-- init.lock
|-- log
`-- yapi-1.8.3
    |-- CHANGELOG.md
    |-- LICENSE
    |-- README.md
    |-- client
    |-- common
    |-- config_example.json
    |-- doc
    |-- exts
    |-- nodemon.json
    |-- npm-debug.log
    |-- package.json
    |-- plugin.json
    |-- server
    |-- static
    |-- test
    |-- webpack.alias.js
    |-- yapi-base-flow.jpg
    |-- ydocfile.js
    `-- ykit.config.js

```



### 4 服务管理

使用pm2管理node服务

4.1 安装pm2

```shell
npm install -g pm2 --registry https://registry.npm.taobao.org

```

4.2 启动服务

```shell
cd /opt/yapi/yapi-1.8.3/server
pm2 start ./app.js

```

4.3 重启服务

```shell
cd /opt/yapi/yapi-1.8.3/server
pm2 restart app.js

```

4.4 停止服务

```
pm2 stop app

```

