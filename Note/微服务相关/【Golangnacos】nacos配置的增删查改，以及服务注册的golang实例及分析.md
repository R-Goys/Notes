## nacos配置的增删查改

-----

先具体来看一段代码，我将逐步分析每一段的作用

```golang
package main

import (
    "fmt"
    "time"

    "github.com/nacos-group/nacos-sdk-go/clients"
    "github.com/nacos-group/nacos-sdk-go/common/constant"
    "github.com/nacos-group/nacos-sdk-go/vo"
)

func main() {
    // 创建ServerConfig
    sc := []constant.ServerConfig{
       *constant.NewServerConfig(
          "192.168.195.129",                  // Nacos服务器的IP地址
          8848,                               // Nacos服务器的端口号
          constant.WithContextPath("/nacos"), // 上下文路径
       ),
    }

    // 创建ClientConfig
    cc := *constant.NewClientConfig(
       constant.WithNamespaceId("ace1b5fe-80c3-4fab-b89a-625f9ff41093"), // 命名空间ID
       constant.WithTimeoutMs(5000),                                     // 超时时间（毫秒）
       constant.WithNotLoadCacheAtStart(true),                           // 启动时不加载缓存
       constant.WithLogDir("/tmp/nacos/log"),                            // 日志目录
       constant.WithCacheDir("/tmp/nacos/cache"),                        // 缓存目录
       constant.WithLogLevel("debug"),                                   // 日志级别
       constant.WithUsername("nacos"),                                   // 用户名
       constant.WithPassword("nacos"),                               // 密码
    )

    // 创建配置客户端
    client, err := clients.NewConfigClient(
       vo.NacosClientParam{
          ClientConfig:  &cc, // 客户端配置
          ServerConfigs: sc,  // 服务器配置切片
       },
    )
    if err != nil {
       panic(err) // 如果创建客户端失败，程序终止并打印错误信息
    }

    // 发布配置
    _, err = client.PublishConfig(vo.ConfigParam{
       DataId:  "test-data",    // 配置的数据ID
       Group:   "test-group",   // 配置的分组
       Content: "hello world!", // 配置的内容
    })
    if err != nil {
       fmt.Printf("PublishConfig err:%+v \n", err) // 如果发布配置失败，打印错误信息
    }

    _, err = client.PublishConfig(vo.ConfigParam{
       DataId:  "test-data-2",  // 第二个配置的数据ID
       Group:   "test-group",   // 第二个配置的分组
       Content: "hello world!", // 第二个配置的内容
    })
    if err != nil {
       fmt.Printf("PublishConfig err:%+v \n", err) // 如果发布配置失败，打印错误信息
    }

    // 等待1秒，确保配置发布完成
    time.Sleep(1 * time.Second)

    // 获取配置
    content, err := client.GetConfig(vo.ConfigParam{
       DataId: "test-data",  // 要获取的配置的数据ID
       Group:  "test-group", // 要获取的配置的分组
    })
    fmt.Println("GetConfig,config :" + content) // 打印获取到的配置内容

    // 监听配置变更
    err = client.ListenConfig(vo.ConfigParam{
       DataId: "test-data",  // 要监听的配置的数据ID
       Group:  "test-group", // 要监听的配置的分组
       OnChange: func(namespace, group, dataId, data string) {
          fmt.Println("config changed group:" + group + ", dataId:" + dataId + ", content:" + data) // 配置变更时打印信息
       },
    })
    if err != nil {
       fmt.Printf("ListenConfig err:%+v \n", err) // 如果监听失败，打印错误信息
    }

    err = client.ListenConfig(vo.ConfigParam{
       DataId: "test-data-2", // 第二个要监听的配置的数据ID
       Group:  "test-group",  // 第二个要监听的配置的分组
       OnChange: func(namespace, group, dataId, data string) {
          fmt.Println("config changed group:" + group + ", dataId:" + dataId + ", content:" + data) // 配置变更时打印信息
       },
    })
    if err != nil {
       fmt.Printf("ListenConfig err:%+v \n", err) // 如果监听失败，打印错误信息
    }

    // 等待1秒，确保监听配置生效
    time.Sleep(1 * time.Second)

    // 修改配置
    _, err = client.PublishConfig(vo.ConfigParam{
       DataId:  "test-data",   // 要修改的配置的数据ID
       Group:   "test-group",  // 要修改的配置的分组
       Content: "test-listen", // 修改后的配置内容
    })
    if err != nil {
       fmt.Printf("PublishConfig err:%+v \n", err) // 如果修改配置失败，打印错误信息
    }

    _, err = client.PublishConfig(vo.ConfigParam{
       DataId:  "test-data-2", // 第二个要修改的配置的数据ID
       Group:   "test-group",  // 第二个要修改的配置的分组
       Content: "test-listen", // 第二个修改后的配置内容
    })
    if err != nil {
       fmt.Printf("PublishConfig err:%+v \n", err) // 如果修改配置失败，打印错误信息
    }

    // 等待2秒，确保配置修改被监听到
    time.Sleep(2 * time.Second)

    // 等待1秒
    time.Sleep(1 * time.Second)

    // 删除配置
    _, err = client.DeleteConfig(vo.ConfigParam{
       DataId: "test-data",  // 要删除的配置的数据ID
       Group:  "test-group", // 要删除的配置的分组
    })
    if err != nil {
       fmt.Printf("DeleteConfig err:%+v \n", err) // 如果删除配置失败，打印错误信息
    }

    // 等待1秒，确保配置删除操作完成
    time.Sleep(1 * time.Second)

    // 取消监听配置变更
    err = client.CancelListenConfig(vo.ConfigParam{
       DataId: "test-data",  // 要取消监听的配置的数据ID
       Group:  "test-group", // 要取消监听的配置的分组
    })
    if err != nil {
       fmt.Printf("CancelListenConfig err:%+v \n", err) // 如果取消监听失败，打印错误信息
    }

    // 搜索配置
    searchPage, _ := client.SearchConfig(vo.SearchConfigParam{
       Search:   "blur", // 搜索模式，这里是模糊搜索
       DataId:   "",     // 数据ID，为空表示不指定
       Group:    "",     // 分组，为空表示不指定
       PageNo:   1,      // 搜索的页码
       PageSize: 10,     // 每页的大小
    })
    fmt.Printf("Search config:%+v \n", searchPage) // 打印搜索结果
}
```



