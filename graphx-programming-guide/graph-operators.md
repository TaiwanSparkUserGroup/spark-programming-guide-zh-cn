# 圖形操作

就像RDDs基本的操作map、filter和reduceByKey一樣，屬性圖形也具備基本操作集合的指令，這些操作採用使用者自訂義的函數並產生新轉換後的特徵和結構的新圖形。在[Graph](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph)中實作了優化的核心操作以及[GraphOps](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps)中定義的表示為核心操作組合的快捷操作。由於Scala中有隱含轉換，故`GraphOps`中的操作可以作為`Graph`的成員直接使用。例如，我們可以透過下方的例子來計算每個節點（定義在`GraphOps`）的內分支度。

```scala
val graph: Graph[(String, String), String]
// Use the implicit GraphOps.inDegrees operator
val inDegrees: VertexRDD[Int] = graph.inDegrees
```

區分核心圖形操作和`GraphOps`的原因是為了能在未來支援不同的圖形表示。每個圖形表示都必須提供核心操作的實作和重複使用定義在`GraphOps`中有用的操作。

## 操作指令清單

以下一些定義在`Graph`和`GraphOps`中的函數摘要，為了簡單起見，用`Graph`的成員做表示。注意，某些函數是已經經過刪簡後的（如預設參數和型別限制皆沒有列出），還有一些較為進階的函數也沒有列出，若是希望了解更多，請閱讀官方的API文件。

```scala
/** Summary of the functionality in the property graph */
class Graph[VD, ED] {
  // Information about the Graph ===================================================================
  val numEdges: Long
  val numVertices: Long
  val inDegrees: VertexRDD[Int]
  val outDegrees: VertexRDD[Int]
  val degrees: VertexRDD[Int]
  // Views of the graph as collections =============================================================
  val vertices: VertexRDD[VD]
  val edges: EdgeRDD[ED]
  val triplets: RDD[EdgeTriplet[VD, ED]]
  // Functions for caching graphs ==================================================================
  def persist(newLevel: StorageLevel = StorageLevel.MEMORY_ONLY): Graph[VD, ED]
  def cache(): Graph[VD, ED]
  def unpersistVertices(blocking: Boolean = true): Graph[VD, ED]
  // Change the partitioning heuristic  ============================================================
  def partitionBy(partitionStrategy: PartitionStrategy): Graph[VD, ED]
  // Transform vertex and edge attributes ==========================================================
  def mapVertices[VD2](map: (VertexID, VD) => VD2): Graph[VD2, ED]
  def mapEdges[ED2](map: Edge[ED] => ED2): Graph[VD, ED2]
  def mapEdges[ED2](map: (PartitionID, Iterator[Edge[ED]]) => Iterator[ED2]): Graph[VD, ED2]
  def mapTriplets[ED2](map: EdgeTriplet[VD, ED] => ED2): Graph[VD, ED2]
  def mapTriplets[ED2](map: (PartitionID, Iterator[EdgeTriplet[VD, ED]]) => Iterator[ED2])
    : Graph[VD, ED2]
  // Modify the graph structure ====================================================================
  def reverse: Graph[VD, ED]
  def subgraph(
      epred: EdgeTriplet[VD,ED] => Boolean = (x => true),
      vpred: (VertexID, VD) => Boolean = ((v, d) => true))
    : Graph[VD, ED]
  def mask[VD2, ED2](other: Graph[VD2, ED2]): Graph[VD, ED]
  def groupEdges(merge: (ED, ED) => ED): Graph[VD, ED]
  // Join RDDs with the graph ======================================================================
  def joinVertices[U](table: RDD[(VertexID, U)])(mapFunc: (VertexID, VD, U) => VD): Graph[VD, ED]
  def outerJoinVertices[U, VD2](other: RDD[(VertexID, U)])
      (mapFunc: (VertexID, VD, Option[U]) => VD2)
    : Graph[VD2, ED]
  // Aggregate information about adjacent triplets =================================================
  def collectNeighborIds(edgeDirection: EdgeDirection): VertexRDD[Array[VertexID]]
  def collectNeighbors(edgeDirection: EdgeDirection): VertexRDD[Array[(VertexID, VD)]]
  def aggregateMessages[Msg: ClassTag](
      sendMsg: EdgeContext[VD, ED, Msg] => Unit,
      mergeMsg: (Msg, Msg) => Msg,
      tripletFields: TripletFields = TripletFields.All)
    : VertexRDD[A]
  // Iterative graph-parallel computation ==========================================================
  def pregel[A](initialMsg: A, maxIterations: Int, activeDirection: EdgeDirection)(
      vprog: (VertexID, VD, A) => VD,
      sendMsg: EdgeTriplet[VD, ED] => Iterator[(VertexID,A)],
      mergeMsg: (A, A) => A)
    : Graph[VD, ED]
  // Basic graph algorithms ========================================================================
  def pageRank(tol: Double, resetProb: Double = 0.15): Graph[Double, Double]
  def connectedComponents(): Graph[VertexID, ED]
  def triangleCount(): Graph[Int, ED]
  def stronglyConnectedComponents(numIter: Int): Graph[VertexID, ED]
}
```

