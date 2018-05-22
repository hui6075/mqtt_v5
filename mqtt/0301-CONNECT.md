## 3.1 CONNECT – 连接请求

客户端到服务端的网络连接建立后，客户端发送给服务端的第一个报文**必须**是CONNECT报文 \[MQTT-3.1.0-1\]。

在一个网络连接上，客户端只能发送一次CONNECT报文。服务端**必须**将客户端发送的第二个CONNECT报文当作协议违规处理并断开客户端的连接 \[MQTT-3.1.0-2\]。有关错误处理的信息请查看4.13节。

有效载荷包含一个或多个编码的字段。包括客户端的唯一标识符，Will主题，Will消息，用户名和密码。除了客户端标识之外，其它的字段都是可选的，基于标志位来决定可变报头中是否需要包含这些字段。

### 3.1.1 CONNECT 固定报头 CONNECT Fixed header

##### 图 3.1 – CONNECT报文的固定报头 CONNECT packet Fixed Header

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
     <td colspan="4" align="center">MQTT报文类型 (1)</td>
     <td colspan="4" align="center">Reserved 保留位</td>
   </tr>
   <tr>
     <td></td>
      <td align="center">0</td>
      <td align="center">0</td>
      <td align="center">0</td>
      <td align="center">1</td>
      <td align="center">0</td>
      <td align="center">0</td>
      <td align="center">0</td>
      <td align="center">0</td>
   </tr>
   <tr>
     <td>byte 2...</td>
     <td colspan="8" align="center">剩余长度</td>
   </tr>
 </table>

**剩余长度字段**

剩余长度等于可变报头的长度加上有效载荷的长度。编码方式为变长字节整数。

### 3.1.2 CONNECT 可变报头 CONNECT Variable header

CONNECT 报文的可变报头按下列次序包含四个字段：协议名（Protocol Name），协议级别（Protocol Level），连接标志（Connect Flags），保持连接（Keep Alive）和属性（Properties）。2.2.2节 描述了属性（Properties）编码规则。

#### 3.1.2.1 协议名 Protocol Name

##### 图 3.2 - 协议名字节 Protocol Name bytes

|        | **说明**     | **7** | **6** | **5** | **4** | **3** | **2** | **1** | **0** |
|--------|--------------|-------|-------|-------|-------|-------|-------|-------|-------|
| 协议名 |
| byte 1 | 长度 MSB (0) | 0     | 0     | 0     | 0     | 0     | 0     | 0     | 0     |
| byte 2 | 长度 LSB (4) | 0     | 0     | 0     | 0     | 0     | 1     | 0     | 0     |
| byte 3 | ‘M’          | 0     | 1     | 0     | 0     | 1     | 1     | 0     | 1     |
| byte 4 | ‘Q’          | 0     | 1     | 0     | 1     | 0     | 0     | 0     | 1     |
| byte 5 | ‘T’          | 0     | 1     | 0     | 1     | 0     | 1     | 0     | 0     |
| byte 6 | ‘T’          | 0     | 1     | 0     | 1     | 0     | 1     | 0     | 0     |

协议名是表示协议名*MQTT*的UTF-8编码的字符串。MQTT规范的后续版本不会改变这个字符串的偏移和长度。

支持多种协议的服务端使用协议名字段判断数据是否为MQTT报文。协议名**必须**是UTF-8字符串“MQTT”。如果服务端不愿意接受CONNECT但希望表明其MQTT服务端身份，**可以**发送包含原因码为0x84（不支持的协议版本）的CONNACK报文，然后**必须**关闭网络连接 \[MQTT-3.1.2-1\]。

> **非规范评注**
>
> 数据包检测工具，例如防火墙，可以使用协议名来识别MQTT流量。

#### 3.1.2.2 协议版本 Protocol Version

##### 图 3.3 - 协议级别字节 Protocol Version byte

|          | **说明** | **7** | **6** | **5** | **4** | **3** | **2** | **1** | **0** |
|----------|----------|-------|-------|-------|-------|-------|-------|-------|-------|
| 协议级别 |
| byte 7   | Level(4) | 0     | 0     | 0     | 0     | 0     | 1     | 0     | 1     |

客户端使用一个字节无符号数表示协议修订级别。MQTT v5.0的协议版本字段为5（0x05）。

支持多版本MQTT协议的服务端使用*协议版本*字段判定客户端正使用的MQTT协议版本。如果协议版本不是5且服务端不愿意接受此CONNECT报文，**可以**发送包含原因码0x84（不支持的协议版本）的CONNACK报文，然后**必须**关闭网络连接 \[MQTT-3.1.2-2\]。

#### 3.1.2.3 连接标志 Connect Flags

连接标志字节包含一些用于指定MQTT连接行为的参数。它还指出有效载荷中的字段是否存在。

##### 图 3.4 - 连接标志位 Connect Flag bits

