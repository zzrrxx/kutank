### 简介

---

IP 是一种无连接的协议. 操作在使用分组交换的链路层（如以太网）上。此协议会尽最大努力交付数据包。

尽最大努力意味着：
<b> IP 协议不保证数据的可靠传输, 没有流量控制机制, 不保证传输序列(意味着 IP 数据包会在传输过程中乱序), 没有接受确认 (ACK) 机制， 也没有重传机制.</b>

----

### 主要功能

----

IP 协议提供了两个基本的功能 <b>寻址(Address)</b> 和 <b>分片(Fragmentation)</b>. 

这里我们先简单看一下概念，后面会详细分析具体是如何实现的.

---

#### 寻址(Address)

---

IP 协议定义了 32 bit 的 IP 地址. IP 协议的实现模块会根据 IP 数据包头部中的目的地址发送/转发数据包到目的地址. 

1. 数据包在由源地址到目的地址的过程中，可能会经过多个中间节点, 这些节点需要使用适当的路由协议转发数据包。也可能丢弃数据包。
2. IP 协议实现模块认为每个 IP 数据包都是独立的，任意两个 IP 数据包之间没有任何联系.

IP 协议还提供了以下机制来配置 IP 协议提供的服务：
1. Type of Service: 用来保证 IP 协议服务的的质量. 

2. Time to Live: 指定一个 IP 数据包的生命周期的上限. 这个值由发送发设置，该数据包没经过一个中间节点，该数据包中的 Time to Live 值便减少1，当这个值变为零并且还未到达目的地址，该数据包会在该中间节点被丢弃.

3. Options: 一些控制位，在某些情况下会被使用.

4. Header Checksum: 用来检测数据包在传输过程中是否发生了错误. 如果在某个节点发现数据出错，数据会在该节点被丢弃.

   ---

#### 分片(Fragmentation)

----

由于不同的物理链路对于数据的最大长度有着不同的限制，因此 IP 数据在传输过程中可能会被拆分为多个较小的数据包进行传输.

发送方可以将数据包设置为 ”don‘t fragment", 这种数据包会被丢弃当数据包的大小大于当前物理链路的最大数据包限制.

----

### IPv4 Header 格式

----