---

### Serverconfig

`ServerConfig`主要用于配置Nacos服务器的信息。它告诉客户端如何连接到Nacos服务器，包括服务器的IP地址、端口号和上下文路径等。通过`ServerConfig`，客户端可以找到并连接到Nacos服务器，这里以切片的形式，是为了可以保证多个选择，并且具有可扩展性。

```go
sc := []constant.ServerConfig{
    *constant.NewServerConfig(
        "192.168.195.129", // Nacos服务器的IP地址
        8848,              // Nacos服务器的端口号
        constant.WithContextPath("/nacos"), // 上下文路径
    ),
}
```
这段代码配置了一个Nacos服务器，使得客户端可以连接到该服务器。

### ClientConfig
`ClientConfig`用于配置客户端的行为和属性，包括命名空间ID、超时时间、日志级别、缓存目录等。`ClientConfig`中的命名空间ID（`NamespaceId`）指定了客户端要操作的特定命名空间。通过命名空间ID，客户端可以确保只管理属于该命名空间的配置和服务。

例如：
```go
cc := *constant.NewClientConfig(
    constant.WithNamespaceId("ace1b5fe-80c3-4fab-b89a-625f9ff41093"), // 命名空间ID
    constant.WithTimeoutMs(5000),
    constant.WithNotLoadCacheAtStart(true),
    constant.WithLogDir("/tmp/nacos/log"),
    constant.WithCacheDir("/tmp/nacos/cache"),
    constant.WithLogLevel("debug"),
    constant.WithUsername("nacos"),
    constant.WithPassword("nacos"),
)
```
这段代码配置了客户端的行为，特别是指定了要操作的命名空间ID。



- **ServerConfig**：用于配置Nacos服务器的信息，告诉客户端如何连接到Nacos服务器。
- **ClientConfig**：用于配置客户端的行为和属性，包括命名空间ID，告诉客户端要操作哪个命名空间中的配置和服务。

通过这两个配置，客户端可以正确地连接到Nacos服务器，并操作指定命名空间中的配置和服务。以下是代码中两者的使用关系：

```go
client, err := clients.NewConfigClient(
    vo.NacosClientParam{
        ClientConfig:  &cc, // 客户端配置，包括命名空间ID等
        ServerConfigs: sc,  // 服务器配置切片，用于连接Nacos服务器
    },
)
```

这里就创建了一个客户端用于访问服务器特定的命名空间。

**命名空间是什么？**

**AI答 + 我的理解**：在Nacos中，命名空间（Namespace）是一种用于隔离配置和服务的机制。通过命名空间，可以将不同的配置和服务分隔开来，避免它们之间的冲突和干扰。命名空间在Nacos中通常用于多环境（如开发环境、测试环境、生产环境）的配置管理，或者用于不同的项目或团队之间的配置隔离。

比如我的代码里面，命名空间ID是 `"ace1b5fe-80c3-4fab-b89a-625f9ff41093"`。这个ID是一个唯一的标识符，用于指定一个特定的命名空间。通过这个命名空间ID，客户端可以操作属于该命名空间的配置和服务。

-----

### 添加配置

