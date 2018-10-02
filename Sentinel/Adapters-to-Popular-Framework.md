## Contents

- [Web Servlet](#web-servlet)
- [Dubbo](#dubbo)
- [Spring Boot / Spring Cloud](#spring-cloud)
- [gRPC](#grpc)
- [Apache RocketMQ](#apache-rocketmq)

## Web Servlet

Sentinel provides Servlet filter integration to enable flow control for web requests. Add the following dependency in `pom.xml` (if you are using Maven):

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-web-servlet</artifactId>
    <version>x.y.z</version>
</dependency>
```

To use the filter, you can simply configure your `web.xml` with:

```xml
<filter>
	<filter-name>SentinelCommonFilter</filter-name>
	<filter-class>com.alibaba.csp.sentinel.adapter.servlet.CommonFilter</filter-class>
</filter>

<filter-mapping>
	<filter-name>SentinelCommonFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

When a request is blocked, Sentinel servlet filter will give a default page indicating the request blocked.
If customized block page is set (via `WebServletConfig.setBlockPage(blockPage)` method),
the filter will redirect the request to provided URL. You can also implement your own
block handler (the `UrlBlockHandler` interface) and register to `WebCallbackManager`.

## Dubbo

[Sentinel Dubbo Adapter](https://github.com/dubbo/dubbo-sentinel-support) provides service consumer filter and provider filter for Dubbo services. Add the following dependency in `pom.xml` (if you are using Maven):

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-dubbo-adapter</artifactId>
    <version>x.y.z</version>
</dependency>
```

The two filters are enabled by default. Once you add the dependency, the Dubbo services and methods will become protected resources in Sentinel, which can leverage Sentinel's flow control and guard ability when rules are configured. 

If you don't want to enable the filter, you can manually disable it. For example:

```xml
<!-- disable the Sentinel Dubbo consumer filter -->
<dubbo:consumer filter="-sentinel.dubbo.consumer.filter"/>
```

The guarded resource can be both service interface and service method:

- Service interface：resourceName format is `interfaceName`，e.g. `com.alibaba.csp.sentinel.demo.dubbo.FooService`
- Service method：resourceName format is `interfaceName:methodSignature`，e.g. `com.alibaba.csp.sentinel.demo.dubbo.FooService:sayHello(java.lang.String)`

Since version 0.1.1, Sentinel Dubbo Adapter supports global fallback configuration.
The global fallback will handle exceptions and give replacement result when blocked by
flow control, degrade or system load protection. You can implement your own `DubboFallback` interface
and then register to `DubboFallbackRegistry`. If no fallback is configured, Sentinel will wrap the `BlockException`
then directly throw it out. Besides, we can also leverage [Dubbo mock mechanism](http://dubbo.apache.org/#!/docs/user/demos/local-mock.md?lang=en-us) to provide fallback implementation of degraded Dubbo services.

For Sentinel's best practice in Dubbo, please refer to [Sentinel: the flow sentinel of Dubbo](http://dubbo.incubator.apache.org/#!/blog/sentinel-introduction-for-dubbo.md?lang=en-us).

For more details of Dubbo filter, see [here](https://dubbo.incubator.apache.org/#/docs/dev/impls/filter.md?lang=en-us).

## Spring Cloud

Sentinel Spring Cloud Starter provides out-of-box integration with Sentinel for Spring Cloud applications and services.

Please refer to [Spring Cloud Alibaba](https://github.com/spring-cloud-incubator/spring-cloud-alibabacloud).

## gRPC

Sentinel provides integration with [gRPC Java](https://github.com/grpc/grpc-java). Sentinel gRPC Adapter provides client and server interceptor for gRPC services. Add the following dependency in `pom.xml` (if you are using Maven):

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-grpc-adapter</artifactId>
    <version>x.y.z</version>
</dependency>
```

To use Sentinel gRPC Adapter, you simply need to register the `Interceptor` to your client or server. The client sample:

```java
public class ServiceClient {

    private final ManagedChannel channel;

    ServiceClient(String host, int port) {
        this.channel = ManagedChannelBuilder.forAddress(host, port)
            .intercept(new SentinelGrpcClientInterceptor()) // Add the client interceptor.
            .build();
        // Init your stub here.
    }
}
```

The server sample；

```java
import io.grpc.Server;

Server server = ServerBuilder.forPort(port)
     .addService(new MyServiceImpl()) // Add your service.
     .intercept(new SentinelGrpcServerInterceptor()) // Add the server interceptor.
     .build();
```

> Note that currently the interceptor only supports unary methods in gRPC.
  In some circumstances (e.g. asynchronous call), the RT metrics might not be accurate.

## Apache RocketMQ

In Apache RocketMQ, when message consumers are consuming messages, there may a sudden inflow of messages, whether using pull or push mode. If all the messages were handled at this time, it would be likely to cause the system to be overloaded and then affect stability. However, in fact, there may be no messages coming within a few seconds. If redundant messages are directly discarded, the system's ability to process the message is not fully utilized. We hope that the sudden inflow of messages can be spread over a period of time, so that the system load can be kept on the stable level while processing as many messages as possible, thus achieving the effect of “shaving the peaks and filling the valley”.

![shaving the peaks and filling the valley](https://github.com/alibaba/Sentinel/wiki/image/mq-traffic-peak-clipping-en.png) 

Sentinel provides a feature for this kind of scenario: [Rate Limiter](https://github.com/alibaba/Sentinel/wiki/Flow-Shaping:-Pace-Limiter), which can spread a large number of sudden request inflow in a uniform rate manner, let the request pass at a fixed interval. It is often used to process burst requests instead of rejecting them. This avoids traffic spurs causing system overloaded. Moreover, the pending requests will be queued and processed one by one. When the request is estimated to exceed the maximum queuing timeout, it will be rejected immediately.

For example, we configure the rule with uniform rate limiting mode and QPS count is 5, which indicates messages are consumed at fixed interval (200 ms) and pending messages will wait (virtual queue). We also set the maximum queuing timeout is 5s, then all requests estimated to exceed the timeout will be rejected immediately.

![Uniform rate](https://github.com/alibaba/Sentinel/wiki/image/uniform-speed-queue.png)

Developer can set rules to different groups and topics (e.g. resouceName is `groupName:topicName`), set the control behaviour to `RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER`, then the messages can be handled at a fixed rate. Here is an example for the rule:

```java
private void initFlowControlRule() {
    FlowRule rule = new FlowRule();
    rule.setResource(KEY); // resource name can be `groupName:topicName`
    rule.setCount(5); // Indicates the interval between two adjacent requests is 200 ms.
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    rule.setLimitApp("default");

    // Enable rate limiting (uniform). This can ensure fixed intervals between two adjacent calls.
	// In this example, intervals between two incoming calls (message consumption) will be 200 ms constantly.
    rule.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER);
    // If more requests are coming, they'll be put into the waiting queue.
    // In this example, the max timeout is 5s.
    rule.setMaxQueueingTimeMs(5 * 1000);
    FlowRuleManager.loadRules(Collections.singletonList(rule));
}
```

When using Sentinel with RocketMQ Client, developers should manually wrap their code of handling messages with Sentinel API. Here is a sample for pull consumer: [Sentinel RocketMQ Demo](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo/sentinel-demo-rocketmq).