Sentinel 控制台作为 Sentinel 的一大利器，提供了多个维度的监控和规则配置功能。Sentinel 客户端目前已可用于生产环境，但若希望在生产环境中使用 Sentinel 控制台还需要进行一些改造。本文将介绍如何对 Sentinel 控制台进行改造以便在生产环境中使用。

在生产环境中使用 Sentinel 控制台只需要两步改造：

1. 改造推送逻辑，支持向规则数据源进行推送
2. 改造监控逻辑，支持监控数据持久化

## 动态规则数据源

Sentinel 的 [动态规则数据源](https://github.com/alibaba/Sentinel/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%99%E6%89%A9%E5%B1%95) 用于从中读取及写入规则。从 0.2.0 版本开始，Sentinel 将动态规则数据源分为两种类型：读数据源（`ReadableDataSource`）和写数据源（`WritableDataSource`）：

- 读数据源仅负责监听或轮询读取远程存储的变更。
- 写数据源仅负责将规则变更写入到规则源中。

其中读数据源常见的实现方式有:

- pull 模式：客户端主动向某个规则管理中心定期轮询拉取规则，这个规则中心可以是 RDBMS、文件 等。这样做的方式是简单，缺点是可能无法及时获取变更，拉取过于频繁也可能会有性能问题。
- push 模式：规则中心统一推送，客户端通过注册监听器的方式时刻监听变化，比如使用 Nacos、Zookeeper 等配置中心。这种方式有更好的实时性和一致性保证。

在实际的场景中，不同的存储类型对应的数据源类型也不同。对于 push 模式的数据源，一般不支持写入；而 pull 模式的数据源则是可写的。

下面我们分别来分析一下它们结合 Sentinel 控制台的使用场景，以及相应的需要改造的点。

### 原始情况

若应用未注册任何数据源，直接从 Sentinel 控制台推送规则的过程非常简单：

![Original push rules from Sentinel Dashboard](https://cdn.nlark.com/lark/0/2018/png/47688/1536660296273-4f440bba-5b9e-4205-9402-fb6083b66912.png) 

Sentinel 控制台通过 API 将规则推送至客户端并直接更新到内存中。这种情况下应用重启规则就会消失，仅用于简单测试，不能用于生产环境。一般在生产环境中，我们需要在应用端配置规则数据源。

### pull 模式的数据源

pull 模式的数据源（如本地文件、RDBMS 等）一般是可写入的。使用时需要在客户端注册数据源：将对应的读数据源注册至对应的 RuleManager，将写数据源注册至 transport 的 `WritableDataSourceRegistry` 中。以本地文件数据源为例：

```java
public class FileDataSourceInit implements InitFunc {

    @Override
    public void init() throws Exception {
        String flowRulePath = "xxx";

        ReadableDataSource<String, List<FlowRule>> ds = new FileRefreshableDataSource<>(
            flowRulePath, source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>() {})
        );
        // 将可读数据源注册至 FlowRuleManager.
        FlowRuleManager.register2Property(ds.getProperty());

        WritableDataSource<List<FlowRule>> wds = new FileWritableDataSource<>(flowRulePath, this::encodeJson);
        // 将可写数据源注册至 transport 模块的 WritableDataSourceRegistry 中.
        // 这样收到控制台推送的规则时，Sentinel 会先更新到内存，然后将规则写入到文件中.
        WritableDataSourceRegistry.registerFlowDataSource(wds);
    }

    private <T> String encodeJson(T t) {
        return JSON.toJSONString(t);
    }
}
```

本地文件数据源会定时轮询文件的变更，读取规则。这样我们既可以在应用本地直接修改文件来更新规则，也可以通过 Sentinel 控制台推送规则。以本地文件数据源为例，推送过程如下图所示：

![Push rules from Sentinel Dashboard to local file](https://cdn.nlark.com/lark/0/2018/png/47688/1536660311826-addf4ff6-9fc9-4586-ba8b-4caf3a91457d.png) 

首先 Sentinel 控制台通过 API 将规则推送至客户端并更新到内存中，接着注册的写数据源会将新的规则保存到本地的文件中。使用 pull 模式的数据源时一般不需要对 Sentinel 控制台进行改造。

### push 模式的数据源

对于 push 模式的数据源（如远程配置中心），推送的操作不应由 Sentinel 数据源进行，而应该经控制台进行推送，数据源仅负责获取配置中心推送的配置并更新到本地。

假设写入的操作也由数据源进行，那么 Sentinel 客户端收到控制台推送的规则后，将新的规则更新到内存中，同时将规则推送至远程的配置中心。此时，数据源监听到配置中心推送过来的新规则，又一次更新到内存中。也就是说应用在本地更新完规则并推送到远程后，又要接收变更并更新一次，这样显然是不合理的。因此推送规则正确做法应该是 **配置中心控制台/Sentinel 控制台 → 配置中心 → Sentinel 数据源 → Sentinel**，而不是经 Sentinel 数据源推送至配置中心。这样的流程就非常清晰了：

![Remote push rules to config center](https://cdn.nlark.com/lark/0/2018/png/47688/1536660393347-c5bc2ad6-0d00-4871-8b9b-388f437611ef.png) 

注意由于不同的生产环境可能使用不同的数据源，从 Sentinel 控制台推送至配置中心的实现需要用户自行改造。以 ZooKeeper 为例，我们可以按照如下步骤进行改造（假设推送维度为应用维度）：

1. 实现一个公共的 ZooKeeper 客户端用于推送规则，在 Sentinel 控制台配置项中需要指定 ZooKeeper 的地址，启动时即创建 ZooKeeper Client。
2. 我们需要针对每个应用（appName），每种规则设置不同的 path（可随时修改）；或者约定大于配置（如 path 的模式统一为 `/sentinel_rules/{appName}/{ruleType}`，e.g. `sentinel_rules/appA/flowRule`）。
3. 规则配置页需要进行相应的改造，直接针对**应用维度**进行规则配置；修改同个应用多个资源的规则时可以批量进行推送，也可以分别推送。Sentinel 控制台将规则缓存在内存中（如 `InMemFlowRuleStore`），可以对其进行改造使其支持应用维度的规则缓存（key 为 appName），每次添加/修改/删除规则都先更新内存中的规则缓存，然后需要推送的时候从规则缓存中获取全量规则，然后通过上面实现的 Client 将规则推送到 ZooKeeper 即可。
4. 应用客户端需要注册对应的读数据源以监听变更，可以参考 [相关文档](https://github.com/alibaba/Sentinel/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%99%E6%89%A9%E5%B1%95)。

## 监控数据持久化

Sentinel 会记录资源访问的秒级数据（若没有访问则不进行记录）并保存在本地日志中，具体格式请见 [秒级监控日志文档](https://github.com/alibaba/Sentinel/wiki/%E6%97%A5%E5%BF%97#%E7%A7%92%E7%BA%A7%E7%9B%91%E6%8E%A7%E6%97%A5%E5%BF%97)。Sentinel 控制台通过 [Sentinel 客户端预留的 API](https://github.com/alibaba/Sentinel/wiki/%E5%AE%9E%E6%97%B6%E7%9B%91%E6%8E%A7#%E5%AE%9E%E6%97%B6%E6%9F%A5%E8%AF%A2) 从秒级监控日志中拉取监控数据，并进行聚合。目前 Sentinel 控制台中监控数据聚合后直接存在内存中，未进行持久化，且仅保留最近 5 分钟的监控数据。若需要监控数据持久化的功能，可以自行扩展实现 `MetricsRepository` 接口（0.2.0 版本），然后注册成 Spring Bean 并在相应位置通过 `@Qualifier` 注解指定对应的 bean name 即可。`MetricsRepository` 接口定义了以下功能：

- `save` 与 `saveAll`：存储对应的监控数据
- `queryByAppAndResourceBetween`：查询某段时间内的某个应用的某个资源的监控数据
- `listResourcesOfApp`：查询某个应用下的所有资源

其中默认的监控数据类型为 `MetricEntity`，包含应用名称、时间戳、资源名称、异常数、请求通过数、请求 block 数、平均响应时间等信息。

同时用户可以自行进行扩展，适配 Grafana 等可视化平台，以便将监控数据更好地进行可视化。

## 其它

在生产环境中使用 Sentinel 控制台还需要考虑以下问题：

- 权限控制：生产环境下的权限控制是非常重要的，理论上只有 AppOps 或管理员才有权限去修改对应应用的规则。Sentinel 控制台不提供权限控制功能，需要开发者自行进行改造。

同时也可以到 [Awesome Sentinel](https://github.com/alibaba/sentinel-awesome) 去参考社区用户的一些扩展和解决方案，也欢迎大家将一些比较好的扩展实现添加进来。