```go
// 发布配置
_, err = client.PublishConfig(vo.ConfigParam{
    DataId:  "test-data",    // 配置的数据ID
    Group:   "test-group",   // 配置的分组
    Content: "hello world!", // 配置的内容
})
if err != nil {
    fmt.Printf("PublishConfig err:%+v \n", err) // 如果发布配置失败，打印错误信息
}

_, err = client.PublishConfig(vo.ConfigParam{
    DataId:  "test-data-2",  // 第二个配置的数据ID
    Group:   "test-group",   // 第二个配置的分组
    Content: "hello world!", // 第二个配置的内容
})
if err != nil {
    fmt.Printf("PublishConfig err:%+v \n", err) // 如果发布配置失败，打印错误信息
```

此处实现的功能为发布配置，和nacos控制台的发布配置功能是一样的

![QQ_1737025381798](C:\Users\chenyue\Desktop\QQ_1737025381798.png)

注意要选择相应的命名空间。

-----

### 监视配置

```go
// 获取配置
content, err := client.GetConfig(vo.ConfigParam{
    DataId: "test-data",  // 要获取的配置的数据ID
    Group:  "test-group", // 要获取的配置的分组
})
fmt.Println("GetConfig,config :" + content) // 打印获取到的配置内容

// 监听配置变更
err = client.ListenConfig(vo.ConfigParam{
    DataId: "test-data",  // 要监听的配置的数据ID
    Group:  "test-group", // 要监听的配置的分组
    OnChange: func(namespace, group, dataId, data string) {
       fmt.Println("config changed group:" + group + ", dataId:" + dataId + ", content:" + data) // 配置变更时打印信息
    },
})
```

这里实现了获取配置，以及对配置的监听，为什么这里可以监听到之后配置的变更呢？这里我认为应该是在函数内部使用了goroutine来实现的，一旦检测到有变化，监听器就会发出响应。

-----

### 修改/删除配置

```go
_, err = client.PublishConfig(vo.ConfigParam{
    DataId:  "test-data",   // 要修改的配置的数据ID
    Group:   "test-group",  // 要修改的配置的分组
    Content: "test-listen", // 修改后的配置内容
})
```

```go
_, err = client.DeleteConfig(vo.ConfigParam{
    DataId: "test-data",  // 要删除的配置的数据ID
    Group:  "test-group", // 要删除的配置的分组
})
if err != nil {
    fmt.Printf("DeleteConfig err:%+v \n", err) // 如果删除配置失败，打印错误信息
}
```

至于修改以及删除的逻辑就很简单了，这里不多赘述

------

## nacos服务注册

先看代码示例，在接下来我会详细讲解每一步

