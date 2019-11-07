---
title:  "go gRPC基础"
date:   2019-11-06 19:31:00 +0800
categories: gRPC
header:
  overlay_image: /assets/images/banners/grpc-banner.svg
  teaser: /assets/images/banners/grpc-banner.svg
comments: true
---

gRPC文档学习笔记

## gRPC Basics - Go

学习目标：
- 定义服务
- 编译生成server和client代码
- 使用Go gRPC API编写简单的客户端和服务端来使用service

### 为何使用gRPC

本篇文档使用的例子是一个简单的路由应用，客户端可以通过该服务获取路由信息，总结路由信息，交换路由信息。
一次编写proto文件就可以生成服务端和客户端代码，而且凡是支持gRPC的语言都可以互相通信。

### Example

```shell
cd $GOPATH/src/google.golang.org/grpc/examples/route_guide
```

#### 定义服务

使用`service`声明一个服务：
```proto
service RouteGuide {
  ...
}
```

然后再service的方法体中定义`rpc` methods，声明请求类型和返回体类型，gRPC允许定义四种类型的服务方法，在该例子中都出现了。

- *A simple RPC* 客户端发送请求，等待服务端返回响应，看似一个普通函数的调用。
```proto
rpc GetFeature(Point) returns (Feature) {}
```

- *A server-side streaming RPC* 服务端流式RPC，客户端发送请求，服务器返回一个响应流，读取消息序列，直到消息读取完毕。
```proto
rpc ListFeatures(Rectangle) returns (stream Feature) {}
```

- *A client-side streaming RPC* 客户端流式RPC，客户端发送消息序列。客户端完成信息写入后，会等待服务端处理这个消息序列。
```proto
rpc RecordRoute(stream Point) returns (RouteSummary) {}
```

- *A bidirectional streaming RPC* 双向流式RPC，双方通信都是发送消息序列。服务端可以等待客户端所有消息发送完毕在处理返回，也可以收到消息就处理返回。
```proto
rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
```

#### 定义类型

```proto
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}
```

#### 编译

```shell
protoc -I routeguide/ routeguide/route_guide.proto --go_out=plugins=grpc:routeguide
```

#### 服务端

- 实现服务接口。
- 启动gRPC服务，监听客户端请求。

#### 实现RouteGuide

服务端routeGuideServer结构体实现了生成的RouterGuideServer接口。

```go
type routeGuideServer struct {
  // ...
}

func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
  // ...
}

func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
  // ...
}

func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
  // ...
}

func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
  // ...
}
```

##### Simple RPC

```go
func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
  for _. feature := range s.saveFeatures {
    if proto.Equal(feature.Location, point) {
      return feature, nil
    }
  }

  //  No feature was found, return an unnamed feature
  return &pb.Feature{"", point}, nil
}
```
该方法接收context对象和客户端point请求作为参数，返回Feature对象和错误信息作为响应。在方法体中填充Feature信息，返回给客户端。

##### Server-side streaming RPC

```go
func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
  for _, feature := range s.saveFeatures {
    if inRange(feature.Location, rect) {
      if err := stream.Send(feature); err != nil {
        return err
      }
    }
  }

  return nil
}
```
ListFeature方法是服务端流式RPC方法，所以需要向客户端返回多个信息。
与简单RPC相比，服务端流式RPC接收的参数是请求对象和`RouteGuide_ListFeaturesServer`对象，后者用于写入responses。
在这个方法中，发送了多个Feature对象，最后如果没有错误发生，则返回一个nil error。

##### 客户端流式RPC

```go
func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteSrever) error {
  var pointCount, featureCount, distance int32
  var lastPoint *pb.Point
  startTime := time.Now()
  for {
    point, err := stream.Recv()
    if err == io.EOF {
      endTime := time.Now()
      return stream.SendAndClose(&pb.RouteSummary{
        PointCount: pointCount,
        FeatureCount: featureCount,
        Distance: distance,
        ElapsedTime: int32(endTime.Sub(startTime).Seconds()),
      })
    }
    if err != nil {
      return err
    }
    pointCount++
    for _, feature := range s.savedFeatures {
      if proto.Equal(feature.Location, point) {
        featureCount++
      }
    }
    if lastPoint != nil {
      distance += clacDistance(lastPoint, point)
    }
    lastPoint = point
  }
}
```
客户端流式RPC相对复杂一点，服务端获取一连串由客户端发送来的`Point`，处理后返回一个`RouteSummary`信息表示这些点描绘的路径。
这个方法不需要接收请求参数，而是直接获取一个`RouteGuide_RecordRouteServer`流对象，服务端可以使用这个流对象读写信息，使用
`Recv()`方法可以接收客户端发送的多个消息，使用`SendAndClose()`方法可以返回一个单一的响应，并且关闭与客户端的连接。
服务端使用`Recv()`方法循环读取客户端发送的请求对象，直到客户端不再发送数据，服务端在读取每一个请求对象时都需要判断该方法返回的
error信息，如果没有发生错误，则表示`stream`对象仍然是好的，如果错误信息是`io.EOF`，则表示客户端停止发送数据，服务端可以处理
收集的这些数据，返回一个`RouteSummary`对象。对于其它形式的error，只需要直接返回错误信息即可，这些错误信息会被转换为对应的RPC
状态。

