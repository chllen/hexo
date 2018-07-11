---
title: CentOS安装NodeJS
date: 2018-07-11 14:53:09
categories:
- node.js
tags:
- node.js
- js
---

前言：
>在CentOS下安装NodeJS有以下几种方法。使用的CentOS版本为7.4。CentOS其他版本的NodeJS安装大同小异，也可以参看本文的方法。

### 直接部署

#### 首先安装wget
>yum install -y wget

#### 下载nodejs最新的bin包
可以在[下载页面](https://nodejs.org/en/download/)中找到下载地址,找到Linux Binaries (x64)版本地址。依次执行指令：
>cd /usr/local
mkdir node  
cd node  
wget https://nodejs.org/dist/v10.6.0/node-v10.6.0-linux-x64.tar.xz

### 解压包
>xz -d node-v10.6.0-linux-x64.tar.xz  
tar -xf node-v9.3.0-linux-x64.tar

### 部署bin文件
先确认你nodejs的路径，我这里的路径为/usr/local/node/node-v10.6.0-linux-x64/bin。确认后依次执行
>ln -s /usr/local/node/node-v10.6.0-linux-x64/bin /usr/bin/node  
ln -s /usr/local/node/node-v10.6.0-linux-x64/bin /usr/bin/npm  

注意：ln的源件名和连接文件名都是绝对路劲,路径不对会报-bash: /usr/bin/node: Too many levels of symbolic links的错误
ps:
用来为文件创件连接，连接类型分为硬连接和符号连接两种。  
符号链接：符号链接相当于windows下的快捷方式，这个你懂得吧。windows下的快捷方式，如果源文件删除了，快捷方式就没用啦。  
硬链接：是指当你建立了一个硬链接之后，就算有一个被删除了，用另一个也可以访问到你要访问的数据。也就是说源文件被删除了，用硬链接产生的文件也可以访问要访问的数据。符号链接是不行的哦，会报错。不信自己打上去试试。实践实践再实践。  

### 编译部署
#### 安装gcc，make，openssl，wget  
>yum install -y gcc make gcc-c++ openssl-devel wget  

#### 下载源代码包
同样的，你可以在下载页面https://nodejs.org/en/download/中找到下载地址,下载Source Code源码包。然后执行指令
>wget https://nodejs.org/dist/v10.6.0/node-v10.6.0.tar.gz

#### 解压源代码包
>tar -xf node-v9.3.0.tar.gz

#### 编译
进入源代码所在路径
>cd node-v9.3.0

先执行配置脚本  
>./configure

编译与部署
>make && make install  

接着就是等待编译完成。如果内存不够会报o(╥﹏╥)o

### 测试
>node -v  
npm  

如果正确输出版本号，则部署OK。/usr/lib –放置了node_modules，即nodejs的各种模块

### 升级node.js和npm
npm中有一个模块叫做“n”，专门用来管理node.js版本的。  
更新到最新的稳定版只需要在命令行中打下如下代码：
>npm install -g n  

在n的安装目录下，我这边的目录是 /usr/local/node/node-v10.6.0-linux-x64/bin

>./n v6.2.0
![](01.png)
如需最新版本则用./n latest 可以安装最新版本

当然，n后面也可以跟具体的版本号：./n v6.2.0  

版本切换
./n v10.6.0
不存在时安装，存在时切换






