# Debian云主机跑Transmission

以下所有命令都以普通用户身份运行。若不会设置sudoers，也可以用root运行，但是不推荐。

安装BT客户端：

    sudo apt install transmission-daemon

安装完成后，transmission会以预设的用户名运行于后台。我们来改成用自己的普通用户名运行。

先停止transmission：

    sudo systemctl stop transmission-daemon

用root身份修改 `/lib/systemd/system/transmission-daemon.service` 文件：

    #将user配置改成自己的用户名
    user=<用户名>

保存后，刷新配置：

    sudo systemctl daemon-reload

启动transmission服务：

    sudo systemctl start transmission-daemon

启动后默认的配置会自动生成。再停止服务：

    sudo systemctl stop transmission-daemon

现在修改默认配置。先建一个bt文件夹用于存放下载和共享的文件：

    mkdir ~/bt

修改配置文件 `~/.config/transmission-daemon/settings.json`  
以下配置可以酌情修改：

下载的文件存放在这个文件夹：

    "download-dir": "/home/<用户名>/bt",

#监听端口：

    "peer-port": 16413,

远程控制监听地址：

    "rpc-bind-address": "127.0.0.1",

下载限速，单位是 K字节每秒：

    "speed-limit-down": 400,

是否开启下载限速：

    "speed-limit-down-enabled": true,

上传限速：

    "speed-limit-up": 800,

是否开启上传限速：

    "speed-limit-up-enabled": true,

保存后启动服务：

    sudo systemctl start transmission-daemon

以下是操控transmission后台服务的常用命令：

查看服务运行情况：

    transmission-remote -si

添加磁力：注意要有引号

    transmission-remote -a '磁力链接'

查看下载情况：

    transmission-remote -l

查看某个文件的下载上传节点： \<id\> 为上面 -l 命令显示的文件编号

    transmission-remote -t <id> -ip

查看某个文件的详情：

    transmission-remote -t <id> -i

删除某个文件的共享和下载：注意如果下载已完成，该命令只会取消共享而不会删除文件

    transmission-remote -t <id> -r

共享某个文件：

先把文件复制到前面建的共享文件夹 `~/bt`：

    cp <文件路径> ~/bt

然后制作种子文件：

    transmission-create ~/bt/<文件名>

检查一下种子文件：

    transmission-show ~/bt/<文件名.torrent>

记下此命令输出的磁力链接。

添加种子到后台：

    transmission-remote -a ~/bt/<文件名.torrent>

此时transmission-daemon会先检验新添加的种子，然后共享就开始了，
可用命令 `transmission-remote -l` 查看。共享开始后种子文件就没用了，可以删除。

如果不记得磁力链接，可以用命令查看：

    transmission-remote -t <id> -i

把磁力链接发给朋友即可。

