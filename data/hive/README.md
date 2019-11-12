# Hive

## Hive 概述

Hive为Hadoop提供了一个SQL接口。 Hive可以被认为是一种编译器，它将SQL（严格来说，Hive查询语言 - HQL，SQL的一种变体）转换为一组MapReduce / Tez / Spark 作业。 因此，Hive非常有助于非程序员使用Hadoop基础架构。 原来，Hive只有一个引擎，即MapReduce。 但是在最新版本中，Hive还支持Spark和Tez作为执行引擎。 这使得Hive成为探索性数据分析的绝佳工具。

基于mapreduce的hive，整个架构图如下：

![](../../.gitbook/assets/data/hive/hive.png)

Driver - 接收查询的组件。 该组件实现了会话句柄的概念，并提供了在JDBC / ODBC接口上的执行和获取数据的api模型。

Compiler - *编译器*解析query的组件，对不同的查询块和查询表达式进行语义分析，最终通过从metastore获取表和分区的信息生成执行计划execution plan。

Metastore - *元数据库*存储仓库中各种表和分区的所有结构信息的组件，包括列和列类型信息，读取和写入数据所需的序列化程序和反序列化程序以及存储数据的相应HDFS文件。

Execution Engine - *执行引擎*执行编译器创建的执行计划的组件。 该计划是一个基于stages的DAG。 执行引擎管理计划的这些不同阶段之间的依赖关系，并在适当的系统组件上执行这些阶段。
