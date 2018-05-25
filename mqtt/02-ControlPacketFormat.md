# 第二章 MQTT控制报文格式 MQTT Control Packet format

## 目录

- [第一章 - 介绍](01-Introduction.md)
- [第二章 – MQTT控制报文格式](02-ControlPacketFormat.md)
- [第三章 – MQTT控制报文](03-ControlPackets.md)
- [第四章 – 操作行为](04-OperationalBehavior.md)
- [第五章 – 安全（非规范）](05-Security.md)
- [第六章 – 使用WebSocket作为网络层](06-WebSocket.md)
- [第七章 – 一致性](07-Conformance.md)
- [附录B - 强制性规范声明（非规范）](08-AppendixB.md)
- [附录C - MQTT v5.0新特性总结（非规范）](09-AppendixC.md)

## 2.1 MQTT控制报文结构 Structure of an MQTT Control Packet

MQTT协议通过交换预定义的MQTT控制报文来通信。这一节描述这些报文的格式。

MQTT控制报文由三部分组成，按照下图描述的顺序：

##### 图 2-1 - MQTT控制报文的结构 Structure of an MQTT Control Packet

<table>
  <tr>
    <td width=224 align="center">Fixed Header固定报头，所有控制报文都包含</td>
  </tr>
  <tr>
    <td align="center">Variable Header 可变报头，部分控制报文包含</td>
  </tr>
  <tr>
    <td align="center">Payload 有效载荷，部分控制报文包含</td>
  </tr>
</table>

### 2.1.1 固定报头 Fixed header

如下图所示，每个 MQTT 控制报文都包含一个固定报头。

##### 图 2-2 - 固定报头的格式 Fixed Header format

<table style="text-align:center">
  <tr>
    <th width=130>比特位</td>
    <th width=65>7</th>
    <th width=65>6</th>
    <th width=65>5</th>
    <th width=65>4</th>
    <th width=65>3</th>
    <th width=65>2</th>
    <th width=65>1</th>
    <th width=65>0</th>
  </tr>
  <tr>
    <td>byte 1</td>
    <td colspan="4" align="center">MQTT控制报文的类型</td>
    <td colspan="4" align="center">用于指定控制报文类型的标志位</td>
  </tr>
  <tr>
    <td>byte 2...</td>
    <td colspan="8" align="center">剩余长度</td>
  </tr>
</table>

### 2.1.2 MQTT控制报文的类型 MQTT Control Packet type

**位置：** 第1个字节，二进制位7-4。

表示为4位无符号值，这些值的定义见下表。

##### 表 2-1 - MQTT控制报文的类型 MQTT Control Packet types

<table>
  <tr>
    <th>名字</th>
    <th>值</th>
    <th>报文流动方向</th>
	<th>描述</th>
  </tr>
  <tr>
    <td>Reserved</td>
	<td>0</td>
	<td>禁止</td>
	<td>保留</td>
  </tr>
  <tr>
    <td>CONNECT</td>
	<td>1</td>
	<td>客户端到服务端</td>
	<td>客户端请求连接服务端</td>
  </tr>
  <tr>
    <td>CONNACK</td>
	<td>2</td>
	<td>服务端到客户端</td>
	<td>连接报文确认</td>
  </tr>
  <tr>
    <td>PUBLISH</td>
	<td>3</td>
	<td>两个方向都允许</td>
	<td>发布消息</td>
  </tr>
  <tr>
    <td>PUBACK</td>
	<td>4</td>
	<td>两个方向都允许</td>
	<td>QoS 1消息发布收到确认</td>
  </tr>
  <tr>
    <td>PUBREC</td>
	<td>5</td>
	<td>两个方向都允许</td>
	<td>发布收到（保证交付第一步）</td>
  </tr>
  <tr>
    <td>PUBREL</td>
	<td>6</td>
	<td>两个方向都允许</td>
	<td>发布释放（保证交付第二步）</td>
  </tr>
  <tr>
    <td>PUBCOMP</td>
	<td>7</td>
	<td>两个方向都允许</td>
	<td>QoS 2消息发布完成（保证交互第三步）</td>
  </tr>
  <tr>
    <td>SUBSCRIBE</td>
	<td>8</td>
	<td>客户端到服务端</td>
	<td>客户端订阅请求</td>
  </tr>
  <tr>
    <td>SUBACK</td>
	<td>9</td>
	<td>服务端到客户端</td>
	<td>订阅请求报文确认</td>
  </tr>
  <tr>
    <td>UNSUBSCRIBE</td>
	<td>10</td>
	<td>客户端到服务端</td>
	<td>客户端取消订阅请求</td>
  </tr>
  <tr>
    <td>UNSUBACK</td>
	<td>11</td>
	<td>服务端到客户端</td>
	<td>取消订阅报文确认</td>
  </tr>
  <tr>
    <td>PINGREQ</td>
	<td>12</td>
	<td>客户端到服务端</td>
	<td>心跳请求</td>
  </tr>
  <tr>
    <td>PINGRESP</td>
	<td>13</td>
	<td>服务端到客户端</td>
	<td>心跳响应</td>
  </tr>
  <tr>
    <td>DISCONNECT</td>
	<td>14</td>
	<td>两个方向都允许</td>
	<td>断开连接通知</td>
  </tr>
  <tr>
    <td>AUTH</td>
	<td>15</td>
	<td>两个方向都允许</td>
	<td>认证信息交换</td>
  </tr>
