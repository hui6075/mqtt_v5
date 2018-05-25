## 3.14 DISCONNECT – 断开通知

DISCONNECT报文是客户端发给服务端的最后一个MQTT控制报文。表示客户端为什么断开网络连接的原因。客户端和服务端在关闭网络连接之前**可以**发送一个DISCONNECT报文。如果在客户端没有首先发送包含原因码为0x00（正常断开）DISCONNECT报文并且连接包含遗嘱消息的情况下，遗嘱消息会被发布。更多细节，参考3.1.2.5节。

服务端**不能**发送DISCONNECT报文，直到它发送了包含原因码小于0x80的CONNACK报文之后 \[MQTT-3.14.0-1\]。

### 3.14.1 DISCONNECT 固定报头 DISCONNECT Fixed Header

##### 图例 3-35 – DISCONNECT 报文固定报头 DISCONNECT packet Fixed Header

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
     <td colspan="4" align="center">MQTT控制报文类型 (14)</td>
     <td colspan="4" align="center">保留位</td>
   </tr>
   <tr>
       <td></td>
       <td align="center">1</td>
       <td align="center">1</td>
       <td align="center">1</td>
       <td align="center">0</td>
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

服务端或客户端**必须**验证所有的保留位都被设置为0，如果他们不为0，发送包含原因码为0x81（无效报文）的DISCONNECT报文，如4.13节所述 \[MQTT-3.14.1-1\]。

**剩余长度字段**  
等于可变报头的长度，编码为变长字节整数。

### 3.14.2 DISCONNECT 可变报头 DISCONNECT Variable Header

DISCONNECT报文的可变报头按顺序包含以下字段：断开原因码，属性（Properties）。属性的编码规则如2.2.2节所述。

#### 3.14.2.1 DISCONNECT 断开原因码 Disconnect Reason Code

可变报头的第1个字节是断开原因码。如果剩余长度小于1，则表示使用原因码0x00（正常断开）。

单字节无符号断开原因码字段如下所示。

表 3-10 – 断开原因值 Disconnect Reason Code

