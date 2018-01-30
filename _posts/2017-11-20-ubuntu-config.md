---
layout: post
title: Ubuntu 配置
categories: Linux
description: 主要工作环境迁移到 Ubuntu 后，进行一些必要的配置，方便使用。
keywords: Linux, Ubuntu
---

安装完 Ubuntu 16.04 LTS 后进行一些必要的配置，方便使用。

## 将你的硬件时钟设置为本地时区

```sh
sudo timedatectl set-local-rtc 1
```

## 更改源为阿里云

在 应用程序 > 软件和更新 中更改源为阿里云

## 安装aptitude

```sh
sudo apt-get install aptitude
```

## 安装vim

```sh
sudo apt-get install vim-gnome
```

vim配置

```sh
vim ~/.vimrc
```

添加

```
" si 基于autoindent的一些改进
set smartindent
" ts 编辑时一个TAB字符占多少个空格的位置
set tabstop=4
" sw 使用每层缩进的空格数
set shiftwidth=4
" et 是否将输入的TAB自动展开成空格。开启后要输入TAB，需要Ctrl-V<TAB>
set expandtab
" 显示行号
set number

filetype indent on
filetype on
filetype plugin on

" 快速按下fd退出insert模式
inoremap fd <esc>
```

## 安装git

```sh
sudo apt-get install git
```

设置name，email，push

```sh
git config --global user.name <name>
git config --global user.email <email>
git config --global push.default simple
```

使用ssh，生成公钥后在github上添加SSH keys

```sh
ssh-keygen -t rsa -C <email>
```

## 安装zsh & oh-my-zsh

```sh
# 安装zsh
sudo apt-get install zsh

# 更改当前用户默认的shell为zsh
chsh -s $(which zsh)

# 安装oh-my-zsh
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

# 修改主题为agnoster
vim ~/.zshrc
# 主题改为如下
# ZSH_THEME="agnoster"

# 安装agnoster依赖的字体，否则会出现乱码
sudo apt-get install fonts-powerline

# 注销后生效
```

## 全局智能代理

方案: iptables白名单 + redsocks2 + shadowsocks-libev(可选)

新建脚本update-cn-rules.sh，用于更新国内白名单

```sh
#!/bin/sh

# -c 断点续传 -N 比较服务器文件时间戳
wget -c -N http://ftp.apnic.net/stats/apnic/delegated-apnic-latest

# 筛选
cat delegated-apnic-latest | awk -F '|' '/CN/&&/ipv4/ {print $4 "/" 32-log($5)/log(2)}' | cat > ./cn_rules.conf
```

编译redsocks2

```sh
git clone https://github.com/semigodking/redsocks.git

cd redsocks

# 编译前先安装依赖
# 注意，redsocks2依赖libevent2，OpenSSL or PolarSSL，但暂时不支持OpenSSL1.1
# 一些较新的发行版默认已经安装了libevent-2.1.6，而libevent-2.1.6依赖于OpenSSL1.1，所以默认OpenSSL也是1.1版本
# 在不降级OpenSSL的情况下没办法编译使用redsocks2，好在还有PolarSSL可选，需安装libpolarssl-dev

# 如果系统的OpenSSL是1.0版本，可以按如下命令安装依赖，然后直接make编译

sudo apt-get install libevent-dev libssl-dev
make

# 如果系统的OpenSSL是1.1版本，则改用PolarSSL编译，可以按如下命令安装依赖，然后编译

sudo apt-get install libevent-dev libpolarssl-dev
make USE_CRYPTO_POLARSSL=true

# 编译完成会生成redsocks2可执行文件
```

新建执行redsocks2的特殊用户

之所以不用当前用户执行，是因为后面iptables的规则需要过滤掉redsocks2本身外发的流量

```sh
sudo adduser --shell /bin/false --no-create-home reds2
```

启动命令

```sh
sudo -u reds2 ./redsocks2 -c redsocks.conf
```

redsocks.conf 配置文件