<table>                        

### 2.1.3 标志 Flags

固定报头第1个字节的剩余的4位 \[3-0\]包含每个 MQTT 控制报文类型特定的标志如下表所示。表格中任何标记为“保留”的标志位，都是保留给以后使用的，**必须**设置为表格中列出的值 \[MQTT-2.1.3-1\]。如果收到非法的标志，此报文被当做无效报文。有关错误处理的详细信息见 4.8节 \[MQTT-2.2.2-2\]。

##### 表 2-2 - 标志位 Flag Bits

<table>
  <tr>
    <th width=100>控制报文</th>
    <th>固定报头标志</th>
    <th>Bit 3</th>
	<th>Bit 2</th>
	<th>Bit 1</th>
	<th>Bit 0</th>
  </tr>
  <tr>
    <td>CONNECT</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>CONNACK</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>PUBLISH</td>
	<td>Used in MQTT v5.0</td>
	<td>DUP</td>
	<td colspan="2" align="center">QoS</td>
	<td>RETAIN</td>
  </tr>
  <tr>
    <td>PUBACK</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>PUBREC</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>PUBREL</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
  </tr>
  <tr>
    <td>PUBCOMP</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>SUBSCRIBE</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
  </tr>
  <tr>
    <td>SUBACK</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>UNSUBSCRIBE</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
  </tr>
  <tr>
    <td>UNSUBACK</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>PINGREQ</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>PINGRESP</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>DISCONNECT</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>AUTH</td>
	<td>Reserved</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
</table>

- DUP<sup>1</sup> =控制报文的重复分发标志
- QoS<sup>2</sup> = PUBLISH报文的服务质量等级
- RETAIN<sup>3</sup> = PUBLISH报文的保留标志
PUBLISH控制报文中的DUP, QoS和RETAIN标志的描述见 3.3.1节。

### 2.1.4 剩余长度 Remaining Length

**位置：** 从第2个字节开始。

剩余长度（Remaining Length）是一个变长字节整数，用来表示当前控制报文剩余部分的字节数，包括可变报头和负载的数据。剩余长度不包括用于编码剩余长度字段本身的字节数。MQTT控制报文总长度等于固定报头的长度加上剩余长度。

## 2.2 可变报头 Variable header

某些 MQTT 控制报文包含一个可变报头部分。它在固定报头和有效载荷之间。可变报头的内容根据报文类型的不同而不同。可变报头的报文标识符（Packet Identifier）字段存在于在多个类型的报文里。

### 2.2.1 报文标识符 Packet Identifier
部分类型MQTT控制报文的可变报头部分包含了2个字节的报文标识符字段。这些MQTT控制报文类型为：PUBLISH报文（当QoS>0时），PUBACK，PUBREC，PUBREC，PUBREL，PUBCOMP，SUBSCRIBE，SUBACK，UNSUBSCRIBE，UNSUBACK。

需要报文标识符的MQTT控制报文如下表所示。

##### 表 2-3 - 包含报文标识符的MQTT控制报文 MQTT Control Packets that contain a Packet Identifier