##### Bidirectional streaming RPC

```go
func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
  for {
    in, err := stream.Recv()
    if err == io.EOF {
      return nil
    }
    if err != nil {
      return err
    }
    key := serialize(in.Location)
    // ... look for notes to be sent to client
    for _, note := range s.routeNotes[key] {
      if err := stream.Send(note); err != nil {
        return err
      }
    }
  }
}
```
`stream`同上一个方法一样，可以被用来读写信息，不同的是客户端在发送数据流的同时，服务端也可以返回数据流，服务端使用
`Send()`方法来发送流数据，客户端和服务端都可以按顺序获取对方发送的数据。流对双方发送的信息分开处理。

#### 启动服务

当所有的RPC接口都被实现之后，开始启动一个gRPC服务，用于监听客户端连接，处理客户端发送来的RPC调用请求。
```go
flag.Parse()
lis, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))
if err != nil {
  log.Fatalf("failed to listen: %v", err)
}
grpcServer := grpc.NewServer()
pb.RegisterRouteGuideServer(grpcServer, &routeGuideServer{})
// ... determine whether to use TLS
grpcServer.Serve(lis)
```
- 启动tpc监听，指定监听端口。
- 创建gRPC服务端实例。
- 注册服务实现。
- 调用`Serve()`方法，启动RPC服务监听，杀死进程或调用`Stop()`方法来停止服务。

#### 创建客户端

##### Creating a stub

调用RPC服务之前需要先使用`grpc.Dial()`方法和服务端建立RPC连接。
```go
conn, err := grpc.Dial(*serverAddr)
if err != nil {
  // ...
}
defer conn.Close()
```
可以使用`DialOptions`设置认证信息。
建立连接之后，需要一个客户端去调用RPC，通过调用protoc编译后生成的`NewRouteGuideClient`方法来获取一个client。
```go
client := pb.NewRouteGuideClient(conn)
```

##### Calling service methods

现在来看看如何调用service提供的RPC方法，gRPC-Go提供的RPC方法都是阻塞同步方式的，客户端必须等待服务端返回响应才做下一步处理。

1. 简单RPC调用

简单到如同调用一个本地方法。
```go
feature, err := client.GetFeature(context.Background(), &pb.Point{409146138, -746188906})
if err != nil {
  // ...
}
```
调用参数包含一个`context.Context`对象，服务端可以根据这一参数做更多事情。

1. 服务端流式RPC

```go
rect := &pb.Rectangle{
  // ...
} // initialize a pb.Rectangle
stream, err := client.ListFeatures(context.Backgound(), rect)
if err != nil {
  // ...
}
for {
  feature, err := stream.Recv()
  if err == io.EOF {
    break
  }
  if err != nil {
    log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
  }
  log.Println(feature)
}
```

调用服务端流式RPC，返回的不是一个消息对象，而是一个流，通过这个流可以读取到服务端发送的一系列消息对象。

1. 客户端流式RPC

```go
// Create a random number of random points
r := rand.New(rand.NewSource(time.Now().UnixNano()))
pointCount := int(r.Int31n(100)) + 2 // Traverse at least two points
var points []*pb.Point
for i := 0; i < pointCount; i++ {
  points = append(points, randomPoint(r))
}
log.Printf("Traversing %d points.", len(points))
stream, err := client.RecordRoute(context.Background())
if err != nil {
  log.Fatalf("%v.RecordRoute(_) = _, %v", client, err)
}
for _, point := range points {
  if err := stream.Send(point); err != nil {
    if err == io.EOF {
      break
    }
    log.Fatalf("%v.Send(%v) = %v", stream, point, err)
  }
}
reply, err := stream.CloseAndRecv()
if err != nil {
  log.Fatalf("%v.CloseAndRecv() got error %v, want %v", stream, err, nil)
}
log.Printf("Route summary: %v", reply)
```

与服务端流式发送想对应的，客户端流式调用RPC最后在后的服务端响应时，需要调用`CloseAndRecv()`方法。

1. 双向流式RPC

```go
stream, err := client.RouteChat(context.Background())
waitc := make(chan struct{})
go func() {
  for {
    in, err := stream.Recv()
    if err == io.EOF {
      // read done.
      close(waitc)
      return
    }
    if err != nil {
      log.Fatalf("Failed to receive a note: %v" ,err)
    }
    log.Printf("Got message %s at point(%d, %d)", in.Message, in.Location.Latitude, in.Location.Longtitude)
  }
}()
for _, note := range notes {
  if err := stream.Send(note); err != nil {
    log.Fatalf("Failed to send to note: %v" ,err)
  }
}
stream.CloseSend()
<-waitc
```

#### Try it out

```shell
$ go run server/server.go
$ go run client/client.go
```
