<img src="https://user-images.githubusercontent.com/9434884/43697219-3cb4ef3a-9975-11e8-9a9c-73f4f537442d.png" alt="Sentinel Logo" height="40%" width="40%">

# Sentinel: The Flow Sentinel of Your App

# What Is Sentinel?

As distributed systems becoming increasingly popular, the stability between services is becoming more important than ever before. Sentinel is a library that takes "flow" as the breakthrough point, and covers multiple fields including flow control, concurrency, circuit breaking, and load protection to protect service stability.

## History of Sentinel
* 2012, Sentinel was first used in Alibaba with the main purpose of flow control.
* 2013-2017, Sentinel grew fast and became a basic component for all applications in Alibaba. It was used in more than 4000 applications and covers almost all core e-commerce scenarios. 
* 2018，Sentinel evolves into an open source project.

# Key Concepts

## Resource
Resource is a key concept in Sentinel. It can be anything, such as a service, a method, or even a code snippet.

Once it is wrapped by Sentinel API, it is a defined as a resource and can apply for protections provided by Sentinel.

## Rules
The way Sentinel protects resources is defined by rules, such as flow control, concurrency, and circuit breaking rules. Rules can be dynamically changed, and take effect in real time.

# Features and Principles 
## Flow Control
Sentinel can enable applications to handle random incoming requests according to the appropriate shape as needed, as illustrated below:
<img src="https://github.com/alibaba/Sentinel/wiki/image/limitflow.gif" width="640" height="360" />

## Principles of Flow Control
Flow control is based on the following statistics:
* Calling path between resources;
* Runtime metrics such as QPS, thread pool, and system load;
* Desired actions to take, such as reject or queue.

Sentinel allows applications to combine all these statistics in a flexible manner. 

## Circuit Breaking and Concurrency

Circuit breaking is used to detect failures and encapsulates the logic of preventing a failure from constantly reoccurring during maintenance, temporary external system failure or unexpected system difficulties. 

To tackle this problem, [Netflix](https://github.com/Netflix/Hystrix/wiki#what-problem-does-hystrix-solve) chose to use threads and thread-pools to achieve isolation. The main benefit of thread pools is that the isolation is clean.

Sentinel uses the following principles to implement circuit breaking:

### Control max concurrency threads
Instead of using thread pools, Sentinel reduces the impact of unstable resources by restricting the number of concurrent threads.

When the response time of a resource becomes longer, threads will start to be occupied. When the number of threads accumulates to a certain amount, new incoming requests will be rejected. Vice versa, when the resource restores and becomes stable, the occupied thread will be released as well, and new requests will be accepted. 

By restricting concurrent threads instead of thread pools, you no longer need to pre-allocate the size of the thread pools and can thus avoid the computational overhead such as the queueing, scheduling, and context switching.

### Degradation 
Besides restricting the concurrent threads, downgrade unstable resources according to their response time is also an effective way to implement circuit breaking. When the response time of a resource is too long, all access to the resource will be rejected in the specified time window.

## Load Protection
Sentinel can be used to protect your server in case the system load goes too high. It helps you achieve a good balance between system load and incoming requests and the system can be restored very quickly.

# How Sentinel works
* Provides APIs and adaptations to popular frameworks to define resources;
* Collects runtime information and traces the call path of resources;
* Defines rules in real time;
* Provides real-time monitoring and extension rules configuration.


# Index

* [How to use](https://github.com/alibaba/Sentinel/wiki/How-to-Use)
* [How it works](https://github.com/alibaba/Sentinel/wiki/How-it-works)
* [Flow Control](https://github.com/alibaba/Sentinel/wiki/Flow-Control)
* [Circuit Breaking](https://github.com/alibaba/Sentinel/wiki/Circuit-Breaking)
* [System Protection](https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%B4%9F%E8%BD%BD%E4%BF%9D%E6%8A%A4)
* [Metrics](https://github.com/alibaba/Sentinel/wiki/Metrics)
* [Dashboard](https://github.com/alibaba/Sentinel/wiki/Dashboard)
* [Dynamic Rules](https://github.com/alibaba/Sentinel/wiki/Dynamic-Rule-Configuration)
* [Adapters to Popular Framework](https://github.com/alibaba/Sentinel/wiki/Adapters-to-Popular-Framework)
* [Logs](https://github.com/alibaba/Sentinel/wiki/Logs)


