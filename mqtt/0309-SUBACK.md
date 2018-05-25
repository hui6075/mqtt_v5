## 3.9 SUBACK – 订阅确认

服务端发送SUBACK报文给客户端，用于确认它已收到并且正在处理SUBSCRIBE报文。

SUBACK报文包含一个原因码列表，用于指定授予的最大QoS等级或SUBSCRIBE报文所请求的每个订阅发生的错误。

### 3.9.1 SUBACK 固定报头 SUBACK Fixed Header

##### 图 3-24 – SUBACK报文固定报头 SUBACK Packet Fixed Header

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
     <td colspan="4" align="center">MQTT控制报文类型 (9)</td>
     <td colspan="4" align="center">保留位</td>
   </tr>
   <tr>
       <td></td>
       <td align="center">1</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
     </tr>
   <tr>
     <td>byte 2</td>
     <td colspan="8" align="center">剩余长度</td>
   </tr>
 </table>

**剩余长度字段**  
可变报头长度加上有效载荷长度，编码为变长字节整数。

### 3.9.2 SUBACK 可变报头 SUBACK Variable Header

SUBACK报文可变报头按顺序包含以下字段：所确认的SUBSCRIBE报文标识符，属性（Properties）。

##### 图例 3.25 – SUBACK报文可变报头

#### 3.9.2.1 SUBACK 属性 SUBACK Properties

##### 3.9.2.1.1 属性长度 Property Length

SUBACK可变报头中的属性长度被编码为变长字节整数。

##### 3.9.2.1.2 原因字符串 Reason String

**31 (0x1F)** ，原因字符串（Reason String）标识符。  
跟随其后的是UTF-8编码的字符串，表示此次响应相关的原因。此原因字符串（Reason String）是为诊断而设计的可读字符串，**不应该**被客户端所解析。

服务端使用此值向客户端提供附加信息。如果加上原因字符串之后的SUBACK报文长度超出了客户端指定的最大报文长度，则服务端**不能**发送此原因字符串 \[MQTT-3.9.2-1\]。包含多个原因字符串将造成协议错误（Protocol Error）。

##### 3.9.2.1.3 用户属性 User Property

**38 (0x26)** ，用户属性（User Property）标识符。  
跟随其后的是UTF-8字符串对。此属性可用于向客户端提供包括诊断信息在内的附加信息。如果加上用户属性之后的SUBACK报文长度超出了客户端指定的最大报文长度，则服务端**不能**发送此属性 \[MQTT-3.9.2-2\]。用户属性（User Property）允许出现多次，以表示多个名字/值对，且相同的名字可以多次出现。

##### 图 3-23 - SUBACK报文可变报头 SUBACK packet Variable Header

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
    <td colspan="9" align="center">报文标识符 MSB</td>
  </tr>
  <tr>
    <td>byte 2</td>
    <td colspan="9" align="center">报文标识符 LSB</td>
  </tr>
</table>

### 3.9.3 SUBACK 载荷 SUBACK Payload

有效载荷包含一个原因码列表。每个原因码对应SUBSCRIBE报文中的一个被确认的主题过滤器。SUBACK报文中的原因码顺序**必须**与SUBSCRIBE报文中的主题过滤器顺序相匹配 \[MQTT-3.9.3-1\]。

表 3-8 - 订阅原因码 Subscribe Reason Codes

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
	<td>授予QoS等级0</td>
	<td>订阅被接受且最大QoS等级为0。可能低于所请求的QoS等级。</td>
  </tr>
  <tr>
    <td>1</td>
    <td>0x01</td>
	<td>授予QoS等级1</td>
	<td>订阅被接受且最大QoS等级为1。可能低于所请求的QoS等级。</td>
  </tr>
  <tr>
    <td>2</td>
    <td>0x02</td>
	<td>授予QoS等级2</td>
	<td>订阅被接受且最大QoS等级为2。可能低于所请求的QoS等级。</td>
  </tr>
  <tr>
    <td>128</td>
    <td>0x80</td>
	<td>未指明错误</td>
	<td>订阅未被接受，且服务端不愿意透露原因或没有适用的原因码。</td>
  </tr>
  <tr>
    <td>131</td>
    <td>0x83</td>
	<td>实现特定错误</td>
	<td>SUBSCRIBE有效但不被服务端所接受。</td>
  </tr>
  <tr>
    <td>135</td>
    <td>0x87</td>
	<td>未授权</td>
	<td>客户端未被授权做此订阅。</td>
  </tr>
  <tr>
    <td>143</td>
    <td>0x8F</td>
	<td>主题过滤器无效</td>
	<td>主题过滤器格式正确，但不被允许。</td>
  </tr>
  <tr>
    <td>145</td>
    <td>0x91</td>
	<td>报文标识符已被占用</td>
	<td>指定的报文标识符正在被使用中。</td>
  </tr>
  <tr>
    <td>151</td>
    <td>0x97</td>
	<td>超出配额</td>
	<td>已超出实现限制或管理限制。</td>
  </tr>
  <tr>
    <td>158</td>
    <td>0x9E</td>
	<td>共享订阅不支持</td>
	<td>服务端不支持此客户端进行共享订阅。</td>
  </tr>
  <tr>
    <td>161</td>
    <td>0xA1</td>
	<td>订阅标识符不支持</td>
	<td>服务端不支持订阅标识符；订阅标识符不被接受。</td>
  </tr>
  <tr>
    <td>162</td>
    <td>0xA2</td>
	<td>通配符订阅不支持</td>
	<td>服务端不支持通配符订阅；订阅未被接受。</td>
  </tr>
</table>

服务端发送SUBACK报文时**必须**对收到的每一个主题过滤器设置一种原因码 \[MQTT-3.9.3-2\]。

> **非规范评注**
>
> 对于SUBSCRIBE报文中的每个主题过滤器，总有一个对应的原因码。如果原因码不是针对某个特定的主题过滤器（比如0x91（报文标识符已占用）），则对每个主题过滤器都使用此原因码。


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


