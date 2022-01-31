# Debian运行Aria2后台服务

Aria2在默认设置下运行时，会启动下载并在完成后退出。Aria2也支持以后台服务的模式在Linux系统下运行，提供 XML RPC 和 JSON RPC 接口供前台程序进行操控。本文以 Debian Linux 系统为例，简单介绍下这种玩法。

请将下文中的 <用户名> 替换为你的实际用户名。请将 vi 命令替换成你喜欢的文本编辑器，比如 nano。

先安装软件：

    sudo apt install aria2 curl jq

建一个放配置的文件夹：

    mkdir /home/<用户名>/.aria2 

再建一个放下载的文件夹：

    mkdir /home/<用户名>/aria2

创建一个空的进度文件：

    touch /home/<用户名>/.aria2/session

创建后台服务的配置文件：

    vi /home/<用户名>/.aria2/aria2.conf

内容是如下：

    #下载文件夹
    dir=/home/<用户名>/aria2
    #启动后从这里读取进度
    input-file=/home/<用户名>/.aria2/session
    #日志文件
    log=/home/<用户名>/.aria2/aria2.log
    #这个太难解释了—。—
    bt-detach-seed-only=true
    #BT加载元数据
    bt-load-saved-metadata=true
    #BT保存元数据到torrent文件
    bt-save-metadata=true
    #BT做种前不验证以前下载的文件的完整性
    bt-seed-unverified=true
    #IPv4 DHT节点文件
    dht-file-path=/home/<用户名>/.aria2/dht.dat
    #IPv6 DHT节点文件
    dht-file-path6=/home/<用户名>/.aria2/dht6.dat
    #UDP监听端口
    dht-listen-port=50414
    #开启IPv6 DHT
    enable-dht6=true
    #TCP监听端口
    listen-port=50414
    #上传限速
    max-overall-upload-limit=800K
    #分享率达到指定值后停止做种
    seed-ratio=0.0
    #允许RPC远程操控
    enable-rpc=true
    #远程操控RPC监听端口
    rpc-listen-port=6800
    #以后台服务模式运行
    daemon=true
    #文件分配方法
    file-allocation=falloc
    #强制保存
    force-save=true
    #日志等级
    log-level=notice
    #下载限速
    max-overall-download-limit=400K
    #进度保存在这个文件
    save-session=/home/<用户名>/.aria2/session
    #配置结束。更多选项请查官方文档。

创建后台服务描述文件：

    sudo vi /usr/local/lib/systemd/system/aria2.service

内容如下：

    [Unit]
    Description=Aria2
    After=network-online.target
    [Install]
    WantedBy=multi-user.target
    [Service]
    User=<用户名>
    ExecStart=aria2c
    Type=forking

配置就到这里。下面安装后台服务：

    sudo systemctl enable aria2

启动服务：

    sudo systemctl start aria2

现在Aria2就在后台运行了。用 curl 测试一下RPC：

    curl -H 'Content-Type: application/json' -d '{"jsonrpc": "2.0", "id": "1", "method": "aria2.tellActive", "params": []}' 'http://[::1]:6800/jsonrpc' | jq

末尾的 jq 命令目的是将结果整理得易读。命令太复杂，搞个简易的脚本：

    sudo vi /usr/local/bin/adm.sh

内容为：

    #!/bin/bash
    function help () {
      echo "adm.sh is a helper script to interact with aria2 daemon through JSON RPC. It calls curl to pass method and arguments to aria2."
      echo "Usage: adm.sh <method> <arg1>,<arg2>,... <jqs>"
      echo "       adm.sh -h|--help"
      echo "Some useful methods and args:"
      echo "  aria2.addUri        [<uri>]"
      echo "  aria2.remove        <gid>"
      echo "  aria2.tellStatus    <gid>"
      echo "  aria2.getPeers      <gid>"
      echo "  aria2.tellActive"
      echo "  aria2.tellStopped   <m>,<n>"
      echo "  aria2.getGlobalStat"
      echo "  aria2.shutdown"
      echo "Default is tellActive if you run me without argument."
    }
    [ "$1" == "-h" ] || [ "$1" == "--help" ] && {
      help
      exit 0
    }
    RPC_METHOD=$1
    RPC_PARAMS="$2"
    JQ_ARG="$3"
    UUID=$(cat /proc/sys/kernel/random/uuid)
    [ $RPC_METHOD ] || RPC_METHOD=aria2.tellActive
    curl -H 'Content-Type: application/json' -d '{"jsonrpc":"2.0", "id":"'$UUID'", "method":"'${RPC_METHOD}'","params":['${RPC_PARAMS}']}'  'http://[::1]:6800/jsonrpc' | jq "$JQ_ARG"

修改下权限：

    sudo chmod 755 /usr/local/bin/adm.sh

查看使用方法：

    adm.sh -h
   
