# GraphX编程指南

GraphX是一個新的(alpha) Spark API，它用於圖(Graph)和平行圖(graph-parallel)的計算。GraphX透過引入[Resilient Distributed Property Graph](property-graph.md)：一種帶有節點和邊屬性的有向多重圖，來擴展Spark RDD。為了支援圖的運算，GraphX公開一系列基本操作(例如：subGraph, joinVertices和aggregateMessages)和Pregel API的優化。此外，GraphX也持續增加圖的演算法還有簡化分析圖的工具(Builder)。

從社群媒體到語言模型，數量和重要性不斷成長的圖形資料推動了許多'graph-parallel'系統(例如：Giraph和GraphLab)的發展。
通过限制可表达的计算类型和引入新的技术来划分和分配图，这些系统可以高效地执行复杂的图形算法，比一般的`data-parallel`系统快很多。

![data parallel vs graph parallel](../img/data_parallel_vs_graph_parallel.png)

然而，通过这种限制可以提高性能，但是很难表示典型的图分析途径（构造图、修改它的结构或者表示跨多个图的计算）中很多重要的stages。另外，我们如何看待数据取决于我们的目标，并且同一原始数据可能有许多不同表和图的视图。

![表和图](../img/tables_and_graphs.png)

结论是，图和表之间经常需要能够相互移动。然而，现有的图分析管道必须组成`graph-parallel`和`data- parallel`系统`，从而实现大数据的迁移和复制并生成一个复杂的编程模型。

![图分析路径](../img/graph_analytics_pipeline.png)

GraphX项目的目的就是将`graph-parallel`和`data-parallel`统一到一个系统中，这个系统拥有一个唯一的组合API。GraphX允许用户将数据当做一个图和一个集合（RDD），而不需要
而不需要数据移动或者复杂。通过将最新的进展整合进`graph-parallel`系统，GraphX能够优化图操作的执行。

* [開始](getting-started.md)
* [属性图](property-graph.md)
* [图操作符](graph-operators.md)
* [Pregel API](pregel-api.md)
* [图构造者](graph-builders.md)
* [顶点和边RDDs](vertex-and-edge-rdds.md)
* [图算法](graph-algorithms.md)
* [例子](examples.md)
