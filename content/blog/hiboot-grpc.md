---
title: "构建Go语言版本的gRPC高性能接口"
date: 2018-10-31T12:29:40+06:00
type: post
image: images/blog/hiboot-grpc/header.png
author: 邓冰寒
tags: ["Go", "Golang", "grpc", "hiboot"]
---

## 背景

作为一个网络应用开发者，我们知道接口的重要性，接口是网络应用的基石。接口关系到应用之间如何交互。而网络通讯的性能是我们非常关心的，那么接下来我将谈谈如何构建Go语言版本的gRPC高性能接口。

## 什么是[gRPC](https://grpc.io)？

RPC是英语Remote Procedure Call的缩写，意思是远程程序调用。gRPC是一个跨语言的RPC框架，几乎原生支持所有主流语言。在gRPC中，客户端应用程序可以直接在不同机器上的服务器应用程序上调用方法，就像它是本地对象一样，使您更容易创建分布式应用程序和服务。 与许多RPC系统一样，gRPC基于定义服务的思想，指定可以使用其参数和返回类型远程调用的方法。 在服务器端，服务器实现此接口并运行gRPC服务器来处理客户端调用。 在客户端，客户端有一个stub（简称为一些语言的客户端），提供与服务器相同的方法。gRPC客户端和服务器可以在各种环境中运行和交互，从Google内部的服务器到您自己的应用，并且可以使用任何gRPC支持的语言编写。 所以，例如，您可以轻松地创建一个Java开发的服务，使用Go，Python或Ruby中的客户端。

