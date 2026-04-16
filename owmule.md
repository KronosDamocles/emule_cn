# OpenWrt跑电骡

## 目录

- 前言
- 准备
  - 硬件设备
  - 预备知识
  - U盘挂载
  - 安装Entware
  - 创建新用户
- 安装
- 配置
- 运行
- 重启
- 鸣谢

## 前言

本文介绍在 [OpenWrt](https://openwrt.org/) 系统里跑 [aMule](https://amule.org/) 软件。
OpenWrt玩法很多，各种修改版五花八门，本文仅提供一种利用官方 OpenWrt + [Entware](https://entware.net/) 并用普通权限用户跑骡子的玩法。

先劝退一波：极度不建议在主路由器里跑电骡，路由器就让它仅仅完成路由和防火墙的功能就好，没必要加码其它应用，效率低负担重风险高。
网页，数据库，流媒体，电骡BT等等都放到下级设备里更安全易用。

不玩不舒服的老司机请继续。。。

## 准备

### 硬件设备

- 能插U盘，且已经安装好OpenWrt操作系统的路由器或其它设备。OpenWrt系统的安装就不在本文范围内了。
- U盘一只，用来创建文件系统挂载到 /opt 安装Entware.

### 预备知识

您至少须具备Linux基础知识，并且能通过`ssh`登陆OpenWrt.
除非有特别说明，本文所有命令都用*root*身份执行，命令中需要替换的部分用尖括号`<>`标注。

OpenWrt从25.12版本起改用 [apk](https://wiki.alpinelinux.org/wiki/Alpine_Package_Keeper)
作为软件包管理器，本文中OpenWrt的软件安装均用`apk`操作，
未升级的 [opkg](https://openwrt.org/docs/guide-user/additional-software/opkg) 系统请自行升级。
Entware目前仍然使用旧的opkg包管理器，电骡相关工具都用`opkg`安装。

国内OpenWrt镜像仓库可极大提高软件下载速度，下面换用 [科大镜像](https://mirrors.ustc.edu.cn/help/openwrt.html)：

    cd /etc/apk/repositories.d
    cp distfeeds.list distfeeds.bak
    sed -i 's/downloads.openwrt.org/mirrors.ustc.edu.cn\/openwrt/g' \
        distfeeds.list

### U盘挂载

先来安装一些软件：

    apk update
    apk add block-mount e2fsprogs fdisk \
        kmod-fs-ext4 kmod-usb-storage-uas

安装完毕后插入U盘，系统会自动识别，通常为 `/dev/sda`.
然后创建分区：

    fdisk /dev/sda

此时会进入`fdisk`操作界面，常用操作有：

- `g` - 创建gpt分区表
- `n` - 新建分区，分区类型选 83 Linux
- `w` - 保存并退出
- `q` - 不保存退出

分区大小不用填，默认是一个分区占据全部空间，通常为 `/dev/sda1`. 然后创建文件系统：

    mkfs.ext4 /dev/sda1

创建挂载点并挂载文件系统：

    mkdir /opt
    mount -t ext4 /dev/sda1 /opt

在 `/etc/config/fstab` 里添加以下段落以使系统在启动后自动挂载：

    config 'mount' 
        option target '/opt' 
        option device '/dev/sda1'

可以重启一下确保自动挂载正常。

### 安装Entware

用 `uname -m` 命令查看一下设备的处理器架构，然后去
[这个页面](https://github.com/Entware/Entware/wiki/Alternative-install-vs-standard)
下载对应的安装脚本。
绝大多数路由器应该用 `https://bin.entware.net/armv7sf-k3.2/installer/generic.sh`.
然后安装 Entware, 安装过程中软件下载有点慢，耐心等待：）

    sh generic.sh

安装完成后按如下配置Entware.

创建文件 `/etc/profile.d/99-local.sh` 内容为：

    [ -r /opt/etc/profile ] && . /opt/etc/profile

换用Entware [北外镜像](https://mirrors.bfsu.edu.cn/help/entware/):

    cp /opt/etc/opkg.conf /opt/etc/opkg.conf.bak
    sed -i -e 's/http:/https:/g' \
        -e 's/bin.entware.net/mirrors.bfsu.edu.cn\/entware/g' \
        /opt/etc/opkg.conf

编辑 `/opt/etc/init.d/rc.unslung`, 在 `ACTION=$1` 这一行的下面加上两行:

    [ "$ACTION" = 'boot' ] && ACTION=start
    [ "$ACTION" = 'shutdown' ] && ACTION=stop

让系统启动/关机时执行Entware里安装的服务：

    ln -s /opt/etc/init.d/rc.unslung /etc/rc.d/S99rc.unslung
    ln -s /opt/etc/init.d/rc.unslung /etc/rc.d/K00rc.unslung

Entware的安装与配置到此结束。

### 创建新用户

用root身份跑骡子是强烈不推荐的。现在创建一个新用户 `janedoe` 专门跑骡子。

在 `/etc/passwd` 的末尾加一行：

    janedoe:x:1000:1000:janedoe:/opt/home/janedoe:/bin/sh

在 `/etc/group` 的末尾加一行：

    janedoe:x:1000:

创建文件夹并修改权限：

    mkdir /opt/home/janedoe
    chown janedoe:janedoe /opt/home/janedoe
    chmod 750 /opt/home/janedoe

安装 sudo:

    apk add sudo

准备工作到此结束。

## 安装

在Entware里安装 amule:

    opkg update
    opkg install amule

这会安装三个重要的程序：
- `amuled` - 电骡后台服务，执行骡子全部功能；
- `amulecmd` - 电骡命令行操控工具；
- `amuleweb` - 电骡网页操控工具。

## 配置

先用下面的命令切换到 `janedoe` 账户：

    sudo -u janedoe -i

然后创建两个目录：

    mkdir -p ~/amule/incoming
    mkdir -p ~/amule/temp

运行一下amuled:

    amuled

运行会报错退出，但是默认配置会生成在这个目录里： `~/.aMule`.  
进入 .aMule 目录：

    cd ~/.aMule

下载最新服务器和kad节点文件到当前目录里：

    wget -O server.met "http://upd.emule-security.org/server.met"
    wget -O nodes.dat "http://upd.emule-security.org/nodes.dat"

生成密码的md5值：

    echo -n '<电骡密码>' | md5sum | cut -d ' ' -f 1

输出结果类似于：
`d30d85deadf34dcb3745c594e1b809b8`  
记下来备用。  
下面修改amuled的配置文件 amule.conf 。以下三项是必须修改的设置：

    #接受外部连接必须是1，不然无法操控
    AcceptExternalConnections=1
    #骡子监听远程操控的地址
    ECAddress=127.0.0.1
    #远程登录密码的md5值，把前面记下的md5贴过来
    ECPassword=d30d85deadf34dcb3745c594e1b809b8

以下是一些常用的可选设置，可以不改，建议按实际情况设置。

    #设置昵称
    Nick=JaneDoe
    #最大下载速度，单位KBytes/s，下同。
    MaxDownload=200
    #最大上传速度
    MaxUpload=400
    #每个上传线程的预计速度。这个数值并不限制每个线程的速度上限，
    #而是用于计算上传线程数=MaxUpload/SlotAllocation  
    SlotAllocation=100
    #TCP监听端口
    Port=4662
    #UDP监听端口
    UDPPort=4662
    #从当前服务器自动更新服务器列表。最好关掉
    AddServerListFromServer=0
    #是否连接服务器。我通常关掉
    ConnectToED2K=1
    #临时文件夹，前面创建过
    TempDir=/opt/home/janedoe/amule/temp
    #下载完成后移入文件夹
    IncomingDir=/opt/home/janedoe/amule/incoming
    #下面是amuleweb的配置
    [WebServer]
    #是否开启网页操控功能
    Enabled=1
    #远程登录密码的md5值，把前面记下的md5贴过来
    Password=d30d85deadf34dcb3745c594e1b809b8
    #网页服务监听端口
    Port=4711

改好配置后保存退回到命令行。
根据刚才改好的配置生成amulecmd的配置文件 remote.conf:

    amulecmd --create-config-from=amule.conf 

~/.aMule 目录里的配置工作就完成了。
输入 `exit` 退回到root身份.

为了使系统以普通用户身份自动启动amuled，须编辑 `/opt/etc/init.d/S57amuled`, 修改以下三项设置：

    PROCS=/opt/bin/amuled
    ARGS=""
    PREARGS="sudo -u janedoe"

## 运行

启动和停止 amuled, 注意这些命令仍旧需要用root身份执行，但骡子是以 janedoe 身份运行的。

    /opt/etc/init.d/S57amuled start
    /opt/etc/init.d/S57amuled stop

上面两个命令会在系统启动/关机时自动执行。若不想让电骡随系统启动，执行：

    cd /opt/etc/init.d && mv S57amuled K42amuled

检查 amuled 是否在运行：

    /opt/etc/init.d/S57amuled check

用 `amulecmd` 命令来操控amuled.  
如果用 janedoe 身份执行amulecmd，会直接进入操作界面，不需要密码。
用其他身份执行amulecmd会要求输入密码。常用的命令有：

    help 帮助
    status 状态
    statistics 统计
    add 添加ed2k链接
    cancel 取消任务
    search 搜索
    exit 退出

具体用法自行查阅： `help <命令>`

若想跳过密码，比如用root身份跑amulecmd不想让它提示敲密码，需要将janedoe的remote.conf复制过来：

    mkdir /root/.aMule
    cp /opt/home/janedoe/.aMule/remote.conf /root/.aMule

## 远程

**警告**：amule的远程通信内容没有加密，用户名、密码、共享内容等都是明文发送的，仅适合在本地局域网内使用。
同时，要确保主路由器的防火墙挡住了从外网到amule远程操控端口的连接，默认端口为 4711 和 4712。

如果在前面的配置中开启了 WebServer, 在浏览器里输入电骡的地址和端口：

    http://<IP地址>:4711/amuleweb/

输入密码后点Submit按钮就能进入操控界面了。

用 amulecmd 也可以远程，需要先把前面的 `ECAddress=` 配置项目留空，然后重启电骡使其生效。
从另一台设备上用amulecmd远程登陆电骡：

    amulecmd -h <IP地址> -p 4712

*提示*：用 `-P ''` (大写字母P传空值参数)可以强制amulecmd提示输入密码。
更多参数及用法请看 `amulecmd --help`.
amulecmd的配置也可以放到 `remote.conf` 中，命令行参数的优先级高于配置文件。

用 `amulegui` 也可以远程，图形界面留给读者自行研究。

## 重启

amuled一直存在内存占用逐渐升高的问题，怀疑有内存泄漏。
一个简单的解决方法是用一个定时任务每天/周自动重启amuled.
下面介绍利用 cron 的定时任务自动重启amuled的设置方法。

创建一个文本文件 `/opt/bin/amuled-recycle`, 内容如下。
这段脚本的意思是，若电骡在运行则重启它。

    #!/bin/sh
    [ -x /opt/etc/init.d/S57amuled ] && \
    /opt/etc/init.d/S57amuled check >/dev/null 2>&1 && \
    /opt/etc/init.d/S57amuled restart >/dev/null 2>&1
    true

修改权限为可执行文件：

    chmod 755 /opt/bin/amuled-recycle

设置 `crontab` 在每天5点执行上面的脚本:

    crontab -e

在编辑界面输入以下一行内容，然后保存退出。

    0 5 * * * /opt/bin/amuled-recycle

以上即是设置每天5点重启电骡。若只在每周日5点重启，请将上面第三颗 `*` 改成 `0`.
其它定时设定请查阅 [crontab说明文档](https://pubs.opengroup.org/onlinepubs/007904975/utilities/crontab.html).

## 鸣谢

本文初版 [OpenWRT跑电骡教程](https://telegra.ph/%E5%9C%A8-OpenWRT-%E4%B8%8A%E4%BD%BF%E7%94%A8-aMule-04-01)
由群友 Abola 发表于 telegraph. 本文进行了大量增补与修改。

---
结束 2026年3月
更新于 2026年4月
