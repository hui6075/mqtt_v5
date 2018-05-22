## 3.2 CONNACK – 确认连接请求 Connect acknowledgement

CONNACK报文由服务端所发送，作为对来自客户端的CONNECT报文的响应。服务端在发送任何除AUTH以外的报文之前**必须**先发送包含原因码为0x00（成功）的CONNACK报文 /[MQTT-3.2.0-1/]。服务端在一次网络连接中**不能**发送多个CONNACK报文 /[MQTT-3.2.0-2/]。

如果客户端在合理的时间内没有收到服务端的CONNACK报文，客户端**应该**关闭网络连接。*合理* 的时间取决于应用的类型和通信基础设施。

### 3.2.1 CONNACK 固定报头 CONNACK Fixed Header

固定报头的格式见图 3-7 的描述。

##### 图 3-7 – CONNACK 报文固定报头 CONNACK packet Fixed Header

  <table style="text-align:center">
     <tr>
       <td width=120 align="center"><strong>Bit</strong></td>
       <td width=60 align="center"><strong>7</strong></td>
       <td width=60 align="center"><strong>6</strong></td>
       <td width=60 align="center"><strong>5</strong></td>
       <td width=60 align="center"><strong>4</strong></td>
       <td width=60 align="center"><strong>3</strong></td>
       <td width=60 align="center"><strong>2</strong></td>
       <td width=60 align="center"><strong>1</strong></td>
       <td width=60 align="center"><strong>0</strong></td>
     </tr>
     <tr>
       <td>byte 1</td>
       <td colspan="4" align="center">MQTT报文类型 (2)</td>
       <td colspan="4" align="center">Reserved 保留位</td>
     </tr>
     <tr>
       <td></td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
     </tr>
     <tr>
       <td>byte 2...</td>
       <td colspan="8" align="center">剩余长度 (2)</td>
     </tr>
     <tr>
       <td></td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">0</td>
       <td align="center">1</td>
       <td align="center">0</td>
     </tr>
   </table>

**剩余长度字段**

表示可变报头的长度。对于CONNACK报文这个值等于2。

### 3.2.2 CONNACK 可变报头 CONNACK Variable Header

CONNACK报文的可变报头按顺序包含以下字段：连接确认标志（Connect Acknowledge Flags），连接原因码（Reason Code），属性（Properties）。属性的编码规则如2.2.2节 所描述。

#### 3.2.2.1 连接确认标志 Connect Acknowledge Flags

第1个字节是连接确认标志，位7-1是保留位且**必须**设置为0 \[MQTT-3.2.2-1\]。

第0（SP）位是会话存在标志（Session Present Flag）。

###### 3.2.2.1.1 会话存在 Session Present

**位置：** 连接确认标志（Connect Acknowledge Flags）的第0位。

会话存在（Session Present）标志通知客户端，服务端是否正在使用此客户标识符之前连接的会话状态（Session State）。会话存在标志使服务端和客户端在是否有已存储的会话状态上保持一致。

如果服务端接受一个新开始（Clean Start）为1的连接，服务端在CONNACK报文中除了把原因码设置为0x00（成功）之外，还**必须**把会话存在标志设置为0 \[MQTT-3.2.2-2\]。

如果服务端接受一个新开始（Clean Start）为0的连接，并且服务端已经保存了此客户标识符（ClientID）的会话状态（Session State），服务端在CONNACK报文中**必须**把会话存在标志设置为1。否则，服务端**必须**把会话存在标志设置为0。无论如何，服务端在CONNACK报文中**必须**把原因码设置为0x00（成功） \[MQTT-3.2.2-3\]。

如果客户端从服务端接收到的会话存在标志值与预期的不同，客户端做如下处理：
-    如果客户端没有保存的会话状态，但收到会话存在标志为1，客户端**必须**关闭网络连接 \[MQTT-3.2.2-4\]。 如果希望重新开始一个新的会话，客户端可以使用新开始（Clean Start）为1并重新连接服务端。
-    如果客户端保存了会话状态，但收到的会话存在标志为0，客户端若要继续此网络连接，它**必须**丢弃其保存的会话状态 \[MQTT-3.2.2-5\]。

如果服务端发送的CONNACK报文中原因码非0，它**必须**把会话存在标志设置为0 \[MQTT-3.2.2-6\]。

#### 3.2.2.2 连接原因码 Connect Reason Code

可变报头中第2个字节是连接原因码（Reason Code）。

