## 测试环境

- CPU: Intel(R) Xeon(R) CPU E5-2650 v2 @ 2.60GHz (32 cores)
- OS: Linux 2.6.32-220.23.2.ali927.el5.x86_64

Java 版本：
```
java version "1.8.0_77"
Java(TM) SE Runtime Environment (build 1.8.0_77-b03)
Java HotSpot(TM) 64-Bit Server VM (build 25.77-b03, mixed mode)
```

## 吞吐量对比

所有的吞吐量测试都基于 JMH 编写。

### 单线程吞吐量

JVM 参数：`-Xmn256m -Xmx1024m -XX:+UseConcMarkSweepGC`

测试单线程模式下接入 Sentinel 与不接入 Sentinel 吞吐量的对比。我们通过执行一些 CPU 密集型操作（如小数组排序）来模拟不同 QPS 下的情况，测试逻辑见 [SentinelEntryBenchmark](https://github.com/alibaba/Sentinel/blob/master/sentinel-benchmark/src/main/java/com/alibaba/csp/sentinel/benchmark/SentinelEntryBenchmark.java)。相关结果如下：

| 数组长度 | Baseline (QPS) | With Sentinel (QPS) | 性能损耗 |
| -------- | -------- | -------- | -------- |
| length=25 | 604589.916 | 401687.765 | 33.56% |
| length=50 | 221307.617 | 192407.832 | 13.06% |
| length=100 | 97673.228 | 91535.146 | 6.28% |
| length=200 | 43742.960 | 42711.129 | 2.36% |
| length=500 | 15332.924 | 15171.024 | 1.06% |
| length=1000 | 7012.848 | 6984.036 | 0.41% |

可以看到在单机 QPS 非常大的时候（20W+），Sentinel 带来的性能损耗会比较大。这种情况业务逻辑本身的耗时非常小，而 Sentinel 一系列的统计、检查操作会消耗一定的时间。常见的场景有缓存读取操作。

![benchmark-single-thread-high-qps](https://user-images.githubusercontent.com/9434884/44128904-18baa1f6-a078-11e8-8dab-877afd9266eb.png)

而单机 QPS 在 5W 以下的时候，Sentinel 的性能损耗就比较小了，对大多数场景来说都适用。

![benchmark-single-thread-medium-qps](https://user-images.githubusercontent.com/9434884/44148815-56d2dcb6-a0cc-11e8-8d08-e4bf164735a9.png)

### 多线程吞吐量影响

在相同逻辑（对 length=25 的数组进行 shuffle 并排序）的情况下，测试不同线程数下的 Sentinel entry 的性能表现：

![Multi Thread Single Entry](https://user-images.githubusercontent.com/9434884/44252102-a3cab900-a22d-11e8-8e37-365d2931574d.png)

## 内存占用情况

测试场景：6000 entry 循环跑（目前最多支持 6000 个 entry）

- 单线程不断循环运行：内存占用约 185 MB
- 8 线程不断循环运行：内存占用约 1 GB （并发持续非常高的时候，底层的 LongAdder 内存占用会很高，因其用空间换时间）
