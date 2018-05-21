# 第二章 MQTT控制报文格式 MQTT Control Packet format

## 目录

- [第一章 - 介绍](01-Introduction.md)
- [第二章 – MQTT控制报文格式](02-ControlPacketFormat.md)
- [第三章 – MQTT控制报文](03-ControlPackets.md)
- [第四章 – 操作行为](04-OperationalBehavior.md)
- [第五章 – 安全（非规范）](05-Security.md)
- [第六章 – 使用WebSocket](06-WebSocket.md)
- [第七章 – 一致性目标](07-Conformance.md)
- [附录B - 强制性规范声明（非规范）](08-AppendixB.md)
- [附录C - Appendix C. MQTT v5.0新特性总结（非规范）](09-AppendixC.md)

## 2.1 MQTT控制报文结构 Structure of an MQTT Control Packet

MQTT协议通过交换预定义的MQTT控制报文来通信。这一节描述这些报文的格式。

MQTT控制报文由三部分组成，按照下图描述的顺序：

##### 图 2.1 –MQTT控制报文的结构

| Fixed header   固定报头，所有控制报文都包含  |
|---------------------------------------------|
| Variable header   可变报头，部分控制报文包含 |
| Payload   有效载荷，部分控制报文包含         |

### 2.1.1 固定报头 Fixed header

如下图所示，每个 MQTT 控制报文都包含一个固定报头。

##### 图 2.2 -固定报头的格式

<table style="text-align:center">
  <tr>
    <td align="center"><strong>Bit</strong></td>
    <td align="center"><strong>7</strong></td>
    <td align="center"><strong>6</strong></td>
    <td align="center"><strong>5</strong></td>
    <td align="center"><strong>4</strong></td>
    <td align="center"><strong>3</strong></td>
    <td align="center"><strong>2</strong></td>
    <td align="center"><strong>1</strong></td>
    <td align="center"><strong>0</strong></td>
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

##### 表 2.1 -控制报文的类型 

| **名字**    | **值** | **报文流动方向** | **描述**                            |
|-------------|--------|------------------|-------------------------------------|
| Reserved    | 0      | 禁止             | 保留                                
| CONNECT     | 1      | 客户端到服务端   | 客户端请求连接服务端                
| CONNACK     | 2      | 服务端到客户端   | 连接报文确认                        
| PUBLISH     | 3      | 两个方向都允许   | 发布消息                            
| PUBACK      | 4      | 两个方向都允许   | QoS 1消息发布收到确认               
| PUBREC      | 5      | 两个方向都允许   | 发布收到（保证交付第一步）          |
| PUBREL      | 6      | 两个方向都允许   | 发布释放（保证交付第二步）          |
| PUBCOMP     | 7      | 两个方向都允许   | QoS 2消息发布完成（保证交互第三步） |
| SUBSCRIBE   | 8      | 客户端到服务端   | 客户端订阅请求                      
| SUBACK      | 9      | 服务端到客户端   | 订阅请求报文确认                    
| UNSUBSCRIBE | 10     | 客户端到服务端   | 客户端取消订阅请求                  
| UNSUBACK    | 11     | 服务端到客户端   | 取消订阅报文确认                    
| PINGREQ     | 12     | 客户端到服务端   | 心跳请求                            
| PINGRESP    | 13     | 服务端到客户端   | 心跳响应                            
| DISCONNECT  | 14     | 两个方向都允许   | 断开连接通知                        
| AUTH        | 15     | 两个方向都允许   | 认证信息交换                        

### 2.1.3 标志 Flags

固定报头第1个字节的剩余的4位 \[3-0\]包含每个 MQTT 控制报文类型特定的标志如下表所示。表格中任何标记为“保留”的标志位，都是保留给以后使用的，**必须**设置为表格中列出的值 \[MQTT-2.1.3-1\]。如果收到非法的标志，此报文被当做无效报文。有关错误处理的详细信息见 4.8节 \[MQTT-2.2.2-2\]。

#####  表 2.2 - 标志位 Flag Bits

