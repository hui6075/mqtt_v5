## 3.3 PUBLISH – 发布消息 Publish message

PUBLISH报文是指从客户端向服务端或者服务端向客户端传输一个应用消息。

### 3.3.1 PUBLISH 固定报头 PUBLISH Fixed Header

##### 图 3-8 – PUBLISH报文固定报头 PUBLISH packet Fixed Header

<table>
   <tr>
     <th width="95">Bit</th>
     <th width="55">7</th>
     <th width="55">6</th>
     <th width="55">5</th>
     <th width="55">4</th>
     <th width="55">3</th>
     <th width="55">2</th>
     <th width="55">1</th>
     <th width="55">0</th>
   </tr>
   <tr>
     <td>byte 1</td>
     <td colspan="4" align="center"> MQTT控制报文类型 (3)</td>
     <td align="center">DUP</td>
     <td align="center" colspan="2">QoS等级</td>
     <td align="center">RETAIN</td>
   </tr>
    <tr>
       <td></td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">1</td>
       <td align="center">X</td>
       <td align="center">X</td>
       <td align="center">X</td>
       <td align="center">X</td>
     </tr>
   <tr>
     <td>byte 2...</td>
     <td colspan="8" align="center">剩余长度</td>
   </tr>
 </table>

#### 3.3.1.1 重发标志 DUP

**位置：** 第1个字节，第3位

如果DUP标志被设置为0，表示这是客户端或服务端第一次请求发送这个PUBLISH报文。如果DUP标志被设置为1，表示这可能是一个早前报文请求的重发。

客户端或服务端请求重发一个PUBLISH报文时，**必须**将DUP标志设置为1 [MQTT-3.3.1-1]。对于QoS为0的消息，DUP标志**必须**设置为0 [MQTT-3.3.1-2]。

服务端发送PUBLISH报文给订阅者时，收到（入站）的PUBLISH报文的DUP标志的值不会被传播。发送（出站）的PUBLISH报文与收到（入站）的PUBLISH报文中的DUP标志是独立设置的，它的值**必须**单独的根据发送（出站）的PUBLISH报文是否是一个重发来确定 [MQTT-3.3.1-3]。

> **非规范评注**
>
> 接收者收到一个DUP标志位1的MQTT控制报文时，不能假设它看到了一个这个报文之前的一个副本。

> **非规范评注**
>
> 需要特别指出的是，DUP标志关注的是MQTT控制报文本身，与它包含的应用消息无关。当使用QoS 1时，客户端可能会收到一个DUP标志为0的PUBLISH 报文，这个报文包含一个它之前收到过的应用消息的副本，但是用的是不同的报文标识符。 2.2.1节提供了有关报文标识符的更多信息。

#### 3.3.1.2 服务质量等级 QoS

**位置：** 第1个字节，第2-1位。

这个字段表示应用消息分发的服务质量等级保证。服务质量等级在下表中列出。

##### 表格 3-2 - 服务质量定义 QoS definitions

<table>
  <tr>
    <th width=100>QoS值</th>
	<th>Bit 2</th>
	<th>Bit 1</th>
	<th width=360>说明</th>
  </tr>
  <tr>
    <td align="center">0</td>
	<td align="center">0</td>
	<td align="center">0</td>
	<td>最多分发一次</td>
  </tr>
  <tr>
    <td align="center">1</td>
	<td align="center">0</td>
	<td align="center">1</td>
	<td>至少分发一次</td>
  </tr>
  <tr>
    <td align="center">2</td>
	<td align="center">1</td>
	<td align="center">0</td>
	<td>只分发一次</td>
  </tr>
  <tr>
    <td align="center">-</td>
	<td align="center">1</td>
	<td align="center">1</td>
	<td>保留位</td>
  </tr>
</table>

如果服务端在对客户端响应的CONNACK报文中包含了最大服务质量（Maximum QoS）且服务端收到的PUBLISH报文的QoS大于此最大服务质量，服务端发送包含原因码为0x9B（不支持的QoS等级）的DISCONNECT报文，如4.13节所述。