![IP Header Format](https://img-blog.csdnimg.cn/20200228235134234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)

接下来，我们逐一介绍一下每个字段的含义：

1. Version: 4 bit. 表明当前 IP 协议的版本.
2. IHL:  是 Internet Hander Length 的缩写，长度为 4 bit. 它的值是一个整数，指向 IP 数据包中所承载的实际数据的开始字节. 单位为 4 字节.
3. Type of Service: 8 bit. 表明当前数据包所期待的服务类型。 8 个 bit 的含义分别如下：
	- bits 0-2： 表明数据包的优先级
	- bit 3: 0 代表 Normal-Delay,            1 代表 Low-Delay
	- bit 4: 0 代表 Normal-Throughput，1 代表 High-Throughput
	- bit 5: 0 代表 Normal-Reliable,        1 代表 High-Reliable
	- bits 6-7： 保留
	这个字段很少使用，这里仅供参考.
4. Total Length: 16 bit.  表明当前数据包的总长度, 包括 IP 头和数据. 单位是字节. 最大长度为 65535 字节, 
5. Identification: 16 bit. 一个由发送者设置的 id 值，用来帮助将分片后的数据包整合为一个数据包. 后边详细介绍
6. Flags: 3 bit. 每个 bit 的含义如下:
	- bit 0: 保留
	- bit 1: (DF) 0 代表 May Fragment, 1 代表 Don't Fragment
	- bit 2: (MF) 0 代表 Last Fragment, 1 代表 More Fragments
7. Fragment Offset: 13 bit. 表明当前分片在所有分片中的位置. 单位为 8 字节.第一个分片该字段的值为 0.
8. Time to Live: 8 bit. 
9. Protocol: 8 bit. 表明当前 IP 数据包中所承载的数据的协议. 也就是位于 IP 协议之上的协议.
10. Header Checksum: 16 bit. 在数据包传输过程中，该值会随着 IP头部的值的改变而改变. 比如: Time to Live 变化，或者 IP 包被分片，这种情况下，都需要重新计算 checksum.
11. Source Address, Destination Address: 各占 32 bit.
12. Options: 可变长度字段. 由于 Options 比较复杂，下面详细描述
13. Padding: 可变长字段. 用来将 IP 数据包长度填充为 32 bit 的整数倍. 填冲的 bit 位的值均为 0.

----

### Options

---

该字段在 IP 头部属于可选字段，可以出现也可以不出现. 

Options 字段中可以包含 0 到多个 Option，对于每个 Option 它的合法格式有以下两种:
1. 使用一个单独的 Option-Type 作为该字段的值
2. Option-Type + Option-Length + Option-Data 作为该字段的值

一个字节的 Option-Length 表明 后续 Option-Data 的长度. 单位也为字节.
##### Option-Type:
该类型长度为 8 bit. 每个 bit 的含义如下:
- bit 0: copied field, 
- bit 1-2: option class
- bit 3-7: option number

##### Copied field:
表明当需要对原始数据包进行分片时，该 Option 是否需要拷贝到每个分片中去. 
- 0 代表 not copied
- 1 代表 copied

##### Option class:
- 0: 代表 Control
- 1: 保留
- 2: 代表 Debugging and measurement
- 3: 保留

##### 当前可用的 Option
| Class | Number | Length | Description                                                  |
| ----- | ------ | ------ | ------------------------------------------------------------ |
| 0     | 0      | -      | End of Option list. 占用一个字节.没有长度                    |
| 0     | 1      | -      | No operation. 占用一个字节.没有长度                          |
| 0     | 2      | 11     | Security.                                                    |
| 0     | 3      | var.   | Loose Source Routing. 使用源发送者所提供的信息路由当前数据包 |
| 0     | 9      | var.   | Strict Source Routing. 使用源发送者所提供的信息路由当前数据包 |
| 0     | 7      | var.   | Record Route. 用于跟踪当当前数据包的路由                     |
| 0     | 8      | 4      | Stream ID. 用于包含 Stream ID                                |
| 2     | 4      | var.   | Internet Timestamp                                           |

----

### 抓包分析

---

#### 未分片IP包

----

经过上面的学习，我们来尝试着分析一个 Wireshark 抓到的 IP 数据包.

![Wireshakr IP Package](https://img-blog.csdnimg.cn/20200229004900793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)

原始二进制数据为：
45 00 00 28 fa f1 40 00 32 06 ec 59 a1 75 ff 00
c0 a8 00 66

1. 第一个字段为 Version，这里是 IPv4 协议，因此第一个字节便是 4.
2. 第二个字段为 IHL， 我们看到它的值是 5， 由于该字段的单位为 4 字节，因此实际的数据是 20. 也就是说我们这个 IP包的包头长度为 20 字节. 说明一点， IP头的最小长度便是 20 字节。
3. 第三个字段为 Type of Service. 很明显 wireshark 使用的描述和 RFC 中定义不同. 不过也没关系，该字段的值为 0. 也就是未使用.
4. 第四个字段为 Total Length, 表明当前 IP包的整体长度为 40 字节。 20 字节的 IP 头部 和 20 字节的 TCP 数据.
5. 第五个字段是  Identification
6. 第六个字段是 Flags，它的值是 010 (二进制)， 表明不要对当前 IP 数据包分片，且当前分片是 last frament.
7. 第七个字段是 Fragment Offset, 由于该 IP 数据包未被分片，因此 我们看到 Fragment Offset 字段为 0.
8. 第八个字段是 Time to Live, 它的值是 50.
9. 第九个字段是 Protocol, 表明当前 IP 数据包中所承载的数据是 TCP 协议的数据包.
10. 第十个字段是 Checksum
11. 第十一个字段和十二个字段分别代表源地址和目的地址.
12. Padding: 截图中未体现出来. 由于该数据包的实际长度为 50 字节，并非是 32 bit 的整数倍，因此需要使用 Padding 字段来填充该 IP 数据包. 计算得知 padding 应为 2 个字节，值为 0x0000.

-----

 #### 分片 IP包

---

接下来我们来看看分片的 IP 数据包是怎么样的.
这里我们将会看到一个分成三个分片的数据包. 这里我们只关注分片相关的几个字段，其他字段参照上面分析.

##### 第一个分片



![IP Fragmentation: First Fragment](https://img-blog.csdnimg.cn/20200229012959984.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)

这里我们需要关注的字段有: Identification, Flags, 和 Fragment Offset 三个字段.

且看上图，这是一个IP包被分为三片中的第一片. 
1. 我们知道 Identification 的值是一个 Id，并没有具体意义，暂且略过.
2. Flags 的值为 001(二进制), 根据 Flags 的定义我们知道:
	- 第二个 bit 为 0， 代表 May Fragment，说明该数据包可以被分片
	- 第三个 bit 为 1， 代表 More Fragment, 说明该数据包不是原始数据包分片中的最后一个.
3. Fragment Offset 为 0， 代表当前分片是原始数据包分片中的第一个.

##### 第二个分片



![IP Fragmentation: Second Fragment](https://img-blog.csdnimg.cn/20200229013900360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)

4. 注意到当前分片的 Identification 值与第一个分片的值相同.
5. Flags 的值为 001(二进制), 根据 Flags 的定义我们知道:
	- 第二个 bit 为 0， 代表 May Fragment，说明该数据包可以被分片
	- 第三个 bit 为 1， 代表 More Fragment, 说明该数据包不是原始数据包分片中的最后一个.
6. Fragment Offset 为 1480. 这里需要注意 wireshark 显示的是计算过之后的值，如果你看它的原始数据的画你会发现该位的实际值为 .00010111001(二进制), 由于该字段的单位为 8 byte，因此等同于 00010111001 * 8 也就是 wireshark 中显示的那个值了. 由第一张图可以看到，第一个分片中的数据长度为 1480，因此第二个分片中的 Fragment offset 便对应于原始数据包中从 1480 位置除开始的数据. 数据长度依然为 1480.

##### 最后一个分片



![IP Fragmentation：Last Fragment](https://img-blog.csdnimg.cn/20200229015123364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9ydWl4aWFuZzExMTE=,size_16,color_FFFFFF,t_70)

7. 注意到当前分片的 Identification 值与前两个分片的值相同.
8. Flags 的值为 000(二进制), 根据 Flags 的定义我们知道:
	- 第二个 bit 为 0， 代表 May Fragment，说明该数据包可以被分片
	- 第三个 bit 为 0， 代表 Last Fragment, 说明该数据包是最后一个分片.
9. Fragment Offset 为 2960. 由于前两个分片的数据长度均为 1480，这个分片中的数据自然要从 2960 处开始了.

总结一下 IP 分片：
10. 如果将一个数据包的 Flags 中的 Don't Flag 标志设置为 1， 那么该 IP 数据包不能被分片
11. 对于需要进行分片的数据包而言，分片规则如下：
	-  对于第一个分片， Fragment Offset 设置为 0, 需要设置 More Fragment
	-  对于中间分片, 需要设置 More Fragment, 且 Fragment Offset 为之前所有分片中数据长度之和除以 8.
	-  对于最后一个分片， 需要设置 Last Fragment, 且 Fragment Offset 为之前所有分片数据长度之和除以 8
	-  同一个 IP 数据包分片之后的 IP 数据包需要设置相同的 Identification.

END!