| **控制报文** | **固定报头标志**   | **Bit 3**       | **Bit 2**       | **Bit 1**       | **Bit 0**          |
|--------------|--------------------|-----------------|-----------------|-----------------|--------------------|
| CONNECT      | Reserved           | 0               | 0               | 0               | 0                  |
| CONNACK      | Reserved           | 0               | 0               | 0               | 0                  |
| PUBLISH      | Used in MQTT v5.0  | DUP<sup>1</sup> | QoS<sup>2</sup> | QoS<sup>2</sup> | RETAIN<sup>3</sup> |
| PUBACK       | Reserved           | 0               | 0               | 0               | 0                  |
| PUBREC       | Reserved           | 0               | 0               | 0               | 0                  |
| PUBREL       | Reserved           | 0               | 0               | 1               | 0                  |
| PUBCOMP      | Reserved           | 0               | 0               | 0               | 0                  |
| SUBSCRIBE    | Reserved           | 0               | 0               | 1               | 0                  |
| SUBACK       | Reserved           | 0               | 0               | 0               | 0                  |
| UNSUBSCRIBE  | Reserved           | 0               | 0               | 1               | 0                  |
| UNSUBACK     | Reserved           | 0               | 0               | 0               | 0                  |
| PINGREQ      | Reserved           | 0               | 0               | 0               | 0                  |
| PINGRESP     | Reserved           | 0               | 0               | 0               | 0                  |
| DISCONNECT   | Reserved           | 0               | 0               | 0               | 0                  |

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

##### 表 2.3 -包含报文标识符的MQTT控制报文 MQTT Control Packets that contain a Packet Identifier

| **MQTT控制报文** | **报文标识符字段**     |
|------------------|------------------------|
| CONNECT          | 不需要                 |
| CONNACK          | 不需要                 |
| PUBLISH          | 需要（如果QoS > 0） |
| PUBACK           | 需要                   |
| PUBREC           | 需要                   |
| PUBREL           | 需要                   |
| PUBCOMP          | 需要                   |
| SUBSCRIBE        | 需要                   |
| SUBACK           | 需要                   |
| UNSUBSCRIBE      | 需要                   |
| UNSUBACK         | 需要                   |
| PINGREQ          | 不需要                 |
| PINGRESP         | 不需要                 |
| DISCONNECT       | 不需要                 |
| AUTH             | 不需要                 |

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

Client | Server
---|---
PUBLISH | Packet Identifier=0x1234---
--PUBLISH | Packet Identifier=0x1234
PUBACK | Packet Identifier=0x1234---
--PUBACK | Packet Identifier=0x1234

### 2.2.2 属性 Properties

CONNECT，CONNACK，PUBLISH，PUBACK，PUBREC，PUBREL，PUBCOMP，SUBSCRIBE，SUBACK，UNSUBACK，DISCONNECT和AUTH报文可变报头的最后一部分是一组属性。CONNECT报文的遗嘱（Will）属性字段中也包含了一组可选的属性。

属性字段由属性长度和所有属性组成。

#### 2.2.2.1 属性长度 Property Length

属性长度被编码为变长字节整数。属性长度不包含用于编码属性长度自身的字节数，但包含所有属性的长度。如果没有任何属性，**必须**由属性长度为零的字段来指示 [MQTT-2.2.2-1]。

#### 2.2.2.2 属性 Property

一个属性包含一段数据和一个定义了属性用途和数据类型的标识符。标识符被编码为变长字节整数。任何控制报文，如果包含了：对于该报文类型无效的标识符，或者错误类型的数据，都是无效报文。收到无效报文时，服务端或客户端使用包含原因码0x81（无效报文）CONNACK或DISCONNECT报文进行错误处理，如4.13节 所述。标识符排序不分先后。

##### 表 2.4 – 属性 Properties

<div class=WordSection1 style='layout-grid:15.6pt'>

