# 连接Kad

如果骡子没能自动连上 Kad，绝大多数原因是没有有效的联系人（Kad节点）。可以尝试以下三个方法。

1. 找一个热门资源下载。在下载的过程中骡子会自动找到其它骡子作为 Kad 联系人。

2. 下载 Kad 节点文件，放到骡子的 config 文件夹里，然后重启骡子。
   下载地址： [http://upd.emule-security.org/nodes.dat][2] .
   或者直接在骡子里添加这个链接： [ed2k://|nodeslist|http://upd.emule-security.org/nodes.dat|/][1]

3. 手工填入一个 Kad 节点的 IP 地址和端口。可以到社区讨论群里索要。注意该节点必须是 HighID。

一篇更详细的教程： [https://emule.ironward.org/conn.html][3]

[1]: ed2k://|nodeslist|http://upd.emule-security.org/nodes.dat|/
[2]: http://upd.emule-security.org/nodes.dat
[3]: https://emule.ironward.org/conn.html
