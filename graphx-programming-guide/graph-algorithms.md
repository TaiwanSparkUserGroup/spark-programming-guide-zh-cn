# 圖形演算法

GraphX具備一系列的圖形演算法來簡化圖形分系的任務。這些演算法都在`org.apache.spark.graphx.lib`，可以直接透過`Graph`中的`GraphOps`來取得。這章節將會描述如何使用這些圖形演算法。

## PageRank

PageRank是用來衡量一個圖形中每個節點的重要程度，假設有一條從u到v的邊，這條邊稱為u給v的重要性指標。例如，一個Twitter使用者有許多追隨者，如此一來，可以認為這名使用者相當重要。

在GraphX中的[PageRank object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.PageRank$)實作了靜態和動態PageRank的方法。靜態的PageRank會在固定的次數內運行，而動態的PageRank則會一直運行，直到收斂。[GraphOps](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps)允許直接呼叫這些方法。

GraphX內有一個範例，可以讓我們直接在社群媒體資料集上運行PageRank演算法。使用者的資料在`graphx/data/users.txt`，使用者之間的關係在`graphx/data/followers.txt`中。我們可以透過以下來計算出每個使用者的PageRank。

```scala
// Load the edges as a graph
val graph = GraphLoader.edgeListFile(sc, "graphx/data/followers.txt")
// Run PageRank
val ranks = graph.pageRank(0.0001).vertices
// Join the ranks with the usernames
val users = sc.textFile("graphx/data/users.txt").map { line =>
  val fields = line.split(",")
  (fields(0).toLong, fields(1))
}
val ranksByUsername = users.join(ranks).map {
  case (id, (username, rank)) => (username, rank)
}
// Print the result
println(ranksByUsername.collect().mkString("\n"))
```

## 连通体算法

连通体算法用id标注图中每个连通体，将连通体中序号最小的顶点的id作为连通体的id。例如，在社交网络中，连通体可以近似为集群。GraphX在[ConnectedComponents object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.ConnectedComponents$)
中包含了一个算法的实现，我们通过下面的方法计算社交网络数据集中的连通体。

```scala
/ Load the graph as in the PageRank example
val graph = GraphLoader.edgeListFile(sc, "graphx/data/followers.txt")
// Find the connected components
val cc = graph.connectedComponents().vertices
// Join the connected components with the usernames
val users = sc.textFile("graphx/data/users.txt").map { line =>
  val fields = line.split(",")
  (fields(0).toLong, fields(1))
}
val ccByUsername = users.join(cc).map {
  case (id, (username, cc)) => (username, cc)
}
// Print the result
println(ccByUsername.collect().mkString("\n"))
```

## 三角形计数算法

一个顶点有两个相邻的顶点以及相邻顶点之间的边时，这个顶点是一个三角形的一部分。GraphX在[TriangleCount object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.TriangleCount$)
中实现了一个三角形计数算法，它计算通过每个顶点的三角形的数量。需要注意的是，在计算社交网络数据集的三角形计数时，`TriangleCount`需要边的方向是规范的方向(srcId < dstId),
并且图通过`Graph.partitionBy`分片过。

```scala
// Load the edges in canonical order and partition the graph for triangle count
val graph = GraphLoader.edgeListFile(sc, "graphx/data/followers.txt", true).partitionBy(PartitionStrategy.RandomVertexCut)
// Find the triangle count for each vertex
val triCounts = graph.triangleCount().vertices
// Join the triangle counts with the usernames
val users = sc.textFile("graphx/data/users.txt").map { line =>
  val fields = line.split(",")
  (fields(0).toLong, fields(1))
}
val triCountByUsername = users.join(triCounts).map { case (id, (username, tc)) =>
  (username, tc)
}
// Print the result
println(triCountByUsername.collect().mkString("\n"))
```
