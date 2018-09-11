1 **源码目录**

| 文件夹      | 说明  |
|-------------|-------|
| assembly    | 构建单一的Jar包 mvn clean -DskipTests package -Pdist或者mvn clean -DskipTests -Pinclude-all package -Pdist |
| bin         | 提供了两个Linux shell脚本carbon-spark-shell和carbon-spark-sql，能够在local mode下快捷体验 |
|build|编译工程的命令，以及生成版本文件的脚本工具|
| common      | 公共模块，目前仅包含日志部分 |
| conf        | 配置文件carbon.properties.template, dataload.properties.template |
| core        | 核心模块, 包含了查询模块代码与一些基础的模型类型和工具类1.core：表、字典、索引等逻辑结构模型，以及字典缓存、MDK生成、数据解压缩、文件读写等2.Scan：实现查询功能，包含查询数据扫描、表达式计算和数据过滤、详单查询行结果收集、查询执行工具类等 |
|datamap|数据地图，主要包括Index DataMap(bloom, lucene, minmax example)和mv DataMap|
| dev         | 开发者工具, Java/scala Code style，findbugs配置文件 |
| docs        | 文档维护 |
| examples    | 可运行的功能演示例子，包括：flink, spark, spark2 |
| format      | CarbonData文件格式定义，使用Apache Thrift定义 |
| hadoop      | Hadoop接口实现，例如：CarbonInputFormat, CarbonRecordReader等 |
| integration | 集成模块1.Spark-common: spark与spark2能重用代码的模块 2.Spark： CarbonContext,sqlparser, optimizer, sparkplan3.Spark2: CarbonSession, CarbonEnv, CarbonScan, CarbonSource4.Spark-common-test：spark与spark2能重用的测试用例 |
|licenses-binary|licenses|
| processing  | 数据加载模块InputProcessorStep: 数据输入接收DataConverteProcessorStep：数据转换(类型转换，字典编码) SortProcessorStep：节点内排序DataWriterProcessorStep：生成Carbondata，Carbonindex文件 |
|store|包括sdk和search mode|
|streaming|流式入库|