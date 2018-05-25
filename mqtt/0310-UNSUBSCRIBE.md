## 3.10 UNSUBSCRIBE –取消订阅请求

客户端发送UNSUBSCRIBE报文给服务端，用于取消订阅主题。

### 3.10.1 UNSUBSCRIBE 固定报头 UNSUBSCRIBE Fixed Header

##### 图 3-28 – UNSUBSCRIBE报文固定报头

<table style="text-align:center">
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
     <td colspan="4" align="center">MQTT控制报文类型 (10)</td>
     <td colspan="4" align="center">保留位</td>
   </tr>
   <tr>
       <td></td>
       <td align="center">1</td>
       <td align="center">0</td>
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

UNSUBSCRIBE固定报头的第3，2，1，0位是保留位且**必须**分别设置为0，0，1，0。服务端必须认为任何其它的值都是不合法的并关闭网络连接 \[MQTT-3.10.1-1\]。

**剩余长度字段**  
等于可变报头长度（2字节）加上有效载荷长度，编码为变长字节整数。

### 3.10.2 UNSUBSCRIBE 可变报头 UNSUBSCRIBE Variable Header

UNSUBSCRIBE报文可变报头按顺序包含以下字段：报文标识符和属性（Properties）。2.2.1节提供了有关报文标识符的更多信息。属性的编码规则，如2.2.2节所述。

#### 3.10.2.1 UNSUBSCRIBE 属性 UNSUBSCRIBE Properties

##### 3.10.2.1.1 属性长度 Property Length

SUBSCRIBE可变报头中属性的长度被编码为变长字节整数。

##### 3.10.2.1.2 用户属性 User Property

**38 (0x26)** ，用户属性（User Property）标识符。  
跟随其后的是一个UTF-8字符串对。

用户属性允许出现多次，以表示多个名字/值对。相同的名字可以出现多次。

> **非规范评注**
>
> UNSUBSCRIBE报文中的用户属性可以被客户端用来向服务端发送订阅相关的属性。本规范不定义这些属性的意义。

### 3.10.3 3.10.3 UNSUBSCRIBE 载荷 UNSUBSCRIBE Payload

UNSUBSCRIBE报文有效载荷包含一列客户端希望取消订阅的主题过滤器。UNSUBSCRIBE报文中的主题过滤器**必须**为1.5.4节所述的UTF-8编码字符串 \[MQTT-3.10.3-1\] ，且连续填充。

UNSUBSCRIBE报文有效载荷**必须**包含至少一个主题过滤器 \[MQTT-3.10.3-2\]。不包含有效载荷的UNSUBSCRIBE报文将造成协议错误（Protocol Error）。错误处理信息，参考4.13节。

> **非规范示例**
>
> 图 3-30 展示了UNSUBSCRIBE报文的载荷示例，包括两个主题过滤器 “a/b”和“c/d”。

##### 图 3-30 - 载荷字节格式非规范示例 Payload byte format non-normative example

<table>
  <tr>
    <th></th>
    <th>说明</th>
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
    <td colspan="10">主题过滤器</td>
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
    <td colspan="10">主题过滤器</td>
  </tr>
  <tr>
    <td>byte 6</td>
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
    <td>byte 7</td>
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
    <td>byte 8</td>
    <td>‘c’ (0x63)</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 9</td>
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
    <td>byte 10</td>
    <td>‘d’ (0x64)</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
  </tr>
</table>

### 3.10.4 UNSUBSCRIBE 行为 UNSUBSCRIBE Actions

服务端**必须**对客户端的UNSUBSCRIBE报文中提供的主题过滤器（不管是否包含通配符）逐个字符与当前持有的主题过滤器集进行比较。如果任何过滤器完全匹配，则**必须**删除其拥有的订阅 \[MQTT-3.10.4-1\]，否则不会进行额外的处理。

当服务端收到UNSUBSCRIBE报文：
-    它**必须**停止添加为了交付给客户端的与主题过滤器相匹配的任何新消息 \[MQTT-3.10.4-2\]。
-    它**必须**完成任何已经开始发送给客户端的、与主题过滤器相匹配的、QoS等级为1或2的消息 \[MQTT-3.10.4-3\]。
-    它**可以**继续交付任何为交付给客户端而缓存的消息。

服务端**必须**发送UNSUBACK报文以响应客户端的UNSUBSCRIBE请求 \[MQTT-3.10.4-4\]。UNSUBACK报文**必须**包含和UNSUBSCRIBE报文相同的报文标识符。即使没有删除任何主题订阅，服务端也**必须**发送一个UNSUBACK响应 [MQTT-3.10.4-5]。

如果服务端收到的UNSUBSCRIBE报文包含多个主题过滤器，服务端**必须**当做收到一系列多个UNSUBSCRIBE报文来处理--除了将它们的响应组合为单个SUBACK响应 \[MQTT-3.10.4-6\]。

如果某个主题过滤器代表一个共享订阅，此会话将被从该共享订阅中删除。如果此会话是该共享订阅所关联的唯一会话，该共享订阅被删除。共享订阅的处理，参考4.8.2节。


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


