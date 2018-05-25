## 3.8 SUBSCRIBE - 订阅请求

客户端向服务端发送SUBSCRIBE报文用于创建一个或多个订阅。每个订阅（Subscription）注册客户端所感兴趣的一个或多个主题。服务端向客户端发送PUBLISH报文以转发被发布到符合这些订阅主题的应用消息。SUBSCRIBE报文同样（为每个订阅）指定了服务端可以向其发送的应用消息最大QoS等级。

### 3.8.1 固定报头 SUBSCRIBE Fixed Header

##### 图 3-20 – SUBSCRIBE报文固定报头 SUBSCRIBE packet Fixed Header

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
     <td colspan="4" align="center">MQTT控制报文类型 (8)</td>
     <td colspan="4" align="center">保留位</td>
   </tr>
   <tr>
       <td></td>
       <td align="center">1</td>
       <td align="center">0</td>
       <td align="center">0</td>
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

SUBSCRIBE报文固定报头第3，2，1，0比特位是保留位，**必须**被设置为0，0，1，0。服务端必须将其他的任何值都当做是不合法的并关闭网络连接 \[MQTT-3.8.1-1\]。

**剩余长度字段**  
表示可变报头的长度加上有效载荷的长度，被编码为变长字节整数。

### 3.8.2 可变报头 SUBSCRIBE Variable Header

SUBSCRIBE报文可变报头按顺序包含以下字段：报文标识符（Packet Identifier），属性（Properties）。2.2.1节提供了更多关于报文标识符的信息。属性的编码规则如2.2.2节所述。

> **非规范示例**
>
> 图3-19展示了一个包含报文标识符为10，且没有属性的SUBSCRIBE可变报头。

##### 图 3-19 – SUBSCRIBE 可变报头示例 SUBSCRIBE Variable Header example

<table>
  <tr>
    <th width="120"></th>
    <th width="280">说明</th>
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
    <td colspan="10">报文标识符</td>
  </tr>
  <tr>
    <td>byte 1</td>
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
    <td>byte 2</td>
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
    <td>byte 3</td>
    <td>属性长度 (0)</td>
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

#### 3.8.2.1 SUBSCRIBE 属性 SUBSCRIBE Properties

##### 3.8.2.1.1 属性长度 Property Length

SUBSCRIBE报文可变报头中的属性长度被编码为变长字节整数。

##### 3.8.2.1.2 订阅标识符 Subscription Identifier

**11 (0x0B)** ，订阅标识符（Subscription Identifier）标识符。  
跟随其后的是一个变长字节整数表示订阅标识符。订阅标识符取值范围从1到268,435,455。订阅标识符的值为0或包含多个订阅标识符将造成协议错误（Protocol Error）。

订阅标识符与SUBSCRIBE报文所创建或修改的订阅（Subscription）相关联。如果包含订阅标识符，它将与订阅一起被存储。如果未指定此属性，则订阅被存储时将不包含订阅标识符。

更多关于订阅标识符的处理信息，参考3.8.3.1节。

##### 3.8.2.1.3 用户属性 User Property

**38 (0x26)** ，用户属性（User Property）标识符。  
跟随其后的是UTF-8字符串对。

用户属性允许出现多次，以表示多个名字/值对。同样的名字允许出现多次。

> **非规范评注**
>
> SUBSCRIBE报文的用户属性可以被客户端用来向服务端发送订阅相关的属性。本规范不定义这些属性的意义。

### 3.8.3 SUBSCRIBE 载荷 SUBSCRIBE Payload

SUBSCRIBE报文的载荷包含一列主题过滤器，指明客户端希望订阅的主题。主题过滤器**必须**为UTF-8 编码的字符串 \[MQTT-3.8.3-1\]。每个主题过滤器之后跟着一个订阅选项（Subscription Options）字节。 

载荷**必须**包含至少一个主题过滤器/订阅选项对 \[MQTT-3.8.3-2\]。不包含载荷的SUBSCRIBE报文将造成协议错误（Protocol Error）。错误处理信息，参考4.13节。

#### 3.8.3.1 订阅选项 Subscription Options

订阅选项的第0和1比特代表最大服务质量字段。此字段给出服务端可以向此客户端发送的应用消息的最大QoS等级。最大服务质量字段为3将造成协议错误（Protocol Error）。

订阅选项的第2比特表示非本地（No Local）选项。值为1，表示应用消息不能被转发给发布此消息的客户标识符 \[MQTT-3.8.3-3\]。共享订阅时把非本地选项设为1将造成协议错误（Protocol Error） \[MQTT-3.8.3-4\]。

订阅选项的第3比特表示发布保留（Retain As Published）选项。值为1，表示向此订阅转发应用消息时保持消息被发布时设置的保留（RETAIN）标志。值为0，表示向此订阅转发应用消息时把保留标志设置为0。当订阅建立之后，发送保留消息时保留标志设置为1。