## 屬性操作

像是RDD的`map`的操作一樣，如下列所示：

```scala
class Graph[VD, ED] {
  def mapVertices[VD2](map: (VertexId, VD) => VD2): Graph[VD2, ED]
  def mapEdges[ED2](map: Edge[ED] => ED2): Graph[VD, ED2]
  def mapTriplets[ED2](map: EdgeTriplet[VD, ED] => ED2): Graph[VD, ED2]
}
```
每個操作執行後都會產生一個新的圖形，其節點或邊的屬性都會經過使用者所定義的`map`操作而改變。

> 注意，在經過這些操作下，是不會影響到圖形的結構。這些操作有一個重要特色，就是它會重複利用原始圖形結構的索引值。下面的兩段程式碼在目的是相同的，但是第一段並不會保存結構的索引值，這樣無法從GraphX中優化。

```scala
val newVertices = graph.vertices.map { case (id, attr) => (id, mapUdf(id, attr)) }
val newGraph = Graph(newVertices, graph.edges)
```
> 另一種方法是透過[mapVertices](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@mapVertices[VD2]((VertexId,VD)⇒VD2)(ClassTag[VD2]):Graph[VD2,ED])來保存索引。

```scala
val newGraph = graph.mapVertices((id, attr) => mapUdf(id, attr))
```

這些操作經常用來初始化作為特定計算或處理不必要的屬性的圖形。例如，給一個具有外分支度（Out-degree）屬性節點的圖形，用於PageRank。

```scala
// Given a graph where the vertex property is the out degree
val inputGraph: Graph[Int, String] =
  graph.outerJoinVertices(graph.outDegrees)((vid, _, degOpt) => degOpt.getOrElse(0))
// Construct a graph where each edge contains the weight
// and each vertex is the initial PageRank
val outputGraph: Graph[Double, Double] =
  inputGraph.mapTriplets(triplet => 1.0 / triplet.srcAttr).mapVertices((id, _) => 1.0)
```

## 結構性的操作

目前的GraphX只有支援一組簡單的結構性操作，我們希望未來能夠增加。下面是基本的結構性操作清單。

```scala
class Graph[VD, ED] {
  def reverse: Graph[VD, ED]
  def subgraph(epred: EdgeTriplet[VD,ED] => Boolean,
               vpred: (VertexId, VD) => Boolean): Graph[VD, ED]
  def mask[VD2, ED2](other: Graph[VD2, ED2]): Graph[VD, ED]
  def groupEdges(merge: (ED, ED) => ED): Graph[VD,ED]
}
```

[reverse](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@reverse:Graph[VD,ED])這個操作會將會反轉圖形內所有邊的方向並回傳反轉後的圖形。例如，這個操作可以用來計算反轉後的PageRank。由於這個操作並不會修改到節點或是邊，也不會改變邊的數量，所以能夠在不搬移或複製資料的情況下有效的實現。

[subgraph](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@subgraph((EdgeTriplet[VD,ED])⇒Boolean,(VertexId,VD)⇒Boolean):Graph[VD,ED])這個操作會利用使用者給予的節點和邊的條件（predicateds），回傳的是圖形是滿足條件的節點和邊，以及滿足節點條件的相連節點。`subgraph`的操作可以在許多情況上限制有興趣的節點和邊或刪除受損的連結。下面的範例就是說明如何刪除受損的連結。