PUBLISH报文的2个QoS比特位**不能**同时设置为1 \[MQTT-3.3.1-4\]。如果服务端或客户端收到QoS 2个比特位都为1的无效PUBLISH报文，使用包含原因码为0x81（无效报文）的DISCONNECT报文关闭网络连接，如4.13节所述。

#### 3.3.1.3 保留标志 RETAIN

**位置：** 第1个字节，第0位。

如果客户端发给服务端的PUBLISH报文的保留（Retain）标志被设置为1，服务端**必须**存储此应用消息，并用其替换此话题下任何已存在的消息 \[MQTT-3.3.1-5\]，以便它可以被分发给未来的匹配此主题名（Topic Name）的订阅者。如果载荷为空，消息可以正常被服务端所处理，但是此话题下的任何保留消息**必须**被丢弃，并且此话题未来的订阅者将不会收到保留消息 \[MQTT-3.3.1-6\]。载荷为空的保留消息将**不能**被存储在服务端 \[MQTT-3.3.1-7\]。

如果客户端发给服务端的PUBLISH报文的保留标志位为0，服务器**不能**把此消息存储为保留消息，也不能丢弃或替换任何已存在的保留消息 \[MQTT-3.3.1-8\]。

如果服务端发送给客户端的CONNACK报文中包含保留可用属性，且属性值为0，但收到的PUBLISH报文中保留标志位为1，服务端使用包含原因码为0x9A（保留不支持）的DISCONNECT报文断开网络连接，如4.13节所述。

当一个新的非共享订阅（Non-shared Subscription）被创建时，每个匹配的话题下的最新保留消息如果存在，将根据保留消息订阅选项（Retain Handling Subscription Option）发送给客户端。这些消息在发送时保留标志被设置为1。保留消息的发送由保留消息处理订阅选项控制，收到订阅时：
-    如果保留消息处理属性被设置为0，服务端**必须**发送主题与客户端订阅的主题过滤器（Topic Filter）相匹配的所有保留消息 \[MQTT-3.3.1-9\]。
-    如果保留消息处理属性被设置为1，如果尚不存在匹配的订阅，服务端**必须**发送主题与客户端订阅的主题过滤器相匹配的所有保留消息。如果已存在相匹配的订阅，服务器**不能**发送这些保留消息 \[MQTT-3.3.1-10\]。
-    如果保留消息处理属性被设置为2，服务器**不能**发送这些保留消息 \[MQTT-3.3.1-11\]。

订阅选项（Subscription Options）的定义，参考3.8.3.1节。

如果服务端收到保留标志设置为1且QoS设置为0的PUBLISH报文，服务端**应该**把此QoS为0的消息存储为其主题下最新的保留消息，但服务端**可以**选择在任何时间丢弃此消息。如果发生丢弃，该主题下将不存在任何保留消息。 

如果某个主题当前的保留消息过期，该主题下将不存在任何保留消息。

服务端转发应用消息时，保留标志位的设置由发布保留（Retain As Published）订阅选项决定。订阅选项的定义，请参考3.8.3.1节。

-    如果发布保留（Retain As Published）订阅选项被设置为0，服务端在转发应用消息时**必须**将保留标志设置为0，而不管收到的PUBLISH报文中保留标志位如何设置的 \[MQTT-3.3.1-12\]。
-    如果发布保留（Retain As Published）订阅选项被设置为1，服务端在转发应用消息时**必须**将保留标志设置为与收到的PUBLISH消息中的保留标志位相同 \[MQTT-3.3.1-13\]。

> **非规范评注**
>
> 对于发布者不定期发送状态消息这个场景，保留消息很有用。新的非共享订阅者将会收到最近的状态。

#### 3.3.1.4 剩余长度 Remaining Length

等于可变报头的长度加上有效载荷的长度，被编码为变长字节整数。

### 3.3.2 PUBLISH可变报头 PUBLISH Variable Header

