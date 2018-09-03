---
title: shadowsocks-libev带simple-obfs混淆的客户端安装
date: 2018-08-29 23:33:26
categories:
- 效率
tags:
- 科学上网
- 效率
- shadowsocks
---

没想到也不能免俗写了一篇科学上网的教程，主要是最近新购入一台主机，连接docker hub和Google的一堆东西不科学上网不行，所以又折腾了几十分钟，为了避免下次重装系统之类的又要重新研究，记录一下步骤吧。

这篇教程仅介绍 Ubuntu 18.04 上客户端的安装，至于服务端，根据相关法律法规，不予显示。

记得跟上次安装比起来，现在客户端的安装步骤简单了不少，基本是傻瓜式的。

### 安装 shadowsocks-libev

[shadowsocks-libev仓库地址](https://github.com/shadowsocks/shadowsocks-libev)

```bash
sudo apt update
sudo apt install shadowsocks-libev
```

### 安装 simple-obfs 插件

[simple-obfs仓库地址](https://github.com/shadowsocks/simple-obfs)

```bash
sudo apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libev-dev asciidoc xmlto automake
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
sudo make install
```

### 配置文件

在` /etc/shadowsocks-libev`目录新增一个 json 文件，例如`obfs.json`，格式如下，照着服务端的配置改就行了，除了`local_address`和`local_port`可以自定义，其他都按服务端的配置来。

```
{
    "server":"11.111.222.33",
    "server_port":443,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"xcvasdfwvcx",
    "timeout":600,
    "udp_timeout":600,
    "method":"chacha20-ietf-poly1305",
    "mode":"tcp_and_udp",
    "fast_open":true,
    "plugin":"obfs-local",
    "plugin_opts":"obfs=tls;obfs-host=itunes.apple.com",
    "workers":1024
}
```

### 启动 ss-client

先在前台运行，如果没问题后面再用守护进程启动，执行下面这条命令后另开一个窗口执行后续步骤。

```bash
/usr/bin/ss-local -c /etc/shadowsocks-libev/obfs.json 
```

### 安装 privoxy

上面的命令启动的是一个socks代理，很多地方不方便使用，用privoxy中转一层，变成http代理，并且可以做智能分流。

```bash
sudo apt install privoxy
curl -skL https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy -o gfwlist2privoxy
bash gfwlist2privoxy '127.0.0.1:1080' # 注意和前面的 local_address 和 local_port 保持一致
sudo mv -f gfwlist.action /etc/privoxy
echo 'actionsfile gfwlist.action' | sudo tee -a /etc/privoxy/config
sudo systemctl restart privoxy
```

执行上述命令以后，privoxy会监听`127.0.0.1:8118`端口做http代理，如果还需要给内网的其他机器使用，修改`/etc/privoxy/config`文件，监听`0.0.0.0:8118`端口，用`systemclt restart privoxy`重启即可。如果发现需要走代理的网站没走代理，编辑`/etc/privoxy/gfwlist.action`文件，按语法添加规则，~~同样是重启后生效~~无需重启就可生效。

### 验证

安装完以后，可以配置环境变量进行验证了。

```bash
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
curl www.google.com
```

### 开机自启动

privoxy安装后默认开机自启动，ss-local可以添加systemd自启动服务

```bash
# @ 后面紧跟 /etc/shadowsocks-libev/ 目录下的json，文件名，例如下面是开机自动读取 obfs.json 文件
sudo systemctl enable shadowsocks-libev-local@obfs 
```

> 祝畅游愉快，少看~~PornHub~~Youtube