<table class=MsoNormalTable border=1 cellspacing=0 cellpadding=0
 style='border-collapse:collapse;border:none;mso-border-alt:solid windowtext .5pt;
 mso-yfti-tbllook:1184;mso-padding-alt:0cm 5.4pt 0cm 5.4pt;mso-border-insideh:
 .5pt solid windowtext;mso-border-insidev:.5pt solid windowtext'>
 <tr style='mso-yfti-irow:0;mso-yfti-firstrow:yes'>
  <td width=65 valign=top style='width:48.55pt;border:solid windowtext 1.0pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Bit</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=85 valign=top style='width:63.65pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>7</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=84 valign=top style='width:63.3pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>6</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=85 valign=top style='width:63.8pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>5</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=47 valign=top style='width:35.45pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>4</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=47 valign=top style='width:35.45pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>3</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=76 valign=top style='width:2.0cm;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>2</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=78 valign=top style='width:58.3pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>1</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=71 valign=top style='width:53.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>0</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:1'>
  <td width=65 valign=top style='width:48.55pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  </td>
  <td width=85 valign=top style='width:63.65pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>User Name Flag<o:p></o:p></span></p>
  </td>
  <td width=84 valign=top style='width:63.3pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>Password Flag<o:p></o:p></span></p>
  </td>
  <td width=85 valign=top style='width:63.8pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>Will Retain<o:p></o:p></span></p>
  </td>
  <td width=95 colspan=2 valign=top style='width:70.9pt;border-top:none;
  border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>Will QoS<o:p></o:p></span></p>
  </td>
  <td width=76 valign=top style='width:2.0cm;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>Will Flag<o:p></o:p></span></p>
  </td>
  <td width=78 valign=top style='width:58.3pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>Clean Start<o:p></o:p></span></p>
  </td>
  <td width=71 valign=top style='width:53.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>Reserved<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:2;mso-yfti-lastrow:yes'>
  <td width=65 valign=top style='width:48.55pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  8<o:p></o:p></span></p>
  </td>
  <td width=85 valign=top style='width:63.65pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>X<o:p></o:p></span></p>
  </td>
  <td width=84 valign=top style='width:63.3pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>X<o:p></o:p></span></p>
  </td>
  <td width=85 valign=top style='width:63.8pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>X<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.45pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>X<o:p></o:p></span></p>
  </td>
  <td width=47 valign=top style='width:35.45pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>X<o:p></o:p></span></p>
  </td>
  <td width=76 valign=top style='width:2.0cm;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>X<o:p></o:p></span></p>
  </td>
  <td width=78 valign=top style='width:58.3pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>X<o:p></o:p></span></p>
  </td>
  <td width=71 valign=top style='width:53.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
</table>

服务端**必须**验证CONNECT控制报文的保留标志位（第0位）是否为0 \[MQTT-3.1.2-3\]，如果不为0则此报文为无效报文。4.13节给出了错误处理信息。

#### 3.1.2.4 新开始 Clean Start

**位置：** 连接标志字节的第1位

这个二进制位表明此次连接是一个新的会话还是一个已存在的会话的延续。4.1节定义了会话状态。

如果收到新开始（Clean Start）为1的CONNECT报文，客户端和服务端**必须**丢弃任何已存在的会话，并开始一个新的会话 \[MQTT-3.1.2-4\]。相应的，CONNACK报文中的会话存在标志设置为0。

如果收到新开始（Clean Start）为0的CONNECT报文，并且存在一个关联此客户标识符的会话，服务端**必须**基于此会话的状态恢复与客户端的通信 \[MQTT-3.1.2-5\]。如果收到新开始（Clean Start）为0的CONNECT报文，并且不存在任何关联此客户标识符的会话，服务端**必须**创建一个新的会话 \[MQTT-3.1.2-6\]。

#### 3.1.2.5 遗嘱标志 Will Flag

**位置：** 连接标志字节的第2位

如果遗嘱标志（Will Flag）被设置为1，表示遗嘱消息**必须**已存储在服务端与此客户标识符相关的会话中 \[MQTT-3.1.2-7\]。遗嘱消息（Will Message）包含遗嘱属性，遗嘱主题和遗嘱载荷字段。遗嘱**必须**在网络连接被关闭、遗嘱延时间隔到期或者会话结束之后被发布，除非服务端收到包含原因码为0x00（正常关闭）的DISCONNECT报文之后删除了遗嘱消息（Will Message），或者一个关于此客户标识符的新的网络连接在遗嘱迟发时间（Will Delay Interval）超时之前被创建 \[MQTT-3.1.2-8\]。

遗嘱消息发布的条件，包括但不限于：
-   服务端检测到了一个I/O错误或者网络故障
-   客户端在保持连接（Keep Alive）的时间内未能通讯
-   客户端在没有发送包含原因码0x00（正常关闭）的情况下关闭了网络连接
-   服务端在没有收到包含原因码0x00（正常关闭）的情况下关闭了网络连接

如果遗嘱标志（Will Flag）被设置为1，遗嘱属性（Will Property）、遗嘱主题（Will Topic）和遗嘱载荷（Will Payload）字段**必须**存在于报文有效载荷中 \[MQTT-3.1.2-9\]。一旦遗嘱消息（Will Message）被发布或者服务端收到包含原因码为0x00（正常关闭）的DISCONNECT报文，遗嘱消息（Will Message）**必须**从服务端的会话中删除 \[MQTT-3.1.2-10\]。

服务端**应该**在网络连接断开并且遗嘱迟发时间（Will Delay Interval）到期，或者会话结束之后立即发布遗嘱消息。服务端关闭或出错的情况下，**可以**在服务重新启动之后发布遗嘱消息（Will Message）。这种情况下从服务端出错到遗嘱发布之间存在一定的延迟。

关于遗嘱延时间隔（Will Delay Interval）的详细信息，请参考3.1.3.2节。

> **非规范评注**
>
> 通过设置晚于会话过期间隔（Session Expiry Interval）的遗嘱迟发时间（Will Delay Interval）并发送包含原因码0x04（包含遗嘱的断开连接），客户端得以发出会话过期（Session Expiry）通告。

#### 3.1.2.6 遗嘱 QoS Will QoS

**位置：** 连接标志字节的第3、4位

这两个比特指定了发布遗嘱消息（Will Message）时的服务质量（QoS）。

如果遗嘱标志（Will Flag）设置为0，遗嘱服务质量（Will QoS）**必须**也设置为0（0x00） \[MQTT-3.1.2-11\]。
如果遗嘱标志设置为1，遗嘱服务质量**可以**被设置为0（0x00），1（0x01）或2（0x02） \[MQTT-3.1.2-12\]。设置为3（0x03）的报文是无效报文。4.13节描述了错误处理信息。

