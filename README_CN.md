# registry-nacos (*这是一个由社区驱动的项目*)

[English](https://github.com/kitex-contrib/registry-nacos/blob/main/README.md)

使用 **nacos** 作为 **Kitex** 的注册中心

##  这个项目应当如何使用?

### 基本使用

#### 服务端

```go
import (
    // ...
    "github.com/kitex-contrib/registry-nacos/registry"
    "github.com/nacos-group/nacos-sdk-go/clients"
    "github.com/nacos-group/nacos-sdk-go/clients/naming_client"
    "github.com/nacos-group/nacos-sdk-go/common/constant"
    "github.com/nacos-group/nacos-sdk-go/vo"
    "github.com/cloudwego/kitex/pkg/rpcinfo"
    // ...
)

func main() {
    // ... 
    r, err := registry.NewDefaultNacosRegistry()
    if err != nil {
        panic(err)
    }
    svr := echo.NewServer(
        new(EchoImpl),
        server.WithServerBasicInfo(&rpcinfo.EndpointBasicInfo{ServiceName: "echo"}),
        server.WithRegistry(r),
    )
    if err := svr.Run(); err != nil {
        log.Println("server stopped with error:", err)
    } else {
        log.Println("server stopped")
    }
    // ...
}
```

#### 客户端

```go
import (
    // ...
    "github.com/kitex-contrib/registry-nacos/resolver"
    "github.com/nacos-group/nacos-sdk-go/clients"
    "github.com/nacos-group/nacos-sdk-go/clients/naming_client"
    "github.com/nacos-group/nacos-sdk-go/common/constant"
    "github.com/nacos-group/nacos-sdk-go/vo"
    // ...
)

func main() {
    // ...
    r, err := resolver.NewDefaultNacosResolver()
    if err != nil {
       panic(err) 
    }
    client, err := echo.NewClient("echo", client.WithResolver(r))
    if err != nil {
        log.Fatal(err)
    }
    // ...
}
```

### 自定义 Nacos Client 配置

#### 服务端

```go
import (
    // ...
    "github.com/kitex-contrib/registry-nacos/registry"
    "github.com/nacos-group/nacos-sdk-go/clients"
    "github.com/nacos-group/nacos-sdk-go/clients/naming_client"
    "github.com/nacos-group/nacos-sdk-go/common/constant"
    "github.com/nacos-group/nacos-sdk-go/vo"
    // ...
)
func main() {
    // ...
    sc := []constant.ServerConfig{
	    *constant.NewServerConfig("127.0.0.1", 8848),
    }
    
    cc := constant.ClientConfig{
        NamespaceId:         "public",
        TimeoutMs:           5000,
        NotLoadCacheAtStart: true,
        LogDir:              "/tmp/nacos/log",
        CacheDir:            "/tmp/nacos/cache",
        LogLevel:            "info",
        Username:            "your-name",
        Password:            "your-password",
    }
    
    cli, err := clients.NewNamingClient(
        vo.NacosClientParam{
            ClientConfig:  &cc,
            ServerConfigs: sc,
        },
    )
    if err != nil {
        panic(err)
    }

    svr := echo.NewServer(new(EchoImpl),
        server.WithServerBasicInfo(&rpcinfo.EndpointBasicInfo{ServiceName: "echo"}),
        server.WithRegistry(registry.NewNacosRegistry(cli)),
    )
    if err := svr.Run(); err != nil {
        log.Println("server stopped with error:", err)
    } else {
        log.Println("server stopped")
    }
    // ...
}
```

#### 客户端

```go
import (
    // ...
    "github.com/kitex-contrib/registry-nacos/resolver"
    "github.com/nacos-group/nacos-sdk-go/clients"
    "github.com/nacos-group/nacos-sdk-go/clients/naming_client"
    "github.com/nacos-group/nacos-sdk-go/common/constant"
    "github.com/nacos-group/nacos-sdk-go/vo"
    // ...
)
func main() {
    // ... 
    sc := []constant.ServerConfig{
        *constant.NewServerConfig("127.0.0.1", 8848),
    }
    cc := constant.ClientConfig{
        NamespaceId:         "public",
        TimeoutMs:           5000,
        NotLoadCacheAtStart: true,
        LogDir:              "/tmp/nacos/log",
        CacheDir:            "/tmp/nacos/cache",
        LogLevel:            "info",
        Username:            "your-name",
        Password:            "your-password",
    }
    
    cli, err := clients.NewNamingClient(
        vo.NacosClientParam{
        ClientConfig:  &cc,
        ServerConfigs: sc,
        },
	)
    if err != nil {
        panic(err)
    }
    client, err := echo.NewClient("echo", client.WithResolver(resolver.NewNacosResolver(cli))
    if err != nil {
        log.Fatal(err)
    }
    // ...
}
```

### 环境变量

| 变量名 | 变量默认值 | 作用 |
| ------------------------- | ---------------------------------- | --------------------------------- |
| serverAddr               | 127.0.0.1                          | nacos 服务器地址 |
| serverPort               | 8848                               | nacos 服务器端口            |
| namespace                 |                                    | nacos 中的 namespace Id |

### 更多信息

更多示例请参考 [example](example)

## 兼容性
该包使用 Nacos1.x 客户端，Nacos2.0 和 Nacos1.0 服务端完全兼容该版本. [详情](https://nacos.io/zh-cn/docs/v2/upgrading/2.0.0-compatibility.html)

主要贡献者： [baiyutang](https://github.com/baiyutang)