PUBLISH报文可变报头按顺序包含：主题名（Topic Name），报文标识符（Packet Identifier），属性（Properties）。属性的编码规则如2.2.2节所述。

#### 3.3.2.1 主题名 Topic Name

主题名（Topic Name）用于识别有效载荷数据应该被发布到哪一个信息通道。

主题名**必须**是PUBLISH报文可变报头的第一个字段。它**必须**是 1.5.4节定义的UTF-8编码的字符串 \[MQTT-3.3.2-1\]。

PUBLISH报文中的主题名**不能**包含通配符 \[MQTT-3.3.2-2\]。

服务端发送给订阅客户端的PUBLISH报文中的主题名**必须**匹配该订阅的主题过滤器（Topic Filter）， 如4.7节所定义的匹配过程 \[MQTT-3.3.2-3\]。然而，由于服务端允许将主题名映射为其他名字，主题名可能与原始PUBLISH报文中的主题名不同。

发送端可以使用主题别名（Topic Alias）以便减少PUBLISH报文的长度。主题别名如3.3.2.3.4节所述。主题名长度为0且没有主题别名，将造成协议错误（Protocol Error）。

#### 3.3.2.2 报文标识符 Packet Identifier

只有当QoS等级是1或2时，报文标识符（Packet Identifier）字段才能出现在PUBLISH报文中。2.2.1节提供了有关报文标识符的更多信息。

#### 3.3.2.3 PUBLISH 属性 PUBLISH Properties

##### 3.3.2.3.1 属性长度 Property Length

PUBLISH报文可变报头中的属性长度被编码为变长字节整数。

##### 3.3.2.3.2 载荷格式指示 Payload Format Indicator

