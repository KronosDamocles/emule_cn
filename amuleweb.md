# 浏览器远程操控amuled

amuleweb 是 amule 自带的远程操控工具，它既是amule客户端，又是网页服务器，为用户提供操作amuled的网页界面。
通信的结构图如下：

    浏览器 --> amuleweb --> amuled

关于 amuled 的架设请看 [Debian命令行跑amule](./amule.md)

**警告**：amuleweb 对通信内容没有加密，用户名、密码、共享内容等都是明文发送的，所以仅适合在本地局域网内使用。
如果需要跨 Internet 使用，需要用其它网页服务器提供安全连接并反向代理amuleweb流量。

## 本地或局域网内使用 amuleweb

在 amule 配置文件 `/home/<用户名>/.aMule/amule.conf` 里，
找到 `[WebServer]` 段落，修改以下配置：

    Enabled=1
    Password=<密码的MD5值>
    Port=<端口号>

其中密码的MD5可用以下命令获得：

    echo -n "amuleweb的密码" | md5sum | cut -d ' ' -f 1

端口号是 amuleweb 的网页服务监听端口，任意未占用的端口即可。如果 amule 是用普通账户运行的，端口号必须大于 1024.

保存配置后，重启 amule-daemon 服务：

    sudo systemctl restart amule-daemon

现在去浏览器里输入 "`http://localhost:<端口号>/amuleweb/`" 应该就能进入 amuleweb 了。

## amuleweb 通信加密

跨互联网直接访问 amuleweb 是不推荐的，因为通信内容没有加密。为了给 amuleweb 的流量加密，我们需要架设一个 HTTPS 网站，然后把 /amuleweb 路径的流量反向代理给 amuleweb 的监听端口。
购买域名和证书，架设 HTTPS 网站的具体步骤就不介绍了，网上很多教程。本文只给出用 caddy 做反代的配置如下：

    yourdomain.com {
      ......
      reverse_proxy /amuleweb* 127.0.0.1:4682
      ......
    }

其中 127.0.0.1:4682 是 amuleweb 的服务地址和端口。如果 amuleweb 不在caddy本地，请把 127.0.0.1 替换成 amuleweb 的 IP.

现在，用浏览器访问 amuleweb 的地址： "`https://yourdomain.com/amuleweb/`"

## 禁止Internet直连amuleweb

amuleweb 不支持指定监听的IP地址，它始终监听本机所有的IP(0.0.0.0)。为安全起见，应禁止从Internet直接访问amuleweb的监听端口。
一般的防火墙软件都能做到这一点。这里给出 iptables 的设置命令。

    iptables -A INPUT -d <公网IP> -p tcp --dport 4682 -j DROP

其中 <公网IP> 是amuleweb所在机器的公网IPv4地址。

**提示**：为了使 iptables 的配置在重启机器后仍然有效，需要安装 iptables-persistent 软件包。
