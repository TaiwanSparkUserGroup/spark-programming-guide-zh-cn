# 圖形操作（Graph Operators）

就像RDDs基本的操作map、filter和reduceByKey一樣，屬性圖形也具備一些基本的運算子，這些運算子採用使用者自訂義的函數並產生新轉換後的特徵和結構的新圖形。在[Graph](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph)中實作了優化後的核心運算子以及[GraphOps](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps)中定義的表示為核心運算子組合的快捷運算子。由於Scala中有隱式轉換，故`GraphOps`中的運算子可以作為`Graph`的成員直接使用。例如，我們可以透過下方的例子來計算每個頂點（定義在`GraphOps`）的內分支度。

```scala
val graph: Graph[(String, String), String]
// Use the implicit GraphOps.inDegrees operator
val inDegrees: VertexRDD[Int] = graph.inDegrees
```

區分核心圖形操作和`GraphOps`的原因是為了能在未來支援不同的圖形表示。每個圖形表示都必須提供核心操作的實作和重複使用定義在`GraphOps`中有用的操作。

## 運算子的摘要清單（Summary List Of Operators）

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

# 屬性運算子（Property Operators）

像是RDD的`map`運算子一樣，如下列所示：

```scala
class Graph[VD, ED] {
  def mapVertices[VD2](map: (VertexId, VD) => VD2): Graph[VD2, ED]
  def mapEdges[ED2](map: Edge[ED] => ED2): Graph[VD, ED2]
  def mapTriplets[ED2](map: EdgeTriplet[VD, ED] => ED2): Graph[VD, ED2]
}
```
每個運算子執行後都會產生一個新的圖形，其頂點或邊的屬性都會經過使用者所定義的`map`函數而改變。

> 注意，在經過這些操作下，是不會影響到圖形的結構。這些運算子有一個重要特色，就是它會重複利用原始圖形結構的索引值。下面的兩段程式碼目的上是相同的，但是第一段並不會保存結構的索引值，這樣將無法讓GraphX系統優化。

> ```scala
val newVertices = graph.vertices.map { case (id, attr) => (id, mapUdf(id, attr)) }
val newGraph = Graph(newVertices, graph.edges)
```
> 另一種方法是透過[mapVertices](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@mapVertices[VD2]((VertexId,VD)⇒VD2)(ClassTag[VD2]):Graph[VD2,ED])來保存索引。

> ```scala
val newGraph = graph.mapVertices((id, attr) => mapUdf(id, attr))
```

這些運算子經常用來初始化作為特定計算或處理不必要的屬性的圖形。例如，給一個具有外分支度（Out-degree）屬性頂點的圖形，用於PageRank。

```scala
// Given a graph where the vertex property is the out degree
val inputGraph: Graph[Int, String] =
  graph.outerJoinVertices(graph.outDegrees)((vid, _, degOpt) => degOpt.getOrElse(0))
// Construct a graph where each edge contains the weight
// and each vertex is the initial PageRank
val outputGraph: Graph[Double, Double] =
  inputGraph.mapTriplets(triplet => 1.0 / triplet.srcAttr).mapVertices((id, _) => 1.0)
```

# 結構性運算子（Structural Operators）

目前的GraphX只有支援一組簡單的結構性運算子，我們希望未來能夠增加。下面列出了基本的結構性運算子。

```scala
class Graph[VD, ED] {
  def reverse: Graph[VD, ED]
  def subgraph(epred: EdgeTriplet[VD,ED] => Boolean,
               vpred: (VertexId, VD) => Boolean): Graph[VD, ED]
  def mask[VD2, ED2](other: Graph[VD2, ED2]): Graph[VD, ED]
  def groupEdges(merge: (ED, ED) => ED): Graph[VD,ED]
}
```

[reverse](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@reverse:Graph[VD,ED])：此運算子將會反轉圖形內所有邊的方向並回傳反轉後的圖形。例如，這個操作可以用來計算反轉後的PageRank。由於這個操作並不會修改到頂點或是邊，也不會改變邊的數量，所以能夠在不搬移或複製資料的情況下有效率地實現。