#### 3.1.2.7 遗嘱保留 Will Retain

**位置：** 连接标志字节的第5位

此位指定遗嘱消息（Will Message）在发布时是否会被保留。

如果遗嘱标志被设置为0，遗嘱保留（Will Retain）标志也**必须**设置为0 \[MQTT-3.1.2-13\]。如果遗嘱标志被设置为1时，如果遗嘱保留被设置为0，则服务端**必须**将遗嘱消息当做非保留消息发布 \[MQTT-3.1.2-14\]。如果遗嘱保留被设置为1，则服务端**必须**将遗嘱消息当做保留消息发布 \[MQTT-3.1.2-15\]。

#### 3.1.2.8 用户名标志 User Name Flag

**位置：** 连接标志字节的第7位

如果用户名标志（User Name Flag）被设置为0，有效载荷中**不能**包含用户名字段 \[MQTT-3.1.2-16\]。如果用户名标志被设置为0，有效载荷中**必须**包含用户名字段 \[MQTT-3.1.2-17\]。

#### 3.1.2.9 密码标志 Password Flag

**位置：** 连接标志字节的第6位

如果密码标志（Password Flag）被设置为0，有效载荷中**不能**包含密码字段 \[MQTT-3.1.2-18\]。如果密码标志被设置为1，有效载荷中**必须**包含密码字段 \[MQTT-3.1.2-19\]。

> **非规范评注**
>
> 相比MQTT v3.1.1，此版本协议允许在没有用户名的情况下发送密码。这表明密码除了作为口令之外还可以有其他用途。

#### 3.1.2.10 保持连接 Keep Alive

##### 图 3.5 - 保持连接字节 Keep Alive bytes

| **Bit** | **7**                   | **6** | **5** | **4** | **3** | **2** | **1** | **0** |
|---------|-------------------------|-------|-------|-------|-------|-------|-------|-------|
| byte 9  | 保持连接 Keep Alive MSB |
| byte 10 | 保持连接 Keep Alive LSB |

保持连接（Keep Alive）使用双字节整数来表示以秒为单位的时间间隔。它是指在客户端传输完成一个MQTT控制报文的时刻到发送下一个报文的时刻，两者之间允许空闲的最大时间间隔。客户端负责保证控制报文发送的时间间隔不超过保持连接的值。如果没有任何其它的MQTT控制报文可以发送，客户端**必须**发送一个PINGREQ 报文 \[MQTT-3.1.2-20\]。

如果服务端返回的CONNACK报文中包含服务端保持连接（Server Keep Alive），客户端**必须**使用此值代替其发送的保持连接（Keep Alive） \[MQTT-3.1.2-21\]。

不管保持连接的值是多少，客户端任何时候都可以发送PINGREQ报文，并且使用PINGRESP报文判断网络和服务端的活动状态。

如果保持连接的值非零，并且服务端在1.5倍的保持连接时间内没有收到客户端的控制报文，它**必须**断开客户端的网络连接，并判定网络连接已断开 \[MQTT-3.1.2-22\]。

客户端发送了PINGREQ报文之后，如果在合理的时间内仍没有收到PINGRESP报文，它**应该**关闭到服务端的网络连接。

保持连接（Keep Alive）值为零的结果是关闭保持连接（Keep Alive）机制。如果保持连接（Keep Alive）值为零，客户端不必按照任何特定的时间发送MQTT控制报文。

> **非规范评注**
>
> 服务端可能因为其他原因断开客户端连接，比如服务端将要关闭服务。设置保持连接（Keep Alive）不保证客户端将一直保持连接状态。

> **非规范评注**
>
> 保持连接的实际值是由应用指定的，一般是几分钟。允许的最大值是18小时12分15秒。

#### 3.1.2.11 CONNECT 属性 CONNECT Properties

##### 3.1.2.11.1 属性长度 Property Length

CONNECT报文可变报头中的属性（Properties）长度被编码为变长字节整数。

##### 3.1.2.11.2 会话过期间隔 Session Expiry Interval

**17 (0x11)**，会话过期间隔（Session Expiry Interval）标识符。  
跟随其后的是用四字节整数表示的以秒为单位的会话过期间隔（Session Expiry Interval）。包含多个会话过期间隔（Session Expiry Interval）将造成协议错误（Protocol Error）。

如果会话过期间隔（Session Expiry Interval）值未指定，则使用0。如果设置为0或者未指定，会话将在网络连接（Network Connection）关闭时结束。 

如果会话过期间隔（Session Expiry Interval）为0xFFFFFFFF (UINT_MAX)，则会话永不过期。

如果网络连接关闭时会话过期间隔（Session Expiry Interval）大于0，则客户端与服务端**必须**存储会话状态 \[MQTT-3.1.2-23\]。

> **非规范评注**
>
> 客户端或服务端可能会因为中断运行导致会话时钟某些时间未运行。这将导致会话的删除被延迟。

更多关于会话的信息参考4.1节。关于会话存储的状态的详细和限制参考4.1.1节。

当会话过期时，客户端和服务端无需以原子操作的方式删除会话状态。

> **非规范评注**
>
> 把新开始（Clean Start）设置为1且会话过期间隔（Session Expiry Interval）设置为0，等同于在MQTT v3.1.1中把清理会话（CleanSession）设置为1。把新开始（Clean Start）设置为0且不设置会话过期间隔（Session Expiry Interval），等同于在MQTT v3.1.1中把清理会话标志设置为0。

> **非规范评注**
>
> 当希望只处理连接上服务端之后才发布的消息，客户端应该把新开始（Clean Start）设置为1且会话过期间隔（Session Expiry Interval）设置为0，这样客户端就不会收到它连接之前被服务端所发布的消息，并且需要每次连接上服务端时重新订阅其感兴趣的主题。

