When an entry is created, a series of slots are created as well. These slots have different responsibilities, some for tracing, some for collecting and calculating run-time information, some for controlling flow, some for authorization, and so on. You can customize your own logic by adding slots based on the following 3 fundamental slots.

<img src="https://github.com/alibaba/Sentinel/blob/master/doc/image/slots.gif" width="600" />

##  NodeSelectorSlot: Trace nodes and form a calling tree in the memory
```java
 ContextUtil.enter("entrance1", "appA", null, EntryType.IN);
 Entry nodeA = SphU.entry("nodeA");
 if (nodeA != null) {
    nodeA.exit();
 }
 ContextUtil.exit();
``` 
The code above will generate the following structure in memory:
```
 	     machine-root
                 /     
                /
         EntranceNode1
              /
             /   
      DefaultNode(nodeA)
```
Each DefaultNode is identified by both the resource id and entery name. In other words, one resource id may have multiple default nodes distinguished by the entery names declared in ContextUtil.enter(name)
```java
  ContextUtil.enter("entrance1", "appA", null, EntryType.IN);
  Entry nodeA = SphU.entry("nodeA");
  if (nodeA != null) {
    nodeA.exit();
  }
  ContextUtil.exit();
  ContextUtil.enter("entrance2", "appA", null, EntryType.IN);
  nodeA = SphU.entry("nodeA");
  if (nodeA != null) {
    nodeA.exit();
  }
  ContextUtil.exit();

``` 
The code above will generate the following structure in memory:
``` 
                   machine-root
                   /         \
                  /           \
          EntranceNode1   EntranceNode2
                /               \
               /                 \
       DefaultNode(nodeA)   DefaultNode(nodeA)
``` 
The calling trace can be displayed by calling `curl http://localhost:8719/tree?type=root`.
``` 
EntranceNode: machine-root(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
-EntranceNode1: Entrance1(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
--nodeA(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
-EntranceNode2: Entrance1(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)
--nodeA(t:0 pq:1 bq:0 tq:1 rt:0 prq:1 1mp:0 1mb:0 1mt:0)

t:threadNum  pq:passQps  bq:blockedQps  tq:totalQps  rt:averageRt  prq: passRequestQps 1mp:1m-passed 1mb:1m-blocked 1mt:1m-total
``` 

##  ClusterNodeBuilderSlot: Build cluster node named by resource
This slot maintains resource runtime statistics (response time, QPS, thread, count, exception), and a list of origin callers' statistics. The origin caller's name is marked by {@code Context.enter(entryname, origin)}.  


The information can be displayed by calling the following URL: 
`curl http://localhost:8719/origin?id=xxxx` 
``` 
id: nodeA
idx origin  threadNum passedQps blockedQps totalQps aRt   1m-passed 1m-blocked 1m-total 
1   caller1 0         0         0          0        0     0         0          0        
2   caller2 0         0         0          0        0     0         0          0        
```                        

## StatisticSlot: Collect runtime statistics 
 - Cluster node: Total statistics of a cluster node of the resource ID;  
 - Origin node: Statistics of a cluster node from different callers;   
 - Default node: The entry of the node, marked by context entry name and resource ID;
 - The total sum statistics of incoming entrances.
