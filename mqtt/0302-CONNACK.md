## 3.2 CONNACK – 确认连接请求 Connect acknowledgement

CONNACK报文由服务端所发送，作为对来自客户端的CONNECT报文的响应。服务端在发送任何除AUTH以外的报文之前**必须**先发送包含原因码为0x00（成功）的CONNACK报文 \[MQTT-3.2.0-1\]。服务端在一次网络连接中**不能**发送多个CONNACK报文 \[MQTT-3.2.0-2\]。

如果客户端在合理的时间内没有收到服务端的CONNACK报文，客户端**应该**关闭网络连接。*合理* 的时间取决于应用的类型和通信基础设施。

### 3.2.1 CONNACK 固定报头 CONNACK Fixed Header

固定报头的格式见图 3-7的描述。

##### 图 3-7 – CONNACK 报文固定报头 CONNACK packet Fixed Header

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

CONNACK报文的可变报头按顺序包含以下字段：连接确认标志（Connect Acknowledge Flags），连接原因码（Reason Code），属性（Properties）。属性的编码规则如2.2.2节所描述。

#### 3.2.2.1 连接确认标志 Connect Acknowledge Flags

第1个字节是连接确认标志，位7-1是保留位且**必须**设置为0 \[MQTT-3.2.2-1\]。

第0（SP）位是会话存在标志（Session Present Flag）。

##### 3.2.2.1.1 会话存在 Session Present

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

#### 3.2.2.3 CONNACK属性 CONNACK Properties

##### 3.2.2.3.1 属性长度 Property Length

CONNACK报文可变报头中的属性长度，编码为变长字节整数。

##### 3.2.2.3.2 会话过期间隔 Session Expiry Interval

**17 (0x11)**，会话过期间隔（Session Expiry Interval）标识符。  
跟随其后的是用四字节整数表示的以秒为单位的会话过期间隔（Session Expiry Interval）。包含多个会话过期间隔（Session Expiry Interval）将造成协议错误（Protocol Error）。

如果会话过期间隔（Session Expiry Interval）值未指定，则使用CONNECT报文中指定的会话过期时间间隔。服务端使用此属性通知客户端它使用的会话过期时间间隔与客户端在CONNECT中发送的值不同。更详细的关于会话过期时间的描述，请参考3.1.2.11.2节。

##### 3.2.2.3.3 接收最大值 Receive Maximum

**33 (0x21)**，接收最大值（Receive Maximum）描述符。  
跟随其后的是由双字节整数表示的最大接收值。包含多个接收最大值或接收最大值为0将造成协议错误（Protocol Error）。

服务端使用此值限制服务端愿意为该客户端同时处理的QoS为1和QoS为2的发布消息最大数量。没有机制可以限制客户端试图发送的QoS为0的发布消息。

如果没有设置最大接收值，将使用默认值65535。

关于接收最大值的详细使用，参考4.9节流控部分。

##### 3.2.2.3.4 最大服务质量 Maximum QoS

**36 (0x24)**，最大服务质量（Maximum QoS）标识符。  
跟随其后的是用一个字节表示的0或1。包含多个最大服务质量（Maximum QoS）或最大服务质量既不为0也不为1将造成协议错误。如果没有设置最大服务质量，客户端可使用最大QoS为2。

如果服务端不支持Qos为1或2的PUBLISH报文，服务端**必须**在CONNACK报文中发送最大服务质量以指定其支持的最大QoS值 \[MQTT-3.2.2-9\]。即使不支持QoS为1或2的PUBLISH报文，服务端也**必须**接受请求QoS为0、1或2的SUBSCRIBE报文 \[MQTT-3.2.2-10\]。

如果从服务端接收到了最大QoS等级，则客户端**不能**发送超过最大QoS等级所指定的QoS等级的PUBLISH报文 \[MQTT-3.2.2-11\]。服务端接收到超过其指定的最大服务质量的PUBLISH报文将造成协议错误（Protocol Error）。这种情况下应使用包含原因码为0x9B（不支持的QoS等级）的DISCONNECT报文进行处理，如4.13节所述。