> **非规范评注**
>
> 某些客户端使用的网络可能只能提供断断续续的连接，这种客户端可以使用较短的会话过期间隔（Session Expiry Interval）以便在网络再次可用后重新连接到服务端时获得持续的消息交付。如果客户端不再重新连接，且允许会话过期，应用消息将会丢失。

> **非规范评注**
>
> 某个客户端设置较长的会话过期间隔（Session Expiry Interval）或设置会话不过期，即要求服务端为其保持会话到其下一次连接上服务端之后。只有打算在一段时间之后将会重连服务端时，客户端才应该设置较长的会话过期间隔（Session Expiry Interval）。当客户端认定其将来不会使用本次会话时，应该在断开时把会话过期间隔（Session Expiry Interval）设置为0。

> **非规范评注**
>
> 客户端应当使用CONNACK报文中的会话存在（Session Present）来判定服务端是否存储了其会话。

> **非规范评注**
>
> 客户端应当以服务端返回的会话存在（Session Present）标志来判定会话是否已过期，而不是客户端自己实现的会话过期状态。如果客户端自己实现会话过期状态，则需要将会话应当被删除的时间作为会话状态的一部分而存储。

##### 3.1.2.11.3 接收最大值 Receive Maximum

**33 (0x21)**，接收最大值（Receive Maximum）标识符。  
跟随其后的是由双字节整数表示的最大接收值。包含多个接收最大值或接收最大值为0将造成协议错误（Protocol Error）。

客户端使用此值限制客户端愿意同时处理的QoS为1和QoS为2的发布消息最大数量。没有机制可以限制服务端试图发送的QoS为0的发布消息。

接收最大值只将被应用在当前网络连接。如果没有设置最大接收值，将使用默认值65535。

关于接收最大值的详细使用，参考4.9节流控。

##### 3.1.2.11.4 最大报文长度 Maximum Packet Size

**39 (0x27)**，最大报文长度（Maximum Packet Size）标识符。  
跟随其后的是由四字节整数表示的客户端愿意接收的最大报文长度（Maximum Packet Size），如果没有设置最大报文长度（Maximum Packet Size），则按照协议由固定报头中的剩余长度可编码最大值和协议报头对数据包的大小做限制。

包含多个最大报文长度（Maximum Packet Size）或者最大报文长度（Maximum Packet Size）值为0将造成协议错误。

> **非规范评注**
>
> 客户端如果选择了限制最大报文长度，应该为最大报文长度设置一个合理的值。

如2.1.4节所述，最大报文长度是MQTT控制报文的总长度。客户端使用最大报文长度通知服务端其所能处理的单个报文长度限制。

服务端**不能**发送超过最大报文长度（Maximum Packet Size）的报文给客户端 \[MQTT-3.1.2-24\]。收到长度超过限制的报文将导致协议错误，客户端发送包含原因码0x95（报文过大）的DISCONNECT报文给服务端，详见4.13节。

当报文过大而不能发送时，服务端**必须**丢弃这些报文，然后当做应用消息发送已完成处理 \[MQTT-3.1.2-25\]。

共享订阅的情况下，如果一条消息对于部分客户端来说太长而不能发送，服务端可以选择丢弃此消息或者把消息发送给剩余能够接收此消息的客户端。

> **非规范评注**
>
> 服务端可以把那些没有发送就被丢弃的报文放在*死信队列*上，或者执行其他诊断操作。具体的操作超出了本规范的范围。

##### 3.1.2.11.5 主题别名最大值 Topic Alias Maximum

**34 (0x22)**，主题别名最大值（Topic Alias Maximum）标识符。  
跟随其后的是用双字节整数表示的主题别名最大值（Topic Alias Maximum）。包含多个主题别名最大值（Topic Alias Maximum）将造成协议错误（Protocol Error）。没有设置主题别名最大值属性的情况下，主题别名最大值默认为零。

此值指示了客户端能够接收的来自服务端的主题别名（Topic Alias）最大数量。客户端使用此值来限制本次连接可以拥有的主题别名的数量。服务端在一个PUBLISH报文中发送的主题别名**不能**超过客户端设置的主题别名最大值（Topic Alias Maximum） \[MQTT-3.1.2-26\]。值为零表示本次连接客户端不接受任何主题别名（Topic Alias）。如果主题别名最大值（Topic Alias）没有设置，或者设置为零，则服务端**不能**向此客户端发送任何主题别名（Topic Alias） \[MQTT-3.1.2-27\]。

##### 3.1.2.11.6 请求响应信息 Request Response Information

**25 (0x19)**，请求响应信息（Request Response Information）标识符。  
跟随其后的是用一个字节表示的0或1。包含多个请求响应信息（Request Response Information），或者请求响应信息（Request Response Information）的值既不为0也不为1会造成协议错误（Protocol Error）。如果没有请求响应信息（Request Response Information），则请求响应默认值为0。

客户端使用此值向服务端请求CONNACK报文中的响应信息（Response Information）。值为0，表示服务端**不能**返回响应信息 \[MQTT-3.1.2-28\]。值为1，表示服务端**可以**在CONNACK报文中返回响应信息。

> **非规范评注**
>
> 即使客户端请求响应信息（Response Information），服务端也可以选择不发送响应信息（Response Information）。

更多关于请求/响应信息的内容，请参考4.10节。

##### 3.1.2.11.7 请求问题信息 Request Problem Information

**23 (0x17)**，请求问题信息（Request Problem Information）标识符。  
跟随其后的是用一个字节表示的0或1。包含多个请求问题信息（Request Problem Information），或者请求问题信息（Request Problem Information）的值既不为0也不为1会造成协议错误（Protocol Error）。如果没有请求问题信息（Request Problem Information），则请求问题默认值为1。

