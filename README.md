# MQTT协议草案5.0中文版

by [hui6075](https://github.com/hui6075)

**最新版本: v0.0.1 2018-05-18** (部分3.3.1版本内容翻译引用 [@mcxiaoke](https://github.com/mcxiaoke/mqtt))

## 文档地址

- [中文翻译项目](https://github.com/hui6075/mqtt_v5)

## 概述

MQTT是一个客户端-服务端架构的发布/订阅模式的消息传输协议。它的设计思想是轻巧、开放、简单、规范，易于实现。这些特点使得它对很多场景来说都是很好的选择，特别是对于受限的环境如机器与机器的通信（M2M）以及物联网环境（IoT）。

MQTT协议5.0版本在3.1.1版本的基础上增加了会话/消息延时功能、原因码、主题别名、in-flight流控、属性、共享订阅等功能，增加了用于增强认证的AUTH报文。

## 目录

**发现任何翻译问题或格式问题欢迎提PR帮忙完善。**

- [项目自述](README.md)
- [前言](mqtt/00-Preface.md)
- [目录](mqtt/00-Contents.md)
- [第一章 - MQTT介绍](mqtt/01-Introduction.md)
- [第二章 – MQTT控制报文格式](mqtt/02-ControlPacketFormat.md)
- [第三章 – MQTT控制报文](mqtt/03-ControlPackets.md)
	- [3.1 CONNECT – 连接请求]
	- [3.2 CONNACK – 确认连接请求]
	- [3.3 PUBLISH – 发布消息]
	- [3.4 PUBACK –发布确认]
	- [3.5 PUBREC – 发布已接收（QoS 2，第一步）]
	- [3.6 PUBREL – 发布释放（QoS 2，第二步）]
	- [3.7 PUBCOMP – 发布完成（QoS 2，第三步）]
	- [3.8 SUBSCRIBE - 订阅请求]
	- [3.9 SUBACK – 订阅确认]
	- [3.10 UNSUBSCRIBE –取消订阅请求]
	- [3.11 UNSUBACK – 取消订阅确认]
	- [3.12 PINGREQ – PING请求]
	- [3.13 PINGRESP – PING响应]
	- [3.14 DISCONNECT – 断开连接]
	- [3.15 AUTH – 交换认证]
- [第四章 – 操作行为](mqtt/04-OperationalBehavior.md)
- [第五章 – 安全（非规范）](mqtt/05-Security.md)
- [第六章 – 使用WebSocket作为网络层](mqtt/06-WebSocket.md)
- [第七章 – 一致性](mqtt/07-Conformance.md)
- [附录B - 强制性规范声明（非规范）](mqtt/08-AppendixB.md)
- [附录C - MQTT v5.0新特性总结（非规范）](mqtt/09-AppendixC.md)

------

## 文档链接

>最新版本: v0.0.1

文档|链接
----|----
中文版 Docx | [MQTT v5.0 草案中文版](https://github.com/hui6075/mqtt_v5/blob/master/protocol/mqtt-v5.0-zh_cn.docx)
中文版 PDF | [MQTT v5.0 草案中文版](https://github.com/hui6075/mqtt_v5/blob/master/protocol/mqtt-v5.0-zh_cn.pdf)
英文版 HTML | [MQTT Version 5.0 cs01](http://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html)
英文版 PDF | [MQTT Version 5.0 cs01](http://docs.oasis-open.org/mqtt/mqtt/v5.0/cs01/mqtt-v5.0-cs01.pdf)


## 许可协议

- [署名-非商业性使用-相同方式共享 4.0 国际](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode)

------

## 联系方式

* Github: <https://github.com/hui6075>
* Email: [hui6075@outlook.com](mailto:hui6075#outlook.com)

## 开源项目

* Mosquitto集群: <https://github.com/hui6075/mosquitto-cluster>

------