如果服务端收到包含遗嘱的QoS超过服务端处理能力的CONNECT报文，服务端**必须**拒绝此连接。服务端应该使用包含原因码为0x9B（不支持的QoS等级）的CONNACK报文进行错误处理，随后**必须**关闭网络连接。4.13节所述 \[MQTT-3.2.2-12\]。

> **非规范评注**
>
> 客户端不必支持QoS为1和2的PUBLISH报文。客户端只需将其发送的任何SUBSCRIBE报文中的QoS字段限制在其支持的最大服务质量以内即可。

##### 3.2.2.3.5 保留可用 Retain Available

**37 (0x25)**，保留可用（Retain Available）标识符。  
跟随其后的是一个单字节字段，用来声明服务端是否支持保留消息。值为0表示不支持保留消息，为1表示支持保留消息。如果没有设置保留可用字段，表示支持保留消息。包含多个保留可用字段或保留可用字段值不为0也不为1将造成协议错误（Protocol Error）。

如果服务端收到一个包含保留标志位1的遗嘱消息的CONNECT报文且服务端不支持保留消息，服务端**必须**拒绝此连接请求，且**应该**发送包含原因码为0x9A（不支持保留）的CONNACK报文，随后**必须**关闭网络连接 \[MQTT-3.2.2-13\]。

从服务端接收到的保留可用标志为0时，客户端**不能**发送保留标志设置为1的PUBLISH报文 \[MQTT-3.2.2-14\]。如果服务端收到这种PUBLISH报文，将造成协议错误（Protocol Error），此时服务端**应该**发送包含原因码为0x9A（不支持保留）的DISCONNECT报文，如4.13节所述。

##### 3.2.2.3.6 最大报文长度 Maximum Packet Size

**39 (0x27)**，最大报文长度（Maximum Packet Size）标识符。  
跟随其后的是由四字节整数表示的服务端愿意接收的最大报文长度（Maximum Packet Size）。如果没有设置最大报文长度，则按照协议由固定报头中的剩余长度可编码最大值和协议报头对数据包的大小做限制。

包含多个最大报文长度（Maximum Packet Size），或最大报文长度为0将造成协议错误（Protocol Error）。

如2.1.4节所述，最大报文长度是MQTT控制报文的总长度。服务端使用最大报文长度通知客户端其所能处理的单个报文长度限制。

客户端不能发送超过最大报文长度（Maximum Packet Size）的报文给服务端 \[MQTT-3.2.2-15\]。收到长度超过限制的报文将导致协议错误，此时服务端应该发送包含原因码0x95（报文过长）的DISCONNECT报文给客户端，详见4.13节。

##### 3.2.2.3.7 分配客户标识符 Assigned Client Identifier

**18 (0x12)**，分配客户标识符（Assigned Client Identifier）标识符。  
跟随其后的是UTF-8编码的分配客户标识符（Assigned Client Identifier）字符串。包含多个分配客户标识符将造成协议错误（Protocol Error）。

服务端分配客户标识符的原因是CONNECT报文中的客户标识符长度为0。

如果客户端使用长度为0的客户标识符（ClientID），服务端**必须**回复包含分配客户标识符（Assigned Client Identifier）的CONNACK报文。分配客户标识符**必须**是没有被服务端的其他会话所使用的新客户标识符 \[MQTT-3.2.2-16\]。

##### 3.2.2.3.8 主题别名最大值 Topic Alias Maximum

**34 (0x22)**，主题别名最大值（Topic Alias Maximum）标识符。  
跟随其后的是用双字节整数表示的主题别名最大值（Topic Alias Maximum）。包含多个主题别名最大值（Topic Alias Maximum）将造成协议错误（Protocol Error）。没有设置主题别名最大值属性的情况下，主题别名最大值默认为零。