客户端使用此值指示遇到错误时是否发送原因字符串（Reason String）或用户属性（User Properties）。 

如果请求问题信息的值为0，服务端**可以**选择在CONNACK或DISCONNECT报文中返回原因字符串（Reason String）或用户属性（User Properties），但**不能**在除PUBLISH，CONNACK或DISCONNECT之外的报文中发送原因字符串（Reason String）或用户属性（User Properties） \[MQTT-3.1.2-29\]。如果此值为0，并且在除PUBLISH，CONNACK或DISCONNECT之外的报文中收到了原因字符串（Reason String）或用户属性（User Properties），客户端将发送一个包含原因码0x82（协议错误）的DISCONNECT报文给服务端，如4.13节所述。

如果此值为1，服务端**可以**在任何被允许的报文中返回原因字符串（Reason String）或用户属性（User Properties）。

##### 3.1.2.11.8 用户属性 User Property

**38 (0x26)**，用户属性（User Property）标识符。  
跟随其后的是UTF-8字符串对。

用户属性（User Property）可以出现多次，表示多个名字/值对。相同的名字可以出现多次。 

> 非规范评注
>
> CONNECT报文中的用户属性可以被用来发送客户端到服务端的连接相关的属性。这些属性的意义本规范不做定义。

##### 3.1.2.11.9 认证方法 Authentication Method

**21 (0x15)**，认证方法（Authentication Method）标识符。
跟随其后的是一个UTF-8编码的字符串，包含了扩展认证的认证方法（Authentication Method）名称。包含多个认证方法将造成协议错误（协议错误）。
如果没有认证方法，则不进行扩展验证。参考4.12节。

如果客户端在CONNECT报文中设置了认证方法，则客户端在收到CONNACK报文之前**不能**发送除AUTH或DISCONNECT之外的报文 \[MQTT-3.1.2-30\]。

##### 3.1.2.11.10 认证数据 Authentication Data

**22 (0x16)**，认证数据（Authentication Data）标识符。
跟随其后的是二进制的认证数据。没有认证方法却包含了认证数据（Authentication Data），或者包含多个认证数据（Authentication Data）将造成协议错误（Protocol Error）。 

认证数据的内容由认证方法定义，关于扩展认证的更多信息，请参考4.12节。

#### 3.1.2.12 可变报头非规范示例 Variable Header non-normative example

##### 图 3.6 - 可变报头示例

<table class=MsoNormalTable border=1 cellspacing=0 cellpadding=0
 style='border-collapse:collapse;border:none;mso-border-alt:solid windowtext .5pt;
 mso-yfti-tbllook:1184;mso-padding-alt:0cm 5.4pt 0cm 5.4pt;mso-border-insideh:
 .5pt solid windowtext;mso-border-insidev:.5pt solid windowtext'>
 <tr style='mso-yfti-irow:0;mso-yfti-firstrow:yes'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>说明</span></b><b style='mso-bidi-font-weight:normal'><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>7</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>6</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>5</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>4</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>3</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>2</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>1</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border:solid windowtext 1.0pt;
  border-left:none;mso-border-left-alt:solid windowtext .5pt;mso-border-alt:
  solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>0</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:1'>
  <td width=638 colspan=10 valign=top style='width:478.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>协议名</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Protocol Name<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:2'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  1<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>长度</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Length MSB (0)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:3'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  2<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>长度</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Length LSB (4)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:4'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  3<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>‘M’<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:5'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  4<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>‘Q’<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:6'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  5<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>‘T’<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:7'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  6<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>‘T’<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:8'>
  <td width=638 colspan=10 valign=top style='width:478.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>Protocol
  Version<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:9'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  style='font-family:黑体;mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;
  mso-fareast-language:ZH-CN'>说明</span></b><b style='mso-bidi-font-weight:normal'><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>7</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>6</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>5</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>4</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>3</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>2</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>1</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><b><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>0</span></b><b
  style='mso-bidi-font-weight:normal'><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p></o:p></span></b></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:10'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  7<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>版本</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Version (5)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:11'>
  <td width=638 colspan=10 valign=top style='width:478.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>连接标志</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Connect Flags</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:12;height:139.5pt'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  8<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal style='line-height:150%'><span style='font-family:黑体;
  mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;mso-ansi-language:
  DE;mso-fareast-language:ZH-CN'>用户名标志</span><span lang=DE style='font-family:
  "Arial","sans-serif";mso-ansi-language:DE'>User Name Flag (1)<o:p></o:p></span></p>
  <p class=MsoNormal style='line-height:150%'><span style='font-family:黑体;
  mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;mso-ansi-language:
  DE;mso-fareast-language:ZH-CN'>密码标志</span><span lang=DE style='font-family:
  "Arial","sans-serif";mso-ansi-language:DE'>Password Flag (1)<o:p></o:p></span></p>
  <p class=MsoNormal style='line-height:150%'><span style='font-family:黑体;
  mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;mso-fareast-language:
  ZH-CN'>遗嘱保留标志</span><span lang=EN-US style='font-family:"Arial","sans-serif"'>Will
  Retain (0)<o:p></o:p></span></p>
  <p class=MsoNormal style='line-height:150%'><span style='font-family:黑体;
  mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;mso-fareast-language:
  ZH-CN'>遗嘱服务质量</span><span lang=EN-US style='font-family:"Arial","sans-serif";
  mso-fareast-language:ZH-CN'>Will QoS (01)<o:p></o:p></span></p>
  <p class=MsoNormal style='line-height:150%'><span style='font-family:黑体;
  mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;mso-fareast-language:
  ZH-CN'>遗嘱标志</span><span lang=EN-US style='font-family:"Arial","sans-serif"'>Will
  Flag (1)<o:p></o:p></span></p>
  <p class=MsoNormal style='line-height:150%'><span style='font-family:黑体;
  mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;mso-fareast-language:
  ZH-CN'>新开始</span><span lang=EN-US style='font-family:"Arial","sans-serif"'>Clean
  Start(1)<o:p></o:p></span></p>
  <p class=MsoNormal style='line-height:150%'><i><span style='font-family:黑体;
  mso-ascii-font-family:Arial;mso-hansi-font-family:Arial;mso-fareast-language:
  ZH-CN'>保留</span></i><i><span lang=EN-US style='font-family:"Arial","sans-serif"'>Reserved</span></i><span
  lang=EN-US style='font-family:"Arial","sans-serif"'> (0)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt;height:139.5pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'><o:p>&nbsp;</o:p></span></p>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:13'>
  <td width=638 colspan=10 valign=top style='width:478.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>保持连接</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Keep Alive</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:14'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  9<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>保持连接</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Keep Alive MSB (0)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:15'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  10<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>保持连接</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Keep Alive LSB (10)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:16'>
  <td width=638 colspan=10 valign=top style='width:478.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>属性</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Properties<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:17'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  11<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>长度</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'>Length (5)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:18'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  12<o:p></o:p></span></p>
  </td>
  <td width=248 valign=top style='width:186.2pt;border-top:none;border-left:
  none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>会话过期间隔标识符</span><span
  lang=EN-US style='font-family:"Arial","sans-serif"'> (17)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:19'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  13<o:p></o:p></span></p>
  </td>
  <td width=248 rowspan=4 valign=top style='width:186.2pt;border-top:none;
  border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span style='font-family:黑体;mso-ascii-font-family:Arial;
  mso-hansi-font-family:Arial;mso-fareast-language:ZH-CN'>会话过期间隔</span><span
  lang=EN-US style='font-family:"Arial","sans-serif";mso-fareast-language:ZH-CN'><o:p></o:p></span></p>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>Session
  Expiry Interval (10)<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:20'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  14<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:21'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  15<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
 <tr style='mso-yfti-irow:22;mso-yfti-lastrow:yes'>
  <td width=106 valign=top style='width:79.8pt;border:solid windowtext 1.0pt;
  border-top:none;mso-border-top-alt:solid windowtext .5pt;mso-border-alt:solid windowtext .5pt;
  padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal><span lang=EN-US style='font-family:"Arial","sans-serif"'>byte
  16<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>1<o:p></o:p></span></p>
  </td>
  <td width=35 valign=top style='width:26.6pt;border-top:none;border-left:none;
  border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;
  mso-border-top-alt:solid windowtext .5pt;mso-border-left-alt:solid windowtext .5pt;
  mso-border-alt:solid windowtext .5pt;padding:0cm 5.4pt 0cm 5.4pt'>
  <p class=MsoNormal align=center style='text-align:center'><span lang=EN-US
  style='font-family:"Arial","sans-serif"'>0<o:p></o:p></span></p>
  </td>
 </tr>
