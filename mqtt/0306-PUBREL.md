## 3.6 PUBREL – 发布释放（QoS 2，第二步）

PUBREL报文是对PUBREC报文的响应。它是QoS 2等级协议交换的第三个报文。

### 3.6.1 PUBREL 固定报头 PUBREL Fixed Header

##### 图 3-14 – PUBREL报文固定报头 PUBREL packet Fixed Header

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
     <td colspan="4" align="center">MQTT控制报文类型 (6)</td>
     <td colspan="4" align="center">保留位</td>
   </tr>
   <tr>
       <td></td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">1</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">0</td>
     </tr>
   <tr>
     <td>byte 2</td>
     <td colspan="8" align="center">剩余长度</td>
   </tr>
 </table>

PUBREL固定报头的第3，2，1，0位是保留位，**必须**被设置为0，0，1，0。服务端**必须**将其它的任何值都当做是不合法的并关闭网络连接 [MQTT-3.6.1-1]。

**剩余长度字段**  
表示可变报头的长度，被编码为变长字节整数。

### 3.6.2 PUBREL 可变报头 PUBREL Variable Header

PUBREL报文的可变报头按顺序包含以下字段：所确认的PUBREC报文标识符，PUBREL原因码，属性（Properties）。属性的编码规则如2.2.2节所述。

##### 图 3-15 – PUBREL报文可变报头 PUBREL packet Variable Header

<table>
   <tr>
     <th>Bit</th>
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
     <td>byte 1</td>
     <td colspan="8" align="center">报文标识符 MSB</td>
   </tr>
   <tr>
     <td>byte 2</td>
     <td colspan="8" align="center">报文标识符 LSB</td>
   </tr>
   <tr>
     <td>byte 3</td>
     <td colspan="8" align="center">PUBREL原因码</td>
   </tr>
   <tr>
     <td>byte 4</td>
     <td colspan="8" align="center">属性长度</td>
   </tr>
 </table>

#### 3.6.2.1 PUBREL 原因码 PUBREL Reason Code

可变报头第3字节是PUBREL原因码。如果剩余长度为2，则表示使用原因码0x00 （成功）。

##### 表 3-6 – PUBREL原因码 PUBREL Reason Codes

<table>
  <tr>
    <th>值</th>
    <th>16进制</th>
	<th>原因码名称</th>
	<th>说明</th>
  </tr>
  <tr>
    <td>0</td>
    <td>0x00</td>
	<td>成功</td>
	<td>消息已释放。</td>
  </tr>
  <tr>
    <td>146</td>
    <td>0x92</td>
	<td>报文标识符未发现</td>
	<td>未知的报文标识符。会话恢复阶段这并非错误，但其他时间这表明服务端和客户端的会话状态不匹配。</td>
  </tr>
</table>

客户端或服务端发送PUBREL报文时**必须**设置其中一种PUBREL原因码 \[MQTT-3.6.2-1\]。当原因码为0x00（成功）且没有属性（Properties）时，原因码和属性长度可以被省略。在这种情况下，PUBREL剩余长度为2。

#### 3.6.2.2 PUBREL 属性 PUBREL Properties

##### 3.6.2.2.1 属性长度 Property Length

PUBREL报文可变报头中的属性长度被编码为变长字节整数。如果剩余长度小于4，则表示没有属性长度字段。

##### 3.6.2.2.2 原因字符串 Reason String

**31 (0x1F)** ，原因字符串（Reason String）标识符。  
跟随其后的是UTF-8编码的字符串，表示此次响应相关的原因。此原因字符串（Reason String）是为诊断而设计的可读字符串，不应该被接收端所解析。

发送端使用此值向接收端提供附加信息。如果加上原因字符串之后的PUBREL报文长度超出了接收端指定的最大报文长度（Maximum Packet Size），则发送端**不能**发送此原因字符串 \[MQTT-3.6.2-2\]。包含多个原因字符串将造成协议错误（Protocol Error）。

##### 3.6.2.2.3 用户属性 User Property

**38 (0x26)** ，用户属性（User Property）标识符。  
跟随其后的是UTF-8字符串对。此属性可用于提供包括诊断信息或关于PUBREL的信息。如果加上用户属性之后的PUBREL报文长度超出了接收端指定的最大报文长度（Maximum Packet Size），则发送端**不能**发送此属性 \[MQTT-3.6.2-3\]。用户属性（User Property）允许出现多次，以表示多个名字/值对，且相同的名字可以多次出现。

### 3.6.3 PUBREL 有效载荷 PUBREL Payload

PUBREL报文没有有效载荷。

### 3.6.4 PUBREL 行为 PUBREL Actions

描述见4.3.3节。


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