此值指示了服务端能够接收的来自客户端的主题别名（Topic Alias）最大值。服务端使用此值来限制本次连接可以拥有的主题别名的值。客户端在一个PUBLISH报文中发送的主题别名值**不能**超过服务端设置的主题别名最大值（Topic Alias Maximum） \[MQTT-3.2.2-17\]。值为0表示本次连接服务端不接受任何主题别名（Topic Alias）。如果主题别名最大值（Topic Alias）没有设置，或者设置为0，则客户端**不能**向此服务端发送任何主题别名（Topic Alias） \[MQTT-3.2.2-18\]。

##### 3.2.2.3.9 原因字符串 Reason String

**31 (0x1F)**，原因字符串（Reason String）标识符。  
跟随其后的是UTF-8编码的字符串，表示此次响应相关的原因。此原因字符串（Reason String）是为诊断而设计的可读字符串，**不应该**被客户端所解析。

服务端使用此值向客户端提供附加信息。如果加上原因字符串之后的CONNACK报文长度超出了客户端指定的最大报文长度，则服务端**不能**发送此原因字符串 \[MQTT-3.2.2-19\]。包含多个原因字符串将造成协议错误（Protocol Error）。

> **非规范评注**
>
> 客户端对原因字符串的恰当使用包括：抛出异常时使用此字符串，或者将此字符串写入日志。

##### 3.2.2.3.10 用户属性 User Property

**38 (0x26)**，用户属性（User Property）标识符。  
跟随其后的是UTF-8字符串对。此属性可用于向客户端提供包括诊断信息在内的附加信息。如果加上用户属性之后的CONNACK报文长度超出了客户端指定的最大报文长度，则服务端**不能**发送此属性 \[MQTT-3.2.2-20\]。用户属性（User Property）允许出现多次，以表示多个名字/值对，且相同的名字可以多次出现。

用户属性的内容和意义本规范不做定义。CONNACK报文的接收端**可以**选择忽略此属性。

##### 3.2.2.3.11 通配符订阅可用 Wildcard Subscription Available

**40 (0x28)**，通配符订阅可用（Wildcard Subscription Available）标识符。  
跟随其后的是一个单字节字段，用来声明服务器是否支持通配符订阅（Wildcard Subscriptions）。值为0表示不支持通配符订阅，值为1表示支持通配符订阅。如果没有设置此值，则表示支持通配符订阅。包含多个通配符订阅可用属性，或通配符订阅可用属性值不为0也不为1将造成协议错误（Protocol Error）。

如果服务端在不支持通配符订阅（Wildcard Subscription）的情况下收到了包含通配符订阅的SUBSCRIBE报文，将造成协议错误（Protocol Error）。此时服务端将发送包含原因码为0xA2（通配符订阅不支持）的DISCONNECT报文，如4.13节所述。

服务端在支持通配符订阅的情况下仍然可以拒绝特定的包含通配符订阅的订阅请求。这种情况下，服务端**可以**发送一个包含原因码为0xA2（通配符订阅不支持）的SUBACK报文。

##### 3.2.2.3.12 订阅标识符可用 Subscription Identifier Available

**41 (0x29)**，订阅标识符可用（Subscription Identifier Available）标识符。  
跟随其后的是一个单字节字段，用来声明服务端是否支持订阅标识符（Subscription Identifiers）。值为0表示不支持订阅标识符，值为1表示支持订阅标识符。如果没有设置此值，则表示支持订阅标识符。包含多个订阅标识符可用属性，或订阅标识符可用属性值不为0也不为1将造成协议错误（Protocol Error）。

如果服务端在不支持订阅标识符（Subscription Identifier）的情况下收到了包含订阅标识符的SUBSCRIBE报文，将造成协议错误（Protocol Error）。此时服务端将发送包含原因码为0xA1（订阅标识符不支持）的DISCONNECT报文，如4.13节所述。

##### 3.2.2.3.13 共享订阅可用 Shared Subscription Available