<table class=MsoNormalTable border=0 cellspacing=0 cellpadding=0 width=685
 style='width:513.9pt;border-collapse:collapse;mso-yfti-tbllook:1184;
 mso-padding-alt:0cm 5.4pt 0cm 5.4pt'>
 <tr style='mso-yfti-irow:0;mso-yfti-firstrow:yes;page-break-inside:avoid'>
  <td width=95 colspan=2 valign=top style='width:71.55pt;background:white;
  mso-background-themecolor:background1;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><b><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>标识符</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif";
  mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=188 rowspan=2 valign=top style='width:141.3pt;background:white;
  mso-background-themecolor:background1;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><b><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>属性名</span></b><b><span
  style='font-family:"Arial","sans-serif"'> </span></b><b><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>（用途）</span></b><b style='mso-bidi-font-weight:
  normal'><span lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:
  ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=150 rowspan=2 valign=top style='width:112.6pt;background:white;
  mso-background-themecolor:background1;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><b><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>数据类型</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif";
  mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=251 rowspan=2 valign=top style='width:188.45pt;background:white;
  mso-background-themecolor:background1;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><b><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>报文</span></b><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>/</span></b><b><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>遗嘱属性</span></b><b style='mso-bidi-font-weight:
  normal'><span lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:
  ZH-CN'><o:p></o:p></span></b></p>
  <p class=MsoNormal><b style='mso-bidi-font-weight:normal'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></b></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:1;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;background:white;mso-background-themecolor:
  background1;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><b><span lang=EN-US style='font-family:"Arial","sans-serif"'>Dec</span></b><b><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;background:white;mso-background-themecolor:
  background1;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><b><span lang=EN-US style='font-family:"Arial","sans-serif"'>Hex<o:p></o:p></span></b></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:2;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x01<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>载荷格式说明</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>PUBLISH,
  Will Properties<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:3;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>2<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x02<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>消息过期时间</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>四字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>PUBLISH,
  Will Properties<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:4;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>3<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x03<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>内容类型</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>编码字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>PUBLISH,
  Will Properties<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:5;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>8<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x08<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>响应主题</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>编码字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>PUBLISH,
  Will Properties<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:6;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>9<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x09<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>相关数据</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>二进制数据</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>PUBLISH,
  Will Properties<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:7;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>11<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x0B<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>定义标识符</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>变长字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>PUBLISH,
  SUBSCRIBE<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:8;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>17<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x11<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>会话过期间隔</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>四字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT,
  CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:9;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>18<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x12<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>分配客户标识符</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>编码字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:10;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>19<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x13<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>服务端保活时间</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>双字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:11;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>21<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x15<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>认证方法</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>编码字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT,
  CONNACK, AUTH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:12;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>22<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x16<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>认证数据</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>二进制数据</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT,
  CONNACK, AUTH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:13;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>23<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x17<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>请求问题信息</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:14;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>24<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x18<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>遗嘱延时间隔</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>四字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>Will
  Properties<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:15;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>25<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x19<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>请求响应信息</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:16;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>26<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x1A<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>请求信息</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>编码字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:17;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>28<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x1C<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>服务端参考</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>编码字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK,
  DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:18;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>31<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x1F<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='text-align:justify;text-justify:inter-ideograph'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>原因字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>编码字符串</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK,
  PUBACK, PUBREC, PUBREL, PUBCOMP, SUBACK, UNSUBACK, DISCONNECT, AUTH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:19;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>33<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x21<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>接收最大数量</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>双字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT,
  CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:20;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>34<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x22<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>主题别名最大长度</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>双字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT,
  CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:21;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>35<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x23<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>主题别名</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>双字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>PUBLISH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:22;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>36<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x24<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>最大</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>QoS<o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:23;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>37<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x25<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>保留属性可用性</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:24;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>38<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x26<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>用户属性</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>UTF-8</span><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>字符串对</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT,
  CONNACK, PUBLISH, Will Properties, PUBACK, PUBREC, PUBREL, PUBCOMP,
  SUBSCRIBE, SUBACK, UNSUBSCRIBE, UNSUBACK, DISCONNECT, AUTH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:25;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>39<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x27<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>最大报文长度</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>四字节整数</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNECT,
  CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:26;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>40<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x28<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>通配符订阅可用性</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:27;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>41<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x29<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>订阅标识符可用性</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:28;mso-yfti-lastrow:yes;page-break-inside:avoid'>
  <td width=45 valign=top style='width:33.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>42<o:p></o:p></span></p>
  </td>
  <td width=51 valign=top style='width:38.05pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>0x2A<o:p></o:p></span></p>
  </td>
  <td width=188 valign=top style='width:141.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>共享订阅可用性</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=150 valign=top style='width:112.6pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>字节</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=251 valign=top style='width:188.45pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