<table>
  <tr>
    <th>名字</th>
    <th>值</th>
  </tr>
  <tr>
    <td>CONNECT</td>
    <td>不需要</td>
  </tr>
  <tr>
    <td>CONNACK</td>
    <td>不需要</td>
  </tr>
  <tr>
    <td>PUBLISH</td>
    <td>需要（如果QoS>0）</td>
  </tr>
  <tr>
    <td>PUBACK</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>PUBREC</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>PUBREL</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>PUBCOMP</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>SUBSCRIBE</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>SUBACK</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>UNSUBSCRIBE</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>UNSUBACK</td>
    <td>需要</td>
  </tr>
  <tr>
    <td>PINGREQ</td>
    <td>不需要</td>
  </tr>
  <tr>
    <td>PINGRESP</td>
    <td>不需要</td>
  </tr>
  <tr>
    <td>DISCONNECT</td>
    <td>不需要</td>
  </tr>
  <tr>
    <td>AUTH</td>
    <td>不需要</td>
  </tr>
<table>

QoS 设置为 0 的 PUBLISH 报文**不能**包含报文标识符[MQTT-2.2.1-2]。

客户端每次发送一个新的SUBSCRIBE，UNSUBSCRIBE或者PUBLISH（当QoS>0时）MQTT控制报文时都**必须**分配一个当前未使用的非零报文标识符 [MQTT-2.2.1-3]。

服务端每次发送一个新的PUBLISH（当QoS>0）MQTT控制报文时都**必须**分配一个当前未使用的非零报文标识符 [MQTT-2.2.1-4]。

当客户端处理完这个报文对应的确认后，这个报文标识符就释放可重用。QoS 1的PUBLISH对应的是PUBACK，QoS 2的PUBLISH对应的是包含原因码128以上的PUBCOMP或PUBREC，与SUBSCRIBE或UNSUBSCRIBE对应的分别是SUBACK或UNSUBACK。

PUBLISH，SUBSCRIBE和UNSUBSCRIBE的报文标识符，在一次会话中对于客户端和服务端来说分属于不同的组。某个报文标识符在某一时刻**不能**被多个命令所使用。

PUBACK，PUBREC和PUBREL报文**必须**包含与最初发送的PUBLISH报文相同的报文标识符 [MQTT-2.2.1-5]。类似地，SUBACK和UNSUBACK**必须**包含在对应的SUBSCRIBE和UNSUBSCRIBE报文中使用的报文标识符 [MQTT-2.2.1-6]。

客户端和服务端彼此独立地分配报文标识符。因此，客户端服务端组合使用相同的报文标识符可以实现并发的消息交换。

> **非规范评注**
>
> 客户端发送标识符为0x1234的PUBLISH报文，它有可能会在收到那个报文的PUBACK之前，先收到服务端发送的另一个不同的但是报文标识符也为0x1234的PUBLISH报文。

<table>
  <tr>
    <th width="260">客户端</th>
    <th width="260">服务端</th>
  </tr>
  <tr>
    <td align="left" colspan="2">PUBLISH Packet Identifier = 0x1234---></td>
  </tr>
  <tr>
    <td align="right" colspan="2"><---PUBLISH	Packet Identifier = 0x1234</td>
  </tr>
  <tr>
    <td align="left" colspan="2">PUBACK Packet Identifier = 0x1234---></td>
  </tr>
  <tr>
    <td align="right" colspan="2"><---PUBACK Packet Identifier = 0x1234</td>
  </tr>
</table>

### 2.2.2 属性 Properties

CONNECT，CONNACK，PUBLISH，PUBACK，PUBREC，PUBREL，PUBCOMP，SUBSCRIBE，SUBACK，UNSUBACK，DISCONNECT和AUTH报文可变报头的最后一部分是一组属性。CONNECT报文的遗嘱（Will）属性字段中也包含了一组可选的属性。

属性字段由属性长度和所有属性组成。

#### 2.2.2.1 属性长度 Property Length

属性长度被编码为变长字节整数。属性长度不包含用于编码属性长度自身的字节数，但包含所有属性的长度。如果没有任何属性，**必须**由属性长度为零的字段来指示 [MQTT-2.2.2-1]。

#### 2.2.2.2 属性 Property

一个属性包含一段数据和一个定义了属性用途和数据类型的标识符。标识符被编码为变长字节整数。任何控制报文，如果包含了：对于该报文类型无效的标识符，或者错误类型的数据，都是无效报文。收到无效报文时，服务端或客户端使用包含原因码0x81（无效报文）CONNACK或DISCONNECT报文进行错误处理，如4.13节 所述。标识符排序不分先后。

##### 表 2-4 - 属性 Properties

