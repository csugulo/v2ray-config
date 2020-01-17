# ssserver-conf

shadowsocks服务配置以及Debian/Ubuntu客户端配置，**请勿通过该服务发送垃圾邮件、爬取网页**。

- [ssserver-conf](#ssserver-conf)
  * [1. ssserver服务配置](#1-ssserver----)
  * [2. 安装配置proxychains/shadowsocks](#2-----proxychains-shadowsocks)
    + [2.1 安装](#21---)
    + [2.2 配置sslocal](#22---sslocal)
    + [2.3 配置proxychains](#23---proxychains)
    + [2.4 sslocal开机启动服务](#24-sslocal------)
    + [2.5 测试](#25---)
  * [3. git配置sock5代理](#3-git--sock5--)
  * [4. go get配置sock5代理](#4-go-get--sock5--)

## 1. ssserver服务配置

|server|port|password|method|机房|
|:-:|:-:|:-:|:-:|:-:|
|cloudcone-la-1.painitch.tech|3921|free|aes-256-cfb|洛杉矶|
|aliyun-hk-1.painitch.tech|3921|free|aes-256-cfb|香港|

## 2. 安装配置proxychains/shadowsocks

### 2.1 安装

```
sudo apt update
sudo apt install git proxychains shadowsocks -y
```

### 2.2 配置sslocal

添加sslocal配置文件/etc/shadowsocks/client.json

```
{
    "server":"aliyun-hk-1.painitch.tech",
    "server_port":3921,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"free",
    "timeout":300,
    "method":"aes-256-cfb"
}
```

### 2.3 配置proxychains

修改proxychains配置文件/etc/proxychains.conf
```
# 1. 注释dns转发
# Proxy DNS requests - no leak for DNS data
#proxy_dns
# 2. 将ProxyList中的sock4修改为sock5并修改端口
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  127.0.0.1 1080
```

### 2.4 sslocal开机启动服务

添加sslocal开机启动服务/etc/systemd/system/shadowsocks.service

```
[Unit]
Description=Shadowsocks Client Service
After=network.target
[Service]
Type=simple
User=root
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks/client.json
[Install]
WantedBy=multi-user.target
```

启用服务并重启计算机

```
sudo systemctl enable /etc/systemd/system/shadowsocks.service
sudo reboot
```

### 2.5 测试

```
proxychains curl https://api.ipify.org/?format=json
curl https://api.ipify.org/?format=json
```

## 3. git配置sock5代理

```
git config --global http.proxy 'socks5://127.0.0.1:1080'
# 取消代理
git config --global --unset http.proxy
```

## 4. go get配置sock5代理

添加bash脚本~/bin/go-get-proxy

```
#!/bin/bash
export https_proxy=socks5://127.0.0.1:1080
export http_proxy=socks5://127.0.0.1:1080
go get -u -v $@
unset https_proxy
unset http_proxy
```

为脚本增加执行权限

```
chmod +x ~/bin/go-get-proxy
```

将~/bin文件夹路径加入PATH环境变量

```
# 若默认shell为bash，则修改~/.bashrc
echo 'export PATH=$HOME/bin:/usr/local/bin:$PATH' | tee -a ~/.zshrc
source ~/.zshrc
```

测试

```
go-get-proxy golang.org/x/net golang.org/x/crypto
```