**1 (0x01)**，载荷格式指示（Payload Format Indicator）标识符。  
跟随其后的是单字节的载荷格式指示值，可以是：
-    0 (0x00)，说明载荷是未指定格式的字节，相当于没有发送载荷格式指示。
-    1 (0x01)，说明载荷是UTF-8编码的字符数据。载荷中的UTF-8数据**必须**是按照Unicode [\[Unicode\]](http://www.unicode.org/versions/latest/)的规范和RFC 3629 [\[RFC3629\]](http://www.rfc-editor.org/info/rfc3629)的重申进行编码。

服务端**必须**把接收到的应用消息中的载荷格式指示原封不动的发给所有的订阅者 [MQTT-3.3.2-4]。接收者**可以**验证载荷数据与所指示的格式一致，如果不一致，发送包含原因码为0x99（载荷格式无效）的PUBACK，PUBREC或DISCONNECT报文，如4.13节所述。

##### 3.3.2.3.3 消息过期间隔 Message Expiry Interval

**2 (0x02)**，消息过期间隔（Message Expiry Interval）标识符。  
跟随其后的是四字节整数表示的消息过期间隔（Message Expiry Interval）。

如果消息过期间隔存在，四字节整数表示以秒为单位的应用消息（Application Message）生命周期。如果消息过期间隔（Message Expiry Interval）已过期，服务端还没开始向匹配的订阅者交付该消息，则服务端**必须**删除该订阅者的消息副本 [MQTT-3.3.2-5]。

如果消息过期间隔不存在，应用消息不会过期。
 
服务端发送给客户端的PUBLISH报文中**必须**包含消息过期间隔，值为接收时间减去消息在服务端的等待时间 \[MQTT-3.3.2-6\]。关于状态存储的细节和限制，参考4.1节。

##### 3.3.2.3.4 主题别名 Topic Alias

**35 (0x23)**，主题别名（Topic Alias）标识符。  
跟随其后的是表示主题别名（Topic Alias）值的双字节整数。包含多个主题别名值将造成协议错误（Protocol Error）。

主题别名是一个整数，用来代替主题名对主题进行识别。主题别名可以减小PUBLISH报文的长度，这对某个网络连接中发送的很长且反复使用的主题名来说很有用。

发送端决定是否使用主题别名及别名值如何选取。发送端通过在PUBLISH报文中包含的非0长度主题名和主题别名来设置主题别名映射。接收端正常处理该PUBLISH报文，但同样将指定的主题别名映射到主题名。

如果接收端已经设置了某个主题别名映射，发送端可以发送包含主题别名和长度为0的主题名的PUBLISH报文。接收端把此PUBLISH报文的主题名当做其包含的主题别名所映射的主题名。 

发送端可以通过在同一个网络连接中发送另一个包含同样主题别名和不同非0长度主题名的PUBLISH报文来修改主题别名映射关系。

主题别名映射仅作用于某个网络连接及其生命周期内。接收端**不能**将任何主题别名映射从一个网络连接转发到另一个网络连接 \[MQTT-3.3.2-7\]。

主题别名不允许为0。发送端**不能**发送包含主题别名值为0的PUBLISH报文 [MQTT-3.3.2-8]。

客户端**不能**发送主题别名值大于服务端的CONNACK报文中指定的主题别名最大值（Topic Alias Maximum）的PUBLISH报文 \[MQTT-3.3.2-9\]。客户端**必须**接受所有值大于0且小于等于其发送的CONNECT报文中的主题别名最大值的主题别名 \[MQTT-3.3.2-10\]。

服务端**不能**发送包含主题别名值大于客户端在CONNECT报文中指定的主题别名最大值（Topic Alias Maximum）的PUBLISH报文 \[MQTT-3.3.2-11\]。服务端**必须**接受所有值大于0且小于等于其发送的CONNACK报文中的主题别名最大值的主题别名 \[MQTT-3.3.2-12\]。

客户端和服务端使用的主题别名映射相互独立。因此一般来说，客户端发送给服务端的主题别名值为1的PUBLISH报文和服务端发送给客户端的主题别名值为1的PUBLISH报文，将被映射到不同的主题。

##### 3.3.2.3.5 响应主题 Response Topic

**8 (0x08)**，响应主题（Response Topic）标识符。  
跟随其后的是一个UTF-8编码的字符串，用作响应消息的主题名。响应主题**必须**是按照1.5.4节所定义的UTF-8编码的字符串 [MQTT-3.3.2-13]。响应主题**不能**包含通配符 \[MQTT-3.3.2-14\]。包含多个响应主题将造成协议错误（Protocol Error）。响应主题的存在将消息标识为请求报文。 

更多关于请求/响应的信息，参考4.10节 。

服务端在收到应用消息时**必须**将响应主题原封不动的发送给所有的订阅者 [MQTT-3.3.2-15]。

> **非规范评注**
>
> 包含响应主题的应用消息接收端使用响应主题作为主题名，发送作为响应消息的PUBLISH报文。如果请求消息中包含对比数据，接收端应当在发送作为对此请求消息进行响应的PUBLISH报文中包含此对比数据。

##### 3.3.2.3.6 对比数据 Correlation Data

**9 (0x09)**，对比数据（Correlation Data）标识符。  
跟随其后的是二进制数据。对比数据被请求消息发送端在收到响应消息时用来标识相应的请求。包含多个对比数据将造成协议错误（Protocol Error）。如果没有设置对比数据，则请求方（Requester）不需要任何对比数据。

服务端在收到应用消息时**必须**原封不动的把对比数据发送给所有的订阅者 [MQTT-3.3.2-16]。对比数据只对请求消息（Request Message）的发送端和响应消息（Response Message）的接收端有意义。

> **非规范评注**
>
>  接收端收到包含响应主题和对比数据的应用消息时，发送以响应主题为主题名的PUBLISH报文作为响应消息。客户端在响应消息中应将对比数据作为PUBLISH报文的一部分原封不动的发送出去。

> **非规范评注**
>
>  如果对客户端响应消息中的对比数据所做的任何更改会造成应用程序错误，则应当对对比数据进行加密/哈希，以便接收端能检测到对比数据是否被更改。

更多关于请求/响应的信息，请参考4.10节。

##### 3.3.2.3.7 用户属性 Property Length

**38 (0x26)**，用户属性（User Property）。  
跟随其后的是UTF-8字符串对。用户属性（User Property）允许出现多次，以表示多个名字/值对，且相同的名字可以多次出现。

服务端在转发应用消息到客户端时**必须**原封不动的把所有的用户属性放在PUBLISH报文中 \[MQTT-3.3.2-17\]。服务端在转发应用消息时**必须**保持所有用户属性的先后顺序 \[MQTT-3.3.2-18\]。

> **非规范评注**
>
> 此属性旨在提供一种传递应用层名称-值标签的方法，其含义和解释仅由负责发送和接收它们的应用程序所有。

##### 3.3.2.3.8 订阅标识符 Subscription Identifier

**11 (0x0B)**，订阅标识符（Subscription Identifier）标识符。  
跟随其后的是一个变长字节整数表示的订阅标识符。

订阅标识符取值范围从1到268,435,455。订阅标识符的值为0将造成协议错误。如果某条发布消息匹配了多个订阅，则将包含多个订阅标识符。这种情况下他们的顺序并不重要。

##### 3.3.2.3.9 内容类型 Content Type

**3 (0x03)**， 内容类型（Content Type）标识符。  
跟随其后的是一个以UTF-8格式编码的字符串，用来描述应用消息的内容。内容类型**必须**是UTF-8编码的字符串，如1.5.4节所定义 \[MQTT-3.3.2-19\]。
包含多个内容类型将造成协议错误（Protocol Error）。内容类型的值由发送应用程序和接收应用程序确定。

服务端**必须**把收到的应用消息中的内容类型原封不动的发送给所有的订阅者 \[MQTT-3.3.2-20\]。

> **非规范评注**
>
> UTF-8编码字符串可以使用一个MIME内容类型字符串来描述应用消息的内容。由于发送程序和接收程序负责内容类型字符串的定义和解释，因此MQTT服务端只确保内容类型是有效的UTF-8编码的字符串，不会做其他方面的验证。

> **非规范评注**
>
> 图3-9是一个PUBLISH示例报文，其中主题名为a/b，报文标识符为10，没有属性。

图 3-9 - PUBLISH报文可变报头非规范示例 PUBLISH packet Variable Header non-normative example

<table>
  <tr>
    <th width=120></th>
	<th width=280>说明</th>
	<th>7</th>
	<th>6</th>
	<th>5</th>
	<th>4</th>
	<th>3</th>
	<th>2</th>
	<th>1</th>
	<th>0</th>
  </tr>
  <tr>
    <td align="center" colspan="10">主题名</td>
  </tr>
  <tr>
    <td>byte 1</td>
	<td>长度MSB (0)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>byte 2</td>
	<td>长度LSB (3)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 3</td>
	<td>‘a’ (0x61)</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 4</td>
	<td>‘/’ (0x2F)</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
	<td>1</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 5</td>
	<td>‘b’ (0x62)</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
  </tr>
  <tr>
    <td align="center" colspan="10">报文标识符</td>
  </tr>
  <tr>
    <td>byte 6</td>
	<td>报文标识符MSB (0)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>byte 7</td>
	<td>报文标识符LSB (10)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
  </tr>
  <tr>
    <td align="center" colspan="10">属性长度</td>
  </tr>
  <tr>
    <td>byte 8</td>
	<td>无属性</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
</table>

### 3.3.3 PUBLISH 载荷 PUBLISH Payload

载荷包含将被发布的应用消息。载荷的内容和格式由应用程序指定。有效载荷的长度这样计算：用固定报头中的剩余长度字段的值减去可变报头的长度。包含零长度有效载荷的PUBLISH报文是合法的。

### 3.3.4 PUBLISH 行为 PUBLISH Actions

PUBLISH报文的接收端**必须**按照PUBLISH报文中的QoS等级发送响应报文 \[MQTT-3.3.4-1\]。

表 3-3 - PUBLISH报文的预期响应 Expected PUBLISH packet response

<table>
  <tr>
    <th>服务质量等级</th>
	<th width=280>预期响应</th>
  </tr>
  <tr>
    <td>QoS 0</td>
	<td>无响应</td>
  </tr>
  <tr>
    <td>QoS 1</td>
	<td>PUBACK报文</td>
  </tr>
  <tr>
    <td>QoS 2</td>
	<td>PUBREC报文</td>
  </tr>
</table>

客户端使用PUBLISH报文发送应用消息给服务端，目的是分发到其他订阅匹配的客户端。

服务端使用PUBLISH报文发送应用消息给每一个订阅匹配的客户端。PUBLISH报文包含SUBSCRIBE报文中承载的订阅标识符--如果存在的话。 

客户端使用带通配符的主题过滤器请求订阅时，客户端的订阅可能会重叠，因此发布的消息可能会匹配多个主题过滤器。这种情况下，服务端**必须**按照所有匹配的订阅中最大的QoS等级把消息发送给客户端 \[MQTT-3.3.4-2\]。此外，服务端**可以**为每一个匹配的订阅按照订阅时的QoS等级，把消息副本分发给客户端。

如果客户端收到一个未经请求的应用消息（没有匹配任何订阅），且QoS大于客户端指定的最大服务质量（Maximum QoS），客户端使用包含原因码为0x9B（不支持的QoS等级）的DISCONNECT报文断开连接，如4.13节所述。
 
如果客户端在这些重叠的订阅中指定了订阅标识符，服务端在发布这些订阅相匹配的消息时**必须**包含这些订阅标识符 \[MQTT-3.3.4-3\]。如果服务端对这些重叠的订阅只发送一条相匹配的消息，服务端**必须**在PUBLISH报文中包含所有的相匹配的订阅标识符（如果存在），但没有顺序要求 \[MQTT-3.3.4-4\]。如果服务端对这些重叠的订阅**必须**分别发送相匹配的消息，则每个PUBLISH报文中含与订阅相匹配的订阅标识符（如果存在） \[MQTT-3.3.4-5\]。

可能存在客户端对同一个发布消息做了多次订阅，并且这些订阅中有多个订阅使用了相同的订阅标识符，这种情况下PUBLISH报文将携带多个相同的订阅标识符。 

PUBLISH报文中若包含服务端收到的SUBSCRIBE报文以外的订阅标识符，将造成协议错误（Protocol Error）。从客户端发送给服务端的PUBLISH报文**不能**包含订阅标识符 \[MQTT-3.3.4-6\]。

对于共享订阅，发送给某个客户端的PUBLISH报文中将只包含该客户端的SUBSCRIBE报文中发送的订阅标识符。 

收到PUBLISH报文时，接收端的行为取决于报文的QoS等级，如4.3节所述。

如果PUBLISH报文包含主题别名，接收端按照以下方式进行处理：
1)	主题别名为0或大于最大主题别名（Maximum Topic Alias），将造成协议错误（Protocol Error），接收端使用包含原因码为0x94（主题别名无效）的DISCONNECT报文断开网络连接，如4.13节所述。 
 
