---
title: "Hiboot 属性配置和使用"
date: 2018-11-21T12:29:40+06:00
type: post
image: images/blog/hiboot-properties/header.png
author: 邓冰寒
tags: ["Go", "Golang", "hiboot", "properties"]
---

作为一个网络应用，需要部署到不同都环境下，那配置文件就显得非常重要了，Hiboot允许通过外部配置让你在不同的环境使用同一应用程序的代码，简单说就是可以通过配置文件来注入属性或者修改默认的配置。Hiboot支持多种外部配置方式，并且按照自下而上的优先级配置，即默认属性优先级最低，命令行参数优先级最高，

1. 命令行属性
2. 应用 SetProperty 方法
3. 操作系统环境变量
4. 外部默认配置文件 `config/application.yml`
5. 外部不同环境下的配置文件 `config/application-${app.profiles.active}.yml`
6. 默认属性

## 命令行属性

默认情况下，Hiboot应用会将命令行任何参数（以`--`开头），如 `--xxx=xxx`, 可以使用`自定义参数`，也可以使用Hiboot的`默认参数`。

更多常见的应用属性请浏览[这里](https://hiboot.hidevops.io/cn/web-app/)

比如，我们以Hiboot例子[helloworld为例](https://github.com/hidevopsio/hiboot/tree/master/examples/web/helloworld)， 通过传递Hiboot默认参数server.port配置不同端口为8081

```bash
helloworld --server.port=8081

______  ____________             _____
___  / / /__(_)__  /_______________  /_
__  /_/ /__  /__  __ \  __ \  __ \  __/
_  __  / _  / _  /_/ / /_/ / /_/ / /_     Hiboot Application Framework
/_/ /_/  /_/  /_.___/\____/\____/\__/     https://hidevops.io/hiboot

[INFO] 2018/11/23 18:06 Starting Hiboot web application hiboot on localhost with PID 68499
[INFO] 2018/11/23 18:06 Working directory: /Users/johnd/.gvm/pkgsets/go1.10/hidevops/src/hidevops.io/hiboot
[INFO] 2018/11/23 18:06 The following profiles are active: local, [actuator web]
[INFO] 2018/11/23 18:06 Initializing Hiboot Application
[INFO] 2018/11/23 18:06 Auto configure web starter on web.configuration
[INFO] 2018/11/23 18:06 Auto configure actuator starter on actuator.configuration
[INFO] 2018/11/23 18:06 Mapped "/" onto main.Controller.Get()
[INFO] 2018/11/23 18:06 Mapped "/health" onto actuator.healthController.Get()
[INFO] 2018/11/23 18:06 Hiboot started on port(s) http://localhost:8081
[INFO] 2018/11/23 18:06 Started hiboot in 0.003900 seconds

```

>通过设置方法`SetAddCommandLineProperties(false)`可以关闭命令行属性的设置功能。

```go

// main function
func main() {
	// create new web application and run it
	web.NewApplication(new(Controller)).
		SetAddCommandLineProperties(false).
		Run()
}
```

## 应用 SetProperty 方法

Hiboot也可以通过`SetProperty`方法来配置属性，在`main`函数下面，创建应用示例之后, 可以通过链式调用来设置一个或多个属性。

```go
// main function
func main() {
	// create new web application and run it
	web.NewApplication(new(Controller)).
		SetProperty("server.port", "8082").
		SetProperty("logging.level", "info").
		Run()
}
```

这个时候编译运行运用，应用监听端口为`8082`，日志级别设为`info`。不加任何参数时运行结果如下：

```bash
helloworld

______  ____________             _____
___  / / /__(_)__  /_______________  /_
__  /_/ /__  /__  __ \  __ \  __ \  __/
_  __  / _  / _  /_/ / /_/ / /_/ / /_     Hiboot Application Framework
/_/ /_/  /_/  /_.___/\____/\____/\__/     https://hidevops.io/hiboot

[INFO] 2018/11/23 18:21 Starting Hiboot web application hiboot-app on localhost with PID 68690
[INFO] 2018/11/23 18:21 Working directory: /Users/johnd
[INFO] 2018/11/23 18:21 The following profiles are active: local, [actuator web]
[INFO] 2018/11/23 18:21 Initializing Hiboot Application
[INFO] 2018/11/23 18:21 Auto configure web starter on web.configuration
[INFO] 2018/11/23 18:21 Auto configure actuator starter on actuator.configuration
[INFO] 2018/11/23 18:21 Mapped "/" onto main.Controller.Get()
[INFO] 2018/11/23 18:21 Mapped "/health" onto actuator.healthController.Get()
[INFO] 2018/11/23 18:21 Hiboot started on port(s) http://localhost:8082
[INFO] 2018/11/23 18:21 Started hiboot-app in 0.003534 seconds

```

## 操作系统环境变量

Hiboot应用配置文件支持环境变量配合默认值使用，如监听端口`server.port`设置为`${MY_APP_SERVER_PORT:8080}`, 其中`MY_APP_SERVER_PORT`为环境变量，默认值为`8080`，Hiboot应用会先读取环境变量值，如果为空则取默认值`8080`。

Hiboot应用注入操作系统环境变量有以下几个途径

* 外部配置文件注入
* 标签( `value` 或者 `default`)直接注入

```bash
export MY_APP_SERVER_PORT=8081
```

### 外部配置文件注入

当外部配置文件引用了操作系统的环境变量`MY_APP_SERVER_PORT`，Hiboot应用会获取环境变量`MY_APP_SERVER_PORT`的值，如果没有获取到则使用默认值`8080`, 也就是属性`server.port`的值会注入为`8080`.

```yaml
app:
  project: hidevopsio
  name: hiboot

server:
  port: ${MY_APP_SERVER_PORT:8080}
```

### 标签( `value` 或者 `default`)直接注入

假设你需要注入操作系统的环境变量`MY_APP_SERVER_PORT`到`MyService`的成员变量`ServerPort`，Hiboot应用会通过标签`value`来注入, Hiboot应用先获取环境变量`MY_APP_SERVER_PORT`的值，如获取到的值为`8081`则结构体`MyService`的字段`ServerPort`会被注入为`8081`, 否则会被注入默认值为`8080`。

注意：依赖注入的前提是需要注册构造函数, 想了解更详细的使用方法请参考[Hiboot文档](https://hiboot.hidevops.io/cn/inversion-of-control/)

```go
// HelloService 你的业务结构体
type HelloService struct {
	ServerPort string `value:"${MY_APP_SERVER_PORT:8080}"`
}
```

## 外部默认配置文件

Hiboot应用配置采用YAML格式的配置文件，默认为工作目录下的 `config/application.yml`, 我们以`etcd.endpoints`举例来说明，

### config/application.yml

```yaml
etcd:
  endpoints: 192.168.0.100:2379
```

## 外部不同环境下的配置文件

`etcd.endpoints`可以配置在默认的配置文件`config/application.yml`中，也可以配置在以`etcd`为后缀的配置文件`config/application-etcd.yml`中，当配置属性较多时，分开配置可以更好的管理配置属性。

### config/application-etcd.yml

```yaml
etcd:
  endpoints: 192.168.1.100:2379
```

### config/application-${app.profiles.active}.yml

这里的`${app.profiles.active}`是一个变量如果你用于本地开发，可以定义为`config/application-local.yml`，而在生产环境，当属性`app.profiles.active=prod`，Hiboot应用会自动切换配置文件为`config/application-prod.yml`。

```yaml
etcd:
  endpoints: 192.168.2.100:2379
```

## 默认属性

在[etcd Hiboot starter](https://github.com/hidevopsio/hiboot-data/tree/master/starter/etcd),  我们定义了`etcd`的`properties`，Hiboot会注入默认属性值`127.0.0.1:2379`, 默认属性值的优先级最低。

```go
type properties struct {
	// ...
	Endpoints      []string `json:"endpoints" default:“127.0.0.1:2379”`
	// ...
}

type etcdConfiguration struct {
	app.Configuration
	// 这里设置Go的 tag mapstructure 为 etcd，Hiboot应用会注入外部配置值到properties。
	// 当外部配置文件没找到时，则使用tag default定义的默认值 127.0.0.1:2379
	Properties properties `mapstructure:"etcd"`
}

```

识别二维码加入公众号，获取更多文章。

![hidevops](/images/pa-qrcode.jpg)