[subgraph](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@subgraph((EdgeTriplet[VD,ED])⇒Boolean,(VertexId,VD)⇒Boolean):Graph[VD,ED])：此運算子會利用使用者給予的頂點和邊的條件（predicateds），回傳的是圖形是滿足條件的頂點和邊，以及滿足頂點條件的相連頂點。`subgraph`運算子可以在許多情況上限制有興趣的頂點和邊或刪除受損的連結。下面的範例就是說明如何刪除受損的連結。

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
> 注意，上面的範例中，只有提供頂點的條件。如果沒有給予頂點或邊的條件，`subgraph`運算子預設為`True`，代表不會做任何限制。

[mask](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@mask[VD2,ED2](Graph[VD2,ED2])(ClassTag[VD2],ClassTag[ED2]):Graph[VD,ED])：此運算子會建造一個子圖形，這個子圖形具備輸入圖形的頂點和邊。可以利用`subgraph`運算子限制圖形，然後將其結果作為`mask`的遮罩來限制結果。例如，我們可以先利用有遺失的頂點來運行連通分量演算法，然後再結合`subgraph`及`mask`來取得正確的結果。

```scala
/ Run Connected Components
val ccGraph = graph.connectedComponents() // No longer contains missing field
// Remove missing vertices as well as the edges to connected to them
val validGraph = graph.subgraph(vpred = (id, attr) => attr._2 != "Missing")
// Restrict the answer to the valid subgraph
val validCCGraph = ccGraph.mask(validGraph)
```

[groupEdges](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@groupEdges((ED,ED)⇒ED):Graph[VD,ED])：此運算子會合併平行的邊（如一對頂點之前重複的邊）。在許多應用上，會藉由將平行的邊合併（權值合併）為一條來降低圖形的大小。

# Join運算子（Join Operators）

在許多情況下，必須將外部的資料合併到圖中。例如，我們可能會想將額外的使用者資訊合併到現有的圖中或是想從一個圖中取出資訊加到另一個圖中。這些任務都可以藉由`join`運算子來完成。以下列出`join`運算子主要的功能。

```scala
class Graph[VD, ED] {
  def joinVertices[U](table: RDD[(VertexId, U)])(map: (VertexId, VD, U) => VD)
    : Graph[VD, ED]
  def outerJoinVertices[U, VD2](table: RDD[(VertexId, U)])(map: (VertexId, VD, Option[U]) => VD2)
    : Graph[VD2, ED]
}
```

[joinVertices](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps@joinVertices[U](RDD[(VertexId,U)])((VertexId,VD,U)⇒VD)(ClassTag[U]):Graph[VD,ED])：此運算子會將輸入的RDD和頂點作結合，回傳一個透過使用者定義的`map`函數所轉換後的頂點的圖。若頂點沒有匹配值則會保留其原始值。

> 注意，對於給定的一個頂點，如果RDD有超過一個的值，而只能使用其中一個。因此建議用下列的方法來將結果預設索引值，來保證RDD的唯一性，來大量加速後續的`join`運算。

```scala
val nonUniqueCosts: RDD[(VertexID, Double)]
val uniqueCosts: VertexRDD[Double] =
  graph.vertices.aggregateUsingIndex(nonUnique, (a,b) => a + b)
val joinedGraph = graph.joinVertices(uniqueCosts)(
  (id, oldCost, extraCost) => oldCost + extraCost)
```

除了將使用者自定義的map函數套用到所有的頂點和改變頂點屬性類型外，更一般的[outerJoinVertices](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@outerJoinVertices[U,VD2](RDD[(VertexId,U)])((VertexId,VD,Option[U])⇒VD2)(ClassTag[U],ClassTag[VD2]):Graph[VD2,ED])的用法與`joinVertices`類似。因為並非所有頂點在RDD中都有匹配值，map函數需要一個option型別參數。

```scala
val outDegrees: VertexRDD[Int] = graph.outDegrees
val degreeGraph = graph.outerJoinVertices(outDegrees) { (id, oldAttr, outDegOpt) =>
  outDegOpt match {
    case Some(outDeg) => outDeg
    case None => 0 // No outDegree means zero outDegree
  }
}
```
> 你可能已經注意到，上面的例子中用到了`curry`函數的多參數清單。我們能夠將f(a)(b)寫成f(a,b)，但f(a,b)表示b的型別將不會依賴於a。因此，使用者需要為自定義的函數提供型別的宣告。