2)	如果接收端已创建此主题别名的映射，
a)	如果报文包含的主题名长度为0，接收端使用主题别名对应的主题名处理此报文
b)	如果报文包含的主题名长度不为0，接收端使用此主题名处理此报文，并更新此主题别名映射到此主题名

3)	如果接收端还没有创建此主题别名的映射，
a)	如果报文包含的主题名长度为0，将造成协议错误，接收端使用包含原因码为0x82（协议错误）的DISCONNECT报文断开网络连接，如4.13节所述。
b)	如果报文包含的主题名长度不为0，接收端使用此主题名处理此报文，并为此报文中的主题别名和主题名创建映射关系

> **非规范评注**
>
> 如果服务端向客户端分发应用消息时使用了不同的协议级别（比如MQTT v3.1.1）-- 不支持属性或本规范提供的其他功能，应用消息中的某些信息将丢失，依赖于这些信息的应用程序可能无法正常工作。

客户端在收到服务端的PUBACK，PUBCOMP或包含原因码大于等于128的PUBREC报文之前，**不能**发送数量超过服务端的接收最大值（Receive Maximum）的QoS为1和2的PUBLISH报文 \[MQTT-3.3.4-7\]。服务端在发送PUBACK或PUBCOMP响应之前，如果收到数量超过客户端的接收最大值的QoS为1和2的PUBLISH报文，服务端使用包含原因码为0x93（超出接收最大值）的DISCONNECT报文断开网络连接，如4.13节所述。更多关于流量控制的信息，参考4.9节。