gRPC使用protocol buffers作为其IDL(接口描述）和其底层消息交换格式。

![grpc](/images/blog/hiboot-grpc/grpc-intro.svg)

## 什么是 [Protocol Buffer](https://developers.google.com/protocol-buffers/)

Protocol Buffer( 简称 Protobuf) 是 Google 公司内部的混合语言数据标准，目前已经正在使用的有超过 48,162 种报文格式定义和超过 12,183 个 .proto 文件。他们用于 RPC 系统和持续数据存储系统。

Protocol Buffers 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

## 演示代码详解

我们将通过一个详细的例子来说明如何构建一个gRPC服务端和客户端，其中使用到了[Hiboot](https://github.com/hidevopsio/hiboot)的[grpc starter](https://github.com/hidevopsio/hiboot/tree/master/pkg/starter/grpc)。

我们接下来要安装相关编译工具

## 安装编译工具

```bash
go get google.golang.org/grpc
go get -u -v github.com/golang/protobuf/{proto,protoc-gen-go}
```

## 定义Protocol Buffer

下面我们定义了两个gRPC的服务，`HelloService`和`HolaService`

关于Protocol Buffer更详细资料请参考[这个链接](https://github.com/protocolbuffers/protobuf)

```protobuf

syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.grpc.examples.helloworld";
option java_outer_classname = "HelloWorldProto";

// target package name
package protobuf;

// The greeting service definition.
service HelloService {
  // Sends a hello greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

service HolaService {
  // Sends a hola greeting
  rpc SayHola (HolaRequest) returns (HolaReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}

// The request message containing the user's name.
message HolaRequest {
  string name = 1;
}

// The response message containing the greetings
message HolaReply {
  string message = 1;
}

```

## 服务端代码

### 配置文件 config/application.yml

为什么要配置文件？虽然这是演示代码，但是我们还是安装标准流程来写，也就是我们以能上生产环境的标准来做的。

```yaml
app:
  project: hidevopsio
  name: grpc-server
  profiles:
    include:
    - logging
    - locale
    - grpc

logging:
  level: info
```

### 配置文件 config/application-grpc.yml

```yaml
grpc:
  server:
    enabled: true
    network: tcp
    port: 7575

```

### 服务端Go源代码

为了简单起见，我们将代码写到一个main.go文件里面。

因为使用了[Hiboot](https://github.com/hidevopsio/hiboot)的[grpc starter](https://github.com/hidevopsio/hiboot/tree/master/pkg/starter/grpc)，代码非常简洁。除了业务逻辑，几乎没有多余的代码。

运行以下命令文件生成Go的接口代码

```bash
go:generate protoc -I ../protobuf --go_out=plugins=grpc:../protobuf ../protobuf/helloworld.proto
```

⚠️ 生成的代码是`helloworld.pb.go`，服务端和客户端是共用的。

```go
// if protoc report command not found error, should install proto and protc-gen-go
// find protoc install instruction on http://google.github.io/proto-lens/installing-protoc.html
// go get -u -v github.com/golang/protobuf/{proto,protoc-gen-go}
//go:generate protoc -I ../protobuf --go_out=plugins=grpc:../protobuf ../protobuf/helloworld.proto

package main

import (
	"golang.org/x/net/context"
	"hidevops.io/hiboot/examples/grpc/helloworld/protobuf"
	"hidevops.io/hiboot/pkg/app/web"
	_ "hidevops.io/hiboot/pkg/starter/actuator"
	"hidevops.io/hiboot/pkg/starter/grpc"
)

// server is used to implement protobuf.GreeterServer.
type helloServiceServerImpl struct {
}

func newHelloServiceServer() protobuf.HelloServiceServer {
	return &helloServiceServerImpl{}
}

// SayHello implements helloworld.GreeterServer
func (s *helloServiceServerImpl) SayHello(ctx context.Context, request *protobuf.HelloRequest) (*protobuf.HelloReply, error) {
	// response to client
	return &protobuf.HelloReply{Message: "Hello " + request.Name}, nil
}

func init() {
 	// must: register grpc server
	// please note that holaServiceServerImpl must implement protobuf.HelloServiceServer, or it won't be registered.
	grpc.Server(protobuf.RegisterHelloServiceServer, newHelloServiceServer)
}

func main() {
	// create new web application and run it
	web.NewApplication().Run()
}

```

## 客户端代码

### 客户端配置文件 config/application.yml

```yaml

app:
  project: hidevopsio
  name: grpc-client
  profiles:
    include:
    - logging
    - locale
    - grpc

logging:
  level: info

```

### 客户端配置文件 config/application-grpc.yml

```yaml
grpc:
  client:
    hello-world-service:
      host: localhost
      port: 7575
```

### 客户端Go源代码

我们`grpc.Client("hello-world-service",
		protobuf.NewHelloServiceClient)`注册gRPC客户端，这样即可以在控制器注入客户端，直接调用服务端的接口了。

```go
package controller

import (
	"golang.org/x/net/context"
	"hidevops.io/hiboot/examples/grpc/helloworld/protobuf"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/app/web"
	"hidevops.io/hiboot/pkg/starter/grpc"
)

// controller
type helloController struct {
	// embedded web.Controller
	web.Controller
	// declare HelloServiceClient
	helloServiceClient protobuf.HelloServiceClient
}

// Init inject helloServiceClient
func newHelloController(helloServiceClient protobuf.HelloServiceClient) *helloController {
	return &helloController{
		helloServiceClient: helloServiceClient,
	}
}

// GET /greeter/name/{name}
func (c *helloController) GetByName(name string) (response string) {

	// call grpc server method
	// pass context.Background() for the sake of simplicity
	result, err := c.helloServiceClient.SayHello(context.Background(), &protobuf.HelloRequest{Name: name})

	// got response
	if err == nil {
		response = result.Message
	}
	return
}

func init() {

	// must: register grpc client, the name greeter-client should configured in application.yml
	// see config/application-grpc.yml
	//
	// grpc:
	//   client:
	// 	   hello-world-service:   # client name
	//       host: localhost # server host
	//       port: 7575      # server port
	//
	grpc.Client("hello-world-service",
		protobuf.NewHelloServiceClient)

	// must: register Rest Controller
	app.Register(newHelloController)
}

```

`main.go`

```go
// if protoc report command not found error, should install proto and protc-gen-go
// find protoc install instruction on http://google.github.io/proto-lens/installing-protoc.html
// go get -u -v github.com/golang/protobuf/{proto,protoc-gen-go}
//go:generate protoc -I ../protobuf --go_out=plugins=grpc:../protobuf ../protobuf/helloworld.proto

package main

import (
	_ "hidevops.io/hiboot/examples/grpc/helloworld/greeter-client/controller"
	"hidevops.io/hiboot/pkg/app/web"
	_ "hidevops.io/hiboot/pkg/starter/actuator"
	_ "hidevops.io/hiboot/pkg/starter/logging"
)

func main() {
	// create new web application and run it
	web.NewApplication().Run()
}
```

## 运行服务端代码

```bash
cd $GOPATH/src/hidevops.io/hiboot/examples/grpc/helloworld/greeter-server
go run main.go --logging.level=info
```

服务端代码运行结果：

```bash
______  ____________             _____
___  / / /__(_)__  /_______________  /_
__  /_/ /__  /__  __ \  __ \  __ \  __/
_  __  / _  / _  /_/ / /_/ / /_/ / /_     Hiboot Application Framework
/_/ /_/  /_/  /_.___/\____/\____/\__/     https://hidevops.io/hiboot

[INFO] 2018/11/06 19:50 Starting Hiboot web application grpc-server on localhost with PID 37344
[INFO] 2018/11/06 19:50 Working directory: /Users/johnd/.gvm/pkgsets/go1.10/hidevops/src/hidevops.io/hiboot/examples/grpc/helloworld/greeter-server
[INFO] 2018/11/06 19:50 The following profiles are active: local, [logging locale grpc web]
[INFO] 2018/11/06 19:50 Initializing Hiboot Application
[INFO] 2018/11/06 19:50 Auto configure web starter
[INFO] 2018/11/06 19:50 Auto configure grpc starter
[INFO] 2018/11/06 19:50 Resolving dependencies
[INFO] 2018/11/06 19:50 Injecting dependencies
[INFO] 2018/11/06 19:50 Registered health.server on gRPC server
[INFO] 2018/11/06 19:50 Registered protobuf.helloServiceServer on gRPC server
[INFO] 2018/11/06 19:50 Registered protobuf.holaServiceServer on gRPC server
[INFO] 2018/11/06 19:50 gRPC server listening on: localhost:7575
[INFO] 2018/11/06 19:50 Injected dependencies
[INFO] 2018/11/06 19:50 Mapped "/health" onto actuator.healthController.Get()
[INFO] 2018/11/06 19:50 Hiboot started on port(s) http://localhost:8080
[INFO] 2018/11/06 19:50 Started grpc-server in 0.005122 seconds
```

```bash
cd $GOPATH/src/hidevops.io/hiboot/examples/grpc/helloworld/greeter-client
go run main.go --logging.level=info
```

客户端代码运行结果：

```bash

______  ____________             _____
___  / / /__(_)__  /_______________  /_
__  /_/ /__  /__  __ \  __ \  __ \  __/
_  __  / _  / _  /_/ / /_/ / /_/ / /_     Hiboot Application Framework
/_/ /_/  /_/  /_.___/\____/\____/\__/     https://hidevops.io/hiboot

[INFO] 2018/11/06 19:51 Starting Hiboot web application grpc-client on localhost with PID 37373
[INFO] 2018/11/06 19:51 Working directory: /Users/johnd/.gvm/pkgsets/go1.10/hidevops/src/hidevops.io/hiboot/examples/grpc/helloworld/greeter-client
[INFO] 2018/11/06 19:51 The following profiles are active: local, [logging locale grpc web]
[INFO] 2018/11/06 19:51 Initializing Hiboot Application
[INFO] 2018/11/06 19:51 Auto configure web starter
[INFO] 2018/11/06 19:51 Auto configure grpc starter
[INFO] 2018/11/06 19:51 Auto configure logging starter
[INFO] 2018/11/06 19:51 Resolving dependencies
[INFO] 2018/11/06 19:51 Injecting dependencies
[INFO] 2018/11/06 19:51 gRPC client connected to: localhost:7575
[INFO] 2018/11/06 19:51 Registered gRPC client helloServiceClient
[INFO] 2018/11/06 19:51 gRPC client connected to: localhost:7575
[INFO] 2018/11/06 19:51 Registered gRPC client healthClient
[INFO] 2018/11/06 19:51 gRPC client connected to: localhost:7575
[INFO] 2018/11/06 19:51 Registered gRPC client holaServiceClient
[INFO] 2018/11/06 19:51 Injected dependencies
[INFO] 2018/11/06 19:51 Mapped "/health" onto actuator.healthController.Get()
[INFO] 2018/11/06 19:51 Mapped "/hello/name/{name}" onto controller.helloController.GetByName()
[INFO] 2018/11/06 19:51 Mapped "/hola/name/{name}" onto controller.holaController.GetByName()
[INFO] 2018/11/06 19:51 Hiboot started on port(s) http://localhost:8081
[INFO] 2018/11/06 19:51 Started grpc-client in 0.008635 seconds
```

## 检查服务健康状态

[Hiboot](https://github.com/hidevopsio/hiboot)的[grpc starter](https://github.com/hidevopsio/hiboot/tree/master/pkg/starter/grpc)提供了健康检测功能，我们来验证一下是否正常。

```bash
http localhost:8081/health
```

测试结果如下，grpc的状态为“Up”表示gRPC客户端可以连接到服务端。

```bash
HTTP/1.1 200 OK
Content-Length: 38
Content-Type: application/json; charset=UTF-8
Date: Tue, 06 Nov 2018 11:53:40 GMT

{
    "grpc": {
        "status": "Up"
    },
    "status": "Up"
}

```

最后，我们来验证gRPC的接口

```bash
http ':8081/hello/name/Hiboot gRPC Application'
```

输出了我们预期的结果 -- 一行字符串`Hello Hiboot gRPC Application`

```bash
HTTP/1.1 200 OK
Content-Length: 29
Content-Type: text/plain; charset=utf-8
Date: Tue, 06 Nov 2018 11:59:46 GMT

Hello Hiboot gRPC Application
```

**以上源码可以在[这里](https://github.com/hidevopsio/hiboot/tree/master/examples/grpc/helloworld)找到**

识别二维码加入公众号，获取更多文章。

![grpc](/images/pa-qrcode.jpg)