</table>

### 3.1.3 CONNECT 载荷 CONNECT Payload

CONNECT报文的载荷中包含由可变报头（Variable Header）中的标志确定的一个或多个以长度为前缀的字段。这些字段若存在，**必须**按照客户标识符（Client Identifier）、遗嘱属性（Will Properties）、遗嘱主题（Will Topic）、遗嘱载荷（Will Payload）、用户名（User Name）、密码（Password）的顺序出现 \[MQTT-3.1.3-1\]。

#### 3.1.3.1 客户标识符 Client Identifier

服务端使用客户标识符（ClientID）识别客户端。连接服务端的每个客户端都有唯一的客户标识符（ClientID）。客户端和服务端都**必须**使用客户标识符（ClientID）识别两者之间的 MQTT 会话相关的状态 \[MQTT-3.1.3-2\]。更多关于会话状态的信息请参考4.1节。

客户标识符**必须**存在，且作为CONNECT报文载荷的第一个字段出现 \[MQTT-3.1.3-3\]。

客户标识符**必须**被编码为1.5.4节 中所定义的UTF-8字符串 \[MQTT-3.1.3-4\]。

服务端**必须**允许1到23个字节长的UTF-8编码的客户标识符，客户标识符只能包含这些字符：
"0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"（大写字母、小写字母和数字） \[MQTT-3.1.3-5\]。

服务端**可以**允许编码后超过23个字节的客户标识符 (ClientID)。服务端**可以**允许包含不是上面列表字符
的客户标识符 (ClientID)。

服务端**可以**允许客户端提供一个零字节的客户标识符 (ClientID) ，如果这样做了，服务端**必须**将这看作特殊情况并分配唯一的客户标识符给那个客户端 \[MQTT-3.1.3-6\]。然后它**必须**假设客户端提供了那个唯一的客户标识符，正常处理这个CONNECT报文 \[MQTT-3.1.3-7\]。

如果服务端拒绝了某个客户标识符（ClientID），它**可以**发送包含原因码0x85（客户标识符无效）的CONNACK报文作为对客户端的CONNECT报文的回应，如4.13节所述。之后**必须**关闭网络连接 \[MQTT-3.1.3-8\]。

> **非规范评注**
>
> 客户端在实现时可以提供一个便于生成随机客户标识符的算法。使用此算法时，客户端需要注意避免创建长期孤儿会话。

#### 3.1.3.2 遗嘱属性 Will Properties

如果遗嘱标志（Will Flag）被设置为1，有效载荷的下一个字段是遗嘱属性（Will Properties）。遗嘱属性字段定义了遗嘱消息（Will Message）将何时被发布，以及被发布时的应用消息（Application Message）属性。遗嘱属性包括属性长度和属性。

##### 3.1.3.2.1 属性长度 Property Length