```go
package main

import (
    "fmt"
    "github.com/nacos-group/nacos-sdk-go/v2/clients/naming_client"
    "time"

    "github.com/nacos-group/nacos-sdk-go/v2/clients"
    "github.com/nacos-group/nacos-sdk-go/v2/common/constant"
    "github.com/nacos-group/nacos-sdk-go/v2/model"
    "github.com/nacos-group/nacos-sdk-go/v2/util"
    "github.com/nacos-group/nacos-sdk-go/v2/vo"
)

func main() {
    // 创建 ServerConfig，配置 Nacos 服务器地址、端口及上下文路径
    sc := []constant.ServerConfig{
       *constant.NewServerConfig("192.168.195.129", 8848, constant.WithContextPath("/nacos")),
    }

    // 创建 ClientConfig，配置客户端的基本参数，如命名空间、超时时间、日志路径等
    cc := *constant.NewClientConfig(
       constant.WithNamespaceId("ace1b5fe-80c3-4fab-b89a-625f9ff41093"),
       constant.WithTimeoutMs(5000),
       constant.WithNotLoadCacheAtStart(true),
       constant.WithLogDir("/tmp/nacos/log"),
       constant.WithCacheDir("/tmp/nacos/cache"),
       constant.WithLogLevel("debug"),
       constant.WithUsername("nacos"),     // 用户名
       constant.WithPassword("nacos"), // 密码
    )

    // 创建命名服务客户端
    client, err := clients.NewNamingClient(
       vo.NacosClientParam{
          ClientConfig:  &cc,
          ServerConfigs: sc,
       },
    )

    if err != nil {
       panic(err) // 如果客户端创建失败，则抛出异常
    }

    // 注册服务实例到 Nacos
    registerServiceInstance(client, vo.RegisterInstanceParam{
       Ip:          "10.0.0.10",                          // 服务实例的 IP 地址
       Port:        8848,                                 // 服务实例的端口号
       ServiceName: "test.go",                            // 服务名称
       GroupName:   "group-a",                            // 分组名称
       ClusterName: "cluster-a",                          // 集群名称
       Weight:      10,                                   // 权重
       Enable:      true,                                 // 是否启用
       Healthy:     true,                                 // 是否健康
       Ephemeral:   true,                                 // 是否为临时实例
       Metadata:    map[string]string{"idc": "shanghai"}, // 元数据信息
    })

    //从 Nacos 取消注册服务实例
    deRegisterServiceInstance(client, vo.DeregisterInstanceParam{
       Ip:          "10.0.0.10", // 服务实例的 IP 地址
       Port:        8848,        // 服务实例的端口号
       ServiceName: "demo.go",   // 服务名称
       GroupName:   "group-a",   // 分组名称
       Cluster:     "cluster-a", // 集群名称
       Ephemeral:   true,        // 必须为临时实例
    })

    time.Sleep(1 * time.Second) // 等待 1 秒

    // 批量注册多个服务实例
    batchRegisterServiceInstance(client, vo.BatchRegisterInstanceParam{
       ServiceName: "demo.go", // 服务名称
       GroupName:   "group-a", // 分组名称
       Instances: []vo.RegisterInstanceParam{{
          Ip:          "10.0.0.10",                          // 第一个服务实例的 IP 地址
          Port:        8848,                                 // 第一个服务实例的端口号
          Weight:      10,                                   // 权重
          Enable:      true,                                 // 是否启用
          Healthy:     true,                                 // 是否健康
          Ephemeral:   true,                                 // 是否为临时实例
          ClusterName: "cluster-a",                          // 集群名称
          Metadata:    map[string]string{"idc": "shanghai"}, // 元数据信息
       }, {
          Ip:          "10.0.0.12",                          // 第二个服务实例的 IP 地址
          Port:        8848,                                 // 第二个服务实例的端口号
          Weight:      7,                                    // 权重
          Enable:      true,                                 // 是否启用
          Healthy:     true,                                 // 是否健康
          Ephemeral:   true,                                 // 是否为临时实例
          ClusterName: "cluster-a",                          // 集群名称
          Metadata:    map[string]string{"idc": "shanghai"}, // 元数据信息
       }},
    })

    time.Sleep(1 * time.Second) // 等待 1 秒

    // 根据服务名称、分组名称和集群名称获取服务信息
    getService(client, vo.GetServiceParam{
       ServiceName: "demo.go",             // 服务名称
       GroupName:   "group-a",             // 分组名称
       Clusters:    []string{"cluster-a"}, // 集群名称列表
    })

    // 获取指定服务的所有实例
    selectAllInstances(client, vo.SelectAllInstancesParam{
       ServiceName: "demo.go",             // 服务名称
       GroupName:   "group-a",             // 分组名称
       Clusters:    []string{"cluster-a"}, // 集群名称列表
    })

    // 获取指定服务的健康实例
    selectInstances(client, vo.SelectInstancesParam{
       ServiceName: "demo.go",             // 服务名称
       GroupName:   "group-a",             // 分组名称
       Clusters:    []string{"cluster-a"}, // 集群名称列表
       HealthyOnly: true,                  // 仅获取健康实例
    })

    // 根据加权随机算法获取一个健康实例
    selectOneHealthyInstance(client, vo.SelectOneHealthInstanceParam{
       ServiceName: "demo.go",             // 服务名称
       GroupName:   "group-a",             // 分组名称
       Clusters:    []string{"cluster-a"}, // 集群名称列表
    })

    // 订阅服务变更，当服务信息发生变化时，会触发回调函数
    subscribeParam := &vo.SubscribeParam{
       ServiceName: "demo.go", // 服务名称
       GroupName:   "group-a", // 分组名称
       SubscribeCallback: func(services []model.Instance, err error) {
          fmt.Printf("callback return services:%s \n\n", util.ToJsonString(services)) // 变更时打印服务实例信息
       },
    }
    subscribe(client, subscribeParam)

    // 等待 3 秒，让客户端从服务端拉取变更
    time.Sleep(3 * time.Second)

    // 更新服务实例信息
    updateServiceInstance(client, vo.UpdateInstanceParam{
       Ip:          "10.0.0.11",                          // 更新后的 IP 地址
       Port:        8848,                                 // 服务实例的端口号
       ServiceName: "demo.go",                            // 服务名称
       GroupName:   "group-a",                            // 分组名称
       ClusterName: "cluster-a",                          // 集群名称
       Weight:      10,                                   // 权重
       Enable:      true,                                 // 是否启用
       Healthy:     true,                                 // 是否健康
       Ephemeral:   true,                                 // 是否为临时实例
       Metadata:    map[string]string{"idc": "beijing1"}, // 更新后的元数据信息
    })

    // 等待 3 秒，让客户端从服务端拉取变更
    time.Sleep(3 * time.Second)

    // 取消订阅服务变更
    unSubscribe(client, subscribeParam)

    // 获取指定分组下的所有服务名称列表
    getAllService(client, vo.GetAllServiceInfoParam{
       GroupName: "group-a", // 分组名称
       PageNo:    1,         // 分页页码
       PageSize:  10,        // 每页大小
    })
}

//==========================================以下为函数实现===================================================

// registerServiceInstance 向 Nacos 注册一个服务实例
func registerServiceInstance(client naming_client.INamingClient, param vo.RegisterInstanceParam) {
    // 调用 RegisterInstance 方法注册服务实例
    success, err := client.RegisterInstance(param)
    if !success || err != nil {
       // 如果注册失败，抛出 panic 并打印错误信息
       panic("RegisterServiceInstance failed!" + err.Error())
    }
    // 打印注册参数和结果
    fmt.Printf("RegisterServiceInstance,param:%+v,result:%+v \n\n", param, success)
}

// batchRegisterServiceInstance 向 Nacos 批量注册多个服务实例
func batchRegisterServiceInstance(client naming_client.INamingClient, param vo.BatchRegisterInstanceParam) {
    // 调用 BatchRegisterInstance 方法批量注册服务实例
    success, err := client.BatchRegisterInstance(param)
    if !success || err != nil {
       // 如果批量注册失败，抛出 panic 并打印错误信息
       panic("BatchRegisterServiceInstance failed!" + err.Error())
    }
    // 打印批量注册参数和结果
    fmt.Printf("BatchRegisterServiceInstance,param:%+v,result:%+v \n\n", param, success)
}

// deRegisterServiceInstance 从 Nacos 取消注册一个服务实例
func deRegisterServiceInstance(client naming_client.INamingClient, param vo.DeregisterInstanceParam) {
    // 调用 DeregisterInstance 方法取消注册服务实例
    success, err := client.DeregisterInstance(param)
    if !success || err != nil {
       // 如果取消注册失败，抛出 panic 并打印错误信息
       panic("DeRegisterServiceInstance failed!" + err.Error())
    }
    // 打印取消注册参数和结果
    fmt.Printf("DeRegisterServiceInstance,param:%+v,result:%+v \n\n", param, success)
}

// updateServiceInstance 更新 Nacos 中已注册的服务实例信息
func updateServiceInstance(client naming_client.INamingClient, param vo.UpdateInstanceParam) {
    // 调用 UpdateInstance 方法更新服务实例信息
    success, err := client.UpdateInstance(param)
    if !success || err != nil {
       // 如果更新失败，抛出 panic 并打印错误信息
       panic("UpdateInstance failed!" + err.Error())
    }
    // 打印更新参数和结果
    fmt.Printf("UpdateServiceInstance,param:%+v,result:%+v \n\n", param, success)
}

// getService 从 Nacos 获取指定服务的信息
func getService(client naming_client.INamingClient, param vo.GetServiceParam) {
    // 调用 GetService 方法获取服务信息
    service, err := client.GetService(param)
    if err != nil {
       // 如果获取服务信息失败，抛出 panic 并打印错误信息
       panic("GetService failed!" + err.Error())
    }
    // 打印获取服务信息的参数和服务信息结果
    fmt.Printf("GetService,param:%+v, result:%+v \n\n", param, service)
}

// selectAllInstances 从 Nacos 获取指定服务的所有实例
func selectAllInstances(client naming_client.INamingClient, param vo.SelectAllInstancesParam) {
    // 调用 SelectAllInstances 方法获取所有实例
    instances, err := client.SelectAllInstances(param)
    if err != nil {
       // 如果获取所有实例失败，抛出 panic 并打印错误信息
       panic("SelectAllInstances failed!" + err.Error())
    }
    // 打印获取所有实例的参数和实例信息结果
    fmt.Printf("SelectAllInstance,param:%+v, result:%+v \n\n", param, instances)
}

// selectInstances 从 Nacos 获取指定服务的健康实例
func selectInstances(client naming_client.INamingClient, param vo.SelectInstancesParam) {
    // 调用 SelectInstances 方法获取健康实例
    instances, err := client.SelectInstances(param)
    if err != nil {
       // 如果获取健康实例失败，抛出 panic 并打印错误信息
       panic("SelectInstances failed!" + err.Error())
    }
    // 打印获取健康实例的参数和实例信息结果
    fmt.Printf("SelectInstances,param:%+v, result:%+v \n\n", param, instances)
}

// selectOneHealthyInstance 从 Nacos 获取一个健康实例（使用加权随机算法）
func selectOneHealthyInstance(client naming_client.INamingClient, param vo.SelectOneHealthInstanceParam) {
    // 调用 SelectOneHealthyInstance 方法获取一个健康实例
    instances, err := client.SelectOneHealthyInstance(param)
    if err != nil {
       // 如果获取健康实例失败，抛出 panic 并打印错误信息
       panic("SelectOneHealthyInstance failed!")
    }
    // 打印获取健康实例的参数和实例信息结果
    fmt.Printf("SelectOneHealthyInstance,param:%+v, result:%+v \n\n", param, instances)
}

// subscribe 订阅 Nacos 中指定服务的变化
func subscribe(client naming_client.INamingClient, param *vo.SubscribeParam) {
    // 调用 Subscribe 方法订阅服务变化
    client.Subscribe(param)
}

// unSubscribe 取消订阅 Nacos 中指定服务的变化
func unSubscribe(client naming_client.INamingClient, param *vo.SubscribeParam) {
    // 调用 Unsubscribe 方法取消订阅服务变化
    client.Unsubscribe(param)
}

// getAllService 从 Nacos 获取指定分组下的所有服务名称列表
func getAllService(client naming_client.INamingClient, param vo.GetAllServiceInfoParam) {
    // 调用 GetAllServicesInfo 方法获取所有服务名称列表
    service, err := client.GetAllServicesInfo(param)
    if err != nil {
       // 如果获取服务名称列表失败，抛出 panic 并打印错误信息
       panic("GetAllService failed!")
    }
    // 打印获取服务名称列表的参数和服务名称列表结果
    fmt.Printf("GetAllService,param:%+v, result:%+v \n\n", param, service)
}
```