```scala
// Create an RDD for the vertices
val users: RDD[(VertexId, (String, String))] =
  sc.parallelize(Array((3L, ("rxin", "student")), (7L, ("jgonzal", "postdoc")),
                       (5L, ("franklin", "prof")), (2L, ("istoica", "prof")),
                       (4L, ("peter", "student"))))
// Create an RDD for edges
val relationships: RDD[Edge[String]] =
  sc.parallelize(Array(Edge(3L, 7L, "collab"),    Edge(5L, 3L, "advisor"),
                       Edge(2L, 5L, "colleague"), Edge(5L, 7L, "pi"),
                       Edge(4L, 0L, "student"),   Edge(5L, 0L, "colleague")))
// Define a default user in case there are relationship with missing user
val defaultUser = ("John Doe", "Missing")
// Build the initial Graph
val graph = Graph(users, relationships, defaultUser)
// Notice that there is a user 0 (for which we have no information) connected to users
// 4 (peter) and 5 (franklin).
graph.triplets.map(
    triplet => triplet.srcAttr._1 + " is the " + triplet.attr + " of " + triplet.dstAttr._1
  ).collect.foreach(println(_))
// Remove missing vertices as well as the edges to connected to them
val validGraph = graph.subgraph(vpred = (id, attr) => attr._2 != "Missing")
// The valid subgraph will disconnect users 4 and 5 by removing user 0
validGraph.vertices.collect.foreach(println(_))
validGraph.triplets.map(
    triplet => triplet.srcAttr._1 + " is the " + triplet.attr + " of " + triplet.dstAttr._1
  ).collect.foreach(println(_))
```
> 注意，上面的範例中，只有提供節點的條件。如果沒有給予節點或邊的條件，`subgraph`將會預設為`True`，代表不會做任何限制。

[mask](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@mask[VD2,ED2](Graph[VD2,ED2])(ClassTag[VD2],ClassTag[ED2]):Graph[VD,ED])操作會建造一個子圖形，這個子圖形具備輸入圖形的節點和邊。可以利用`subgraph`去限制圖形，然後將其結果作為`mask`的遮罩來限制結果。例如，我們可以先利用有遺失的節點來運行連通分量演算法，然後再結合`subgraph`及`mask`來取得正確的結果。

```scala
/ Run Connected Components
val ccGraph = graph.connectedComponents() // No longer contains missing field
// Remove missing vertices as well as the edges to connected to them
val validGraph = graph.subgraph(vpred = (id, attr) => attr._2 != "Missing")
// Restrict the answer to the valid subgraph
val validCCGraph = ccGraph.mask(validGraph)
```

[groupEdges](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@groupEdges((ED,ED)⇒ED):Graph[VD,ED])操作會合併平行的邊（如一對節點之前重複的邊）。大許多應用上，會藉由將平行的邊合併（權值合併）為一條來降低圖形的大小。

## 连接操作

在许多情况下，有必要将外部数据加入到图中。例如，我们可能有额外的用户属性需要合并到已有的图中或者我们可能想从一个图中取出顶点特征加入到另外一个图中。这些任务可以用join操作完成。
下面列出的是主要的join操作。

```scala
class Graph[VD, ED] {
  def joinVertices[U](table: RDD[(VertexId, U)])(map: (VertexId, VD, U) => VD)
    : Graph[VD, ED]
  def outerJoinVertices[U, VD2](table: RDD[(VertexId, U)])(map: (VertexId, VD, Option[U]) => VD2)
    : Graph[VD2, ED]
}
```

[joinVertices](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps@joinVertices[U](RDD[(VertexId,U)])((VertexId,VD,U)⇒VD)(ClassTag[U]):Graph[VD,ED])
操作将输入RDD和顶点相结合，返回一个新的带有顶点特征的图。这些特征是通过在连接顶点的结果上使用用户定义的`map`函数获得的。在RDD中没有匹配值的顶点保留其原始值。

注意，对于给定的顶点，如果RDD中有超过1个的匹配值，则仅仅使用其中的一个。建议用下面的方法保证输入RDD的唯一性。下面的方法也会预索引返回的值用以加快后续的join操作。

```scala
val nonUniqueCosts: RDD[(VertexID, Double)]
val uniqueCosts: VertexRDD[Double] =
  graph.vertices.aggregateUsingIndex(nonUnique, (a,b) => a + b)
val joinedGraph = graph.joinVertices(uniqueCosts)(
  (id, oldCost, extraCost) => oldCost + extraCost)
```

除了将用户自定义的map函数用到所有顶点和改变顶点属性类型以外，更一般的[outerJoinVertices](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@outerJoinVertices[U,VD2](RDD[(VertexId,U)])((VertexId,VD,Option[U])⇒VD2)(ClassTag[U],ClassTag[VD2]):Graph[VD2,ED])与`joinVertices`类似。
因为并不是所有顶点在RDD中拥有匹配的值，map函数需要一个option类型。

