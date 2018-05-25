## 3.5 PUBREC – 发布已接收（QoS 2，第一步）

PUBREC报文是对QoS等级2的PUBLISH报文的响应。它是QoS 2等级协议交换的第二个报文。

### 3.5.1 PUBREC 固定报头 PUBREC Fixed Header

##### 图 3-12 – PUBREC报文固定报头 PUBREC packet Fixed Header

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
     <td colspan="4" align="center">MQTT控制报文类型 (5)</td>
     <td colspan="4" align="center">保留位</td>
   </tr>
    <tr>
       <td></td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
     </tr>
   <tr>
     <td>byte 2</td>
     <td colspan="8" align="center">剩余长度 (2)</td>
   </tr>
 </table>

**剩余长度字段**  
表示可变报头的长度，用变长字节整数编码。

### 3.5.2 PUBREC 可变报头 PUBREC Variable Header

PUBREC可变报头按顺序包含以下字段：所确认的PUBLISH报文标识符（Packet Identifier），PUBREC原因码（Reason Code），属性（Properties）。属性的编码规则，如2.2.2节所述。

##### 图 3-13 – PUBREC报文可变报头 PUBREC packet Variable Header

<table style="text-align:center">
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
     <td colspan="8" align="center">报文标识符 MSB</td>
   </tr>
   <tr>
     <td>byte 2</td>
     <td colspan="8" align="center">报文标识符 LSB</td>
   </tr>
   <tr>
     <td>byte 3</td>
     <td colspan="8" align="center">PUBREC原因码</td>
   </tr>
   <tr>
     <td>byte 4</td>
     <td colspan="8" align="center">属性长度</td>
   </tr>
 </table>

#### 3.5.2.1 PUBREC 原因码 PUBREC Reason Code

PUBREC可变报头第3字节是原因码（Reason Code）。如果剩余长度为2，则表示使用原因码0x00（成功）。

##### 表 3-5 – PUBACK原因码 PUBREC Reason Codes

<table>
  <tr>
    <th>值</th>
    <th width="80">16进制</th>
	<th width="150">原因码名称</th>
	<th>说明</th>
  </tr>
  <tr>
    <td>0</td>
    <td>0x00</td>
	<td>成功</td>
	<td>消息被接收。QoS为2的消息已发布。</td>
  </tr>
  <tr>
    <td>16</td>
    <td>0x10</td>
	<td>无匹配的订阅者</td>
	<td>消息被接收，但没有订阅者。只有服务端会发送此原因码。如果服务端得知没有匹配的订阅者，服务端<strong>可以</strong>使用此原因码代替0x00（成功）。</td>
  </tr>
  <tr>
    <td>128</td>
    <td>0x80</td>
	<td>未指明的错误</td>
	<td>接收端不接受此消息，且不愿意透露错误原因或没有适用的原因码。</td>
  </tr>
  <tr>
    <td>131</td>
    <td>0x83</td>
	<td>实现特定错误</td>
	<td>PUBLISH报文有效，但不被接收端所接受。</td>
  </tr>
  <tr>
    <td>135</td>
    <td>0x87</td>
	<td>未授权</td>
	<td>PUBLISH报文未授权。</td>
  </tr>
  <tr>
    <td>144</td>
    <td>0x90</td>
	<td>主题名无效</td>
	<td>主题名格式正确，但未被客户端或服务端所接受。</td>
  </tr>
  <tr>
    <td>145</td>
    <td>0x91</td>
	<td>报文标识符被占用</td>
	<td>报文标识符正被占用。可能表明客户端和服务端之间的会话状态不匹配。</td>
  </tr>
  <tr>
    <td>151</td>
    <td>0x97</td>
	<td>超出配额</td>
	<td>已超出实现限制或管理限制。</td>
  </tr>
  <tr>
    <td>153</td>
    <td>0x99</td>
	<td>载荷格式无效</td>
	<td>载荷格式与载荷格式指示符不匹配。</td>
  </tr>
</table>

服务端或客户端发送PUBREC报文时**必须**设置其中一种原因码 \[MQTT-3.5.2-1\]。当原因码为0x00（成功）且没有属性（Properties）时，原因码和属性长度可以被省略。在这种情况下，PUBREC剩余长度为2。

#### 3.5.2.2 PUBREC 属性 PUBREC Properties

##### 3.5.2.2.1 属性长度 Property Length

PUBREC可变报头的属性长度被编码为变长字节整数。如果剩余长度小于4，则表示没有属性长度字段。

##### 3.5.2.2.2 原因字符串 Reason String

**31 (0x1F)** ，原因字符串（Reason String）标识符。
跟随其后的是UTF-8编码的字符串，表示此次响应相关的原因。此原因字符串（Reason String）是为诊断而设计的可读字符串，**不应该**被接收端所解析。 

发送端使用此值向接收端提供附加信息。如果加上原因字符串之后的PUBREC报文长度超出了接收端指定的最大报文长度（Maximum Packet Size），则发送端**不能**发送此属性 \[MQTT-3.5.2-2\]。包含多个原因字符串将造成协议错误（Protocol Error）。

##### 3.5.2.2.3 用户属性 User Property

**38 (0x26)** ，用户属性（User Property）标识符。
跟随其后的是UTF-8字符串对。此属性可用于提供包括诊断信息在内的附加信息。如果加上用户属性之后的PUBREC报文长度超出了接收端指定的最大报文长度（Maximum Packet Size），则发送端**不能**发送此属性 \[MQTT-3.5.2-3\]。用户属性（User Property）允许出现多次，以表示多个名字/值对，且相同的名字可以多次出现。
 
### 3.5.3 有效载荷 PUBREC Payload

PUBREC报文没有有效载荷。

### 3.5.4 动作 PUBREC Actions

描述见 4.3.3节。


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