-----

### 服务的注册

```go
registerServiceInstance(client, vo.RegisterInstanceParam{
    Ip:          "10.0.0.10",                          // 服务实例的 IP 地址
    Port:        8848,                                 // 服务实例的端口号
    ServiceName: "test.go",                            // 服务名称
    GroupName:   "group-a",                            // 分组名称
    ClusterName: "cluster-a",                          // 集群名称
    Weight:      10,                                   // 权重
    Enable:      true,                                 // 是否启用
    Healthy:     true,                                 // 是否健康
    Ephemeral:   true,                                 // 是否为临时实例
    Metadata:    map[string]string{"idc": "shanghai"}, // 元数据信息
})
```

这里通过调用函数实现服务的注册，其中test.go只是是服务的名称，在注册服务之后，可以通过通过服务实例的ip地址和端口号进行访问，随后访问这些地址上运行的实际程序。

而这个函数是我们自己实现的，我们可以来看看函数的内部：

```go
func registerServiceInstance(client naming_client.INamingClient, param vo.RegisterInstanceParam) {
    // 调用 RegisterInstance 方法注册服务实例
    success, err := client.RegisterInstance(param)
    if !success || err != nil {
       // 如果注册失败，抛出 panic 并打印错误信息
       panic("RegisterServiceInstance failed!" + err.Error())
    }
    // 打印注册参数和结果
    fmt.Printf("RegisterServiceInstance,param:%+v,result:%+v \n\n", param, success)
}
```

