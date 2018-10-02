
* [Introduction](https://github.com/alibaba/Sentinel/wiki/How-to-Use#introduction)
* [Define Resource](https://github.com/alibaba/Sentinel/wiki/How-to-Use#define-resource)
  * ["try" and "catch" Mode](https://github.com/alibaba/Sentinel/wiki/How-to-Use#try-and-catch-mode)
  * [Bool Mode](https://github.com/alibaba/Sentinel/wiki/How-to-Use#bool-mode)

* [Define Rules](https://github.com/alibaba/Sentinel/wiki/How-to-Use#define-rules)
  * [Definitions of Rule](https://github.com/alibaba/Sentinel/wiki/How-to-Use#definitions-of-rule)
  * [HTTP Commands for Rules](https://github.com/alibaba/Sentinel/wiki/How-to-Use#http-commands-for-rules)
  * [Integrate with rule repositories](https://github.com/alibaba/Sentinel/wiki/How-to-Use#integrate-with-rule-repositories)


# Introduction

To use Sentinel, you only need to complete 2 steps:
1. Define resources
2. Define rules

These two steps don’t have to be synchronized. As long as the resources are defined, you can add rules as needed. Multiple rules can be applied to the same resource simultaneously.

Sentinel provides adaptions to popular frameworks as well. After introducing these adapters, services and methods provided by these frameworks are defined as resources by default. 

# Define Resource
You can use one of the following two methods to define resources.
###  "try" and "catch" mode

```java
Entry entry = null;
try {
    entry = SphU.entry(KEY);

    // Your business logic here.
} catch (BlockException ex) {
    // Resource is rejected.

    // Here to handle the block exception
} finally {
    if (entry != null) {
        entry.exit();
    }
}
```
### Bool Mode

```java
  if(SphO.entry("helloworld")){
    try {
      /**
      * code logic
      */
    } finally {
      SphO.exit();
    }
  }else{
    //resource is rejected
    /**
    * logic to handle exception
    */
  }
}
```

### Adapters to Popular Frameworks

For details, please refer to [Adapters to popular frameworks](https://github.com/alibaba/Sentinel/wiki/%E4%B8%BB%E6%B5%81%E6%A1%86%E6%9E%B6%E7%9A%84%E9%80%82%E9%85%8D).

# Define Rules
Sentinel provides APIs for you to modify your rules, which can be integrated with various kinds of rule repository, such as configuration server and SQL.

## Definition of Rules

There are 3 types of rules：**flow control rules**, **circuit breaking rules**, and **system protection rules**.

### Flow control rules (FlowRule)

#### Definition:
key fields：

| Field| Description|Default value|
| :----: | :----| :----|
| resource| resource name, defined in entry() ||
| count| threshold ||
| grade| calculated by QPS or concurrent thread count |QPS|
| limitApp| refer to specified caller|default|
| strategy| by resource itself or other resource (refResource)，or entry (refResource)|resource itself|
| controlBehavior| reject directly，queue，slow start up|reject directly|

Multiple rules can be applied to the same resource.

#### API
API:FlowRuleManager.loadRules() can be used to configure flow control rules.

```java
private static void initFlowQpsRule() {
        List<FlowRule> rules = new ArrayList<FlowRule>();
        FlowRule rule1 = new FlowRule();
        rule1.setResource(KEY);
        // set limit qps to 20
        rule1.setCount(20);
        rule1.setGrade(RuleConstant.FLOW_GRADE_QPS);
        rule1.setLimitApp("default");
        rules.add(rule1);
        FlowRuleManager.loadRules(rules);
    }
```

For more details please refer to [Flow control](https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6).

### Circuit breaking rules (DegradeRule)

Key fields:

| Field| Description|Default value
| :----: | :----| :----|
| resource| resource name ||
| count| threshold ||
| grade| by response time or exception ratio |response time|
| timeWindow| degrade time window||

Multiple rules can be applied to the same resource.

### API
API:DegradeRuleManager.loadRules() can be used to configure degradation rules.

```java
 private static void initDegradeRule() {
        List<DegradeRule> rules = new ArrayList<DegradeRule>();
        DegradeRule rule = new DegradeRule();
        rule.setResource(KEY);
        // set threshold rt, 10 ms
        rule.setCount(10);
        rule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
        rule.setTimeWindow(10);
        rules.add(rule);
        DegradeRuleManager.loadRules(rules);
    }
```
For more details, please refer to [Circuit Break](https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7).

### System protection rules (SystemRule)

Key factors

| Field| Description |Default value
| :----: | :----| :----|
| highestSystemLoad | threshold of Load1 |-1(not valid)|
| avgRt | average response time  |-1(not valid)|
| maxThread |concurrent thread count |-1(not valid)|

### API
API:SystemRuleManager.loadRules()

```java
 private static void initDegradeRule() {
        List<SystemRule> rules = new ArrayList<SystemRule>();
        SystemRule rule = new SystemRule();
        rule.setHighestSystemLoad(10);
        rules.add(rule);
        DegradeRuleManager.loadRules(rules);
    }
```
For more details, please refer to [System Protection](https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%B4%9F%E8%BD%BD%E4%BF%9D%E6%8A%A4).


### HTTP commands for rules
You can also use HTTP commands to configure, query and update Sentinel rules. 

To use these commands, make sure that the following library has been introduced:

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>x.y.z</version>
</dependency>
```

#### Query command

```shell
curl http://localhost:8719/getRules?type=<XXXX>
```

- `type=flow` for flow rules;
- `type=degrade` for circuit breaking rules;
- `type=system` for system protection rules. 

Rules will be returned in JSON format.

#### Modification command

```shell
curl http://localhost:8719/setRules?type=<XXXX>&data=<DATA>
```


## Integrate with rule repositories

[DataSource](https://github.com/alibaba/Sentinel/blob/master/sentinel-extension/src/main/java/com/alibaba/csp/sentinel/datasource/AbstractDataSource.java) is designed to integrate rules to customized repositories and make rules persistent. 

For more details, you can refer to  [Dynamic Rules](https://github.com/alibaba/Sentinel/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%99%E6%89%A9%E5%B1%95).
