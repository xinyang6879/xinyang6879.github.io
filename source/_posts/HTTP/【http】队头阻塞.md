---
title: 【Htpp】队头阻塞
categories: http
tag: http
---

## http1队头阻塞

除了TCP的本身限制外，http1的先进先出（FIFO）的设计导致的。即浏览器在发送了请求后，必须按照发送顺序获取到相应的响应结果。例如请求A、请求B按顺序发送，请求A需要1s响应，请求B需要0.1s响应，即使B请求响应更加快速，但是浏览器依旧需要等待A的响应才能获取到B的响应。这就是http1的队头阻塞

### FIFO

#### 使用原因

- 简化协议实现

    http1设计在早期互联网，顺序处理模型更加简单，减少协议复杂性

- 避免资源竞争

    顺序处理可以避免服务器或客户端因并发处理多个请求导致数据混乱

- 兼容性

    早期浏览器和服务器对并发有限制，因此使用FIFO是权衡之后的结果

#### 表现

请求顺序化和相应顺序化

#### 导致问题

- 队头阻塞

- 性能问题

    高并发情况下，FIFO机制会导致连接利用率低下，无法充分利用带宽

#### 改进尝试

管道化：允许客户端在未接受响应是连续发送多个请求

- 缺点

    - 响应必须按顺序返回
    - 服务器实现复杂，且易因队头阻塞导致性能问题
    - 浏览器默认禁用管道话

因此管道化实际并未解决队头阻塞问题

## http2

### 如何解决

- 二进制分帧

    http2中引入了二进制分帧的概念，即将请求的参数及数据全部分解为一个一个的二进制格式帧，这样可以提升传输和解析的效率。

- 多路复用

    流是一组双向的帧序列，每个流都有一个唯一的标识符，并且可以在同一个连接中并发传输多个流，可以最大程度实现多路复用

    一个流可以由多个帧组成，这些帧按顺序发送，接收方根据流标识符将其重新组合为完整的请求或响应

因此多个http可以同时并发传输或者交错传输，从而减少队头阻塞

### 帧

#### 内容

每个帧有一个固定头部（9字节），包括：

- 长度：Length

    帧负载长度，由24位3个字节大小表示

- 类型：Type

    帧类型，说明格式及其语义，用8位1个字节表示

- 标志：Flags

    指示帧的特定属性或状态，用8位1个字节表示

- 流标识符：Stream Identifier

    表示该帧关联的流，用31为表示，上限为2^31

- 帧数据

    传输的数据内容Payload由帧类型决定

- R
    1位保留字段，未定义，以0x0结尾


#### 帧类型

Pad Length：填充字节长度
E：标识流是否为独占，设置PRIORITY时才有值
Stream Dependency：流的依赖流，设置PRIORITY时才有值
Weight：流优先级权重，设置PRIORITY时才有值
Header Block Fragment：Header块片段
Padding：填充字节长度

- DATA帧

    数据帧主要存储http2数据报文，包含8位填充字节+填充字节长度（PADDED）标记位true时说明由填充字节+Data具体传输的数据

- HEADER帧

    包含Pad Length（填充字节长度）+E（标识流是否为独占，设置PRIORITY时才有值）+Stream Dependency（流的依赖流，设置PRIORITY时才有值）+ Weight（流优先级权重，设置PRIORITY时才有值）+ Header Block Fragment（Header块片段）+ Padding（填充字节长度）

- PROIRITY帧

    发送流的优先级，包含E+Stream Dependency+Weight

- RST_STREAM帧

    当发送错误或取消时，用于终止一个流，包含Error Code，32位错误代码，说明错误原因

- SETTINGS帧

    用于传达连接端点之间的配置参数，标为ACK=0表示被对等的SETTINGS帧使用，不为0时表示FRAME_SIZE_ERROR连接错误

- PUSH_PROMISE帧

    服务端向客户端推送的帧，客户端可以返回RST_STREAM拒绝

- PING帧

    心跳监测，测量发送往还时间，确定连接是否正常，ACK=0表示PING帧响应，1表示PING帧

- GOAWAY帧

    用于关闭连接或发出错误，允许停止接受新的流并完成前面的流处理

    包括R+Last Stream Id + Error Code + Addictional Debug Data

- WINDOW_UPDATE帧

    用于连接和流的流量控制

- CONTINUATION帧

    一种持续帧，用于继续传输Header头块片段。通常Header块比较大，在HEADERS、PUSH_PROMISE、CONTINUATION帧之后继续传输

### 问题

http2只解决了应用层的问题，但是由于其还是基于tcp的，因此tcp硬伤的队头阻塞依旧存在

tcp由于“可靠传输”而包含的按序到达的机制，使得http2的队头阻塞问题仍然存在

## 参考链接

[HTTP 1.x 学习笔记 —— Web 性能权威指南](https://www.cnblogs.com/huansky/p/14420723.html)

[关于队头阻塞（Head-of-Line blocking），看这一篇就足够了](https://zhuanlan.zhihu.com/p/330300133)

[HTTP1.1 对头阻塞和 HTTP2 中对其的解决措施](https://blog.csdn.net/weixin_63951768/article/details/144917352)