遗嘱属性（Will Properties）中的属性长度被编码为可变长字节整数。

##### 3.1.3.2.2 遗嘱延时间隔 Property Length

**24 (0x18)**，遗嘱延时间隔（Will Delay Interval）标识符。  
跟随其后的是由四字节整数表示的以秒为单位的遗嘱延时间隔（Will Delay Interval）。包含多个遗嘱延时间隔将造成协议错误（Protocol Error）。如果没有设置遗嘱延时间隔，遗嘱延时间隔默认值将为0，即不用延时发布遗嘱消息（Will Message）。

服务端将在遗嘱延时间隔（Will Delay Interval）到期或者会话（Session）结束时发布客户端的遗嘱消息（Will Message），取决于两者谁先发生。如果某个会话在遗嘱延时间隔到期之前创建了新的网络连接，则服务端**不能**发送遗嘱消息 \[MQTT-3.1.3-9\]。

> **非规范评注**
>
> 遗嘱时间间隔的一个用途是避免在频繁的网络连接临时断开时发布遗嘱消息，因为客户端往往会很快重新连上网络并继续之前的会话。

> **非规范评注**
>
> 如果某个连接到服务端的网络连接使用已存在的客户标识符，此已存在的网络连接的遗嘱消息将会被发布，除非新的网络连接设置了新开始（Clean Start）为0并且遗嘱延时大于0。如果遗嘱延时为0，遗嘱消息将在网络连接断开时发布。如果新开始为1，遗嘱消息也将被发布，因为此会话已结束。

##### 3.1.3.2.3 载荷格式指示 Payload Format Indicator