<table>
  <tr>
    <th colspan="2">标识符</th>
    <th width=156 rowspan="2">属性名</th>
    <th width=152 rowspan="2">数据类型</th>
	<th rowspan="2">报文/遗嘱属性</th>
  </tr>
  <tr>
    <td><strong>Dec</strong></td>
	<td><strong>Hex</strong></td>
  </tr>
  <tr>
    <td>1</td>
	<td>0x01</td>
	<td>载荷格式说明</td>
	<td>字节</td>
	<td>PUBLISH, Will Properties</td>
  </tr>
  <tr>
    <td>2</td>
	<td>0x02</td>
	<td>消息过期时间</td>
	<td>四字节整数</td>
	<td>PUBLISH, Will Properties</td>
  </tr>
  <tr>
    <td>3</td>
	<td>0x03</td>
	<td>内容类型</td>
	<td>UTF-8编码字符串</td>
	<td>PUBLISH, Will Properties</td>
  </tr>
  <tr>
    <td>8</td>
	<td>0x08</td>
	<td>响应主题</td>
	<td>UTF-8编码字符串</td>
	<td>PUBLISH, Will Properties</td>
  </tr>
  <tr>
    <td>9</td>
	<td>0x09</td>
	<td>相关数据</td>
	<td>二进制数据</td>
	<td>PUBLISH, Will Properties</td>
  </tr>
  <tr>
    <td>11</td>
	<td>0x0B</td>
	<td>定义标识符</td>
	<td>变长字节整数</td>
	<td>PUBLISH, SUBSCRIBE</td>
  </tr>
  <tr>
    <td>17</td>
	<td>0x11</td>
	<td>会话过期间隔</td>
	<td>四字节整数</td>
	<td>CONNECT, CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>18</td>
	<td>0x12</td>
	<td>分配客户标识符</td>
	<td>UTF-8编码字符串</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>19</td>
	<td>0x13</td>
	<td>服务端保活时间</td>
	<td>双字节整数</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>21</td>
	<td>0x15</td>
	<td>认证方法</td>
	<td>UTF-8编码字符串</td>
	<td>CONNECT, CONNACK, AUTH</td>
  </tr>
  <tr>
    <td>22</td>
	<td>0x16</td>
	<td>认证数据</td>
	<td>二进制数据</td>
	<td>CONNECT, CONNACK, AUTH</td>
  </tr>
  <tr>
    <td>23</td>
	<td>0x17</td>
	<td>请求问题信息</td>
	<td>字节</td>
	<td>CONNECT</td>
  </tr>
  <tr>
    <td>24</td>
	<td>0x18</td>
	<td>遗嘱延时间隔</td>
	<td>四字节整数</td>
	<td>Will Properties</td>
  </tr>
  <tr>
    <td>25</td>
	<td>0x19</td>
	<td>请求响应信息</td>
	<td>字节</td>
	<td>CONNECT</td>
  </tr>
  <tr>
    <td>26</td>
	<td>0x1A</td>
	<td>请求信息</td>
	<td>UTF-8编码字符串</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>28</td>
	<td>0x1C</td>
	<td>服务端参考</td>
	<td>UTF-8编码字符串</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>31</td>
	<td>0x1F</td>
	<td>原因字符串</td>
	<td>UTF-8编码字符串</td>
	<td>CONNACK, PUBACK, PUBREC, PUBREL, PUBCOMP, SUBACK, UNSUBACK, DISCONNECT, AUTH</td>
  </tr>
  <tr>
    <td>33</td>
	<td>0x21</td>
	<td>接收最大数量</td>
	<td>双字节整数</td>
	<td>CONNECT, CONNACK</td>
  </tr>
  <tr>
    <td>34</td>
	<td>0x22</td>
	<td>主题别名最大长度</td>
	<td>双字节整数</td>
	<td>CONNECT, CONNACK</td>
  </tr>
  <tr>
    <td>35</td>
	<td>0x23</td>
	<td>主题别名</td>
	<td>双字节整数</td>
	<td>PUBLISH</td>
  </tr>
  <tr>
    <td>36</td>
	<td>0x24</td>
	<td>最大QoS</td>
	<td>字节</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>37</td>
	<td>0x25</td>
	<td>保留属性可用性</td>
	<td>字节</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>38</td>
	<td>0x26</td>
	<td>用户属性</td>
	<td>UTF-8字符串对</td>
	<td>CONNECT, CONNACK, PUBLISH, Will Properties, PUBACK, PUBREC, PUBREL, PUBCOMP, SUBSCRIBE, SUBACK, UNSUBSCRIBE, UNSUBACK, DISCONNECT, AUTH</td>
  </tr>
  <tr>
    <td>39</td>
	<td>0x27</td>
	<td>最大报文长度</td>
	<td>四字节整数</td>
	<td>CONNECT, CONNACK</td>
  </tr>
  <tr>
    <td>40</td>
	<td>0x28</td>
	<td>通配符订阅可用性</td>
	<td>字节</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>41</td>
	<td>0x29</td>
	<td>订阅标识符可用性</td>
	<td>字节</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>42</td>
	<td>0x2A</td>
	<td>共享订阅可用性</td>
	<td>字节</td>
	<td>CONNACK</td>
  </tr>