```
base {
    log_debug = off;
    log_info = off;
    //log = stderr;
    log = "file:/home/howie/proxy/redsocks.log";
    //log = "syslog:local7";
    daemon = on;
    redirector = iptables;
}

redsocks {
    local_ip = 127.0.0.1;
    local_port = 12345;
    ip = 127.0.0.1;
    port = 1080;
    type = socks5;
    //ip = <ss server ip>;
    //port = 8989;
    //type = shadowsocks;
    autoproxy = 1;
    timeout = 3;
    //login = "aes-256-gcm";
    //password = "xxxxxxxxxx";
}

autoproxy {
    no_quick_check_seconds = 3600;
    quick_connect_timeout = 1;
}

ipcache {
    cache_size = 4;
    cache_file = "/home/howie/proxy/ipcache.txt";
    stale_time = 7200;
    autosave_interval = 5;
    port_check = 1;
}
```

安装ipset，用于iptables匹配白名单

```sh
sudo apt-get install ipset
```

新建iptables-set.sh

```sh
#!/bin/sh

SOCKS_SERVER=<ss server ip> # SOCKS 服务器的 IP 地址

# Setup the ipset
ipset -N chnroute hash:net maxelem 65536

for ip in $(cat './cn_rules.conf'); do
      ipset add chnroute $ip
done

# 在nat表中新增一个链SHADOWSOCKS
iptables -t nat -N SHADOWSOCKS

# Allow connection to the server
iptables -t nat -A SHADOWSOCKS -d $SOCKS_SERVER -j RETURN

# Allow connection to reserved networks
iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN

# Allow connection to chinese IPs
iptables -t nat -A SHADOWSOCKS -p tcp -m set --match-set chnroute dst -j RETURN
# 如果你想对 icmp 协议也实现智能分流，可以加上下面这一条
# iptables -t nat -A SHADOWSOCKS -p icmp -m set --match-set chnroute dst -j RETURN

# Redirect to Shadowsocks
# 把1081改成你的shadowsocks本地端口
#iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-port 1080
# 如果你想对 icmp 协议也实现智能分流，可以加上下面这一条
# iptables -t nat -A SHADOWSOCKS -p icmp -j REDIRECT --to-port 1081

# 将SHADOWSOCKS链中所有的规则追加到OUTPUT链中
iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS
# 如果你想对 icmp 协议也实现智能分流，可以加上下面这一条
# iptables -t nat -A OUTPUT -p icmp -j SHADOWSOCKS

####################################################################

# redsocks2不要被重定向
iptables -t nat -I SHADOWSOCKS -p tcp -m owner --uid-owner reds2 -j RETURN

# tcp转到 redsocks2 端口
iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 12345
```

新建iptables-flush.sh

```sh
#!/bin/sh

# iptables -t nat -D OUTPUT -p icmp -j SHADOWSOCKS
iptables -t nat -D OUTPUT -p tcp -j SHADOWSOCKS
iptables -t nat -F SHADOWSOCKS
iptables -t nat -X SHADOWSOCKS
ipset destroy chnroute
```

由于redsocks2自带shadowsocks功能，所以可以不安装shadowsocks-libev，但redsocks2自带的ss不支持aes-256-gcm

有需要的可自行安装shadowsocks-libev，然后使用ss-local提供本地socks5代理

```sh
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev -y
sudo apt-get update
sudo apt-get install shadowsocks-libev
```

启动命令

```sh
# & 后台运行作业，使得终端不被占据;
# nohup(no hang up) 退出终端后仍在后台运行
# > ss-local.log 将command的输出重定向到指定文件中
# 2>&1 将标准错误重定向到标准输出
# -u 开启udp转发
nohup ss-local -c ss.json -u > ss-local.log 2>&1 &
#nohup ss-redir -c ss.json -u > ss-redir.log 2>&1 &
```

配置文件ss.json

```
{
    "server": "xx.xx.xx.xx",
    "server_port": 8989,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "xxxxxxxxxx",
    "timeout": 300,
    "method": "aes-256-gcm",
    "fast_open": false,
    "workers": 1
}
```
### 解决DNS污染

停止NetworkManager自带的dnsmasq

```sh
sudo vim /etc/NetworkManager/NetworkManager.conf
```

在dns=dnsmasq前加注释

```
#dns=dnsmasq
```

安装完整版dnsmasq

```sh
sudo apt-get install dnsmasq
```

配置dnsmasq

```sh
sudo vim /etc/dnsmasq.conf
```

在最后添加

```
no-resolv
no-poll
listen-address=127.0.0.1
cache-size=4096
#conf-dir=/etc/dnsmasq.d
# set upstream server to cdns port
#server=127.0.0.1#1053
# BAI DNS
server=106.14.152.170
```