**1 (0x01)**，载荷格式指示（Payload Format Indicator）标识符。  
跟随载荷格式指示（Payload Format Indicator ）之后的可能是：
-    0 (0x00)，表示遗嘱消息（Will Message）是未指定的字节，等同于不发送载荷格式指示。
-    1 (0x01)，表示遗嘱消息（Will Message）是UTF-8编码的字符数据。载荷中的UTF-8数据**必须**按照Unicode规范[\[Unicode\]](http://www.unicode.org/versions/latest/)和RFC 3629 [\[RFC3629\]](http://www.rfc-editor.org/info/rfc3629)中的申明进行编码。

包含多个载荷格式指示（Payload Format Indicator）将造成协议错误（Protocol Error）。服务端**可以**按照格式指示对遗嘱消息（Will Message）进行验证，如果验证失败发送一条包含原因码0x99（载荷格式无效）的CONNACK报文。如4.13节所述。

##### 3.1.3.2.4 消息过期间隔 Message Expiry Interval

**2 (0x02)**，消息过期间隔（Message Expiry Interval）标识符。  
跟随其后的是表示消息过期间隔（Message Expiry Interval）的四字节整数。包含多个消息过期间隔将导致协议错误（Protocol Error）。

如果设定了消息过期间隔（Message Expiry Interval），四字节整数描述了遗嘱消息的生命周期（秒），并在服务端发布遗嘱消息时被当做发布过期间隔（Publication Expiry Interval）。

如果没有设定消息过期间隔，服务端发布遗嘱消息时将不发送消息过期间隔（Message Expiry Interval）。 

##### 3.1.3.2.5 内容类型 Content Type

**3 (0x03)**，内容类型（Content Type）标识符。  
跟随其后的是一个以UTF-8格式编码的字符串，用来描述遗嘱消息（Will Message）的内容。包含多个内容类型（Content Type）将造成协议错误（Protocol Error）。内容类型的值由发送应用程序和接收应用程序确定。

##### 3.1.3.2.6 响应主题 Response Topic

**8 (0x08)**，响应主题（Response Topic）标识符。  
跟随其后的是一个以UTF-8格式编码的字符串，用来表示响应消息的主题名（Topic Name）。包含多个响应主题（Response Topic）将造成协议错误。响应主题的存在将遗嘱消息（Will Message）标识为一个请求报文。 

更多关于请求/响应的内容，参考4.10节。

##### 3.1.3.2.7 对比数据 Correlation Data

**9 (0x09)**，对比数据（Correlation Data）标识符。  
跟随其后的是二进制数据。对比数据被请求消息发送端在收到响应消息时用来标识相应的请求。包含多个对比数据将造成协议错误（Protocol Error）。如果没有设置对比数据，则请求方（Requester）不需要任何对比数据。

对比数据只对请求消息（Request Message）的发送端和响应消息（Response Message）的接收端有意义。

更多关于请求/响应的内容，参考4.10节。

##### 3.1.3.2.8 用户属性 User Property

**38 (0x26)**，用户属性（User Property）标识符。
跟随其后的是一个UTF-8字符串对。用户属性（User Property）可以出现多次，表示多个名字/值对。相同的名字可以出现多次。

服务端在发布遗嘱消息（Will Message）时**必须**维护用户属性（User Properties）的顺序 \[MQTT-3.1.3-10\]。

> **非规范评注**
>
> 此属性旨在提供一种传递应用层名称-值标签的方法，其含义和解释仅由负责发送和接收它们的应用程序所有。

#### 3.1.3.3 遗嘱主题 Will Topic

如果遗嘱标志（Will Flag）被设置为1，遗嘱主题（Will Topic）为载荷中下一个字段。遗嘱主题（Will Topic）**必须**为UTF-8编码的字符串，如1.5.4节 所定义 \[MQTT-3.1.3-11\]。

#### 3.1.3.4 遗嘱载荷 Will Payload

如果遗嘱标志（Will Flag）被设置为1，遗嘱载荷（Will Payload）为载荷中下一个字段。遗嘱载荷定义了将要发布到遗嘱主题（Will Topic）的应用消息载荷，如3.1.2.5节所定义。此字段为二进制数据。

#### 3.1.3.5 用户名 User Name

如果用户名标志（User Name Flag）被设置为1，用户名（User Name）为载荷中下一个字段。用户名**必须**是1.5.4节定义的UTF-8 编码字符串 \[MQTT-3.1.3-12\]。服务端可以将它用于身份验证和授权。

#### 3.1.3.6 密码 Password

如果密码标志（Password Flag）被设置为1，密码（Password）为载荷中下一个字段。密码字段是二进制数据，尽管被称为密码，但可以被用来承载任何认证信息。

### 3.1.4 CONNECT 行为 CONNECT Actions

注意：服务器**可以**在同一个TCP端口或其他网络端点上支持多种协议（包括本协议的早期版本）。如果服务器确定协议是MQTT v5.0，那么它按照下面的方法验证连接请求。

1.	网络连接建立后，如果服务端在合理的时间内没有收到 CONNECT 报文，服务端**应该**关闭这个连接。
2.	服务端**必须**按照3.1节的要求验证CONNECT报文，如果报文不符合规范，服务端关闭网络连接 \[MQTT-3.1.4-1\]。服务端**可以**在关闭网络连接之前发送包含3.1.2.4节所述的0x80及以上原因码的CONNACK报文。 
3.	服务端**可以**检查CONNECT报文的内容是不是满足任何进一步的限制，**应该**执行身份验证和授权检查。如果任何一项检查没通过，服务端**必须**关闭网络连接 \[MQTT-3.1.4-2\]。在关闭网络连接之前，服务端**可以**发送一个合适的包含如3.2节和4.13节所述的0x80及以上原因码的CONNACK报文。

如果验证成功，服务端会执行下列步骤。

1.	如果客户标识符（ClientID）所代表的客户端已经连接到此服务端，那么向原有的客户端发送一个包含原因码为0x8E（会话被接管）的DISCONNECT报文，并且**必须**关闭原有的网络连接 \[MQTT-3.1.4-3\]。如果原有客户端存在遗嘱消息（Will Message），遗嘱消息按照 3.1.2.5节所描述的方式发布。

> **非规范评注**
>
> 如果原有网络连接包含遗嘱消息，且遗嘱延时间隔为0，则遗嘱消息会在此网络连接被关闭时发送。如果原有网络连接会话过期间隔为0，或者新网络连接新开始标志设置为1且原有网络连接包含遗嘱消息，则遗嘱消息会被发送，因为原有会话已结束。

2.	服务端**必须**按照3.1.2.4节所描述的方式对新开始标志进行处理 \[MQTT-3.1.4-4\]。

3.	服务端**必须**使用包含原因码为0x00（成功）的CONNACK报文对客户端的CONNECT报文进行确认 \[MQTT-3.1.4-5\]。

> **非规范评注**
>
> 如果服务端被用来处理商业关键数据，推荐对网络连接进行认证和授权。如果认证和授权成功，服务端可通过发送包含原因码为0x00（成功）的CONNACK报文进行响应，否则建议服务端根本不要发送CONNACK报文，因为这是一种潜在的对MQTT服务端的攻击，可以被用来进行拒绝服务攻击或密码猜测攻击。

4.	开始消息分发和保持连接状态监视。

允许客户端在发送CONNECT报文之后立即发送其它的MQTT控制报文；客户端不需要等待服务端的CONNACK报文。如果服务端拒绝了CONNECT报文，它**不能**处理客户端在CONNECT报文之后发送的任何除AUTH以外的报文 \[MQTT-3.1.4-6\]。

> **非规范评注**
>
> 客户端通常会等待CONNACK报文。然而，如果在收到CONNACK报文之前就自由的发送其它MQTT控制报文将会简化客户端的实现，因为它不必监督连接的状态。如果连接被拒绝了，客户端在接收CONNACK报文之前发送的任何数据将不会被服务端所处理。

> **非规范评注**
>
> 选择在收到CONNACK报文之前就发送MQTT控制报文的客户端将不知道服务端所存在的约束以及会话是否被使用。

> **非规范评注**
>
> 服务端在对某个客户端完成认证之前，可以选择限制读取该客户端的网络数据或者关闭该客户端的网络连接。这是一种避免拒绝服务攻击的方法。


### 第三章目录 MQTT控制报文

- [3.0 Contents – MQTT控制报文](03-ControlPackets.md)
- [3.1 CONNECT – 连接请求](0301-CONNECT.md)
- [3.2 CONNACK – 确认连接请求](0302-CONNACK.md)
- [3.3 PUBLISH – 发布消息](0303-PUBLISH.md)
- [3.4 PUBACK –发布确认](0304-PUBACK.md)
- [3.5 PUBREC – 发布已接收（QoS 2，第一步）](0305-PUBREC.md)
- [3.6 PUBREL – 发布释放（QoS 2，第二步）](0306-PUBREL.md)
- [3.7 PUBCOMP – 发布完成（QoS 2，第三步）](0307-PUBCOMP.md)
- [3.8 SUBSCRIBE - 订阅请求](0308-SUBSCRIBE.md)
- [3.9 SUBACK – 订阅确认](0309-SUBACK.md)
- [3.10 UNSUBSCRIBE –取消订阅](0310-UNSUBSCRIBE.md)
- [3.11 UNSUBACK – 取消订阅确认](0311-UNSUBACK.md)
- [3.12 PINGREQ – PING请求](0312-PINGREQ.md)
- [3.13 PINGRESP – PING响应](0313-PINGRESP.md)
- [3.14 DISCONNECT – 断开通知](0314-DISCONNECT.md)
- [3.15 AUTH – 认证交换](0315-AUTH.md)

### 项目主页

- [MQTT v5.0协议草案中文版](https://github.com/hui6075/mqtt_v5)

