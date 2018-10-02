# 0.2.0

This is a milestone version providing new cool features like **asynchronous invocation support** and **flow control by frequent (hot spot) parameters**.

**Note**: this is the last milestone version that supports JDK 1.6. Since next milestone version, the minimum JDK version will be 1.7.

- Add support for asynchronous invocation entry ([#146](https://github.com/alibaba/Sentinel/pull/146))
- Flow control by frequent (hot spot) parameters ([#156](https://github.com/alibaba/Sentinel/pull/156))
- Support extensible slot chain builder using SPI mechanism ([#145](https://github.com/alibaba/Sentinel/pull/145))
- Refactor data source hierarchy: spilt into readable and writable data source ([#124](https://github.com/alibaba/Sentinel/pull/124))
- Add callback registry for statistic slot for extensions
- Add Redis data source extension ([#102](https://github.com/alibaba/Sentinel/pull/102), [@tigerMoon](https://github.com/tigerMoon))
- Add MetricsRepository interface and reuse original in-memory implementation for Sentinel Dashboard ([#126](https://github.com/alibaba/Sentinel/pull/126))
- Access control for white-list / black-list (AuthorityRule)
- Support JDK-based proxy in Sentinel annotation support ([#111](https://github.com/alibaba/Sentinel/pull/111))
- Enhance exception tracing in Sentinel Dubbo Adapter
- Enhance FileRefreshableDataSource (check file modified) and support FileWritableDataSource ([#125](https://github.com/alibaba/Sentinel/pull/125), [@yfh0918](https://github.com/yfh0918))
- Add RequestOriginParser for Sentinel Web Servlet Integration to support extracting request origin
- Refactor LeapArray to reuse code for current bucket
- Enhance log and null check for rule managers
- Add benchmark for Sentinel
- Extract MetricsReader from MetricSearcher ([#103](https://github.com/alibaba/Sentinel/pull/103), [@refactormachine](https://github.com/refactormachine))
- Fix bugs for Sentinel internal error when context count exceeds MAX_CONTEXT_NAME_SIZE ([#152](https://github.com/alibaba/Sentinel/pull/152))
- Fix bugs for degrade rule checking (fixes [#109](https://github.com/alibaba/Sentinel/issues/109) and [#128](https://github.com/alibaba/Sentinel/issues/128))
- Fix bug for error file separator regex in Windows environment (fixes [#52](https://github.com/alibaba/Sentinel/issues/52))
- Fix bug for missing `maxQueueingTimeMs` in equals and hashCode method of FlowRule (fixes [#99](https://github.com/alibaba/Sentinel/issues/99))
- Fix bug for system rule page in Sentinel Dashboard (fixes [#51](https://github.com/alibaba/Sentinel/issues/51))
- Fix bug for automatic exit of default context when exiting the entry
- Fix bug for probability of metric lose when collecting metrics
- Fix bug for update logic of minRt in metric bucket

Thanks for the contributors: [@kimmking](https://github.com/kimmking), [@refactormachine](https://github.com/refactormachine), [@talshalti](https://github.com/talshalti), [@tigerMoon](https://github.com/tigerMoon), [@yfh0918](https://github.com/yfh0918)

# 0.1.2

This is a bug-fix version, including bug fixes and enhancement:

- Fix bugs for Sentinel internal error when context count exceeds MAX_CONTEXT_NAME_SIZE (fixes [#151](https://github.com/alibaba/Sentinel/issues/151))
- Fix bugs for degrade rule checking (fixes [#109](https://github.com/alibaba/Sentinel/issues/109) and [#128](https://github.com/alibaba/Sentinel/issues/128))
- Fix bug for error file separator regex in Windows environment (fixes [#52](https://github.com/alibaba/Sentinel/issues/52))
- Fix bug for missing `maxQueueingTimeMs` in equals and hashCode method of FlowRule (fixes [#99](https://github.com/alibaba/Sentinel/issues/99))
- Fix bug for system rule page in Sentinel Dashboard (fixes [#51](https://github.com/alibaba/Sentinel/issues/51))
- Fix bug for automatic exit of default context when exiting the entry
- Fix bug for probability of metric lose when collecting metrics
- Fix bug for update logic of minRt in metric bucket
- Support JDK-based proxy in Sentinel annotation support ([#111](https://github.com/alibaba/Sentinel/pull/111))
- Enhance log and null check for rule managers

# 0.1.1

- Add Nacos data source extension ([#8](https://github.com/alibaba/Sentinel/pull/8))
- Use named thread factory for clear identification ([#10](https://github.com/alibaba/Sentinel/pull/10))
- Add ZooKeeper data source extension ([#33](https://github.com/alibaba/Sentinel/pull/33), [@guonanjun](https://github.com/guonanjun))
- Add Sentinel annotation and AspectJ support ([#43](https://github.com/alibaba/Sentinel/pull/43))
- Add default fallback support for Dubbo adapter
- Add Apollo data source extension ([#46](https://github.com/alibaba/Sentinel/pull/46), [@nobodyiam](https://github.com/nobodyiam))
- Optimize for statistic data structures (leap array)
- Enhance error handling when parsing bad dashboard addresses (fixes [#38](https://github.com/alibaba/Sentinel/issues/38))
- Fix bugs about runtime port and heartbeat sending logic (fixes [#20](https://github.com/alibaba/Sentinel/issues/20))
- Refine degrade rule checking and default value of flow rule attributes
- Remove `sentinel-spring-boot-starter` module (replaced by `spring-cloud-starter-sentinel` module in Spring Cloud Alibaba)

Thanks for the contributors: [@guonanjun](https://github.com/guonanjun), [@manzhizhen](https://github.com/manzhizhen), [@nobodyiam](https://github.com/nobodyiam), [@xg1907](https://github.com/xg1907)