修改设置中网络的IPv4首选DNS地址为127.0.0.1，然后查看是否修改成功

```sh
cat /etc/resolv.conf
```

BAI DNS属于私人DNS，目前暂未污染

必要时再启用dnsmasq-china-list + cdns，国内用国内dns，国外用opendns做上游

dnsmasq-china-list项目地址

[https://github.com/felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list)

下载以下2个文件，放到/etc/dnsmasq.d/目录下

```
accelerated-domains.china.conf
bogus-nxdomain.china.conf
```

可选google和apple部分中国可访问地址的配置

```
google.china.conf
apple.china.conf
```

可选自动更新，把下面的文件放到/usr/bin/目录下

```
dnsmasq-update-china-list
```

修改配置

```sh
sudo vim /etc/dnsmasq.conf
# 取消conf-dir前面的注释
#conf-dir=/etc/dnsmasq.d
```

重启dnsmasq

```sh
sudo service dnsmasq restart
```

编译cdns

```sh
git clone https://github.com/semigodking/cdns

cd cdns

# cdns依赖libevent2
# 使用CMake编译，先安装CMake

sudo apt-get install cmake

mkdir build
cd build
cmake ../
make
```

启动命令

```sh
./cdns -c cdns.json
```

配置文件cdns.conf

```
{
    "global": {
    // run as daemon
    "daemon": true,
    // where to send log to: syslog:daemon, stderr, file:/path/file
    "log": "syslog:daemon",
    // pid file
    //"pidfile": "/var/run/cdns.pid",
    // enable or disable debug info
    "log_debug": false
    },
    "cdns": {
        // local server listen address and port
        "listen_ip": "127.0.0.1",
        "listen_port": 1053,
        // Timeout for each DNS request
        "timeout": 2,
        // List of upstream DNS servers
        "servers": [
        {
            "ip_port": "208.67.220.220:443" //OpenDNS
        },
        {
            "ip_port": "106.14.152.170" // BAI DNS
        }
        ]
    }
}
```

新建clean.sh 用于清除dns缓存，重启网络

```sh
#!/bin/sh

sudo /etc/init.d/dns-clean start

sudo service dnsmasq restart

sudo service network-manager restart
```

## 局部代理

有了全局智能代理，没必要使用局部代理，这里只是记录一下，需shadowsocks配合

apt-get使用代理

```sh
vim apt_proxy_conf
```

添加

```
Acquire::http::proxy "http://127.0.0.1:1080/";
Acquire::ftp::proxy "ftp://127.0.0.1:1080/";
Acquire::https::proxy "https://127.0.0.1:1080/";
```

更新时使用配置代理

```sh
sudo apt-get update -c apt_proxy_conf
```

proxychain5代理

```sh
sudo apt-get install proxychains
```

配置

```sh
sudo vim /etc/proxychains.conf
```

添加

```
socks5  127.0.0.1 1080
```

## 安装Chrome

```sh
wget -q -O - https://raw.githubusercontent.com/longhr/ubuntu1604hub/master/linux_signing_key.pub | sudo apt-key add
sudo sh -c 'echo "deb [ arch=amd64 ] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
sudo apt-get update
sudo apt-get install google-chrome-stable
```

## 安装Redshift

```sh
sudo apt-get install redshift-gtk
```

Redshift项目地址

[https://github.com/jonls/redshift](https://github.com/jonls/redshift)

下载配置文件redshift.conf，放到如下目录

```
~/.config/redshift.conf
```

修改几个主要参数

```
; Set the day and night screen temperatures
temp-day=4000
temp-night=3000

; Disable the smooth fade between temperatures when Redshift starts and stops.
; 0 will cause an immediate change between screen temperatures.
; 1 will gradually apply the new screen temperature over a couple of seconds.
fade=0

[manual]
lat=22.5
lon=114.0
```

运行，在托盘图标勾选Autostart

## 小键盘灯默认开启

刚装完ubuntu默认是开启的，但是之后经过升级内核或者其他一些操作后有可能会失效，具体原因不明

```sh
sudo apt-get install numlockx
```

Gnome(GDM3)桌面暂时没办法解决，详见如下的bug报告

[https://bugs.launchpad.net/ubuntu/+source/gdm3/+bug/1727466](https://bugs.launchpad.net/ubuntu/+source/gdm3/+bug/1727466)

Unity(lightdm)桌面

```
sudo vim /etc/rc.local
```

在 exit 0 前添加

```
if [-x /usr/bin/numlockx ]; then
numlockx on
fi
```

lightdm登录界面小键盘灯默认开启

```sh
sudo vim /usr/share/lightdm/lightdm.conf.d/50-unity-greeter.conf
```

最后一行添加

```
greeter-setup-script=/usr/bin/numlockx on
```

## 安装dropbox

Add Dropbox’s repository key

```sh
sudo apt-key adv --keyserver pgp.mit.edu --recv-keys 5044912E
```

Add Dropbox’s repository

```sh
sudo sh -c 'echo "deb http://linux.dropbox.com/ubuntu $(lsb_release -sc) main" >> /etc/apt/sources.list.d/dropbox.list'
```

update and install Dropbox

```sh
sudo apt-get update
sudo aptitude install nautilus-dropbox
```

有可能系统中的包版本高于dropbox依赖的包，使用aptitude安装，选第二个方案，降低版本

start Dropbox deamon with -i(install) option

```sh
https_proxy="http://127.0.0.1:1080" dropbox start -i
```

## 安装jekyll

安装ruby开发包

```sh
sudo apt-get install ruby ruby-dev
```

安装jekyll和bundler

```sh
sudo gem install jekyll bundler
```

进入Gemfile所在目录，更新依赖

```sh
bundle install
```

过程可能会报错，我的具体情况是 commonmarker 依赖 cmake, nokogiri 依赖 zlib

```sh
sudo apt-get install cmake
sudo apt-get install zlib1g-dev  
```

启动本地服务

```sh
bundle exec jekyll serve --baseurl=""
```

## 安装virtualbox

```sh
sudo sh -c 'echo "deb http://download.virtualbox.org/virtualbox/debian xenial contrib" >> /etc/apt/sources.list.d/virtualbox.list'
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
sudo apt-get update
sudo apt-get install virtualbox-5.2
```

## 安装docker-ce

确保APT能使用https方式工作，并且CA证书已安装

```sh
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```

添加官方GPG密钥

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

添加REPO并更新

```sh
echo "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```

或者直接追加进sources.list中

```sh
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
```

更新

```sh
sudo apt-get update
```

安装docker-ce

```sh
sudo apt-get install docker-ce
```

Docker Hub加速

```sh
sudo vim /etc/docker/daemon.json
```

添加镜像加速地址

```
{
    "registry-mirrors": ["https://40fcvoar.mirror.aliyuncs.com"]
}
```

重启docker服务

```sh
sudo service docker restart
```

安装docker-machine

```sh
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && chmod +x /tmp/docker-machine && sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```


## 系统备份 & 还原

备份

```sh
# /media/howie/00081B2600000D13/ubuntu.tar 为备份文件所存放的位置，根据需求自己更改

sudo tar -zvcpf /media/howie/00081B2600000D13/ubuntu.tar.gz \
--exclude=/proc \
--exclude=/sys \
--exclude=/dev \
--exclude=/run \
--exclude=/mnt \
--exclude=/tmp \
--exclude=/media \
--exclude=/cdrom \
--exclude=/lost+found \
--exclude=/var/log \
--exclude=/var/cache/apt/archives \
--exclude=/home/*/.gvfs \
--exclude=/home/*/.cache \
--exclude=/home/*/.local/share/Trash \
/

# (可选)记录备份过程的log和error

> /media/howie/00081B2600000D13/ubuntu.log 2> /media/howie/00081B2600000D13/ubuntu.error
```

还原

通过live cd进入试用ubuntu

查看分区情况

```sh
lsblk
```

挂载备份文件所在分区，一般会自动挂载在media下，自行查看

```sh
sudo mount /dev/sda5 /mnt
```

删除旧系统所有数据

```sh
sudo rm -rf /mnt/*
```

挂载UFI分区

```sh
sudo mkdir -p /mnt/boot/efi
sudo mount /dev/sda1 /mnt/boot/efi
```

恢复

```sh
sudo tar -zxvpf /media/ubuntu-gnome/00081B2600000D13/ubuntu.tar.gz -C /mnt --numeric-owner
```

新建目录

```sh
cd /mnt
sudo mkdir proc sys dev run mnt tmp var/log var/cache/apt/archives
```