这里直接通过调用client的方法来为我们注册服务，而这个`client`可以和nacos服务器进行通信，可以注册、取消注册、更新服务实例的信息，以及从 Nacos 服务器获取这些信息。

----

### 取消服务的注册

```go
deRegisterServiceInstance(client, vo.DeregisterInstanceParam{
    Ip:          "10.0.0.10", // 服务实例的 IP 地址
    Port:        8848,        // 服务实例的端口号
    ServiceName: "demo.go",   // 服务名称
    GroupName:   "group-a",   // 分组名称
    Cluster:     "cluster-a", // 集群名称
    Ephemeral:   true,        // 必须为临时实例
})
```

取消注册服务，以上均为必须要传递的参数，用于匹配将要取消注册的服务实例

**实现函数**：

```go
func deRegisterServiceInstance(client naming_client.INamingClient, param vo.DeregisterInstanceParam) {
    // 调用 DeregisterInstance 方法取消注册服务实例
    success, err := client.DeregisterInstance(param)
    if !success || err != nil {
       // 如果取消注册失败，抛出 panic 并打印错误信息
       panic("DeRegisterServiceInstance failed!" + err.Error())
    }
    // 打印取消注册参数和结果
    fmt.Printf("DeRegisterServiceInstance,param:%+v,result:%+v \n\n", param, success)
}
```

和注册的方式类似，直接通过调用cilent的方法，并传递需要取消注册的服务实例的必要参数，用于定位。

-----

### 批量注册

```go
batchRegisterServiceInstance(client, vo.BatchRegisterInstanceParam{
    ServiceName: "demo.go", // 服务名称
    GroupName:   "group-a", // 分组名称
    Instances: []vo.RegisterInstanceParam{{
       Ip:          "10.0.0.10",                          // 第一个服务实例的 IP 地址
       Port:        8848,                                 // 第一个服务实例的端口号
       Weight:      10,                                   // 权重
       Enable:      true,                                 // 是否启用
       Healthy:     true,                                 // 是否健康
       Ephemeral:   true,                                 // 是否为临时实例
       ClusterName: "cluster-a",                          // 集群名称
       Metadata:    map[string]string{"idc": "shanghai"}, // 元数据信息
    }, {
       Ip:          "10.0.0.12",                          // 第二个服务实例的 IP 地址
       Port:        8848,                                 // 第二个服务实例的端口号
       Weight:      7,                                    // 权重
       Enable:      true,                                 // 是否启用
       Healthy:     true,                                 // 是否健康
       Ephemeral:   true,                                 // 是否为临时实例
       ClusterName: "cluster-a",                          // 集群名称
       Metadata:    map[string]string{"idc": "shanghai"}, // 元数据信息
    }},
})
```