客户端**不能**延迟发送任何报文，除了PUBLISH报文--如果已发送且没有收到确认的PUBLISH报文数量已达到服务端的接收最大值（Receive Maximum） \[MQTT-3.3.4-8\]。接收最大值只应用于当前网络连接。

> **非规范评注**
>
> 客户端可以选择发送少于服务端接收最大值的未经确认的PUBLISH报文，尽管它可以发送更多数量的报文。

> **非规范评注**
>
> 客户端可以选择暂停发送QoS为0的报文，当其暂停发送了QoS为1和2的PUBLISH报文。

> **非规范评注**
>
> 如果客户端在收到CONNACK之前发送QoS为1或QoS为2的PUBLISH报文，客户端有可能被服务器断开连接，因为它发送了超过服务端接收最大值数量的发布报文。

服务端在接收到客户端的PUBACK，PUBCOMP或包含原因码大于等于128的PUBREC报文之前，**不能**发送数量超过客户端的接收最大值（Receive Maximum）的QoS为1和2的PUBLISH报文 \[MQTT-3.3.4-9\]。客户端在发送PUBACK或PUBCOMP响应之前，如果收到数量超过服务端的接收最大值的QoS为1和2的PUBLISH报文，客户端使用包含原因码为0x93（超出接收最大值）的DISCONNECT报文断开网络连接，如4.13节所述。更多关于流量控制的信息，参考4.9节 。