```scala
val outDegrees: VertexRDD[Int] = graph.outDegrees
val degreeGraph = graph.outerJoinVertices(outDegrees) { (id, oldAttr, outDegOpt) =>
  outDegOpt match {
    case Some(outDeg) => outDeg
    case None => 0 // No outDegree means zero outDegree
  }
}
```

你可能已经注意到了，在上面的例子中用到了curry函数的多参数列表。虽然我们可以将f(a)(b)写成f(a,b)，但是f(a,b)意味着b的类型推断将不会依赖于a。因此，用户需要为定义
的函数提供类型标注。

```scala
val joinedGraph = graph.joinVertices(uniqueCosts,
  (id: VertexID, oldCost: Double, extraCost: Double) => oldCost + extraCost)
```

## 相邻聚合（Neighborhood Aggregation）

图分析任务的一个关键步骤是汇总每个顶点附近的信息。例如我们可能想知道每个用户的追随者的数量或者每个用户的追随者的平均年龄。许多迭代图算法（如PageRank，最短路径和连通体）
多次聚合相邻顶点的属性。

为了提高性能，主要的聚合操作从`graph.mapReduceTriplets`改为了新的`graph.AggregateMessages`。虽然API的改变相对较小，但是我们仍然提供了过渡的指南。

### 聚合消息(aggregateMessages)

GraphX中的核心聚合操作是[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])。
这个操作将用户定义的`sendMsg`函数应用到图的每个边三元组(edge triplet)，然后应用`mergeMsg`函数在其目的顶点聚合这些消息。

```scala
class Graph[VD, ED] {
  def aggregateMessages[Msg: ClassTag](
      sendMsg: EdgeContext[VD, ED, Msg] => Unit,
      mergeMsg: (Msg, Msg) => Msg,
      tripletFields: TripletFields = TripletFields.All)
    : VertexRDD[Msg]
}
```

用户自定义的`sendMsg`函数是一个[EdgeContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.EdgeContext)类型。它暴露源和目的属性以及边缘属性
以及发送消息给源和目的属性的函数([sendToSrc](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.EdgeContext@sendToSrc(msg:A):Unit)和[sendToDst](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.EdgeContext@sendToDst(msg:A):Unit))。
可将`sendMsg`函数看做map-reduce过程中的map函数。用户自定义的`mergeMsg`函数指定两个消息到相同的顶点并保存为一个消息。可以将`mergeMsg`函数看做map-reduce过程中的reduce函数。
[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])
操作返回一个包含聚合消息(目的地为每个顶点)的`VertexRDD[Msg]`。没有接收到消息的顶点不包含在返回的`VertexRDD`中。

另外，[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])
有一个可选的`tripletFields`参数，它指出在`EdgeContext`中，哪些数据被访问（如源顶点特征而不是目的顶点特征）。`tripletsFields`可能的选项定义在[TripletFields](http://spark.apache.org/docs/latest/api/java/org/apache/spark/graphx/TripletFields.html)中。
`tripletFields`参数用来通知GraphX仅仅只需要`EdgeContext`的一部分允许GraphX选择一个优化的连接策略。例如，如果我们想计算每个用户的追随者的平均年龄，我们仅仅只需要源字段。
所以我们用`TripletFields.Src`表示我们仅仅只需要源字段。

在下面的例子中，我们用`aggregateMessages`操作计算每个用户更年长的追随者的年龄。

```scala
// Import random graph generation library
import org.apache.spark.graphx.util.GraphGenerators
// Create a graph with "age" as the vertex property.  Here we use a random graph for simplicity.
val graph: Graph[Double, Int] =
  GraphGenerators.logNormalGraph(sc, numVertices = 100).mapVertices( (id, _) => id.toDouble )
// Compute the number of older followers and their total age
val olderFollowers: VertexRDD[(Int, Double)] = graph.aggregateMessages[(Int, Double)](
  triplet => { // Map Function
    if (triplet.srcAttr > triplet.dstAttr) {
      // Send message to destination vertex containing counter and age
      triplet.sendToDst(1, triplet.srcAttr)
    }
  },
  // Add counter and age
  (a, b) => (a._1 + b._1, a._2 + b._2) // Reduce Function
)
// Divide total age by number of older followers to get average age of older followers
val avgAgeOfOlderFollowers: VertexRDD[Double] =
  olderFollowers.mapValues( (id, value) => value match { case (count, totalAge) => totalAge / count } )
// Display the results
avgAgeOfOlderFollowers.collect.foreach(println(_))
```
当消息（以及消息的总数）是常量大小(列表和连接替换为浮点数和添加)时，`aggregateMessages`操作的效果最好。

### Map Reduce三元组过渡指南

在之前版本的GraphX中，利用[mapReduceTriplets]操作完成相邻聚合。

```scala
class Graph[VD, ED] {
  def mapReduceTriplets[Msg](
      map: EdgeTriplet[VD, ED] => Iterator[(VertexId, Msg)],
      reduce: (Msg, Msg) => Msg)
    : VertexRDD[Msg]
}
```
`mapReduceTriplets`操作在每个三元组上应用用户定义的map函数，然后保存用用户定义的reduce函数聚合的消息。然而，我们发现用户返回的迭代器是昂贵的，它抑制了我们添加额外优化(例如本地顶点的重新编号)的能力。
[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])
暴露三元组字段和函数显示的发送消息到源和目的顶点。并且，我们删除了字节码检测转而需要用户指明三元组的哪些字段实际需要。

