# 第七章 一致性 Conformance

## 目录

- [第一章 - 介绍](01-Introduction.md)
- [第二章 – MQTT控制报文格式](02-ControlPacketFormat.md)
- [第三章 – MQTT控制报文](03-ControlPackets.md)
- [第四章 – 操作行为](04-OperationalBehavior.md)
- [第五章 – 安全（非规范）](05-Security.md)
- [第六章 – 使用WebSocket作为网络层](06-WebSocket.md)
- [第七章 – 一致性](07-Conformance.md)
- [附录B - 强制性规范声明（非规范）](08-AppendixB.md)
- [附录C - MQTT v5.0新特性总结（非规范）](09-AppendixC.md)

MQTT规范定义了MQTT客户端实现和MQTT服务端实现的一致性要求。MQTT实现可以同时作为MQTT客户端和MQTT服务端。

## 7.1 一致性条款 Conformance clauses

### 7.1.1 MQTT服务端一致性条款 MQTT Server conformance clause

服务端的定义，参考术语章节的[服务端（Server）](01-Introduction.md)部分。

MQTT服务端只有满足下面所有的要求才算是符合本规范：

1. 服务端发送的所有MQTT控制报文的格式符合[第二章](02-ControlPacketFormat.md)和[第三章](03-ControlPackets.md)描述的格式。
2. 遵守4.7节描述的主题匹配规则和4.8节匹配的订阅规则。
3. 满足下列章节中所有**必须**级别的要求，明确仅适用于对客户端的除外：
	- [第一章 – 介绍](01-Introduction.md)
	- [第二章 – MQTT控制报文格式](02-ControlPacketFormat.md)
	- [第三章 – MQTT控制报文](03-ControlPackets.md)
	- [第四章 – 操作行为](04-OperationalBehavior.md)
	- [第六章 – 使用WebSocket作为网络层](06-WebSocket.md)
4. 为了能够与任何其他一致的（MQTT）实现进行互操作，无需使用在规范之外定义的任何扩展。

### 7.1.2 MQTT客户端一致性条款 MQTT Client conformance clause

客户端的定义，参考术语章节的[客户端（Client）](01-Introduction.md)部分。

MQTT客户端只有满足下面所有的要求才算是符合本规范：

1. 客户端发送端所有MQTT控制报文的格式符合[第二章](02-ControlPacketFormat.md)和[第三章](03-ControlPackets.md)描述的格式。
2. 满足下列章节中所有**必须**级别的要求，明确仅适用于对服务端的除外：
	- [第一章 – 介绍](01-Introduction.md)
	- [第二章 – MQTT控制报文格式](02-ControlPacketFormat.md)
	- [第三章 – MQTT控制报文](03-ControlPackets.md)
	- [第四章 – 操作行为](04-OperationalBehavior.md)
	- [第六章 – 使用WebSocket作为网络层](06-WebSocket.md)
3. 为了能够与任何其他一致的（MQTT）实现进行互操作，无需使用在规范之外定义的任何扩展。


### 项目主页

- [MQTT协议中文版](https://github.com/hui6075/mqtt_v5)