服务端**不能**延迟发送任何报文，除了PUBLISH报文--如果已发送且没有收到确认的PUBLISH报文数量已到达客户端的接收最大值（Receive Maximum） \[MQTT-3.3.4-10\]。

> **非规范评注**
>
> 服务端可以选择发送少于客户端接收最大值的未经确认的PUBLISH报文，尽管它可以发送更多数量的报文。

> **非规范评注**
>
> 服务端可以选择暂停发送QoS为0的报文，当其暂停发送了QoS为1和2的PUBLISH报文。


### 第三章目录 MQTT控制报文

- [3.0 Contents – MQTT控制报文](03-ControlPackets.md)
- [3.1 CONNECT – 连接服务端](0301-CONNECT.md)
- [3.2 CONNACK – 确认连接请求](0302-CONNACK.md)
- [3.3 PUBLISH – 发布消息](0303-PUBLISH.md)
- [3.4 PUBACK –发布确认](0304-PUBACK.md)
- [3.5 PUBREC – 发布收到（QoS 2，第一步）](0305-PUBREC.md)
- [3.6 PUBREL – 发布释放（QoS 2，第二步）](0306-PUBREL.md)
- [3.7 PUBCOMP – 发布完成（QoS 2，第三步）](0307-PUBCOMP.md)
- [3.8 SUBSCRIBE - 订阅主题](0308-SUBSCRIBE.md)
- [3.9 SUBACK – 订阅确认](0309-SUBACK.md)
- [3.10 UNSUBSCRIBE –取消订阅](0310-UNSUBSCRIBE.md)
- [3.11 UNSUBACK – 取消订阅确认](0311-UNSUBACK.md)
- [3.12 PINGREQ – 心跳请求](0312-PINGREQ.md)
- [3.13 PINGRESP – 心跳响应](0313-PINGRESP.md)
- [3.14 DISCONNECT –断开连接](0314-DISCONNECT.md)
- [3.15 AUTH – 认证交换](0315-AUTH.md)

### 项目主页

- [MQTT v5.0协议草案中文版](https://github.com/hui6075/mqtt_v5)

