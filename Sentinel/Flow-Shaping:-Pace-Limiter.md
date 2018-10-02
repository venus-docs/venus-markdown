## Introduction

If RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER is set, request will pass at a fixed rate. 

When a new request arrives, the flow rule checks whether the interval between the new request and the previous request. If the interval is longer than the value in the rule, the request can pass; otherwise, sentinel will calculate the waiting time for this request. And if the request has to wait longer than the “timeout” value in the rule, the request will be rejected immediately.

Pace limiter is widely used for pulsed flow. When a large amount of traffic arrives, the system does not process all requests at once but handles these requests steadily in the following seconds.

## Demo

(https://github.com/alibaba/Sentinel/blob/master/demo/src/main/java/com/alibaba/csp/sentinel/demo/flow/WarmUpFlowDemo.java) is an example of how to use pace control.

1. `paceFlowRule()` will construct a flow rule using pacing control

```
rule1.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER);
rule1.setCount(10);
rule1.setMaxQueueingTimeMs(20*1000);
```
Note: Pace limiter can be only used in the rules using 'RuleConstant.GRADE_QPS`, otherwise the rule will not take effect. 
Count is set to 10, which means that 10 requests are allowed to pass at the maximum per second,  the average interval of request is 1000/10=100ms; and the time out is set to 20*1000ms, 20 seconds.

2. `simulatePulseFlow()` simulates 100 request arriving at almost the same time

3. output

```
seperated by ",", the first column is the milli seconds, and the second column is the waiting time
....
1528872403887 pass, cost 9348
1528872403986 pass, cost 9469
1528872404087 pass, cost 9570
1528872404187 pass, cost 9642
1528872404287 pass, cost 9770
1528872404387 pass, cost 9848
1528872404487 pass, cost 9970
done
pass:100 block:0
```
All these 100 requests are passed at a fixed interval of 100 ms without blocking

4. Compared with the default behavior
By setting RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER` in `defaultFlowRule()'，10 requests are allowed to pass。