订阅选项的第4和5比特表示保留操作（Retain Handling）选项。此选项指示当订阅建立时，是否发送保留消息。此选项不影响之后的任何保留消息的发送。如果没有匹配主题过滤器的保留消息，则此选项所有值的行为都一样。值可以设置为：  
0 = 订阅建立时发送保留消息  
1 = 订阅建立时，若该订阅当前不存在则发送保留消息  
2 = 订阅建立时不要发送保留消息  
保留操作的值设置为3将造成协议错误（Protocol Error）。

订阅选项的第6和7比特为将来所保留。服务端**必须**把此保留位非0的SUBSCRIBE报文当做无效报文 \[MQTT-3.8.3-5\]。

> **非规范评注**
>
> 非本地（No Local）和发布保留（Retain As Published）订阅选项在客户端把消息发送给其他服务端的情况下，可以被用来实现桥接。

> **非规范评注**
>
> 已存在订阅的情况下不发送保留消息是很有用的，比如重连完成时客户端不确定订阅是否在之前的会话连接中被创建。

> **非规范评注**
>
> 不发送保存的保留消息给新创建的订阅是很有用的，比如客户端希望接收变更通知且不需要知道最初的状态。

> **非规范评注**
>
> 对于某个指示其不支持保留消息的服务端，发布保留和保留处理选项的所有有效值都将得到同样的结果：订阅时不发送任何保留消息，且所有消息的保留标志都会被设置为0。

##### 图 3-20 – SUBSCRIBE报文载荷格式 SUBSCRIBE packet Payload format

<table style="text-align:center">
   <tr>
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
     <td colspan="9">主题过滤器</td>
   </tr>
    <tr>
     <td>byte 1</td>
     <td colspan="8" align="center">长度 MSB</td>
   </tr>
    <tr>
     <td>byte 2</td>
     <td colspan="8" align="center">长度 LSB</td>
   </tr>
   <tr>
     <td>byte 3..N</td>
     <td colspan="8" align="center">主题过滤器（Topic Filter）</td>
   </tr>
   <tr>
     <td colspan="9">订阅选项</td>
   </tr>
   <tr>
       <td></td>
       <td align="center" colspan="2">保留位</td>
       <td align="center" colspan="2">保留处理</td>
       <td align="center">RAP</td>
       <td align="center">NL</td>
       <td align="center" colspan="2">QoS</td>
   </tr>
   <tr>
       <td>byte N+1</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">X</td>
       <td align="center">X</td>
       <td align="center">X</td>
       <td align="center">X</td>
       <td align="center">X</td>
       <td align="center">X</td>
   </tr>
 </table>

RAP指发布保留（Retain as Published）。  
NL指非本地（No Local）。

##### 图 3-21 – 载荷字节格式非规范示例 Payload byte format non-normative example

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
    <td colspan="10">订阅选项</td>
  </tr>
  <tr>
    <td>byte 6</td>
    <td>订阅选项 (1)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
  </tr>
  <tr>
    <td colspan="10">主题过滤器</td>
  </tr>
  <tr>
    <td>byte 7</td>
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
    <td>byte 8</td>
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
    <td>byte 9</td>
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
    <td>byte 10</td>
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
    <td>byte 11</td>
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
  <tr>
    <td colspan="10">订阅选项</td>
  </tr>
  <tr>
    <td>byte 12</td>
    <td>订阅选项 (2)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
  </tr>
</table>

### 3.8.4 SUBSCRIBE 行为 SUBSCRIBE Actions

当服务端收到来自客户端的SUBSCRIBE报文时，**必须**使用SUBACK报文作为相应 [MQTT-3.8.4-1]。SUBACK报文**必须**和待确认的SUBSCRIBE报文有相同的报文标识符 \[MQTT-3.8.4-2\]。

允许服务端在发送SUBACK报文之前就开始发送与订阅相匹配的PUBLISH报文。

如果服务端收到的SUBSCRIBE报文中的一个主题过滤器与当前会话的一个非共享订阅（Non-shared Subscription）相同，那么**必须**使用新的订阅替换现存的订阅 \[MQTT-3.8.4-3\]。新订阅的主题过滤器与之前的订阅相同，但其订阅选项可能不同。如果保留处理选项为0，任何匹配该主题过滤器的保留消息**必须**被重发，但替换订阅不能造成应用消息的丢失 \[MQTT-3.8.4-4\]。

如果服务端收到的非共享主题过滤器（Non-shared Topic Filter）不同于当前会话的任何主题过滤器，一个新的非共享订阅将被创建。如果保留处理选项不为2，所有相匹配的保留消息将发送给客户端。

如果服务端收到的主题过滤器与服务端已存在的某个共享订阅（Shared Subscription）主题过滤器相同，则将此会话添加到该共享订阅中。不发送任何保留消息。