</table>

<p class=MsoNormal><span lang=EN-US><o:p>&nbsp;</o:p></span></p>

</div>

> **非规范评注**
>
> 尽管属性标识符用变长字节整数来表示，但在此版本协议中，所有的标识符均由一个字节来表示。


## 2.3 有效载荷 Payload

某些 MQTT 控制报文在报文的最后部分包含一个有效载荷，这将在第三章论述。对于 PUBLISH 来说有效载荷就是应用消息。

##### 表 2.5 – 包含有效载荷的MQTT控制报文 MQTT Control Packets that contain a Payload

| **MQTT控制报文** | **有效载荷** |
|------------------|--------------|
| CONNECT          | 需要         |
| CONNACK          | 不需要       |
| PUBLISH          | 可选         |
| PUBACK           | 不需要       |
| PUBREC           | 不需要       |
| PUBREL           | 不需要       |
| PUBCOMP          | 不需要       |
| SUBSCRIBE        | 需要         |
| SUBACK           | 需要         |
| UNSUBSCRIBE      | 需要         |
| UNSUBACK         | 不需要       |
| PINGREQ          | 不需要       |
| PINGRESP         | 不需要       |
| DISCONNECT       | 不需要       |

## 2.4 原因码 Reason Code

原因码是一个单字节无符号数，用来指示一次操作的结果。小于0x80的原因码指示某次操作成功完成，通常用0来表示。大于等于0x80的原因码用来指示操作失败。

CONNACK，PUBACK，PUBREC，PUBREL，PUBCOMP，DISCONNECT和AUTH控制报文的可变报头有一个单字节的原因码。SUBACK和UNSUBACK报文的载荷字段包含一个或多个原因码。

原因码如下表所示。

##### 表 2.6 – 原因码 Reason Code

