# 第一章 概述 Introduction

## 1.0 知识产权政策 Intellectual property rights policy

此公开评审草案的发布基于 [OASIS IPR Policy](https://www.oasis-open.org/policies-guidelines/ipr) 的 [Non-Assertion](https://www.oasis-open.org/policies-guidelines/ipr#Non-Assertion-Mode) 模式。

关于实现本规范必不可少的任何专利是否已公开，以及其他的专利许可条款相关的信息，请参考技术委员会网站的知识产权部分(<https://www.oasis-open.org/committees/mqtt/ipr.php>).

## 1.1 MQTT协议的组织结构 Organization of MQTT

本规范分为七个章节：

- [第一章 – 介绍](01-Introduction.md)
- [第二章 – MQTT控制报文格式](02-ControlPacketFormat.md)
- [第三章 – MQTT控制报文](03-ControlPackets.md)
- [第四章 – 操作行为](04-OperationalBehavior.md)
- [第五章 – 安全（非规范）](05-Security.md)
- [第六章 – 使用WebSocket作为网络层](06-WebSocket.md)
- [第七章 – 一致性](07-Conformance.md)
- [附录B - 强制性规范声明（非规范）](08-AppendixB.md)
- [附录C - MQTT v5.0新特性总结（非规范）](09-AppendixC.md)

## 1.2 术语 Terminology
        
本规范中用到的关键字 **必须** MUST，**不能** MUST NOT，**要求** REQUIRED，**将会** SHALL，**不会** SHALL NOT，**应该** SHOULD，**不应该** SHOULD NOT，**推荐** RECOMMENDED，**可以** MAY，**可选** OPTIONAL 都是按照 IETF RFC 2119 [\[RFC2119\]](#anchor-RFC2119) 中的描述解释。

**网络连接 Network Connection**

MQTT使用的底层传输协议基础设施。

-   客户端使用它连接服务端。
-   它提供有序的、可靠的、双向字节流传输。

例子见4.2节。

**应用消息 Application Message**
MQTT协议通过网络传输应用数据。应用消息通过MQTT传输时，它们有关联的服务质量（QoS）和主题（Topic）。

**客户端 Client**  
使用MQTT的程序或设备。客户端总是通过网络连接到服务端。它可以

- 	打开连接到服务端的网络连接
-   发布应用消息给其它相关的客户端
-   订阅以请求接受相关的应用消息
-   取消订阅以移除接受应用消息的请求
-   关闭连接到服务端的网络连接

**服务端 Server**  
一个程序或设备，作为发送消息的客户端和请求订阅的客户端之间的中介。服务端

-   接受来自客户端的网络连接
-   接受客户端发布的应用消息
-   处理客户端的订阅和取消订阅请求
-   转发应用消息给符合条件的已订阅客户端
-   关闭来自客户端的网络连接

**会话 Subscription**  
客户端和服务端之间的状态交互。一些会话持续时长与网络连接一样，另一些可以在客户端和服务端的多个连续网络连接间扩展。

**订阅 Subscription**  
订阅包含一个主题过滤器（Topic Filter）和一个最大的服务质量（QoS）等级。订阅与单个会话（Session）关联。会话可以包含多于一个的订阅。会话的每个订阅都有一个不同的主题过滤器。

**共享订阅 Shared Subscription**  
一个共享订阅包含一个主题过滤器（Topic Filter）和一个最大的服务质量（QoS）等级。一个共享订阅可以与多个订阅会话相关联，便于支持大范围消息交换模式。一条主题匹配的应用消息只发送给关联到此共享订阅的多个会话中的一个会话。一个会话可以包括多个共享订阅，可以同时包含共享订阅与非共享订阅。

**通配符订阅 Wildcard Subscription**  
通配符订阅是指主题过滤器（Topic Filter）包含一个或多个通配符的订阅。通配符订阅使得一次订阅匹配多个主题名（Topic Name）。4.7节 描述了主题过滤器中的通配符。

**主题名 Topic Name**  
附加在应用消息上的一个标签，服务端已知且与订阅匹配。服务端发送应用消息的一个副本给每一个匹配的客户端订阅。

**主题过滤器 Topic Filter**  
订阅中包含的一个表达式，用于表示相关的一个或多个主题。主题过滤器可以使用通配符。

**MQTT控制报文 MQTT Control Packet**  
通过网络连接发送的信息数据包。MQTT 规范定义了十四种不同类型的MQTT控制报文，其中一个（PUBLISH 报文）用于传输应用消息。

**无效报文 Malformed Packet**  
根据规范不能被正确解析的控制报文。4.13节 描述了如何进行相应的错误处理。

**协议错误 Protocol Error**  
在报文解析之后发现包含协议不允许或与客户端或服务端当前状态不一致的数据的错误。4.13节 描述了如何进行相应的错误处理。

**遗嘱消息 Will Message**  
在网络连接非正常关闭的情况下，由服务端发布的应用消息。3.1.2.5节 描述了遗嘱消息。

## 1.3 规范引用 Normative references

\[RFC2119\]
Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997,  
<http://www.rfc-editor.org/info/rfc2119>

\[RFC3629\]
Yergeau, F., "UTF-8, a transformation format of ISO 10646", STD 63, RFC 3629, DOI 10.17487/RFC3629, November 2003,  
<http://www.rfc-editor.org/info/rfc3629>

\[RFC6455\]
Fette, I. and A. Melnikov, "The WebSocket Protocol", RFC 6455, DOI 10.17487/RFC6455, December 2011,  
<http://www.rfc-editor.org/info/rfc6455>

\[Unicode\]
The Unicode Consortium. The Unicode Standard,  
<http://www.rfc-editor.org/info/rfc2119>

## 1.4 非规范引用 Non-normative references

\[RFC0793\]
Postel, J., "Transmission Control Protocol", STD 7, RFC 793, DOI 10.17487/RFC0793, September 1981,  
<http://www.rfc-editor.org/info/rfc793>

\[RFC5246\]
Dierks, T. and E. Rescorla, "The Transport Layer Security (TLS) Protocol Version 1.2", RFC 5246, DOI 10.17487/RFC5246, August 2008,  
<http://www.rfc-editor.org/info/rfc2119>

\[AES\]
Advanced Encryption Standard (AES) (FIPS PUB 197).  
<https://csrc.nist.gov/csrc/media/publications/fips/197/final/documents/fips-197.pdf>

\[CHACHA20\]
ChaCha20 and Poly1305 for IETF Protocols  
<https://tools.ietf.org/html/rfc7539>

\[FIPS1402\]
Security Requirements for Cryptographic Modules (FIPS PUB 140-2)  
<https://csrc.nist.gov/csrc/media/publications/fips/140/2/final/documents/fips1402.pdf>

\[IEEE 802.1AR\]
IEEE Standard for Local and metropolitan area networks - Secure Device Identity  
<http://standards.ieee.org/findstds/standard/802.1AR-2009.html>

\[ISO29192\]
ISO/IEC 29192-1:2012 Information technology -- Security techniques -- Lightweight cryptography -- Part 1: General  
<https://www.iso.org/standard/56425.html>

\[MQTT NIST\]
MQTT supplemental publication, MQTT and the NIST Framework for Improving Critical Infrastructure Cybersecurity  
<http://docs.oasis-open.org/mqtt/mqtt-nist-cybersecurity/v1.0/mqtt-nist-cybersecurity-v1.0.html>

\[MQTTV311\]
MQTT V3.1.1 Protocol Specification  
<http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html>

\[ISO20922\]
MQTT V3.1.1 ISO Standard (ISO/IEC 20922:2016)  
<https://www.iso.org/standard/69466.html>

\[NISTCSF\]
Improving Critical Infrastructure Cybersecurity Executive Order 13636  
<https://www.nist.gov/sites/default/files/documents/itl/preliminary-cybersecurity-framework.pdf>

\[NIST7628\]
NISTIR 7628 Guidelines for Smart Grid Cyber Security Catalogue  
<https://www.nist.gov/sites/default/files/documents/smartgrid/nistir-7628_total.pdf>

\[NSAB\]
NSA Suite B Cryptography  
<http://www.nsa.gov/ia/programs/suiteb_cryptography/>

\[PCIDSS\]
PCI-DSS Payment Card Industry Data Security Standard  
<https://www.pcisecuritystandards.org/pci_security/>

\[RFC1928\]
Leech, M., Ganis, M., Lee, Y., Kuris, R., Koblas, D., and L. Jones, "SOCKS Protocol Version 5", RFC 1928, DOI 10.17487/RFC1928, March 1996,  
<http://www.rfc-editor.org/info/rfc1928>

\[RFC4511\]
Sermersheim, J., Ed., "Lightweight Directory Access Protocol (LDAP): The Protocol", RFC 4511, DOI 10.17487/RFC4511, June 2006,  
<http://www.rfc-editor.org/info/rfc4511>

\[RFC5280\]
Cooper, D., Santesson, S., Farrell, S., Boeyen, S., Housley, R., and W. Polk, "Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile", RFC 5280, DOI 10.17487/RFC5280, May 2008,  
<http://www.rfc-editor.org/info/rfc5280>

\[RFC6066\]
Eastlake 3rd, D., "Transport Layer Security (TLS) Extensions: Extension Definitions", RFC 6066, DOI 10.17487/RFC6066, January 2011,  
<http://www.rfc-editor.org/info/rfc6066>

\[RFC6749\]
Hardt, D., Ed., "The OAuth 2.0 Authorization Framework", RFC 6749, DOI 10.17487/RFC6749, October 2012,  
<http://www.rfc-editor.org/info/rfc6749>

\[RFC6960\]
Santesson, S., Myers, M., Ankney, R., Malpani, A., Galperin, S., and C. Adams, "X.509 Internet Public Key Infrastructure Online Certificate Status Protocol - OCSP", RFC 6960, DOI 10.17487/RFC6960, June 2013,  
<http://www.rfc-editor.org/info/rfc6960>

\[SARBANES\]
Sarbanes-Oxley Act of 2002.  
<http://www.gpo.gov/fdsys/pkg/PLAW-107publ204/html/PLAW-107publ204.htm>

\[USEUPRIVSH\]
U.S.-EU Privacy Shield Framework  
<https://www.privacyshield.gov>

\[RFC3986\]
Berners-Lee, T., Fielding, R., and L. Masinter, "Uniform Resource Identifier (URI): Generic Syntax", STD 66, RFC 3986, DOI 10.17487/RFC3986, January 2005,  
<http://www.rfc-editor.org/info/rfc3986>

\[RFC1035\]
Mockapetris, P., "Domain names - implementation and specification", STD 13, RFC 1035, DOI 10.17487/RFC1035, November 1987,  
<http://www.rfc-editor.org/info/rfc1035>

\[RFC2782\]
Gulbrandsen, A., Vixie, P., and L. Esibov, "A DNS RR for specifying the location of services (DNS SRV)", RFC 2782, DOI 10.17487/RFC2782, February 2000,  
<http://www.rfc-editor.org/info/rfc2782>

## 1.5 数据表示 Data representations

### 1.5.1 二进制位 Bits

字节中的位从0到7。第7位是最高有效位，第0位是最低有效位。

### 1.5.2 双字节整数 Two Byte Integer

双字节整数是 16 位，使用大端序（big-endian，高位字节在低位字节前面）。这意味着一个 16 位的字在网络上表示为最高有效字节（MSB），后面跟着最低有效字节（LSB）。

### 1.5.3 四字节整数 Four Byte Integer

四字节整数是32位，使用大端序（big-endian，高位字节在低位字节前面）。这意味着一个32位的字在网络上表示为第一个最高有效字节（MSB）后面跟着第一个最低有效字节（LSB），再后面为第二个最高有效字节（MSB）后面跟着第二个最低有效字节（LSB）。

### 1.5.4 UTF-8编码字符串 UTF-8 encoded strings

后面会描述的控制报文中的文本字段编码为UTF-8格式的字符串。UTF-8 \[[RFC3629](#RFC3629)\] 是一个高效的Unicode字符编码格式，为了支持基于文本的通信，它对ASCII字符的编码做了优化。

每一个字符串都有一个两字节的长度字段作为前缀，它给出这个字符串UTF-8编码的字节数，它们在[图例 1.1 UTF-8编码字符串的结构](#_Figure_1.1_Structure) 中描述。因此可以传送的UTF-8编码的字符串大小有一个限制，不能超过 65535字节。

除非另有说明，所有的UTF-8编码字符串的长度都在0到65535字节这个范围内。

##### 图 1-1 - UTF-8编码字符串的结构 Structure of UTF-8 encoded strings

<table>
  <tr>
    <th width=166>二进制位</th>
    <th width=65>7</th>
    <th width=65>6</th>
    <th width=65>5</th>
    <th width=65>4</th>
    <th width=65>3</th>
    <th width=65>2</th>
    <th width=65>1</th>
    <th width=65>0</th>
  </tr>
  <tr>
    <td>byte 1</td>
	<td colspan="9">字符串长度的最高有效字节（MSB）</td>
  </tr>
  <tr>
    <td>byte 2</td>
	<td colspan="9">字符串长度的最低有效字节（LSB）</td>
  </tr>
  <tr>
    <td>byte 3 ...</td>
	<td colspan="9">如果长度大于0，这里是UTF-8编码的字符数据。</td>
  </tr>
</table>

UTF-8编码字符串中的字符数据**必须**是按照Unicode规范 \[[Unicode](#Unicode)\] 定义的和在RFC3629 \[[RFC3629](#RFC3629)\] 中重申的有效的UTF-8格式。特别需要指出的是，这些数据**不能**包含字符码在U+D800和U+DFFF之间的数据。如果服务端或客户端收到了一个包含无效UTF-8字符的控制报文，它**必须**关闭网络连接  \[MQTT-1.5.3-1\]。

UTF-8编码的字符串**不能**包含空字符U+0000。如果客户端或服务端收到了一个包含U+0000的控制报文，它**必须**关闭网络连接 \[MQTT-1.5.3-2\]。

数据中**不应该**包含下面这些Unicode代码点的编码。如果一个接收者（服务端或客户端）收到了包含下列任意字符的控制报文，它**可以**关闭网络连接：

- U+0001和U+001F之间的控制字符
- U+007F和U+009F之间的控制字符
- Unicode规范定义的非字符代码点（例如U+0FFFF）
- Unicode规范定义的保留字符（例如U+0FFFF）

UTF-8编码序列0XEF 0xBB 0xBF总是被解释为U+FEFF（零宽度非换行空白字符），无论它出现在字符串的什么位置，报文接收者都不能跳过或者剥离它  \[MQTT-1.5.3-3\]。

#### 非规范示例 Non normative example

> 例如，字符串 A𪛔 是一个拉丁字母A后面跟着一个代码点U+2A6D4(它表示一个中日韩统一表意文字扩展B中的字符)，这个字符串编码如下：

##### 图 1-2 - UTF-8编码字符串非规范示例 UTF-8 encoded string non normative example

<table>
  <tr>
    <th width=140>比特位</th>
    <th width=65>7</th>
    <th width=65>6</th>
    <th width=65>5</th>
    <th width=65>4</th>
    <th width=65>3</th>
    <th width=65>2</th>
    <th width=65>1</th>
    <th width=65>0</th>
  </tr>
  <tr>
    <td>byte 1</td>
	<td colspan="9" align="center">字符串长度MSB (0x00)</td>
  </tr>
  <tr>
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
    <td>byte 2</td>
	<td colspan="9" align="center">字符串长度LSB (0x05)</td>
  </tr>
  <tr>
    <td></td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 3</td>
	<td colspan="9" align="center">‘A’ (0x41)</td>
  </tr>
  <tr>
    <td></td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 4</td>
	<td colspan="9" align="center">(0xF0)</td>
  </tr>
  <tr>
    <td></td>
	<td>1</td>
	<td>1</td>
	<td>1</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
	<td>0</td>
  </tr>
  <tr>
    <td>byte 5</td>
	<td colspan="9" align="center">(0xAA)</td>
  </tr>
  <tr>
    <td></td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
  </tr>
  <tr>
    <td>byte 6</td>
	<td colspan="9" align="center">(0x9B)</td>
  </tr>
  <tr>
    <td></td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
	<td>1</td>
  </tr>
  <tr>
    <td>byte 7</td>
	<td colspan="9" align="center">(0x94)</td>
  </tr>
  <tr>
    <td></td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>1</td>
	<td>0</td>
	<td>0</td>
  </tr>
</table>

### 1.5.5 变长字节整数 Variable Byte Integer

剩余长度字段使用一个变长字节编码方案，对小于 128 的值它使用单字节编码。更大的值按下面的方式处理。低 7 位有效位用于编码数据，最高有效位用于指示是否有更多的字节。因此每个字节可以编码 128 个数值和一个延续位（continuation bit）。剩余长度字段最大 4 个字节[MQTT-1.5.5-1]，如表 1-1 所示。

表 1-1 - 变长字节整数大小 Size of Variable Byte Integer

<table>
  <tr>
    <th width=100>字节数</th>
    <th>最小值</th>
    <th>最大值</th>
  </tr>
  <tr>
    <td align="center">1</td>
	<td>0 (0x00)</td>
	<td>127 (0x7F)</td>
  </tr>
  <tr>
    <td align="center">2</td>
	<td>128 (0x80, 0x01)</td>
	<td>16,383 (0xFF, 0x7F)</td>
  </tr>
  <tr>
    <td align="center">3</td>
	<td>16,384 (0x80, 0x80, 0x01)</td>
	<td>2,097,151 (0xFF, 0xFF, 0x7F)</td>
  </tr>
  <tr>
    <td align="center">4</td>
	<td>2,097,152 (0x80, 0x80, 0x80, 0x01)</td>
	<td>268,435,455 (0xFF, 0xFF, 0xFF, 0x7F)</td>
  </tr>
</table>

#### 非规范示例 Non normative example

> 非负整数 X 使用变长编码方案的算法如下：
> do <br>
> 　　encodedByte = X MOD 128 <br>
> 　　X = X DIV 128 <br>
> 　　// if there are more data to encode, set the top bit of this byte <br>
> 　　if (X > 0) <br>
> 　　　　encodedByte = encodedByte OR 128 <br>
> 　　endif <br>
> 　　'output' encodedByte <br>
> while (X > 0) <br>
> MOD是模运算，DIV是整数除法，OR是位操作或（C语言中分别是%，/，|）。

#### 非规范示例 Non normative example

> 剩余长度字段的解码算法如下：
> multiplier = 1 <br>
> value = 0 <br>
> do <br>
> 　　encodedByte = 'next byte from stream' <br>
> 　　value += (encodedByte AND 127) * multiplier <br>
> 　　if (multiplier > 128*128*128) <br>
> 　　　　throw Error(Malformed Variable Byte Integer) <br>
> 　　multiplier *= 128 <br>
> while ((encodedByte AND 128) != 0) <br>
> AND 是位操作与（C 语言中的&）

这个算法终止时，value 包含的就是剩余长度的值。

### 1.5.6 二进制数据 Binary Data

二进制数据由一个双字节整数指示其数据长度，因此，二进制数据的长度被限制为0到65,535字节。 

### 1.5.7 UTF-8字符串对 UTF-8 String Pair

UTF-8字符串对由两个UTF-8编码的字符串组成，用来表示名字-值对，第一个字符串表示名字，第二个字符串表示值。

所有的字符串必须遵循UTF-8字符串编码规范 \[MQTT-1.5.7-1\]。如果接受者（客户端或者服务端）接受到一个字符串对，然而其编码并不遵循规范，则此报文为无效报文。4.13节描述了错误处理的信息。

## 1.6 安全 Security

MQTT客户端和服务端实现应该提供认证、授权和安全通信功能，如第5章所描述。强烈建议任何关注于个人身份信息或敏感信息的应用使用这些安全设施。

## 1.7 编辑约定 Editing conventions

本规范用黄色高亮的文本标识一致性声明，每个一致性声明都分配了一个这种格式的引用：\[MQTT-x.x.x-y\]。

## 1.8 变更历史 Editing conventions

### 1.8.1 MQTT v3.1.1

MQTT v3.1.1 是首个OASIS标准版本MQTT **\[MQTTV311\]**。  
MQTT v3.1.1也是ISO/IEC 20922:2016 \[ISO20922\] 标准。

### 1.8.2 MQTT v5.0

MQTT v5.0 在保持MQTT核心不变的基础上添加了大量的新功能。这些功能的主要目标如下：
- 	进一步支持大规模可扩展系统
- 	改进的错误报告
- 	规范化包括容量探索和请求响应在内的通用模式
- 	包括用户属性在内的可扩展机制
- 	改进性能并支持小型客户端

### 项目主页

- [MQTT v5.0协议草案中文版](https://github.com/hui6075/mqtt_v5)
