# P2P 传输

# NAT（Network Address Translator）

NAT 主要做的事情就是地址映射，其作用就是将内网的主机经过 NAT 的映射生成外网的 ip 地址和端口，如下图所示：

![NAT 图解](https://assets.ng-tech.icu/item/D9GAxJ.md.png)

NAT 产生的原因如下：IPV4 的地址不够、处于安全考虑，保证某局域网下的设备的安全。NAT 的种类有以下几种：

- 完全锥型 NAT：所有从同一个内网 IP 和端口号发送过来的请求都会被映射成同一个外网 IP 和端口号，并且任何一个外网主机都可以通过这个映射的外网 IP 和端口号向这台内网主机发送包。
- 地址限制锥型 NAT：它也是所有从同一个内网 IP 和端口号发送过来的请求都会被映射成同一个外网 IP 和端口号。与完全锥形不同的是，外网主机只能够向先前已经向它发送过数据包的内网主机发送包。
- 端口限制锥型 NAT：端口限制锥形 NAT 与限制锥形 NAT 很相似，只不过它包括端口号。也就是说，一台 IP 地址 X 和端口 P 的外网主机想给内网主机发送包，必须是这台内网主机先前已经给这个 IP 地址 X 和端口 P 发送过数据包。
- 对称型 NAT：所有从同一个内网 IP 和端口号发送到一个特定的目的 IP 和端口号的请求，都会被映射到同一个 IP 和端口号。如果同一台主机使用相同的源地址和端口号发送包，但是发往不同的目的地，NAT 将会使用不同的映射。此外，只有收到数据的外网主机才可以反过来向内网主机发送包。

# STUN（Session Traversal Utilities for NAT）

STUN 即 NAT 会话穿透实用工具[RFC5389]，用于进行 UAT 穿透，采用的是典型的客户端/服务端模式。客户端发送请求，服务端进行响应。STUN 是一个 C/S 架构的协议，支持两种传输类型。一种是请求/响应类型，由客户端给服务端发送请求并等待服务端响应；另一种是指示类型，由服务器或者客户端发送指示，另一方不产生响应。两种类型的传输都包含一个 96 位的随机数作为事务 ID（transaction ID），对于请求/响应类型，事务 ID 允许客户端将响应和产生响应的请求连接起来；对于指示类型，事务 ID 通常作为 debugging aid 使用。

STUN 报文和大多数网络类型的格式一样，是以大端编码(big-endian)的，即最高有效位在左边。所有的 STUN 报文都以 20 字节（160 位）的头部开始，后面跟着若干个属性。

## STUN 报文头部

STUN 头部包含了 STUN 消息类型，magic cookie，事务 ID 和消息长度，如下：

```sh
   0                   1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |0 0|     STUN Message Type     |         Message Length        |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                         Magic Cookie                          |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
  |                                                               |
  |                     Transaction ID (96 bits)                  |
  |                                                               |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

# Links

- WebRTC 传输协议 https://www.cnblogs.com/renhui/p/6387118.html