连接原因码（Reason Code）的值如下所示。如果服务端收到一个格式正确的CONNECT报文，但服务端无法完成连接的创建，服务端**可以**发送一个包含适当的连接原因码的CONNACK报文。如果服务端发送了一个包含原因码大于等于128的CONNACK报文，它随后**必须**关闭网络连接 \[MQTT-3.2.2-7\]。

表 3-1 - 连接原因码 Connect Reason Code values

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
	<td>连接被接受。</td>
  </tr>
  <tr>
    <td>128</td>
	<td>0x80</td>
	<td>未指明的错误</td>
	<td>服务端不愿透露的错误，或者没有适用的原因码。</td>
  </tr>
  <tr>
    <td>129</td>
	<td>0x81</td>
	<td>无效报文</td>
	<td>CONNECT报文内容不能被正确的解析。 </td>
  </tr>
  <tr>
    <td>130</td>
	<td>0x82</td>
	<td>协议错误</td>
	<td>CONNECT报文内容不符合本规范。</td>
  </tr>
  <tr>
    <td>131</td>
	<td>0x83</td>
	<td>实现特定错误</td>
	<td>CONNECT有效，但不被服务端所接受。</td>
  </tr>
  <tr>
    <td>132</td>
	<td>0x84</td>
	<td>协议版本不支持</td>
	<td>服务端不支持客户端所请求的MQTT协议版本。</td>
  </tr>
  <tr>
    <td>133</td>
	<td>0x85</td>
	<td>客户标识符无效</td>
	<td>客户标识符有效，但未被服务端所接受。</td>
  </tr>
  <tr>
    <td>134</td>
	<td>0x86</td>
	<td>用户名密码错误</td>
	<td>客户端指定的用户名密码未被服务端所接受。</td>
  </tr>
  <tr>
    <td>135</td>
	<td>0x87</td>
	<td>未授权</td>
	<td>客户端未被授权连接。</td>
  </tr>
  <tr>
    <td>136</td>
	<td>0x88</td>
	<td>服务端不可用</td>
	<td>MQTT服务端不可用。</td>
  </tr>
  <tr>
    <td>137</td>
	<td>0x89</td>
	<td>服务端正忙</td>
	<td>服务端正忙，请重试。</td>
  </tr>
  <tr>
    <td>138</td>
	<td>0x8A</td>
	<td>禁止</td>
	<td>客户端被禁止，请联系服务端管理员。</td>
  </tr>
  <tr>
    <td>140</td>
	<td>0x8C</td>
	<td>无效的认证方法</td>
	<td>认证方法未被支持，或者不匹配当前使用的认证方法。</td>
  </tr>
  <tr>
    <td>144</td>
	<td>0x90</td>
	<td>主题名无效</td>
	<td>遗嘱主题格式正确，但未被服务端所接受。</td>
  </tr>
  <tr>
    <td>149</td>
	<td>0x95</td>
	<td>报文过长</td>
	<td>CONNECT报文超过最大允许长度。</td>
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
	<td>遗嘱载荷数据与载荷格式指示符不匹配。</td>
  </tr>
  <tr>
    <td>154</td>
	<td>0x9A</td>
	<td>不支持保留</td>
	<td>遗嘱保留标志被设置为1，但服务端不支持保留消息。</td>
  </tr>
  <tr>
    <td>155</td>
	<td>0x9B</td>
	<td>不支持的QoS等级</td>
	<td>服务端不支持遗嘱中设置的QoS等级。</td>
  </tr>
  <tr>
    <td>156</td>
	<td>0x9C</td>
	<td>（临时）使用其他服务端</td>
	<td>客户端应该临时使用其他服务端。</td>
  </tr>
  <tr>
    <td>157</td>
	<td>0x9D</td>
	<td>服务端已（永久）移动</td>
	<td>客户端应该永久使用其他服务端</td>
  </tr>
  <tr>
    <td>159</td>
	<td>0x9F</td>
	<td>超出连接速率限制</td>
	<td>超出了所能接受的连接速率限制。</td>
  </tr>
</table>

服务端发送的CONNACK报文**必须**设置一种原因码 \[MQTT-3.2.2-8\]。

> **非规范评注**
>
> 原因码0x80（未指明的错误）可以被用作：服务器知道失败的原因但是并不希望透露给客户端，或者没有其他适用的原因码。
>
> 出于安全考虑，发现CONNECT出错时服务端可以选择不发送CONNACK报文而关闭网络连接。例如，在公网中向未被授权的网络连接告知自身MQTT服务端身份并不明智。


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

### 项目主页

- [MQTT协议中文版](https://github.com/mcxiaoke/mqtt)


