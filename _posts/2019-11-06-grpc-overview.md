---
title:  "什么是gRPC"
date:   2019-11-06 15:59:00 +0800
categories: gRPC
header:
  overlay_image: /assets/images/banners/grpc-banner.svg
  teaser: /assets/images/banners/grpc-banner.svg
comments: true
---

gRPC官网文档学习笔记。主要是翻译。

## 概览

gRPC客户端可以直接调用另一台机器上运行的gRPC服务提供的方法，就如同调用本地方法一样，让构建分布式应用和服务变得更简单。
gRPC的设计理念和众多RPC系统一样，定义服务，提供方法，远程调用时只需要提供方法名，参数和返回值类型。服务端需要实现这些接口，
并启动一个gRPC服务，处理客户端的请求。客户端有一个`stub`，作为服务端定义的方法存根，供客户端调用。

![gRPC-overview](/assets/images/gRPC/gRPC-overview.svg)

gRPC客户端和服务器可以在很多环境下互相通信。例如上图，可以用C++实现客户端，用Java、Ruby、Go或Python等语言实现客户端。

## 使用 Protocol Buffers

gRPC默认使用[protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)，谷歌开源的一种成熟的序列化数据结构的语法机制。
当然gRPC也支持其他数据格式，例如JSON。

使用protocol buffers首先需要在*proto*文件中定义一个数据结构，这种结构被称为*message*，每个message中包含了很多键值对，它们被称作*fields*。
```proto
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```
然后使用你擅长的语言的protocol buffer编译器--`protoc`，去生成数据访问类，提供对各field的存储方法，以及序列化整个数据结构为字节流、解析字节流为数据结构的方法。
如果使用的是C++版的protocol buffers编译器，处理上述例子将会生成一个`Person`类。

再看一个完整的例子：
```proto
// 定义greeter服务
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// 请求结构包含了用户的name
message HelloRequest {
  string name = 1;
}

// 返回消息包含了问候语
message HelloReply {
  string message = 1;
}
```
使用protoc可以根据proto文件生成客户端服务端代码，以及一些用于操作数据的`populating`, `serializing` 和 `retrieving`简单代码。
之后的学习将更细致的分析这些代码。
