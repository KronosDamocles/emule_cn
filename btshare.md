# BT快速共享

首先要有公网地址，IPv4或IPv6均可。国内的网络运营商现在都会下发 IPv6 公网地址给入网设备，各种设备也都能自动配置 IPv6。如果未能获得（以数字2开头的一串用冒号分隔的数字），请自行学习解决。

然后用BT软件比如 qbittorrent，开启 DHT（默认就是开的），制作种子文件 *.torrent，把种子的磁力链接（magnet）发给朋友即可。如果只有公网IPv6，朋友的下载机也需要有公网IPv6。

本地机和路由器的防火墙需要放行BT的监听端口。可以先尝试UPNP，不行的话手工设置防火墙规则或者端口映射。

