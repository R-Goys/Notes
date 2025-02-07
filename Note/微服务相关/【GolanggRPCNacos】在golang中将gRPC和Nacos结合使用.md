## Nacos与gRPC

### 前言

关于这部分，前段时间我在看文档以及视频教程的时候，怎么都想不明白，到底为什么要用gRPC是什么，他在项目中应该充当什么样的角色？Nacos又是如何和他结合的？

于是我就决定去看看一些小项目是如何实现的这个功能，现在将我最近学到的分享给大家。

### 正文

在正文开始之前，我们要先知道Nacos和gRPC在本篇内容中，会涉及到的作用：

#### gRPC

- gRPC 允许服务之间无缝通信，像调用本地函数一样调用远程服务的功能。

#### Nacos

- 服务在启动时将自身注册到 Nacos，包含 IP、端口和服务名称，其他服务可以通过服务名称查询可用实例。

#### 正片

在这里，我们以一个实现了用户注册/登陆的接口为例子，进行讲解。

> micro              		 # 项目根目录
> └── app            		 # 应用逻辑目录
>     ├── api        		 # 定义API接口
>     │   └── user.go		 # 用户相关的API实现
>     ├── client     		 # 客户端功能实现
>     │   ├── nacos  		 # Nacos配置相关
>     │   │   └── nacos.go  	  # Nacos客户端实现
>     │   └── user   		 # 用户客户端功能
>     │       └── proto       	# 用户相关的proto定义
>     │           └── userclient.go    # 用户客户端的实现
>     ├── cmd        		 # 应用入口
>     │   └── main.go 		# 启动程序的主文件
>     ├── Middleware   		# 中间件处理
>     │   └── Token.go 		# 处理Token的中间件
>     ├── model        		# 数据模型定义
>     │   └── model.go 		# 数据结构和模型实现
>     ├── router       		# 路由功能实现
>     │   └── router.go 		# 路由定义和管理
>     └── utils       		 # 工具函数
>         └── jwt.go  		 # JWT处理相关函数
> └── response         		# 响应相关定义
>     ├── Errors.go   		 # 错误处理
>     ├── Rsp.go      		 # 通用响应结构
>     └── rspModel.go 		 # 响应模型定义
> └── server          		 # 服务器端逻辑
>     └── user        		 # 用户功能实现
>         ├── conf    		 # 配置文件
>         │   ├── conf.go		# 配置读取逻辑
>         │   └── db.yaml 	      # 数据库配置文件
>         ├── dao      		# 数据访问对象
>         │   ├── init.go  	    # 初始化数据库
>         │   └── user.go 	     # 用户相关的数据库操作
>         └── proto   		 # proto文件定义
>             ├── user.pb.go           # 编译后的用户proto文件
>             ├── user.proto          # 用户定义的proto文件
>             └── user_grpc.pb.go     # gRPC编译后的用户proto文件
>     └── main.go     		 # 服务器的主入口/同时实现了相应的业务逻辑函数

结构如上，首先我们需要知道，在app层，我们实现了基本的路由逻辑，随后client相当于我们的客户端，我们通过client的grpc客户端和server进行通信，并且拿到server上面实现的业务逻辑函数，从而调用相关的函数，从而实现完整用户的注册/登陆功能。

了解完这个demo的基本结构以后，下面我来细说。

##### proto部分

```protobuf
syntax = "proto3";

package user;

option go_package = "2025/01January/20250121/micro/proto;user";

message RegisterRequest {
  string username = 1;
  string password = 2;
}

message LoginRequest {
  string username = 1;
  string password = 2;
}

message RegisterResponse {
  bool success = 1;
  string message = 2;
}

message LoginResponse {
  bool success = 1;
  string message = 2;
  string token = 3;
}

service User {
  rpc Login(LoginRequest) returns(LoginResponse);
  rpc Register(RegisterRequest) returns(RegisterResponse);
}
```

首先我来解释解释这上面都是什么意思，LoginRequest 以及RegisterRequest都相当于一个请求的结构体，上面带有请求的信息，我们可以看到信息就是用户名和密码，随后就是我们的回复信息Response，上面带有请求的结果以及信息，接下来只需要在控制台输入相应的指令就好了

```cmd
protoc --go_out=. --go-grpc_out=. user.proto
```

随后就会产生相应的go文件，这里产生的文件我们并不需要理会它，这些文件实现了通信的内部逻辑，而且我们由于我们只需要按照规则简单的定义.proto文件就可以了，所以遵循这个规则的我们就可以轻易的实现通信，并且不需要关心内部的通信逻辑。

##### gRPC部分

随后我们需要做的是什么呢？在接下来我们需要将这个文件复制到server端一份，以保证两端可以相互通信，然后我们需要将我们定义在.proto文件中的函数方法进行重写，比如：

```go
type UserServer struct {
	pb.UnimplementedUserServer
}

func (U *UserServer) Register(_ context.Context, req *pb.RegisterRequest) (*pb.RegisterResponse, error) {
	user := model.User{
		Name:     req.Username,
		Password: req.Password,
	}
	if len(user.Password) < 5 || len(user.Password) > 20 {
		return &pb.RegisterResponse{
			Success: false,
			Message: "PasswordLength",
		}, response.ErrPasswordLength
	}
	if len(user.Name) < 5 || len(user.Name) > 20 {
		return &pb.RegisterResponse{
			Success: false,
			Message: "NameLength",
		}, response.ErrNameLength
	}
	if err := dao.Register(user); err != nil {
		return &pb.RegisterResponse{
			Success: false,
			Message: err.Error(),
		}, err
	}
	return &pb.RegisterResponse{
		Success: true,
		Message: "ok",
	}, nil
}
```