**42 (0x2A)**，共享订阅可用（Shared Subscription Available）标识符。  
跟随其后的是一个单字节字段，用来声明服务端是否支持共享订阅（Shared Subscription）。值为0表示不支持共享订阅，值为1表示支持共享订阅。如果没有设置此值，则表示支持共享订阅。包含多个共享订阅可用（Shared Subscription Available），或共享订阅可用属性值不为0也不为1将造成协议错误（Protocol Error）。

如果服务端在不支持共享订阅（Shared Subscription）的情况下收到了包含共享订阅的SUBSCRIBE报文，将造成协议错误（Protocol Error）。此时服务端将发送包含原因码为0x9E（共享订阅不支持）的DISCONNECT报文，如4.13节所述。

##### 3.2.2.3.14 服务端保持连接 Server Keep Alive

**19 (0x13)**，服务端保持连接（Server Keep Alive）标识符。  
跟随其后的是由服务端分配的双字节整数表示的保持连接（Keep Alive）时间。如果服务端发送了服务端保持连接（Server Keep Alive）属性，客户端**必须**使用此值代替其在CONNECT报文中发送的保持连接时间值 [MQTT-3.2.2-21]。如果服务端没有发送服务端保持连接属性，服务端**必须**使用客户端在CONNECT报文中设置的保持连接时间值 [MQTT-3.2.2-22]。包含多个服务端保持连接属性将造成协议错误（Protocol Error）。

> **非规范评注**
>
> 服务端保持连接属性的主要作用是通知客户端它将会比客户端指定的保持连接更快的断开非活动的客户端。

##### 3.2.2.3.15 响应信息 Response Information

**26 (0x1A)**，响应信息（Response Information）标识符。  
跟随其后的是一个以UTF-8编码的字符串，作为创建响应主题（Response Topic）的基本信息。关于客户端如何根据响应信息（Response Information）创建响应主题不在本规范的定义范围内。包含多个响应信息将造成协议错误（Protocol Error）。

如果客户端发送的请求响应信息（Request Response Information）值为1，则服务端在CONNACK报文中发送响应信息（Response Information）为**可选**项。

> 非规范评注
>
> 响应信息通常被用来传递主题订阅树的一个全局唯一分支，此分支至少在该客户端的会话生命周期内为该客户端所保留。请求客户端和响应客户端的授权需要使用它，所以它通常不能仅仅是一个随机字符串。一般把此分支作为特定客户端的订阅树根节点。通常此信息需要正确配置，以使得服务器能返回信息。使用此机制时，具体的信息一般由服务端来进行统一配置，而非由各个客户端自己配置。

更多关于请求/响应的信息，请参考4.10节。

##### 3.2.2.3.16 服务端参考 Server Reference

**28 (0x1C)**，服务端参考（Server Reference）标识符。  
跟随其后的是一个以UTF-8编码的字符串，可以被客户端用来标识其他可用的服务端。包含多个服务端参考（Server Reference）将造成协议错误（Protocol Error）。

服务端在包含了原因码为0x9C（（临时）使用其他服务端）或0x9D（服务端已（永久）移动）的CONNACK报文或DISCONNECT报文中设置服务端参考，如4.13节所述。

关于如何使用服务端参考，请参考4.11节服务端重定向信息。

##### 3.2.2.3.17 认证方法 Authentication Method

**21 (0x15)**，认证方法（Authentication Method）标识符。  
跟随其后的是一个以UTF-8编码的字符串，包含了认证方法（Authentication Method）名。包含多个认证方法将造成协议错误（Protocol Error）。更多关于扩展认证的信息，请参考4.12节。

##### 3.2.2.3.18 认证数据 Authentication Data

**22 (0x16)**，认证数据（Authentication Data）标识符。  
跟随其后的是包含认证数据（Authentication Data）的二进制数据。此数据的内容由认证方法和已交换的认证数据状态定义。包含多个认证数据将造成协议错误（Protocol Error）。更多关于扩展认证的信息，请参考4.12节。

### 3.2.3 CONNACK 载荷 CONNACK Payload

CONNACK报文不包含有效载荷。

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

