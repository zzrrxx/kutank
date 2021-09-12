### 简介
前面的文章中，我们介绍了 MAC Frame 的帧格式。我们知道，在每个 Ethernet Frame 中都分别包含一个 48 bit 的源物理地址和目的物理地址. 对于源地址很容易理解，该地址可以直接从硬件上读取. 但是对于一个网络节点，他怎么知道一个 Frame 的目的物理地址呢? 本文我们将学习 ARP 协议来解答这个问题.

> ARP 协议主要用来完成将网络层协议的地址(比如，IP 地址)解析为物理地址的工作. 

为什么需要将网络层协议的地址转换为物理地址呢？

对于物理层来说，他仅仅能处理 48 bit 的物理地址，而网络层协议往往有自己协议中定义的协议地址，而这些协议的地址往往都不一样. 比如, IP 地址的长度为 32 bit，CHAOS 地址的长度为 16 bit, Xerox PUP 地址的长度为 8 bit.

反过来说，ARP 协议的存在也是必须的. 因为如果一个物理层想要同时支持上述的这些协议，他就不应该依赖于这些协议的实现，而是定义自己的地址格式，再通过一种方式将网络层协议的地址转化为物理地址. 这个方式就是 ARP 协议要实现的功能.

说明一点，接下来的描述将倾向于网络层协议是 IP 来进行描述.

### ARP 包格式
![ARP Packet Format](https://img-blog.csdnimg.cn/20200222155934121.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)
ARP 协议的包结构比较简单，我们直接看一个例子. 通过这个例子来解析每个字段的含义:

首先，我们注意到 ARP 包中 ar$op 字段可能的值有两个 REQUEST 和 REPLY。也就是说 ARP 协议的包大体分为两类，我们一一来看.

##### REQUEST
![ARP Request Packet](https://img-blog.csdnimg.cn/2020022216034585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)
这个包对应的二进制数据为：
00 01 08 00 06 04 00 01 dc a3 33 c4 1e 5a c0 a8
01 01 00 00 00 00 00 00 c0 a8 01 65            

1. 首先，我们看到 Wireshark 抓到的这个包中确实有9个字段，与我们上图中给出的 ARP 包结构完全吻合，并一一对应.
2. Hardware type: Enternet(1) 该字段对应于 ar$hdr, 表明当前硬件地址的类型为 Ethernet 物理地址类型
3. Protocol type: IPv4(0x0800) 该字段对应于 ar$pro, 表明网络层协议为 IP 协议，也就是说这个 ARP 请求包是为了完成将一个 IP 地址解析为物理地址的工作.
4. Hardware size: 6 该字段对应于 ar$hln, 表明物理地址的长度, 这里 6 的单位为 byte，而不是 bit，需要注意.
5.  Protocol size: 4 该字段对应于 ar$pln, 表明网络层协议的长度, 这里 4 的单位为 byte，而不是 bit，需要注意.
6. Opcode：request(1) 该字段对应于 ar$op, 表明当前 ARP 包的类型。这里，这个 ARP 包是一个请求包.
7. Sender MAC Address: 该字段对应于 ar$sha, 表明发送这个ARP 请求包的网络节点的物理地址.
8. Sender IP Address: 该字段对应于 ar$spa, 表明发送这个ARP 请求包的网络节点的网络层地址. 这里应为网络层协议为 IP 协议，因此这里是一个 IP 地址. 自然，不同的网络层协议实现的 ARP 协议，这个字段的长度和值是不同的.
9. Target MAC Address: 该字段对应于 ar\$tha，应为这个 ARP 包是一个请求包且当前网络节点不知道目的地址的物理地址，因此此处填上全0来占位.
10.Target IP Address: 该字段对应于 ar$tpa,  表明想要将该IP地址解析为物理层地址.

总结：
1. 这个 ARP 请求包想要解析 IP 地址 "192.168.1.101" 对应的物理地址
2. 至于 ar$hln 和 ar$pln 的必要性，我们解释一下: ARP 协议用来完成将网络层协议地址解析为物理层地址的功能，而正如我们前文提到的，不同的网络层协议的地址长度是不同的，因此我们需要这两个字段来表明地址的长度。 只有这样，在接收者收到这个 ARP 请求时，才能正确的解析出来.
3. 往往，ARP 请求都是以广播的形式发送。因为在发送这个 ARP 请求的时候，发送节点并不知道接收方的物理层地址. 

 ##### REPLY
![ARP Reply Packet](https://img-blog.csdnimg.cn/20200222162916115.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)
这个包对应的二进制数据为：
00 01 08 00 06 04 00 02 98 fa 9b 17 a8 f8 c0 a8
01 65 dc a3 33 c4 1e 5a c0 a8 01 01            

1. 在接收到 ARP 请求之后，接收者会比较这个ARP 请求中的网络层地址与自身的网络地址是否一致。如果一致，就使用自己的物理层地址构建一个 ARP Reply 包来响应这个请求。
2. 这里，这个响应不再需要以广播的形式发送，因为在接收到的 ARP 请求包中包含了发送方的物理层地址，因此可以直接将响应发送给该网络节点.


END!