<table>
  <tr>
    <th>值</th>
    <th>16进制</th>
	<th>原因码名称</th>
	<th>发送端</th>
	<th>说明</th>
  </tr>
  <tr>
    <td>0</td>
    <td>0x00</td>
	<td>正常断开</td>
	<td>客户端或服务端</td>
	<td>正常关闭连接。不发送遗嘱。</td>
  </tr>
  <tr>
    <td>4</td>
    <td>0x04</td>
	<td>包含遗嘱消息的断开</td>
	<td>客户端</td>
	<td>客户端希望断开但也需要服务端发布它的遗嘱消息。</td>
  </tr>
  <tr>
    <td>128</td>
    <td>0x80</td>
	<td>未指定错误</td>
	<td>客户端或服务端</td>
	<td>连接被关闭，但发送端不愿意透露原因，或者没有其他适用的原因码。</td>
  </tr>
  <tr>
    <td>129</td>
    <td>0x81</td>
	<td>无效的报文</td>
	<td>客户端或服务端</td>
	<td>收到的报文不符合本规范。</td>
  </tr>
  <tr>
    <td>130</td>
    <td>0x82</td>
	<td>协议错误</td>
	<td>客户端或服务端</td>
	<td>收到意外的或无序的报文。</td>
  </tr>
  <tr>
    <td>131</td>
    <td>0x83</td>
	<td>实现指定错误</td>
	<td>客户端或服务端</td>
	<td>收到的报文有效，但根据实现无法进行处理。</td>
  </tr>
  <tr>
    <td>135</td>
    <td>0x87</td>
	<td>未授权</td>
	<td>服务端</td>
	<td>请求没有被授权</td>
  </tr>
  <tr>
    <td>137</td>
    <td>0x89</td>
	<td>服务端正忙</td>
	<td>服务端</td>
	<td>服务端正忙且不能继续处理此客户端的请求。</td>
  </tr>
  <tr>
    <td>139</td>
    <td>0x8B</td>
	<td>服务正关闭</td>
	<td>服务端</td>
	<td>服务正在关闭。</td>
  </tr>
  <tr>
    <td>141</td>
    <td>0x8D</td>
	<td>保持连接超时</td>
	<td>服务端</td>
	<td>连接因为在超过1.5倍的保持连接时间内没有收到任何报文而关闭。</td>
  </tr>
  <tr>
    <td>142</td>
    <td>0x8E</td>
	<td>会话被接管</td>
	<td>服务端</td>
	<td>另一个使用了相同的客户标识符的连接已建立，导致此连接关闭。</td>
  </tr>
  <tr>
    <td>143</td>
    <td>0x8F</td>
	<td>主题过滤器无效</td>
	<td>服务端</td>
	<td>主题过滤器格式正确，但不被服务端所接受。</td>
  </tr>
  <tr>
    <td>144</td>
    <td>0x90</td>
	<td>主题名无效</td>
	<td>客户端或服务端</td>
	<td>主题名格式正确，但不被客户端或服务端所接受。</td>
  </tr>
  <tr>
    <td>147</td>
    <td>0x93</td>
	<td>超出接收最大值</td>
	<td>客户端或服务端</td>
	<td>客户端或服务端收到了数量超过接收最大值的未发送PUBACK或PUBCOMP的发布消息。</td>
  </tr>
  <tr>
    <td>148</td>
    <td>0x94</td>
	<td>主题别名无效</td>
	<td>客户端或服务端</td>
	<td>客户端或服务端收到的PUBLISH报文包含的主题别名大于其在CONNECT或CONNACK中发送的主题别名最大值。</td>
  </tr>
  <tr>
    <td>149</td>
    <td>0x95</td>
	<td>报文过大</td>
	<td>客户端或服务端</td>
	<td>报文长度大于此客户端或服务端的最大报文长度。</td>
  </tr>
  <tr>
    <td>150</td>
    <td>0x96</td>
	<td>消息速率过高</td>
	<td>客户端或服务端</td>
	<td>收到的数据速率太高。</td>
  </tr>
  <tr>
    <td>151</td>
    <td>0x97</td>
	<td>超出配额</td>
	<td>客户端或服务端</td>
	<td>已超出实现限制或管理限制。</td>
  </tr>
  <tr>
    <td>152</td>
    <td>0x98</td>
	<td>管理操作</td>
	<td>客户端或服务端</td>
	<td>连接因为管理操作被关闭。</td>
  </tr>
  <tr>
    <td>153</td>
    <td>0x99</td>
	<td>载荷格式无效</td>
	<td>客户端或服务端</td>
	<td>载荷格式与指定的载荷格式指示符不匹配。</td>
  </tr>
  <tr>
    <td>154</td>
    <td>0x9A</td>
	<td>不支持保留</td>
	<td>服务端</td>
	<td>服务端不支持保留消息。</td>
  </tr>
  <tr>
    <td>155</td>
    <td>0x9B</td>
	<td>不支持的QoS等级</td>
	<td>服务端</td>
	<td>客户端指定的QoS等级大于CONNACK报文中指定的最大QoS等级。 </td>
  </tr>
  <tr>
    <td>156</td>
    <td>0x9C</td>
	<td>（临时）使用其他服务端</td>
	<td>服务端</td>
	<td>客户端应该临时使用其他服务端。</td>
  </tr>
  <tr>
    <td>157</td>
    <td>0x9D</td>
	<td>服务端已（永久）移动</td>
	<td>服务端</td>
	<td>服务端已移动且客户端应该永久使用其他服务端。</td>
  </tr>
  <tr>
    <td>158</td>
    <td>0x9E</td>
	<td>不支持共享订阅</td>
	<td>服务端</td>
	<td>服务端不支持共享订阅。</td>
  </tr>
  <tr>
    <td>159</td>
    <td>0x9F</td>
	<td>超出连接速率限制</td>
	<td>服务端</td>
	<td>此连接因为连接速率过高而被关闭。</td>
  </tr>
  <tr>
    <td>160</td>
    <td>0xA0</td>
	<td>最大连接时间</td>
	<td>服务端</td>
	<td>超出为此连接授予的最大连接时间。</td>
  </tr>
  <tr>
    <td>161</td>
    <td>0xA1</td>
	<td>不支持订阅标识符</td>
	<td>服务端</td>
	<td>服务端不支持订阅标识符；订阅未被接受。</td>
  </tr>
  <tr>
    <td>162</td>
    <td>0xA2</td>
	<td>不支持通配符订阅</td>
	<td>服务端</td>
	<td>服务端不支持通配符订阅；订阅未被接受。</td>
  </tr>
</table>

客户端或服务端发送DISCONNECT报文时**必须**使用一种DISCONNECT原因码 \[MQTT-3.14.2-1\]。如果原因码为0x00（正常断开）且没有属性，原因码和属性长度可以被省略。这种情况下DISCONNECT报文剩余长度为0。