和名字一样，可以想函数中传递多个参数，用于注册服务，但不同的一点是，服务的名称和分组名称独立了出来，这表示注册的服务均属于同一种服务和分组，而此时传递的结构体中包含了多个instance实例(这个参数就是服务单独注册时使用到的结构体的切片)。

**函数实现**

```go
func batchRegisterServiceInstance(client naming_client.INamingClient, param vo.BatchRegisterInstanceParam) {
    // 调用 BatchRegisterInstance 方法批量注册服务实例
    success, err := client.BatchRegisterInstance(param)
    if !success || err != nil {
       // 如果批量注册失败，抛出 panic 并打印错误信息
       panic("BatchRegisterServiceInstance failed!" + err.Error())
    }
    // 打印批量注册参数和结果
    fmt.Printf("BatchRegisterServiceInstance,param:%+v,result:%+v \n\n", param, success)
}
```

无需多说，也是通过调用方法来实现服务注册。

---------

## http实例自动注册服务

先看看代码的实现，然后我会一步一步细说：

```go
package main

import (
    "context"
    "fmt"
    "github.com/nacos-group/nacos-sdk-go/v2/clients/naming_client"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/nacos-group/nacos-sdk-go/v2/clients"
    "github.com/nacos-group/nacos-sdk-go/v2/common/constant"
    "github.com/nacos-group/nacos-sdk-go/v2/vo"
)

func main() {
    // 1. 创建 Nacos 客户端
    client := createNacosClient()

    // 2. 定义服务实例的元数据信息
    serviceName := "servicetest"   // 服务名称
    groupName := "group1"          // 分组名称
    clusterName := "cluster-a"     // 集群名称
    ip := "127.0.0.1"              // 服务实例的 IP 地址（当前虚拟机的 IP 地址）
    port := 8080                   // 服务实例的端口号
    metadata := map[string]string{ // 元数据信息
       "idc": "shanghai",
    }

    // 3. 注册服务实例到 Nacos
    registerServiceInstance(client, vo.RegisterInstanceParam{
       Ip:          ip,
       Port:        uint64(port),
       ServiceName: serviceName,
       GroupName:   groupName,
       ClusterName: clusterName,
       Weight:      10,
       Enable:      true,
       Healthy:     true,
       Ephemeral:   true,
       Metadata:    metadata,
    })

    // 4. 启动 HTTP 服务
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
       fmt.Fprintf(w, "Hello, this is demo.go service at %s:%d!", ip, port)
    })

    server := &http.Server{
       Addr:    fmt.Sprintf("%s:%d", ip, port),
       Handler: nil,
    }

    go func() {
       fmt.Printf("Starting HTTP server at http://%s:%d\n", ip, port)
       if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
          fmt.Printf("HTTP server failed: %v\n", err)
       }
    }()

    // 5. 监听系统信号，实现优雅关闭
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    fmt.Println("Shutting down server...")

    // 6. 取消注册服务实例
    deRegisterServiceInstance(client, vo.DeregisterInstanceParam{
       Ip:          ip,
       Port:        uint64(port),
       ServiceName: serviceName,
       GroupName:   groupName,
       Cluster:     clusterName,
       Ephemeral:   true,
    })

    // 7. 关闭 HTTP 服务
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    if err := server.Shutdown(ctx); err != nil {
       fmt.Printf("HTTP server shutdown failed: %v\n", err)
    }

    fmt.Println("Server exited gracefully")
}

// createNacosClient 创建 Nacos 客户端
func createNacosClient() naming_client.INamingClient {
    // 创建 ServerConfig
    sc := []constant.ServerConfig{
       *constant.NewServerConfig(
          "192.168.195.129",                  // Nacos 服务器的 IP 地址
          8848,                               // Nacos 服务器的端口号
          constant.WithContextPath("/nacos"), // Nacos 服务器的上下文路径
       ),
    }

    // 创建 ClientConfig
    cc := *constant.NewClientConfig(
       constant.WithNamespaceId(""),              // 命名空间 ID
       constant.WithTimeoutMs(5000),              // 超时时间
       constant.WithNotLoadCacheAtStart(true),    // 启动时不加载缓存
       constant.WithLogDir("/tmp/nacos/log"),     // 日志目录
       constant.WithCacheDir("/tmp/nacos/cache"), // 缓存目录
       constant.WithLogLevel("debug"),            // 日志级别
       constant.WithUsername("nacos"),
       constant.WithPassword("nacos"),
    )

    // 创建命名服务客户端
    client, err := clients.NewNamingClient(
       vo.NacosClientParam{
          ClientConfig:  &cc,
          ServerConfigs: sc,
       },
    )
    if err != nil {
       panic("Failed to create Nacos client: " + err.Error())
    }

    return client
}

// registerServiceInstance 向 Nacos 注册一个服务实例
func registerServiceInstance(client naming_client.INamingClient, param vo.RegisterInstanceParam) {
    success, err := client.RegisterInstance(param)
    if !success || err != nil {
       panic("Failed to register service instance: " + err.Error())
    }
    fmt.Printf("Registered service instance: %+v\n", param)
}wd

// deRegisterServiceInstance 从 Nacos 取消注册一个服务实例
func deRegisterServiceInstance(client naming_client.INamingClient, param vo.DeregisterInstanceParam) {
    success, err := client.DeregisterInstance(param)
    if !success || err != nil {
       panic("Failed to deregister service instance: " + err.Error())
    }
    fmt.Printf("Deregistered service instance: %+v\n", param)
}
```