<table class=MsoNormalTable border=0 cellspacing=0 cellpadding=0 width=673
 style='width:504.9pt;border-collapse:collapse;mso-yfti-tbllook:1184;
 mso-padding-alt:0cm 5.4pt 0cm 5.4pt'>
 <tr style='mso-yfti-irow:0;mso-yfti-firstrow:yes;page-break-inside:avoid'>
  <td width=113 colspan=2 valign=top style='width:84.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='margin:0cm;margin-bottom:.0001pt;
  text-align:center'><b><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>原因码</span></b><b><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=211 rowspan=2 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='margin:0cm;margin-bottom:.0001pt;
  text-align:center'><b><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>名称</span></b><b><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=349 rowspan=2 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='margin:0cm;margin-bottom:.0001pt;
  text-align:center'><b><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>报文</span></b><b><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  <p class=MsoNormal align=center style='margin:0cm;margin-bottom:.0001pt;
  text-align:center'><b><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></b></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:1;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='margin:0cm;margin-bottom:.0001pt;
  text-align:center'><b><span lang=EN-US style='font-family:"Arial","sans-serif"'>Decimal</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='margin:0cm;margin-bottom:.0001pt;
  text-align:center'><b><span lang=EN-US style='font-family:"Arial","sans-serif"'>Hex<o:p></o:p></span></b></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:2;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x00<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>成功</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, PUBACK, PUBREC, PUBREL,
  PUBCOMP, UNSUBACK, AUTH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:3;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x00<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>正常断开</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:4;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x00<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>授权的</span><span lang=EN-US style='font-family:
  "Arial","sans-serif"'>QoS 0<o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>SUBACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:5;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x01<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>授权的</span><span lang=EN-US style='font-family:
  "Arial","sans-serif"'>QoS 1<o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>SUBACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:6;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>2<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x02<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>授权的</span><span lang=EN-US style='font-family:
  "Arial","sans-serif"'>QoS 2<o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>SUBACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:7;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>4<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x04<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>包含遗嘱的断开</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:8;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>16<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x10<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>无匹配订阅</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>PUBACK, PUBREC<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:9;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>17<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x11<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>订阅不存在</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>UNSUBACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:10;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>24<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x18<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>继续认证</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>AUTH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:11;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>25<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x19<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>重新认证</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>AUTH<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:12;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>128<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x80<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>未指明的错误</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, PUBACK, PUBREC, SUBACK,
  UNSUBACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:13;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>129<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x81<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>无效报文</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:14;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>130<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x82<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>协议错误</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:15;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>131<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x83<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>实现错误</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, PUBACK, PUBREC, SUBACK,
  UNSUBACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:16;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>132<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x84<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>协议版本不支持</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:17;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>133<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x85<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>客户标识符无效</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:18;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>134<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x86<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>用户名密码错误</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:19;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>135<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x87<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>未授权</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, PUBACK, PUBREC, SUBACK,
  UNSUBACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:20;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>136<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x88<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>服务端不可用</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:21;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>137<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x89<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>服务端正忙</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:22;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>138<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x8A<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>禁止</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:23;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>139<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x8B<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>服务端关闭中</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:24;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>140<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x8C<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>无效的认证方法</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:25;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>141<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x8D<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>保活超时</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:26;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>142<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x8E<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>会话被接管</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:27;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>143<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x8F<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>主题过滤器无效</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>SUBACK, UNSUBACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:28;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>144<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x90<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>主题名无效</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, PUBACK, PUBREC, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:29;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>145<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x91<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>报文标识符已被占用</span><span lang=EN-US
  style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>PUBACK, PUBREC, SUBACK, UNSUBACK<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:30;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>146<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x92<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>报文标识符无效</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>PUBREL, PUBCOMP<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:31;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>147<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x93<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>接收超出最大数量</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:32;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>148<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x94<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>主题别名无效</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:33;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>149<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x95<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>报文过长</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:34;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>150<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x96<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>消息太过频繁</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:35;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>151<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x97<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>超出配额</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, PUBACK, PUBREC, SUBACK,
  DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:36;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>152<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x98<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>管理行为</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:37;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>153<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x99<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>载荷格式无效</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, PUBACK, PUBREC, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:38;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>154<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x9A<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>不支持保留</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:39;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>155<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x9B<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>不支持的</span><span lang=EN-US style='font-family:
  "Arial","sans-serif"'>QoS</span><span style='font-family:黑体;mso-ascii-font-family:
  Arial;mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>等级</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:40;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-size:10.5pt;font-family:"Arial","sans-serif";color:#333333'>156</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x9C<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>（临时）使用其他服务端</span><span lang=EN-US
  style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:41;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-size:10.5pt;font-family:"Arial","sans-serif";color:#333333'>157</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x9D<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>服务端已（永久）移动</span><span lang=EN-US
  style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:42;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-size:10.5pt;font-family:"Arial","sans-serif";color:#333333'>158<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x9E<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>不支持共享订阅</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>SUBACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:43;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-size:10.5pt;font-family:"Arial","sans-serif";color:#333333'>159<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0x9F<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>超出连接速率限制</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>CONNACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:44;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-size:10.5pt;font-family:"Arial","sans-serif";color:#333333'>160<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0xA0<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>最大连接时间</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:45;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-size:10.5pt;font-family:"Arial","sans-serif";color:#333333'>161<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0xA1<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>不支持订阅标识符</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>SUBACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:46;mso-yfti-lastrow:yes;page-break-inside:avoid'>
  <td width=66 valign=top style='width:49.2pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-size:10.5pt;font-family:"Arial","sans-serif";color:#333333'>162<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.3pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0xA2<o:p></o:p></span></p>
  </td>
  <td width=211 valign=top style='width:158.5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>不支持通配符订阅</span><span lang=EN-US style='font-family:
  "Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
  <td width=349 valign=top style='width:261.9pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal style='margin:0cm;margin-bottom:.0001pt'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>SUBACK, DISCONNECT<o:p></o:p></span></p>
  </td>
 </tr>
</table>

> **非规范评注**
>
> 对于原因码0x91（报文标识符已被占用）的处理可以为尝试修复会话、以新会话标志为1重置会话或者判定客户端或服务端实现有缺陷。

### 项目主页

- [MQTT协议中文版](https://github.com/hui6075/mqtt_v5)