> **非规范评注**
>
> DISCONNECT报文用于指示断开的原因，例如没有确认报文（比如QoS等级0的发布消息）或当客户端或服务端不能继续处理连接。

> **非规范评注**
>
> 客户端可以使用这些信息来决定是否重新连接，以及在重新尝试之前应该等待多长时间。

#### 3.14.2.2 DISCONNECT 属性 DISCONNECT Properties

##### 3.14.2.2.1 属性长度 Property Length

DISCONNECT报文可变报头中的属性（Properties）的长度被编码为变长字节整数。如果剩余长度小于2，属性长度使用0。

##### 3.14.2.2.2 会话过期间隔 Session Expiry Interval

**17 (0x11)** ，会话过期间隔（Session Expiry Interval）标识符。  
跟随其后的是用四字节整数表示的以秒为单位的会话过期间隔（Session Expiry Interval）。包含多个会话过期间隔将造成协议错误（Protocol Error）。

如果没有设置会话过期间隔，则使用CONNECT报文中的会话过期间隔。
 
会话过期间隔**不能**由服务端的DISCONNECT报文发送 \[MQTT-3.14.2-2\]。

如果CONNECT报文中的会话过期间隔为0，则客户端在DISCONNECT报文中设置非0会话过期间隔将造成协议错误（Protocol Error）。如果服务端收到这种非0会话过期间隔，则不会将其视为有效的DISCONNECT报文。服务端使用包含原因码为0x82（协议错误）的DISCONNECT报文，如4.13节所述。

##### 3.14.2.2.3 原因字符串 Reason String

**31 (0x1F)** ，原因字符串（Reason String）标识符。  
跟随其后的是UTF-8编码字符串表示断开原因。此原因字符串是为诊断而设计的可读字符串，**不应该**被接收端所解析。 

如果此属性使得DISCONNECT报文的长度超出了接收端指定的最大报文长度，则发送端**不能**发送此属性 [MQTT-3.14.2-3]。包含多个原因字符串将造成协议错误（Protocol Error）。

##### 3.14.2.2.4 用户属性 User Property

**38 (0x26)** ，用户属性（User Property）标识符。  
跟随其后的是UTF-8字符串对。此属性可用于向客户端提供包括诊断信息在内的附加信息。如果加上用户属性之后的DISCONNECT报文长度超出了接收端指定的最大报文长度，则发送端**不能**发送此属性 \[MQTT-3.14.2-4\]。用户属性允许出现多次，以表示多个名字/值对，且相同的名字可以多次出现。

##### 3.14.2.2.5 服务端参考 Server Reference

**28 (0x1C)** ，服务端参考（Server Reference）标识符。  
跟随其后的是一个UTF-8编码字符串，客户端可以使用它来识别其他要使用的服务端。包含多个服务端参考将造成协议错误（Protocol Error）。

服务端发送包含一个服务端参考和原因码0x9C（（临时）使用其他服务端）或0x9D（服务端已（永久）移动）的DISCONNECT报文，如4.13节所述。

关于如何使用服务端参考，参考4.11节服务端重定向。

图 3-24 - DISCONNECT 报文可变报头非规范示例 DISCONNECT packet Variable Header non-normative example

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
    <td colspan="10">断开原因码</td>
  </tr>
  <tr>
    <td>byte 1</td>
    <td></td>
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
    <td colspan="10">属性</td>
  </tr>
  <tr>
    <td>byte 2</td>
    <td>长度 (5)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 3</td>
    <td>会话过期间隔标识符 (17)</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 4</td>
    <td rowspan="4">会话过期间隔标识符 (17)</td>
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
    <td>byte 5</td>
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
    <td>byte 6</td>
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

### 3.14.3 DISCONNECT 有效载荷 DISCONNECT Payload

DISCONNECT报文没有有效载荷。

### 3.14.4 DISCONNECT 行为 DISCONNECT Actions

发送端发送完DISCONNECT报文之后：
-    **不能**再在此网络连接上发送任何MQTT控制报文 \[MQTT-3.14.4-1\]。
-    **必须**关闭网络连接 \[MQTT-3.14.4-2\]。

接收到包含原因码为0x00（成功）的DISCONNECT时，服务端：
-    **必须**丢弃任何与当前连接相关的遗嘱消息，而不发布它 \[MQTT-3.14.4-3\]，如3.1.2.5节所述。

接收到DISCONNECT报文时，接收端：
-    **应该**关闭网络连接


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


