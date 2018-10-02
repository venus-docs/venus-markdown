Sentinel provides real-time metrics, including resource count, response time, concurrency, etc. You can view these metrics on [Sentinel-Dashboard](https://github.com/alibaba/Sentinel/blob/master/sentinel-dashboard/README.md), but you can also use the metrics API to retrieve data.

### Metrics for Resources

1.After an application starts, you can run the following command to fetch the resource list in JSON format.

  `curl http://localhost:8719/clusterNode`

  ```javascript
[
 {"avgRt":0.0, //avg response time per second
 "blockRequest":0, //block count per minute
 "blockedQps":0.0, //block count per second
 "curThreadNum":0, //current concurrency
 "passQps":1.0, // success exit count per second
 "passReqQps":1.0, //pass count per second
 "resourceName":"/registry/machine", resource name
 "timeStamp":1529905824134, //time stamp
 "totalQps":1.0, // total request count per minute
 "totalRequest":193}, 
  ....
]

```

2.Display resource information by resource name

  Run command `curl http://localhost:8719/cnode?id=xxxx` to query a specified resource name. The parameter id corresponds to the resource name. Fuzzy query is supported.

  `$curl http://11.138.222.145:8719/cnode?id=machine`
  ``` 
idx id                                thread    pass      blocked   success    total    aRt   1m-pass   1m-block   1m-all   exeption   
6   /app/aliswitch2/machines.json     0         0         0         0          0        0     0         0          0        0          
7   /app/sentinel-admin/machines.json 0         1         0         1          1        6     0         0          0        0          
8   /identity/machine.json            0         0         0         0          0        0     0         0          0        0          
9   /registry/machine                 0         2         0         2          2        1     192       0          192      0          
10  /app/views/machine.html           0         1         0         1          1        2     0         0          0        0   

```        

3.Display resources' origin info

  `curl http://localhost:8719/origin?id=xxxx`

  ``` 
id: nodeA
idx origin  threadNum passedQps blockedQps totalQps aRt   1m-passed 1m-blocked 1m-total 
1   caller1 0         0         0          0        0     0         0          0        
2   caller2 0         0         0          0        0     0         0          0      
``` 
  Among which, `origin` is defined by calling the following method:
  ```java
Context.enter(resourceName，origin) 
```

### Resource Path 

Any resource which calls the following code explicitly will be regarded as an entranceNode, which is the child of root `machine-root`: 

```java
Context.enter(resourceName，origin) 
```
Run command `curl http://localhost:8719/tree` to display the calling path in a tree form:
``` 
EntranceNode: machine-root(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
-EntranceNode1: Entrance1(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
--nodeA(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
-EntranceNode2: Entrance1(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
--nodeA(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)

t:threadNum  pq:passQps  bq:blockedQps  tq:totalQps  rt:averageRt  prq: passRequestQps 1mp:1m-passed 1mb:1m-blocked 1mt:1m-total
```

### History Resource Metrics

1.Resources metrics log

Resource metrics log is generated in the directory: `${home}\logs\csp\${appName}-${pid}-metrics.log.${date}.xx`. For example, log file name would be: `app-3518-metrics.log.2018-06-22.1`

```
1529573107000|2018-06-21 17:25:07|sayHello(java.lang.String,long)|10|3601|10|0|2
```

| index| example| description|
| :--- | :----: | :---- |
|1| `1529573107000`|time stamp|
|2| `2018-06-21 17:25:07`|date|
|3| `sayHello(java.lang.String,long)`|resource name|
|4| `10`|the count of pass resource per second|
|5| `3601`|the count of blocked resource per second|
|6| `10`|the count of exit resource per second|
|7| `0`|the count of exception per second|
|8| `2`|the avg response time|


2.Block log

Also, the block log will be generated in the dir `${home}\logs\csp\sentinel-block.log`. If no block happens, the log will not be generated.

```
2014-06-20 16:35:10|1|sayHello(java.lang.String,long),FlowException,default,origin|61,0
2014-06-20 16:35:11|1|sayHello(java.lang.String,long),FlowException,default,origin|1,0
```
| Index| Example| Description|
| :--- | :----: | :---- |
|1|`2014-06-20 16:35:10`|timestamp|
|2|`1`|the resource index|
|3| `sayHello(java.lang.String,long)|resource name|
|4| `XXXException`| `FlowException` is blocked by flow rules, `DegradeException` is blocked by degradation rules，`SystemException`is blocked by system rules|
|5| `default`|the limit app defined in effective rule|
|6| `origin`|the origin of the blocked resource. could be empty|
|7| `61,0`|61 the blocked count this second. and 0 can be ignored|

3.Real-time query
```shell
curl http://localhost:8719/metric?startTime=XXXX&endTime=XXXX
```
  [History Resource Metrics](https://github.com/alibaba/Sentinel/wiki/Metrics#history-resource-metrics)

  ```
1529998904000|2018-06-26 15:41:44|abc|100|0|0|0|0
1529998905000|2018-06-26 15:41:45|abc|4|5579|104|0|728
1529998906000|2018-06-26 15:41:46|abc|0|15698|0|0|0
1529998907000|2018-06-26 15:41:47|abc|0|19262|0|0|0
1529998908000|2018-06-26 15:41:48|abc|0|19502|0|0|0
1529998909000|2018-06-26 15:41:49|abc|0|18386|0|0|0
1529998910000|2018-06-26 15:41:50|abc|0|19189|0|0|0
1529998911000|2018-06-26 15:41:51|abc|0|16543|0|0|0
1529998912000|2018-06-26 15:41:52|abc|0|18471|0|0|0
1529998913000|2018-06-26 15:41:53|abc|0|19405|0|0|0
```