如果服务端收到的共享订阅主题过滤器（Shared Subscription Topic Filter）与任何已存在的共享订阅主题过滤器都不同，一个新的共享订阅将被创建。将此会话作为订阅者添加到该共享订阅。不发送任何保留消息。 

更多关于共享订阅的细节，参考4.8节。

如果服务端收到的SUBSCRIBE报文包含多个主题过滤器，服务端**必须**当做收到一系列多个SUBSCRIBE报文来处理--除了将它们的响应组合为单个SUBACK响应 [MQTT-3.8.4-5]。

服务端发送给客户端的SUBACK报文**必须**为每一个主题过滤器/订阅选项对包含一个原因码 \[MQTT-3.8.4-6\]。 此原因码**必须**说明为该订阅授予的最大QoS等级，或指示订阅失败 [MQTT-3.8.4-7]。服务端可能授予了低于订阅者所请求的最大QoS等级。响应该订阅的应用消息QoS等级**必须**为该消息发布时的QoS等级和服务端授予的最大QoS等级二者最小值 [MQTT-3.8.4-8]。在原始消息发布的QoS等级为1，且授予的最大QoS等级为0的情况下，服务端允许发送重复的消息副本给订阅者（？）。

> **非规范评注**
>
> 如果订阅客户端的某个主题过滤器已被授予的最大QoS等级为1，那么匹配此过滤器的QoS等级为0的应用消息按照QoS等级为0分发给此客户端。这意味着客户端最多只能收到该消息的一个副本。另一方面，发布到相同主题的QoS等级为2的消息，其QoS等级被服务端降级为1以便分发给该客户端。因此该客户端可能收到此消息的多个副本。 

> **非规范评注**
>
> 如果订阅客户端被授予的最大QoS等级为0，那么按照QoS等级为2发布的应用消息在繁忙时可能会丢失，但服务端不应该发送重复的消息副本。发布到相同主题的QoS等级为1的消息，分发给该客户端时可能会丢失或重复。

> **非规范评注**
>
> 使用QoS等级2订阅某个主题过滤器，等于是说：*我想要按照消息被发布时的QoS等级接收匹配此过滤器的消息* 。这意味着发布者负责决定消息可以被发布的最大QoS等级，但订阅端可以要求服务端降低该消息的QoS到更适合它的等级。

订阅标识符是服务端的会话状态的一部分，并将在收到PUBLISH报文时返回给客户端。当服务端收到客户端的UNSUBSCRIBE报文时，服务端将此会话标识符从服务端的会话状态中移除：当服务端收到客户端的UNSUBSCRIBE报文，当服务端收到客户端对同样主题过滤器的SUBSCRIBE报文但订阅标识符不同或没有订阅标识符，或者当服务端在CONNACK报文中将会话存在标志设置为0。

订阅标识符不构成客户端的会话状态的一部分。在一个有用的实现中，客户端将订阅标识符与其他客户端状态相关联，此客户端状态将被移除：当客户端取消订阅，当客户端以不同的订阅标识符或没有订阅标识符订阅同样的主题过滤器，或者当客户端收到的CONNACK报文中会话存在标志被设置为0。

服务端在重传的PUBLISH报文中无需使用同一组订阅标识符。客户端可以通过发送包含与当前会话已存在的主题过滤器的SUBSCRIBE报文进行重新订阅。如果客户端在PUBLISH报文初传之后重新订阅并使用了不同的订阅标识符，允许服务端在任何重传中使用初传所包含的订阅标识符，或者在重传中使用此新的订阅标识符。不允许服务端在发送了包含新的订阅标识符的PUBLISH报文之后再次使用旧的订阅标识符。

> **非规范评注**
>
> 使用场景，用以阐述订阅标识符：
> -    客户端实现指示某条发布消息匹配多个订阅的编程接口，客户端实现每次订阅时生成新的订阅标识符。如果返回的发布消息包含多个订阅标识符，则该发布消息匹配多个订阅。
>
> -    客户端实现允许订阅者将消息定向到其相关联的订阅的回调，客户端实现生成映射到唯一回调的订阅标识符。收到某条发布消息时，使用订阅标识符决定触发哪一个回调。
>
> -    客户端实现在发布消息时返回程序用于订阅的主题字符串，为此客户端生成一个唯一标识了该主题过滤器的标识符。收到某条发布消息时，客户端实现使用此标识符查找原始主题过滤器，并将主题过滤器返回给其应用程序。
>
> -    网关（Gateway）将从服务端收到的发布消息转发给向该网关做了订阅的客户端，网关实现维护其收到的每个唯一的订阅过滤器到其收到的一组客户标识符--订阅标识符对的映射，网关对它转发给服务端的每个主题过滤器生成一个唯一的标识符。收到某条发布消息时，网关使用从服务端收到的订阅标识符查找对应的客户标识符--订阅标识符对，并把它们加入发送给客户端的PUBLISH报文中。如果上游服务端因为消息匹配了多个订阅而发送了多个PUBLISH报文，则此行为将反映到客户端。


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


