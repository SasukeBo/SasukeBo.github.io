---
title:  "go语言快速使用gRPC"
date:   2019-11-06 17:00:00 +0800
categories: gRPC
header:
  overlay_image: /assets/images/banners/grpc-banner.svg
  teaser: /assets/images/banners/grpc-banner.svg
comments: true
---

gRPC官网文档学习笔记

## 快速开始

### 先决条件

gRPC需要1.6+版本的go。

### 安装gRPC

```shell
$ go get -u google.golang.org/grpc
```

### 安装Protocol Buffers v3

安装编译器`protoc`，用来生成gRPC服务代码。前往[https://github.com/google/protobuf/releases](https://github.com/google/protobuf/releases)下载适合你的电脑预编译二进制文件，文件名一般为`protoc-<version>-<platform>.zip`。
- 解压文件
- 将protoc二进制文件路径加入到**PATH**环境变量

然后安装go版本的protoc插件。
```shell
$ go get -u github.com/golang/protobuf/protoc-gen-go
```
该插件的二进制路径也需要加入到**PATH**环境变量。
```shell
$ export PATH=$PATH:$GOPATH/bin
```

### Examples

前面安装的`google.golang.org/grpc`包中就包含了许多例子。
尝试编译这个例子。

```shell
$ cd $GOPATH/src/google.golang.org/grpc/examples/helloworld/helloworld
```
gRPC服务定义文件一般为`.proto`文件，protoc 处理`.proto`文件会生成相应的`.pb.go`文件。
当前示例文件夹中已经通过编译`helloworld.proto`文件生成了`helloworld.pb.go`文件，该文件主要包含：
- 生成的client和server代码
- populating/serializing/retrieving `HelloRequest`和`HelloReply`的代码。

#### 运行

- 服务端
```shell
$ cd $GOPATH/src/google.golang.org/grpc/examples/helloworld
$ go run greeter_server/main.go
```
- 客户端
```shell
$ go run greeter_client/main.go
# => Greeting: Hello world
```

#### 尝试修改

学习更多定义gRPC服务可以访问[gRPC Basics: GO](https://grpc.io/docs/tutorials/basic/go/)。
现在只需要知道在刚刚的例子中，server和client的`stub`都有一个**SayHello** RPC方法，服务端接收client发送来的**HelloRequest**参数，
并返回**HelloReply** response给client。
```proto
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

现在给**Greeter**服务增加一个方法，如下：

```proto
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

...
```

#### 编译

```shell
$ protoc -I helloworld/ helloworld/helloword.proto -go_out=plugins=grpc:helloworld
```

#### 更新服务端代码

```golang
// greeter_server/main.go
func (s *server) SayHelloAgain(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
  return &pb.HelloReply{Message: "Hello again " + in.GetName()}, nil
}
```

#### 更新客户端代码

```golang
// greeter_client/main.go
r, err = c.SayHelloAgain(ctx, &pb.HelloRequest{Name: name})
if err != nil {
  log.Fatalf("could not greet: %v", err)
}
log.Printf("Greeting: %s", r.GetMessage())
```

#### 运行

重新启动服务端，运行客户端

## 下一步

学习更多go - gRPC：[gRPC Basics: Go](https://grpc.io/docs/tutorials/basic/go/)
