## Interception logs

All interception logs will be recorded in `{user_home}/logs/csp/sentinel-block.log`:
```
2014-06-20 16:35:10|1|sayHello(java.lang.String,long),FlowException,default,origin|61,0
2014-06-20 16:35:11|1|sayHello(java.lang.String,long),FlowException,default,origin|1,0
```
1. `2014-06-20 16:35:10`，timestamp；
2. `1`，index；
3. `sayHello(java.lang.String,long)`，resource；
4. `XXXException`，what kinds of rules。`FlowException` for flow control，`DegradeException` for circuit break，`SystemException` for system protection
5. `default` for callers defined in rules；
6. `origin`，for the real caller of the request；
7. `61,0`，61 for block times，0 has no meaning, can be ignored

## Metrics log 

Metrics of resources are recorded in `{user_home}/logs/csp/{app_name}_{pid}_metrics.log`:
```
1529573107000|2018-06-21 17:25:07|sayHello(java.lang.String,long)|10|3601|10|0|2
```
1. `1529573107000`，timestamp；
2. `2018-06-21 17:25:07`，format datetime；
3. `sayHello(java.lang.String,long)`，resource；
4. `10`，incoming request count；
5. `3601`，blocked count；
6. `10`，success handled count；
7. `0`，biz exception count；
8. `2`，average response time(ms)。

## Other logs

Other info is recorded in `{user_home}/logs/csp/record.log`.