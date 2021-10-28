# [Spring Cloud Eureka 服务实现不停机（Zero-downtime）部署](https://segmentfault.com/a/1190000022134014)

Eureka Rest api文档：

https://github.com/Netflix/eureka/wiki/Eureka-REST-operations

## 问题

Spring Cloud 项目中一般会用到 Ribbon 作为负载均衡，那么是不是只要保证每个服务部署多台服务器，发布时采用 Rolling Update 分批次部署，保证一部分服务器正常提供服务的同时发布另一部分服务器，Ribbon 就能自动切换，保证服务的不间断？然而并不是。

## 产生原因

所有服务的状态保存在注册中心，即 Eureka Server。一个服务要想获取其他服务的实例列表和状态，需要通过 Eureka Client 定时从 Eureka Server 中获取并缓存下来，默认时间间隔是30秒。Eureka Client 和 Eureka Server 是通过 HTTP 协议通信，请求由 Eureka Client 发起，而不是基于长连接或者 Eureka Server 主动推送，所以无法立即知道其他服务状态变更。

即使同一个服务部署多台机器，每台机器依次发布，当其中一个服务实例重启时，服务调用方是无法第一时间知道的，所以还是会调用到这台暂时无法提供服务的实例上。这样会造成短暂的访问失败，这段时间也会对正在使用产品的用户造成一定的影响。

## 解决方案

基于以上的原因，在部署应用时应该按照以下步骤进行（为了简单起见，假设一个应用部署两个实例）：

1. 将服务的一个实例在注册中心的状态设置为 DOWN
2. 等待一段时间，直到其他服务缓存刷新，不再调用到这台服务器上
3. 停止服务，更新代码，重新启动，等待，直到启动成功

完成后，再重复以上步骤部署另一个实例。

### 第一步：修改服务实例状态为 DOWN

有两种方案可以修改实例的状态，选择其一即可：

1. 直接调用 Eureka Server API 修改：PUT /eureka/apps/{appID}/{instanceID}/status?value=DOWN
2. 调用服务实例对应的 actuator endpoint：`/service-registry`

我更偏向使用方法二，对应的命令：

```bash
curl -H "Content-Type:application/json" -X POST http://{host:port}/actuator/service-registry?status=DOWN
```

如果 actuator endpoint 加了 Spring Security Basic 认证，则还需要加上用户名和密码：

```bash
curl -H "Content-Type:application/json" -X POST -u {username}:{password} http://{host:port}/actuator/service-registry?status=DOWN
```

### 第二步：等待其他服务缓存刷新

具体要等多久，其他调用者的请求才会不再访问到这台状态为 DOWN 的实例？这里涉及到三个配置项：

- `eureka.client.registryFetchIntervalSeconds` Eureka 客户端每隔多久去 Eureka 服务器拉取最新的注册信息，默认值 30（秒）。
- `ribbon.ServerListRefreshInterval` Ribbon 的缓存刷新间隔时间，默认 30000（毫秒）。Eureka 客户端拉取到最新注册信息后，Ribbon、Feign 等组件不会立即生效，是因为 Ribbon 还有一层缓存。
- `eureka.server.responseCacheUpdateIntervalMs` Eureka Server 返回最新的注册信息的接口缓存刷新时间间隔，默认 30000（毫秒）。有时候会看到 Eureka 页面和 `/eureka/apps` 接口的服务状态不一致，就是因为 `/eureka/apps` 接口默认会有 30 秒缓存。

在默认情况下，当一个服务状态改为 DOWN，最长可能需要 30+30+30 秒，所有的缓存才会刷新，其他调用者才不会调用到这个状态为 DOWN 的实例。这就意味着修改服务实例状态为 DOWN 后需要等待 90 秒，才能进行下一步操作。

为了让部署时间缩短，可以将以上三个配置项都修改为5秒：

Eureka Server：

```yaml
eureka:
  server:
    responseCacheUpdateIntervalMs: 5000
```

Eureka Client（即各个服务）：

```yaml
ribbon:
  ServerListRefreshInterval: 5000
eureka:
  client:
    registryFetchIntervalSeconds: 5
```

完成以上配置，部署时将实例状态设为 DOWN 后，只需要等待 15 秒即可停止进程：

```bash
sleep 15s
```

### 第三步：实例部署

这一步主要需要注意

- 尽量不要使用 `kill -9 pid` 强制杀掉进程，而应该使用 `kill pid` 或者 `kill -15 pid` 关闭进程。使用 `kill pid` 或者 `kill -15 pid` 关闭进程之前，Eureka Client 会给 Eureka Server 请求删除自己，后续服务再次启动后会重新注册为 UP 状态。如果使用 `kill -9 pid` 强制杀掉进程，Eureka Client 没有办法注销自己，Eureka Server 就不知道该实例已下线，直到长时间收不到心跳才会删除该实例。如果在 Eureka Server 删除实例之前实例启动了，那么它的状态还是会保持 DOWN 状态。如果确实需要用到 `kill -9 pid` 强制杀掉进程，那么服务重启后需要再通过第一步的方式将实例状态设为 UP。
- 服务启动后，需要等待并确认启动成功后，才可以开始部署下一台服务器。这里我们可以定时去请求 Spring Boot 提供的 actuator endpoint `/health` 接口，例如每隔 1 秒请求一次，直到接口可以正常访问，即可认为服务启动成功。