</table>

> **非规范评注**
>
> 尽管属性标识符用变长字节整数来表示，但在此版本协议中，所有的标识符均由一个字节来表示。


## 2.3 有效载荷 Payload

某些MQTT控制报文在报文的最后部分包含一个有效载荷，这将在第三章论述。对于PUBLISH来说有效载荷就是应用消息。

##### 表 2-5 包含有效载荷的MQTT控制报文 MQTT Control Packets that contain a Payload

<table>
  <tr>
    <th>MQTT控制报文</th>
    <th>有效载荷</th>
  </tr>
  <tr>
    <td>CONNECT</td>
	<td>需要</td>
  </tr>
  <tr>
    <td>CONNACK</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>PUBLISH</td>
	<td>可选</td>
  </tr>
  <tr>
    <td>PUBACK</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>PUBREC</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>PUBREL</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>PUBCOMP</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>SUBSCRIBE</td>
	<td>需要</td>
  </tr>
  <tr>
    <td>SUBACK</td>
	<td>需要</td>
  </tr>
  <tr>
    <td>UNSUBSCRIBE</td>
	<td>需要</td>
  </tr>
  <tr>
    <td>UNSUBACK</td>
	<td>需要</td>
  </tr>
  <tr>
    <td>PINGREQ</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>PINGRESP</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>DISCONNECT</td>
	<td>不需要</td>
  </tr>
  <tr>
    <td>AUTH</td>
	<td>不需要</td>
  </tr>
<table>

## 2.4 原因码 Reason Code

原因码是一个单字节无符号数，用来指示一次操作的结果。小于0x80的原因码指示某次操作成功完成，通常用0来表示。大于等于0x80的原因码用来指示操作失败。

CONNACK，PUBACK，PUBREC，PUBREL，PUBCOMP，DISCONNECT和AUTH控制报文的可变报头有一个单字节的原因码。SUBACK和UNSUBACK报文的载荷字段包含一个或多个原因码。

原因码如下表所示。

##### 表 2-6 - 原因码 Reason Code

