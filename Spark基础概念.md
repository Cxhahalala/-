# Spark基础概念

## Spark官方概念

Apache Spark 是用于**大规模数据**处理的**统一分析引擎**。

在提出Spark的论文中，同样提出了RDD的概念，RDD,即弹性分布式数据集，是一种分布式内存抽象。

![img](http://114.132.222.183:5244/d/HLP/chengxin/spark/1.png)

Spark可以计算多种各类的数据，包括结构化数据，半结构化数据，非结构化等类型。

同时Spark也支持多种语言，如Java，Python,Scala,R以及SQL语言来开发应用程序来计算数据。

因此，Spark被称为**统一分析引擎**。

## Spark发展历程

- 2009年起源于加州伯克利分校
- 2010Spark开源
- 2013年Spark被捐献给Apache
- 2014年称为Apache顶级项目
- 2016年发布Spark2.0
- 2019年发布Spark3.0

## Spark与Hadoop中的MapReduce对比

![img](http://114.132.222.183:5244/d/HLP/chengxin/spark/2.png)

Spark在计算方面相比MapReduce有巨大的性能优势，因为Spark是基于内存进行计算，相较于MapReduce会少很多IO开销。

但Spark只做计算，并不是说替代了Hadoop，充其量只能说Spark在计算方面可以代替MapReduce。

Spark一般是与Hadoop配合使用。

## Spark四大特点

### Speed

Apache Spark支持内存计算，通过DAG(有向无环图)执行引擎支持无环数据流，官方宣称在内存中比Hadoop的MapReduce快100倍，
在硬盘中比MapReduec快10倍。

在处理方式上，Spark可以将中间处理结果存储到内存中，并且Spark提供了丰富的算子（API）,可以由Spark程序中完成复杂任务，而MapReduce却只有Map和Reduce两个算子。

### Easy of Use

Spark支持多种语言，如Java,Python,Scala。

Spark的代码是简单的，但概念较多。

![img](http://114.132.222.183:5244/d/HLP/chengxin/spark/3.png)

从上图可以看出，即使没有学过Spark的语法，也是很容易读懂代码的含义。

### Gengrality

![img](http://114.132.222.183:5244/d/HLP/chengxin/spark/4.png)

Spark的核心Spark Core支持多种语言编写。

除了Spark Core，Spark还提供了Spark Sql,Spark Streaming,MLlib和Graphx在内的多个工具库。

Spark Sql:处理结构化数据。

Spark Streaming:处理流式数据。

MLlib:处理机器学习计算。

GraphX:处理图计算。

可以看到，Spark不仅支持多种变成语言，而且也能处理多种类型的数据，通用性强。

### Runs EveryWhere

**运行方式**

Spark可以运行在Hadoop的yarn资源管理框架上，也可以运行在Mesos资源管理框架上，并且也支持Standalone的独立运行模式，

也可以运行在云Kubernetes。

![img](http://114.132.222.183:5244/d/HLP/chengxin/spark/Spark%E8%BF%90%E8%A1%8C%E6%96%B9%E5%BC%8F.png)

**数据源**

Spark可以从多种数据源获取数据，如Hadoop的hdfs,Hbase，Mysql，Kafka。

![img](http://114.132.222.183:5244/d/HLP/chengxin/spark/Spark%E6%94%AF%E6%8C%81%E6%95%B0%E6%8D%AE%E6%BA%90.png)



## Spark框架模块

- Spark Core：Spark最底层的模块，所有其它模块均建立Spark Core之上。提供Spark的核心功能，Spark Core以RDD为数据抽象，提供多种语言api。
- SparkSql：处理结构化数据模块，针对离线数据处理。
- SparkStreaming:基于Spark Sql在线流式数据处理。
- MLlib:机器学习计算。
- GrahpX:图计算。

## 大数据处理类型

- 批量数据处理：时间跨度数十分钟到数小时。
- 历史数据的交互式查询：时间跨度在数10秒到数分钟之间。
- 实时数据流的数据处理：时间跨度在数百毫秒到数秒。