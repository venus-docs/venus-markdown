## 1 Introduction

Requests arriving at pulse may drag down long idle systems even though it can handle much more requests in stable period. This situation usually happens in scenarios that require extra time to initialize, for example, db establishes a connection; connects to a remote service, and so on.

That’s why we need “warm up”.

Sentinel's "warm-up" implementation is based on the [Guava](https://github.com/google/guava/blob/master/guava/src/com/google/common/util/concurrent/SmoothRateLimiter.java)'s  algorithm. However, Guava’s implementation focus on adjusting the request interval, in other words, a "Leaky bucket". Sentinel pays more attention to controlling the count of incoming requests per second without calculating its interval, it is more like a “Token bucket.”

The count of remaining tokens in the bucket is used to measure the system utility. Suppose a system can handle b requests per second, then every second b tokens will be added into the bucket until the bucket is full. When system processes a request, it takes a token out from the bucket. The more tokens left in the bucket, the lower the utilization of the system; when the token in the token bucket is above a certain threshold, we call it in a "saturation" state.

Base on Guava’s theory, there is a linear equation we can write this in the form y = m * x + b where y (a.k.a y(x)), or qps(q)), is our expected QPS given a saturated period (e.g. 3 minutes in), m is the rate of change from our cold (minimum) rate to our stable (maximum) rate, x (or q) is the occupied token.

## 2 Demo
[Demo](https://github.com/alibaba/Sentinel/blob/master/demo/src/main/java/com/alibaba/csp/sentinel/demo/flow/WarmUpFlowDemo.java) is a demo of "warm up".

1. `warmupFlowRule()` sets up a warm up rule
```
    rule1.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_WARM_UP);
    rule1.setWarmUpPeriodSec(10); //times the system needs to warm up 
```

2. 'coldSystem()'  simulates the cold system by setting 'qpsInColdPeriod'。

3. 'simulateTraffic()' simulates a large amount of requests

4. the out put will be like:
```
Large amount of traffic is coming
88 send qps is: 2061
1528883307808,total:2061, pass:9, block:2053
87 send qps is: 3699
1528883308808,total:3699, pass:7, block:3692
86 send qps is: 3898
1528883309808,total:3898, pass:7, block:3893
85 send qps is: 3713
1528883310808,total:3713, pass:7, block:3708
84 send qps is: 3756
1528883311808,total:3756, pass:8, block:3749
83 send qps is: 3750
1528883312808,total:3750, pass:9, block:3741
82 send qps is: 3492
1528883313806,total:3492, pass:10, block:3482
81 send qps is: 3923
1528883314808,total:3923, pass:11, block:3913
80 send qps is: 3176
1528883315820,total:3176, pass:13, block:3163
79 send qps is: 3729
1528883316821,total:3729, pass:22, block:3708
78 send qps is: 3534
1528883317820,total:3534, pass:20, block:3514
```

to the 10th second, the system becomes stable, and allow 20 requests to pass per second