<table>
  <tr>
    <th colspan="2">原因码</th>
    <th rowspan="2">名称</th>
    <th rowspan="2">报文</th>
  </tr>
  <tr>
    <td><strong>Dec</strong></td>
	<td><strong>Hex</strong></td>
  </tr>
  <tr>
    <td>0</td>
	<td>0x00</td>
	<td>成功</td>
	<td>CONNACK, PUBACK, PUBREC, PUBREL, PUBCOMP, UNSUBACK, AUTH</td>
  </tr>
  <tr>
    <td>0</td>
	<td>0x00</td>
	<td>正常断开</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>0</td>
	<td>0x00</td>
	<td>授权的QoS 0</td>
	<td>SUBACK</td>
  </tr>
  <tr>
    <td>1</td>
	<td>0x01</td>
	<td>授权的QoS 1</td>
	<td>SUBACK</td>
  </tr>
  <tr>
    <td>2</td>
	<td>0x02</td>
	<td>授权的QoS 2</td>
	<td>SUBACK</td>
  </tr>
  <tr>
    <td>4</td>
	<td>0x04</td>
	<td>包含遗嘱的断开</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>16</td>
	<td>0x10</td>
	<td>无匹配订阅</td>
	<td>PUBACK, PUBREC</td>
  </tr>
  <tr>
    <td>17</td>
	<td>0x11</td>
	<td>订阅不存在</td>
	<td>UNSUBACK</td>
  </tr>
  <tr>
    <td>24</td>
	<td>0x18</td>
	<td>继续认证</td>
	<td>AUTH</td>
  </tr>
  <tr>
    <td>25</td>
	<td>0x19</td>
	<td>重新认证</td>
	<td>AUTH</td>
  </tr>
  <tr>
    <td>128</td>
	<td>0x80</td>
	<td>未指明的错误</td>
	<td>CONNACK, PUBACK, PUBREC, SUBACK, UNSUBACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>129</td>
	<td>0x81</td>
	<td>无效报文</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>130</td>
	<td>0x82</td>
	<td>协议错误</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>131</td>
	<td>0x83</td>
	<td>实现错误</td>
	<td>CONNACK, PUBACK, PUBREC, SUBACK, UNSUBACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>132</td>
	<td>0x84</td>
	<td>协议版本不支持</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>133</td>
	<td>0x85</td>
	<td>客户标识符无效</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>134</td>
	<td>0x86</td>
	<td>用户名密码错误</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>135</td>
	<td>0x87</td>
	<td>未授权</td>
	<td>CONNACK, PUBACK, PUBREC, SUBACK, UNSUBACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>136</td>
	<td>0x88</td>
	<td>服务端不可用</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>137</td>
	<td>0x89</td>
	<td>服务端正忙</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>138</td>
	<td>0x8A</td>
	<td>禁止</td>
	<td>CONNACK</td>
  </tr>
  <tr>
    <td>139</td>
	<td>0x8B</td>
	<td>服务端关闭中</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>140</td>
	<td>0x8C</td>
	<td>无效的认证方法</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>141</td>
	<td>0x8D</td>
	<td>保活超时</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>142</td>
	<td>0x8E</td>
	<td>会话被接管</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>143</td>
	<td>0x8F</td>
	<td>主题过滤器无效</td>
	<td>SUBACK, UNSUBACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>144</td>
	<td>0x90</td>
	<td>主题名无效</td>
	<td>CONNACK, PUBACK, PUBREC, DISCONNECT</td>
  </tr>
  <tr>
    <td>145</td>
	<td>0x91</td>
	<td>报文标识符已被占用</td>
	<td>PUBACK, PUBREC, SUBACK, UNSUBACK</td>
  </tr>
  <tr>
    <td>146</td>
	<td>0x92</td>
	<td>报文标识符无效</td>
	<td>PUBREL, PUBCOMP</td>
  </tr>
  <tr>
    <td>147</td>
	<td>0x93</td>
	<td>接收超出最大数量</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>148</td>
	<td>0x94</td>
	<td>主题别名无效</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>149</td>
	<td>0x95</td>
	<td>报文过长</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>150</td>
	<td>0x96</td>
	<td>消息太过频繁</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>151</td>
	<td>0x97</td>
	<td>超出配额</td>
	<td>CONNACK, PUBACK, PUBREC, SUBACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>152</td>
	<td>0x98</td>
	<td>管理行为</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>153</td>
	<td>0x99</td>
	<td>载荷格式无效</td>
	<td>CONNACK, PUBACK, PUBREC, DISCONNECT</td>
  </tr>
  <tr>
    <td>154</td>
	<td>0x9A</td>
	<td>不支持保留</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>155</td>
	<td>0x9B</td>
	<td>不支持的QoS等级</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>156</td>
	<td>0x9C</td>
	<td>（临时）使用其他服务端</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>157</td>
	<td>0x9D</td>
	<td>服务端已（永久）移动</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>158</td>
	<td>0x9E</td>
	<td>不支持共享订阅</td>
	<td>SUBACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>159</td>
	<td>0x9F</td>
	<td>超出连接速率限制</td>
	<td>CONNACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>160</td>
	<td>0xA0</td>
	<td>最大连接时间</td>
	<td>DISCONNECT</td>
  </tr>
  <tr>
    <td>161</td>
	<td>0xA1</td>
	<td>不支持订阅标识符</td>
	<td>SUBACK, DISCONNECT</td>
  </tr>
  <tr>
    <td>162</td>
	<td>0xA2</td>
	<td>不支持通配符订阅</td>
	<td>SUBACK, DISCONNECT</td>
  </tr>
</table>

> **非规范评注**
>
> 对于原因码0x91（报文标识符已被占用）的处理可以为尝试修复会话、以新会话标志为1重置会话或者判定客户端或服务端实现有缺陷。

### 项目主页

- [MQTT v5.0协议草案中文版](https://github.com/hui6075/mqtt_v5)


