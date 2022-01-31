# Debian命令行跑amule

## 目录

- 前言
- 准备
- 安装
- 配置
- 运行

## 前言 

本文介绍Debian系Linux操作系统命令行环境下运行amule软件。其它发行版希望骡友补充。
有固定公网IP的节点在Kad网络中是很有价值的，即使不共享任何文件，也会承担文件源和关键字的索引功能，以及lowID伙伴(buddy)的功能。希望有Linux云主机的骡友一起云养骡，共同维护共享精神。 

## 准备 

首先得有基本的Linux命令行操作能力。对主机的配置要求很低，单核心512M内存10G存储盘这种低端的VPS就足够共享一些小体型文件了。需要有root权限。 

## 安装 

用root权限(含sudo,下同)执行这个命令：

    apt install amule-daemon amule-utils

其实就是安装那两个软件包。Daemon包里的核心程序是amuled，它于后台运行，执行电骡的全部功能。Utils包的核心程序是amulecmd，它是一个通过命令行来操控amuled的工具。

## 配置 

建议用普通账户运行amule. 除非有说明，下文中的命令都是用这个普通账户执行的，需要替换的文字用尖括号标注<>。
先创建两个目录：

    mkdir -p ~/amule/incoming
    mkdir -p ~/amule/temp

运行一下amuled:

    amuled

运行会报错退出，但是默认配置会生成在这个目录里： `~/.aMule`
进入 .aMule 目录：

    cd ~/.aMule

下载最新服务器和kad节点文件到当前目录里：

    wget "http://upd.emule-security.org/server.met"
    wget "http://upd.emule-security.org/nodes.dat"

生成密码的md5值：

    echo -n "<任意密码>" | md5sum -

输出结果类似于：
`d30d85deadf34dcb3745c594e1b809b8  -`

短横线前面那段就是密码的md5值，记下来备用。至于命令中用什么密码都无所谓，后面不会用到。 
下面修改amuled的配置文件 amule.conf 。不会命令行编辑的可以在本地修改好再上传。
以下三项是必须修改的设置：

    #接受外部连接必须是1，不然无法操控
    AcceptExternalConnections=1
    #骡子监听远程操控的地址
    ECAddress=127.0.0.1
    #远程登录密码的md5值，把前面记下的md5贴过来
    ECPassword=d30d85deadf34dcb3745c594e1b809b8

以下是一些常用的可选设置，可以不改，建议按实际情况设置。

    #设置昵称
    Nick=JanDoe
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
    TempDir=/home/<账户名>/amule/temp
    #下载完成后移入文件夹
    IncomingDir=/home/<账户名>/amule/incoming

改好配置后保存退回到命令行。
根据刚才改好的配置生成amulecmd的配置文件 remote.conf:

    amulecmd --create-config-from amule.conf 

~/.aMule 目录里的配置工作就完成了。
最后用root权限修改这个配置文件
`/etc/default/amule-daemon`:

    #用于跑骡子的普通账户名
    AMULED_USER="<账户名>"

保存后配置完成。 

## 运行 

用root权限运行：
启动amuled后台服务：

    systemctl start amule-daemon

查看后台服务是否在运行：

    systemctl status amule-daemon

查看日志：

    journalctl -u amule-daemon

异常退出后重启服务：

    systemctl restart amule-daemon

amule-daemon服务会随系统自动启动。 
用普通账户运行amulecmd来操作amuled:

    amulecmd

如果顺利进入命令行模式，说明amuled正常运行且可以用amulecmd来操作它。常用的命令有：

    help 帮助
    status 状态
    statistics 统计
    add 添加ed2k链接
    cancel 取消任务
    search 搜索
    exit 退出

具体用法自行查阅： `help <命令>`

------------------------------------------------------------------------

结束

2020年1月