这部分实现了相应的服务端的逻辑，就相当于我们在单层架构中的service层的作用，在该方法中，我们调用了dao层的函数，以便于实现完整的注册逻辑。

既然重写了相应的方法函数，这并不是完事大吉了，我们还需要在真正运行代码的时候让两端进行通信，此时就需要我们创建相应的grpc客户端和服务端。

```go
	// 设置监听地址和端口
	listener, err := net.Listen("tcp", ":10001")
	if err != nil {
		log.Fatalf("Failed to listen: %v", err)
	}
	
	grpcServer := grpc.NewServer(grpc.Creds(insecure.NewCredentials()))

	// 注册服务
	pb.RegisterUserServer(grpcServer, &UserServer{})
```

考虑到此处我们的重点是gRPC通信和nacos服务注册，我们就采取了不安全的通信方式。

此处我们设置了监听的端口，UserServer是实现了相对应的接口的结构体，使用 `pb.RegisterUserServer` 函数将 `UserServer` 实例注册到 gRPC 服务器。这使得通过 gRPC 客户端调用该服务时，可以调用您在 `UserServer` 中实现的方法。

此处还不着急使用nacos注册服务，我们先来看看客户端如何通信。

```go
	// 连接到 gRPC 服务
	UserConn, err = grpc.Dial(addr, grpc.WithInsecure())

	if err != nil {
		log.Fatalf("failed to connect to gRPC service: %v", err)
		return
	}

	UserClient = pb.NewUserClient(UserConn)
```

此处的addr是我们对应的server的地址+端口号，也是采取了不安全的通信，然后这个UserClient可以这样理解，他作为一个对象，在通信之后，能够作为UserServer对象来调用服务端实现的方法，就是如此，所以我们要在app层创建一个UserClient实例来完成我们的调用。

##### nacos部分

nacos是什么？他是一个服务注册中心，服务在启动时将自身注册到 Nacos，包含 IP、端口和服务名称，其他服务可以通过服务名称查询可用实例，于是我们在**服务端**将当前启动服务的ip端口以及服务名称等注册到了nacos，在客户端，我们就可以通过服务的名称等信息查找相应服务对应的信息，这样我们就不需要以硬编码的形式将服务对应的IP端口写在客户端，而且当我们在服务端改变IP和端口时，也是将最新的IP端口注册到nacos中，这样就不需要担心修改客户端的问题了，因为客户端的IP和端口是以变量的形式保存的，保证了灵活性，而当有多个相同服务注册时，客户端可以从多个可用实例中选择，提高系统的整体性能与可靠性。

下面让我们来看看是如何实现的。

首先是在服务端：

```go
	nacos.RegisterServiceInstance(nacos.Client, vo.RegisterInstanceParam{
		Ip:          "127.0.0.1",                          // 根据实际情况填写
		Port:        10001,                                // gRPC服务的端口
		ServiceName: "UserTest",                           // 服务名称
		GroupName:   "GroupTest",                          // 分组名称
		ClusterName: "cluster-a",                          // 集群名称
		Weight:      10,                                   // 权重
		Enable:      true,                                 // 是否启用
		Healthy:     true,                                 // 是否健康
		Ephemeral:   true,                                 // 是否为临时实例
		Metadata:    map[string]string{"idc": "shanghai"}, // 元数据信息
	})
```

在启动服务端时，我们通过函数调用，将服务的地址等信息注册到nacos的相应位置，关于nacos服务的增删查改可以看看我之前发的博客。

这样就算注册完成了，启动完服务端，我们返回来看客户端：

```go

	param := vo.GetServiceParam{
		ServiceName: "UserTest",            // 替换为你的服务名称
		GroupName:   "GroupTest",           // 根据需要设置
		Clusters:    []string{"cluster-a"}, // 集群名称
	}

	service, err := nacos.Client.GetService(param)
	if err != nil {
		log.Fatalf("can't discover the service: %v", err)
		return
	}

	// 获取第一个服务实例的地址
	if len(service.Hosts) == 0 {
		log.Fatal("no healthy instance found for service 'user'")
		return
	}

	addr := fmt.Sprintf("%s:%d", service.Hosts[0].Ip, service.Hosts[0].Port)
```

此处我们只需要将搜索信息放入param中，然后根据我们的信息查找这个服务是否注册到了nacos中，如果没注册，就说明服务端还没启动，就停止调用，如果发现了这个服务，我们便可以获取这个服务的地址，然后根据地址，连接gRPC服务，这样就算完成了！

最后，启动服务端，将服务注册进nacos，同时开启grpc服务端，监听端口，然后启动客户端，发送post命令，看到令人满意的结果，就算结束了。

如果想要详细的代码，可以看我的github-Some-WORKs仓库

-----

### 结语

此处便是我关于nacos和gRPC如何结合起来实现服务注册调用的想法，如果有问题，欢迎提出来！



