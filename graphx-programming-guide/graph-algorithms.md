# 圖形演算法

GraphX具備一系列的圖形演算法來簡化圖形分系的任務。這些演算法都在`org.apache.spark.graphx.lib`，可以直接透過`Graph`中的[GraphOps](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps)來取得。這章節將會描述如何使用這些圖形演算法。

## PageRank

PageRank是用來衡量一個圖形中每個節點的重要程度，假設有一條從u到v的邊，這條邊稱為u給v的重要性指標。例如，一個Twitter使用者有許多追隨者，如此一來，可以認為這名使用者相當重要。

在GraphX中的[PageRank object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.PageRank$)實作了靜態和動態PageRank的方法。靜態的PageRank會在固定的次數內運行，而動態的PageRank則會一直運行，直到收斂。[GraphOps](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps)允許直接呼叫這些方法。

GraphX內有一個範例，可以讓我們直接在社群媒體資料集上運行PageRank演算法。
使用者的資料在`graphx/data/users.txt`，使用者之間的關係在`graphx/data/followers.txt`中。我們可以透過以下來計算出每個使用者的PageRank。

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

## 連通分量演算法

連通分量演算法利用連通分量中編號最小的節點的ID來作為其的標籤。例如，在社群媒體中，連通分量可以近似為一個群聚。在GraphX中的[ConnectedComponents object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.ConnectedComponents$)有這個演算法實作，我們可以透過下面的範例來完成。

```scala
// Load the graph as in the PageRank example
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

## 三角形計數演算法

若一個節點有兩個相鄰的節點且和它們有邊相連，那麼這個節點就是三角形的一部分。GraphX中的[TriangleCount object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.TriangleCount$)實作了演算法，它計算通過每個節點的三角形數量，用來衡量群聚。需要注意的`TriangleCount`要求邊的方向是按照規定的方向（srcId < dstId）並且圖形是利用`Graph.partitionBy`所切開的。

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