下面的代码用到了`mapReduceTriplets`

```scala
val graph: Graph[Int, Float] = ...
def msgFun(triplet: Triplet[Int, Float]): Iterator[(Int, String)] = {
  Iterator((triplet.dstId, "Hi"))
}
def reduceFun(a: Int, b: Int): Int = a + b
val result = graph.mapReduceTriplets[String](msgFun, reduceFun)
```

下面的代码用到了`aggregateMessages`

```scala
val graph: Graph[Int, Float] = ...
def msgFun(triplet: EdgeContext[Int, Float, String]) {
  triplet.sendToDst("Hi")
}
def reduceFun(a: Int, b: Int): Int = a + b
val result = graph.aggregateMessages[String](msgFun, reduceFun)
```

### 计算度信息

最一般的聚合任务就是计算顶点的度，即每个顶点相邻边的数量。在有向图中，经常需要知道顶点的入度、出度以及总共的度。[GraphOps](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps)
类包含一个操作集合用来计算每个顶点的度。例如，下面的例子计算最大的入度、出度和总度。

```scala
// Define a reduce operation to compute the highest degree vertex
def max(a: (VertexId, Int), b: (VertexId, Int)): (VertexId, Int) = {
  if (a._2 > b._2) a else b
}
// Compute the max degrees
val maxInDegree: (VertexId, Int)  = graph.inDegrees.reduce(max)
val maxOutDegree: (VertexId, Int) = graph.outDegrees.reduce(max)
val maxDegrees: (VertexId, Int)   = graph.degrees.reduce(max)
```
### Collecting Neighbors

在某些情况下，通过收集每个顶点相邻的顶点及它们的属性来表达计算可能更容易。这可以通过[collectNeighborIds](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps@collectNeighborIds(EdgeDirection):VertexRDD[Array[VertexId]])
和[collectNeighbors](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps@collectNeighbors(EdgeDirection):VertexRDD[Array[(VertexId,VD)]])操作来简单的完成

```scala
class GraphOps[VD, ED] {
  def collectNeighborIds(edgeDirection: EdgeDirection): VertexRDD[Array[VertexId]]
  def collectNeighbors(edgeDirection: EdgeDirection): VertexRDD[ Array[(VertexId, VD)] ]
}
```
这些操作是非常昂贵的，因为它们需要重复的信息和大量的通信。如果可能，尽量用`aggregateMessages`操作直接表达相同的计算。

### 缓存和不缓存

在Spark中，RDDs默认是不缓存的。为了避免重复计算，当需要多次利用它们时，我们必须显示地缓存它们。GraphX中的图也有相同的方式。当利用到图多次时，确保首先访问`Graph.cache()`方法。

在迭代计算中，为了获得最佳的性能，不缓存可能是必须的。默认情况下，缓存的RDDs和图会一直保留在内存中直到因为内存压力迫使它们以LRU的顺序删除。对于迭代计算，先前的迭代的中间结果将填充到缓存
中。虽然它们最终会被删除，但是保存在内存中的不需要的数据将会减慢垃圾回收。只有中间结果不需要，不缓存它们是更高效的。这涉及到在每次迭代中物化一个图或者RDD而不缓存所有其它的数据集。
在将来的迭代中仅用物化的数据集。然而，因为图是由多个RDD组成的，正确的不持久化它们是困难的。对于迭代计算，我们建议使用Pregel API，它可以正确的不持久化中间结果。

