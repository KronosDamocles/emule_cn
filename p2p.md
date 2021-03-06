# P2P文件分享

## 提要

P2P共享的概念，发展的两个阶段，DHT的原理，DHT骨干节点。

## 基本概念

Peer to peer file sharing，字面意思就是用户互相分享。
分享的对象未必是文件，只是文件占了绝大多数应用场景。
这一分享方式与传统的 服务器-客户端 模式有本质不同： **分享的文件不来自于服务器**。

## 发展阶段

在P2P分享模式发展的第一阶段，仍然依赖于服务器提供索引功能。
比如电驴服务器，提供两种索引：文件名的关键字到文件hash的对应关系；文件hash到供档者 IP地址:端口 的对应关系。
BT服务器，只提供上述第二种索引。这是BT与电驴的根本区别。

在P2P分享模式发展的第二阶段，对中心化服务器的依赖被彻底移除，服务器的功能由全体用户分担。
这种模式学术上称为 **D**istributed **H**ash **T**able, 即 DHT.
电骡对这种模式的实现被取名为 Kademlia ([论文链接](https://www.scs.stanford.edu/~dm/home/papers/kpos.pdf))。
BT就干脆直接借用了它的学术名 DHT，为避免概念混淆，下文称之为 BT-DHT.
电骡的Kad与BT-DHT原理相同，只是实现方式不同。

P2P分享模式现已发展到了第三阶段，即基于虚拟路由的匿名共享。例如： ipfs, freenet, kmule+i2p 等。
此类分享工具使用有一定门槛，用户基数太小。它们不在本社区关注范围内。

## DHT基本原理

DHT是一张由全体用户组织成的网格，每个用户都是网格里的一个节点。
每个节点有独特的ID，并直接连接若干个节点。每个节点都分担了一部分索引工作。
基本的索引有三个：

- 节点ID -> 节点IP:Port 的索引
- 文件hash -> 供档节点ID 的索引
- 关键字 -> 文件hash 的索引

当某个节点要查找某个文件hash的供档者时，它先检索自己的索引，如果没找到，就询问自己直连的节点，
被询问的节点也执行类似的操作，使检索行为在网格中逐渐扩散，直到得到结果（找到或失败）。
当文件hash与节点ID之间有固定的对应关系时，检索的扩散就具有明确方向了，
它会向 文件hash 与 节点ID 对应最紧密的方向递归检索，效率比广播式查找大大提高。
这一对应关系的最简单的形式，就是 hash = 节点ID。

## DHT骨干节点

在DHT里，有固定IP地址并且长时间在线的节点是非常重要的，它们有求必应，是节点间联系的枢纽，是P2P分享中执行检索操作的桥梁。
即使是低配的云主机，512M内存加几个G的月流量，不共享任何文件，只需开着电骡和BT，就足够承担索引的任务，对DHT的生态起到关键作用。
家用电脑和宽带网络，即使容量高，带宽大，由于IP地址变动会导致索引的变化，它们对DHT的贡献并不大。