```scala
val joinedGraph = graph.joinVertices(uniqueCosts,
  (id: VertexID, oldCost: Double, extraCost: Double) => oldCost + extraCost)
```

# 相鄰聚合（Neighborhood Aggregation）

圖形分析中最關鍵的步驟就是匯集每個頂點周圍的資訊。例如，我們可能想知道每個使用者的追隨者數量或是平均年齡。許多的迭代圖形演算法（如PageRank、最短路徑（Shortest Path）和連通分量（Connected Components））重複的匯集相鄰頂點（如PageRank的值、到來源的最短路徑、最小可到達的頂點id）的資訊。

> 為了改善效能，將主要的聚合運算子從`graph.mapReduceTriplets`改成新的`graph.AggregateMessages`。雖然API的變化不大，但是我們仍然提高轉換的指南。

## 聚合訊息(aggregateMessages)

GraphX中的核心聚合運算是[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])。這個運算子在圖形的每個edge triplet應用一個使用者自定義的`sendMsg`函數，然後也應用`mergeMsg`函數去匯集目標頂點的資訊。

```scala
class Graph[VD, ED] {
  def aggregateMessages[Msg: ClassTag](
      sendMsg: EdgeContext[VD, ED, Msg] => Unit,
      mergeMsg: (Msg, Msg) => Msg,
      tripletFields: TripletFields = TripletFields.All)
    : VertexRDD[Msg]
}
```

使用者自定義的`sendMsg`函數接受一個[EdgeContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.EdgeContext)型別，`EdgeContext`透露了起始和目標的屬性以及傳送訊息給起始和目標屬性的函數（[sendToSrc](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.EdgeContext@sendToSrc(msg:A):Unit)和[sendToDst](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.EdgeContext@sendToDst(msg:A):Unit)）。可以將`sendMsg`視作`map-reduce`中的`map`函數。而使用者自定義的`mergeMsg`函數接受兩個指定的訊息到相同的頂點並產生一個訊息，可以將`mergeMsg`視作`map-reduce`中的`reduce`函數。[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])運算子會回傳一個包含匯集訊息（Msg型別）到指定的每一個頂點的`VertexRDD[Msg]`。沒有接收到訊息的頂點不會包含在回傳的`VertexRDD`中。

