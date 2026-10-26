# 计算机网络配图 AI 重绘记录

2026-09-14 · 内置 imagegen · 白底、深色文字、蓝橙强调色。

14 张原图均保留，新图在正文原位置引用；其余 16 张配图及正文代码保持原样。

完整生成提示词、定点修正提示词、尺寸和 SHA-256 见 [重绘清单](redraw-manifest.json)。

| 新图 | 原图 | 内容调整 |
|---|---|---|
| [OSI 七层模型与 TCP/IP 四层模型](01-osi-tcpip-models.png) | [原图](../external-c276bd38aa148382.png) | 纠正 TCP/IP 四层模型底层名称为网络接口层。 |
| [数据逐层封装与解封装](02-data-encapsulation.png) | [原图](../external-fc201003c61cd06f.png) | 明确物理层与数据链路层；以 FCS 表示帧校验，避免帧尾和 CRC 重复。 |
| [单播、组播与广播对比](03-unicast-multicast-broadcast.png) | [原图](../external-57165229f821110b.png) | 以交换机和广播域呈现范围，避免原图将路由器与局域网广播混淆。 |
| [ARP 广播请求与单播应答](04-arp-request-reply.png) | [原图](../external-f00c4d0fb49f7191.png) | 使用同一局域网交换机示例，区分请求广播与应答单播。 |
| [以太网与 IPv4 场景下的 ARP 报文格式](05-arp-packet-format.png) | [原图](../external-48fe2bc15ddc1424.png) | 区分以太网 EtherType 0x0806 与 ARP 内 IPv4 协议类型 0x0800；删除无关端口图。 |
| [跨网络转发中的 IP 与 MAC 地址变化](06-router-forwarding.png) | [原图](../external-196f1bc780651af3.png) | 依正文地址变化表重构；保留 PC1 至 PC6 路径，去除原图不一致的路由表。 |
| [IPv4 地址与子网掩码按位与计算](07-subnet-mask-and.png) | [原图](../external-04f34677656392a6.png) | 移除原图“对外 IP”“隐藏主机号”等误导描述，保留原计算示例。 |
| [两个 IPv4 子网通过默认网关通信](08-default-gateway.png) | [原图](../external-ab65b6eeec3d0aae.png) | 改为正文中的 192.168.1.0/24 与 192.168.2.0/24，网关使用 .1 和 .254，纠正原图 .0 网关。 |
| [192.168.3.100/29 的子网划分计算](09-subnet-29.png) | [原图](../external-a4825092e19c0145.png) | 纠正原图掩码与按位与结果混排；补充 /29 范围，避免“隐藏主机号”描述。 |
| [IPv4 首部字段与位宽](10-ipv4-header.png) | [原图](../external-e717f2471d31029f.png) | 保留 IPv4 固定首部与可变部分，明确字段位宽及 IHL 单位。 |
| [TCP 三次握手与双方状态变化](11-tcp-three-way-handshake.png) | [原图](../external-a9f88955e38ba089.png) | 规范 TCP 状态名称与确认号，移除含糊的 API 阻塞时机和 ACK 的 ACK 说法。 |
| [TCP 四次挥手与 TIME-WAIT](12-tcp-four-way-close.png) | [原图](../external-bbb05223f7eac037.png) | 纠正原图序号递增与 CLOSE-WAIT 拼写，使用互相独立的 u、v、w 表示两端序列空间。 |
| [TCP 丢包后的三个重复 ACK 与快重传](13-tcp-fast-retransmit.png) | [原图](../external-071e3da919f69f3e.png) | 使用字节序号明确累计 ACK 的含义，准确画出三个重复 ACK 后重传。 |
| [慢开始、拥塞避免与超时后的窗口变化](14-tcp-congestion-control.png) | [原图](../external-ee79e60637e634e4.png) | 修正原图曲线数值错误，明确教学初始窗口、FlightSize 与超时假设。 |

## 核对依据

- [RFC 9293：TCP 状态与序列号](https://www.rfc-editor.org/rfc/rfc9293.html)

- [RFC 5681：快重传与拥塞控制](https://www.rfc-editor.org/rfc/rfc5681.html)

- [RFC 826：ARP 字段与请求应答](https://www.rfc-editor.org/rfc/rfc826.html)

## 验证

逐张查看生成结果并核对文字、数字和箭头；IPv4 标志字段边界、握手服务端 ESTABLISHED 位置已经过第二次 AI 修正。子网计算以 IPv4 按位运算复核。拥塞控制图采用明确标注的教学假设。