-----

### 创建client

第一步便是我们的创建client，此时我们也需要serverconfig和clientconfig

但是注册的client类型却不一样,注意，我们之前crud的时候采取的并不是naming client而是configclient，此处需要注意以下
```go
func createNacosClient() naming_client.INamingClient {
    // 创建 ServerConfig
    sc := []constant.ServerConfig{
       *constant.NewServerConfig(
          "192.168.195.129",                  // Nacos 服务器的 IP 地址
          8848,                               // Nacos 服务器的端口号
          constant.WithContextPath("/nacos"), // Nacos 服务器的上下文路径
       ),
    }

    // 创建 ClientConfig
    cc := *constant.NewClientConfig(
       constant.WithNamespaceId(""),              // 命名空间 ID
       constant.WithTimeoutMs(5000),              // 超时时间
       constant.WithNotLoadCacheAtStart(true),    // 启动时不加载缓存
       constant.WithLogDir("/tmp/nacos/log"),     // 日志目录
       constant.WithCacheDir("/tmp/nacos/cache"), // 缓存目录
       constant.WithLogLevel("debug"),            // 日志级别
       constant.WithUsername("nacos"),
       constant.WithPassword("nacos"),
    )

    // 创建命名服务客户端
    client, err := clients.NewNamingClient(
       vo.NacosClientParam{
          ClientConfig:  &cc,
          ServerConfigs: sc,
       },
    )
    if err != nil {
       panic("Failed to create Nacos client: " + err.Error())
    }

    return client
}
```

其他都是一样的，需要指定注册在哪一个服务器，哪一个命名空间。

### 注册服务到nacos服务器

```go
serviceName := "servicetest"   // 服务名称
groupName := "group1"          // 分组名称
clusterName := "cluster-a"     // 集群名称
ip := "127.0.0.1"              // 服务实例的 IP 地址（当前虚拟机的 IP 地址）
port := 8080                   // 服务实例的端口号
metadata := map[string]string{ // 元数据信息
    "idc": "shanghai",
}

// 3. 注册服务实例到 Nacos
registerServiceInstance(client, vo.RegisterInstanceParam{
    Ip:          ip,
    Port:        uint64(port),
    ServiceName: serviceName,
    GroupName:   groupName,
    ClusterName: clusterName,
    Weight:      10,
    Enable:      true,
    Healthy:     true,
    Ephemeral:   true,
    Metadata:    metadata,
})
```

此处我们需要指定当前服务实例所在的ip，端口，以及服务名称等一系列参数，这里只是想nacos服务器表示了这个服务时`存在`的，随后我们再启动我们的http服务。

### 启动http服务

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, this is demo.go service at %s:%d!", ip, port)
})

server := &http.Server{
    Addr:    fmt.Sprintf("%s:%d", ip, port),
    Handler: nil,
}

go func() {
    fmt.Printf("Starting HTTP server at http://%s:%d\n", ip, port)
    if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
       fmt.Printf("HTTP server failed: %v\n", err)
    }
}()
```

这里无需多说，就是随便启动了一个http服务

### 优雅结束

```go
// 5. 监听系统信号，实现优雅关闭
quit := make(chan os.Signal, 1)
signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
<-quit

fmt.Println("Shutting down server...")

// 6. 取消注册服务实例
deRegisterServiceInstance(client, vo.DeregisterInstanceParam{
    Ip:          ip,
    Port:        uint64(port),
    ServiceName: serviceName,
    GroupName:   groupName,
    Cluster:     clusterName,
    Ephemeral:   true,
})

// 7. 关闭 HTTP 服务
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
if err := server.Shutdown(ctx); err != nil {
    fmt.Printf("HTTP server shutdown failed: %v\n", err)
}

fmt.Println("Server exited gracefully")
```

最后优雅的结束。

-----

## 结语

关于这篇文章，主要写出来也是想逼自己去看看官方的示例，以便于自己更能理解nacos是怎么运作的，在此之前我网上查了很多资料，也问了AI，但都没学到什么东西说实话，毕竟Golang目前的学习环境跟java比起来真的差远了，反而今晚上看了一会官方给的示例让我收获很大....

所以真的得去看官方的文档。

以上就是我今天所学到的内容，希望对你也会有帮助，如果有问题，欢迎给我留言

毕竟我也只是个初学者，如果有错误希望包涵以下~
