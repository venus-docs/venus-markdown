All the rules can be queried or modified in memory and these modifications take effects immediately. Sentinel also provides APIs for you to define your own rules.


### Rule definitions

There are 3 kinds of rules: flow rules, degradation rules, and system rules.

1.Flow Rules

[Flow Rule](https://github.com/alibaba/Sentinel/wiki/Flow-Control-&-Flow-Shaping) has the following fields：

| Field| Description |Default value|
| :----: | :----| :----|
| resource| resource name ||
| count| thredshold ||
| grade| depends on QPS or concurrency(active thread count) |QPS|
| limitApp| whether to count on callers|no|
| strategy| count on the resource itself, or on the related resource(refResource)，or on entry resource(refResource)|resource itself|
| controlBehavior| how to act after block criteria are met. To block immediately, or wait in a queue, or slow warm up|block immediatley|

Multiple rules can be applied to one resource.

2.Degradation Rules
[Degrade](https://github.com/alibaba/Sentinel/wiki/Degrade) has the following fields：

| Field| Description|Default value
| :----: | :----| :----|
| resource| resource name ||
| count| thredshold||
| grade| down grade o RT or qps |RT|
| timeWindow| timewindow when degradation happens||



3. System Protection Rules

### Query active rules

Run the following command to query current active rules：

`curl http://localhost:8719/getRules?type=? ,type=flow` 

If the type is flow, it will return all flow rules in JSON format; if the type is degrade, it will return all degradation rules in JSON format; and if the type is system, it will return system rules.


### Modify rules

You can modify the following rules via the following command：
 
`curl http://localhost:8719/setRules?type=<XXXX>&data=<DATA>`

Data is the rules in JSON format.

### Customize your persistent rules

All the rules' configuration above are stored in memory, which means that rules will be lost if the application restart. We can implement [DataSource](https://github.com/alibaba/Sentinel/blob/master/sentinel-extension/sentinel-datasource-extension/src/main/java/com/alibaba/csp/sentinel/datasource/DataSource.java) to define persist rule logic.

1. Via [Nacos](https://github.com/alibaba/nacos). You can refer to [TODO] for the sample file.
2. Via SQL, Git, You can refer to [TODO] for the sample file.
3. Via Dashboard,  You can refer to [TODO] for the sample file.
4. Via [File](https://github.com/alibaba/Sentinel/blob/master/sentinel-demo/sentinel-demo-dynamic-file-rule/src/main/java/com/alibaba/csp/sentinel/demo/file/rule/FileDataSourceDemo.java) 

