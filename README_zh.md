# WSMC

为 Minecraft Java 启用 WebSocket 支持。
由于大多数 CDN 提供商（至少是他们的免费套餐）不支持原始 TCP 代理，借助这个模组，服务器所有者现在可以将服务器隐藏在 CDN 后面，并让玩家通过 WebSocket 连接，从而防止 DDoS 攻击。

适用于 Minecraft Forge、Neoforge 和 Fabric：
* 1.21.11
* 1.20.5 到 1.21.4，包括。1.21.5 到 1.21.10 已知可正常工作。
* 1.20.2、1.20.3、1.20.4
* 1.20.1
* 1.18.2、1.19.x、1.20

此分支用于 1.21.11，截至 2026/01/12。

此模组独立运行，没有任何依赖项。

## 当此模组安装在服务器上时：
* 服务器将允许玩家通过 WebSocket 连接。
* 玩家仍可以通过原始的TCP协议地址加入。
* 服务器在同一监听端口上接受和处理 TCP 和 WebSocket 连接。
* 如果不在客户端安装此模组，玩家仍可以使用原始的TCP协议地址加入已安装此模组的服务器。
* 服务器可以从 WebSocket 握手获取客户端统计信息（例如，真实 IP）。

## 当此模组安装在客户端时：
* 客户端可以使用类似 `ws://hostname.com:port/path_to_minecraft_endpoint` 的 URI 加入支持 WebSocket 的服务器。
* 客户端可以使用旧语法使用原始的TCP协议地址加入任何服务器，例如 `hostname_or_ip:port`。

## 注意事项
* 此模组不会影响任何游戏玩法。
* 此模组不会修改任何 GUI。
* Vanilla 客户端可以在您安装此模组后加入您的服务器，请注意您拥有的其他模组可能阻止 vanilla 客户端加入。
* 在客户端上安装此模组不会阻止您加入其他原版插件服或模组服。
* 服务器仍然可以获得通过 CDN 代理 WebSocket 加入的玩家的真实 IP。
* 此模组与其他 TCP-WebSocket 代理兼容，例如 websocat。

## 客户端选项
有时 DNS 会为 HTTP 主机名（ws）或 SNI（wss）返回慢速 IP。客户端可能希望控制解析 IP 地址的方式。

客户端可以可选地控制 WebSocket 握手中的 HTTP 主机名和 SNI：
```
使用指定的 http 主机名进行非安全 WebSocket 连接：
ws://host.com@ip.ip.ip.ip

指定 sni 和 http 主机名为相同值（sni-host.com），从 ip.ip.ip.ip 解析服务器 IP：
wss://sni-host.com@ip.ip.ip.ip

设置不同的 sni 和 http 主机名，从 host.com 解析服务器 IP：
wss://sni.com:@host.com[:port]

设置不同的 sni 和 http 主机名，从 sni.com 解析服务器 IP：
wss://:host.com@sni.com[:port]

分别设置 sni、http 主机名和服务器地址
wss://sni.com:host.com@ip.ip.ip.ip
```

端口和路径规范可以同时附加。

## 配置
此模组的配置通过"系统属性"传递。您可以使用命令行中的 `-D` 选项来传递这些参数。

| 属性键                    | 类型     | 用法                                                                                                                                                                                         | 端侧          | 默认值  | 示例   |
|---------------------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------|--------|
| wsmc.disableVanillaTCP    | 布尔值   | 禁用 vanilla TCP 登录和服务器状态。                                                                                                                                                          | 服务器        | false   | true   |
| wsmc.wsmcEndpoint         | 字符串   | 设置 Minecraft 登录和服务器状态的 WebSocket 端点。如果此属性不存在，客户端可以通过任何 WebSocket 端点加入游戏。必须以 / 开头，区分大小写。                                                 | 服务器        | 未设置  | /mc    |
| wsmc.debug                | 布尔值   | 显示调试日志。                                                                                                                                                                               | 服务器 客户端 | false   | true   |
| wsmc.dumpBytes            | 布尔值   | 转储原始 WebSocket 二进制帧。仅当 `wsmc.debug` 设置为 `true` 时才有效。                                                                                                                      | 服务器 客户端 | false   | true   |
| wsmc.maxFramePayloadLength| 整数     | 最大允许的帧负载长度。将此值设置为满足您的模组包的要求，否则 Netty 将抛出错误"Max frame length of x has been exceeded"。                                                                      | 服务器 客户端 | 65536   | 65536  |

## 依赖项
### Forge 版本
* `netty-codec-http` 用于处理 HTTP 和 WebSocket

### Fabric 版本
您需要安装 Fabric Loader，然后安装此模组。Fabric API 是可选的但强烈推荐。

## 对于开发者
要修改和调试代码，首先在 Eclipse IDE 中将"forge"或"fabric"文件夹作为 Gradle 项目导入，然后运行 gradle 任务 `genEclipseRuns`。

Windows 用户需要用 `.\` 和 `..\` 分别替换 `./` 和 `../`。

代码库使用 Minecraft 官方映射。

在服务器端，如果客户端通过 WebSocket 加入，则其握手请求可通过 vanilla 的 `net.minecraft.network.Connection` 类访问。
要获取此类信息，请将 Connection 实例转换为 IConnectionEx，然后调用 `IConnectionEx.getWsHandshakeRequest()`。

这对于在 Minecraft 服务器位于反向代理（例如 CDN）后面时获取原始请求的信息很有用。
例如，header `X-Forwarded-For` 和 `CF-IPCountry` 分别表示客户端 IP 地址和客户端国家代码。

### 编译 Fabric 构件
```
git clone https://github.com/rikka0w0/wsmc.git
cd wsmc/fabric
./gradlew build
```

要在 Fabric 中调试，可能需要创建 `run/config/fabric_loader_dependencies.json` 并包含以下内容：
```
{
  "version": 1,
  "overrides": {
    "wsmc": {
      "-depends": {
        "minecraft": "IGNORED",
        "fabricloader": "IGNORED"
      }
    }
  }
} 
```

### 编译 Forge 构件
```
git clone https://github.com/rikka0w0/wsmc.git
cd wsmc/forge
./gradlew build
```

### 指定 JRE 路径（自 1.18.1 起，Minecraft 需要 Java 17）：
```
./gradlew -Dorg.gradle.java.home=/path_to_jdk_directory <commands>
```
* 自 1.18.1 起，Minecraft 需要 Java 17
* 自 1.20.5 起，Minecraft 需要 Java 21