另外，[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])接受一個可選的參數`tripletFields`，它顯示出在`EdgeContext`中，哪些資料是可被存取的（如起始頂點的屬性，而目標頂點的屬性無法）。`tripletsFields`可能的值都定義在[TripletFields](http://spark.apache.org/docs/latest/api/java/org/apache/spark/graphx/TripletFields.html)中，預設值為`TripleetFields.All`，其說明使用者自定義的`sendMsg`可存取`EdgeContext`的任何欄位。`tripletFields`參數可用來通知GraphX只有部分的`EdgeContext`需要允許GraphX選擇一個優化的`Join`策略。舉例，如果我們想要計算每個使用者的追隨者平均年齡，我們只需要起始的欄位，所以我們只需要用`TripletFields.Src`來表示我們只需要起始的欄位。

> 在早期GraphX的版本，我們利用位元碼檢測，作為`TripletFields.Src`的值，然而我們發現這樣有點不太可靠，所以挑選了更明確的用法。

在以下的範例中，我們用`aggregateMessages`運算子來計算每個使用者年長的追隨者的平均年齡。

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
> 當訊息（或是訊息的總數）是固定常數（如福點數和加法，而不是串列和串接）時，`aggregateMessages`的效果會最好。

## Map Reduce三元组过渡指南

在早期GraphX的版本中，利用[mapReduceTriplets](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@mapReduceTriplets[A](mapFunc:org.apache.spark.graphx.EdgeTriplet[VD,ED]=>Iterator[(org.apache.spark.graphx.VertexId,A)],reduceFunc:(A,A)=>A,activeSetOpt:Option[(org.apache.spark.graphx.VertexRDD[_],org.apache.spark.graphx.EdgeDirection)])(implicitevidence$10:scala.reflect.ClassTag[A]):org.apache.spark.graphx.VertexRDD[A])運算子來完成相鄰聚合（Neighborhood Aggregation）。

```scala
class Graph[VD, ED] {
  def mapReduceTriplets[Msg](
      map: EdgeTriplet[VD, ED] => Iterator[(VertexId, Msg)],
      reduce: (Msg, Msg) => Msg)
    : VertexRDD[Msg]
}
```

[mapReduceTriplets](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@mapReduceTriplets[A](mapFunc:org.apache.spark.graphx.EdgeTriplet[VD,ED]=>Iterator[(org.apache.spark.graphx.VertexId,A)],reduceFunc:(A,A)=>A,activeSetOpt:Option[(org.apache.spark.graphx.VertexRDD[_],org.apache.spark.graphx.EdgeDirection)])(implicitevidence$10:scala.reflect.ClassTag[A]):org.apache.spark.graphx.VertexRDD[A])運算子接受每個三元組應用於使用者自定義的`map`函數，且能夠產生利用使用者自定義的`reduce`函數來匯集訊息。然而，我們發現使用者返回的迭代器是昂貴的，且它禁止我們添加額外的優化功能（如區域頂點的重新編號）。在[aggregateMessages](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@aggregateMessages[A]((EdgeContext[VD,ED,A])⇒Unit,(A,A)⇒A,TripletFields)(ClassTag[A]):VertexRDD[A])中，我們介紹了`EdgeContext`，它透露三元組欄位和函數來更明確的傳送訊息給起始和目標的頂點。因此，我們移除了位元碼檢測，而要求使用者明確的指出三元組的哪些欄位是實際上使用的。

以下是利用了`mapReduceTriplets`的範例：

```scala
val graph: Graph[Int, Float] = ...
def msgFun(triplet: Triplet[Int, Float]): Iterator[(Int, String)] = {
  Iterator((triplet.dstId, "Hi"))
}
def reduceFun(a: Int, b: Int): Int = a + b
val result = graph.mapReduceTriplets[String](msgFun, reduceFun)
```

也等效於以下使用`aggregateMessages`的範例：

```scala
val graph: Graph[Int, Float] = ...
def msgFun(triplet: EdgeContext[Int, Float, String]) {
  triplet.sendToDst("Hi")
}
def reduceFun(a: Int, b: Int): Int = a + b
val result = graph.aggregateMessages[String](msgFun, reduceFun)
```

## 計算分支度（degree）資訊

最一般的聚合任務就是計算每一個頂點的分支度數，就是每個頂點相鄰邊的數量。在有向圖中，經常需要知道頂點的內分支度、外分支度及分支度的總數。[GraphOps](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps)類別中，有一系列的運算子來計算每個頂點的分支度。例如，以下的範例是計算最大的內分支度、外分支度和分支度的總數。

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
## Collecting Neighbors

在某些情形下，透過收集每個頂點相鄰的頂點及他們的屬性來代替計算會更容易。這可以透過[collectNeighborIds](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps@collectNeighborIds(EdgeDirection):VertexRDD[Array[VertexId]])運算子完成。

```scala
class GraphOps[VD, ED] {
  def collectNeighborIds(edgeDirection: EdgeDirection): VertexRDD[Array[VertexId]]
  def collectNeighbors(edgeDirection: EdgeDirection): VertexRDD[ Array[(VertexId, VD)] ]
}
```

> 這些運算子是非常昂貴的，因為需要複製資訊及大量的通訊。如果可能，盡量使用`aggregateMessages`來直接替代相同的計算。

## 暫存與否

在Spark中，RDDs在預設下是不會一直存在記憶體中。為了避免重複運算，當要多次使用它們，則必須明確的將它們暫存起來。而Graphs在GraphX中的行為就像是RDDs一樣。當Graphs需要被多次使用，記得先呼叫`Graph.cache()`。

在迭代運算中，為了得到最佳的效能，不暫存是必須的。在預設情況下，暫存的RDDs和Graphs會一直保留在記憶體中，直到記憶體將它們釋放（利用LRU演算法）。對於迭代運算中，先前計算的結果也會暫存在記憶體中。雖然最終都會被釋放，但是暫存不需要的資料在記憶體中會減慢垃圾回收（Garbage collection）速度。若中間產生出來的結果不暫存，則會提升整體的效率。這牽扯到每次迭代中實體化一個Graph或者RDD，且不暫存其他的資料集，在未來的迭代中僅僅使用實體化的資料集。**對於迭代的計算，我們推薦Pregel API，它能適時的將中間結果釋放。**
