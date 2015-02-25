# Pregel API

圖本身是遞迴型的資料結構，因為頂點的屬性依賴於其鄰居的屬性，這些鄰居的屬性又依賴於其鄰居的屬性。結論是，許多重要的圖形演算法，都需要重複的重新計算每個頂點的屬性，直到滿足某個確定的條件。一系列的Graph-parallel抽象體已經背提出來代表這些迭代型演算法。GraphX公開了一個Pregel-like的運算子，它是廣泛使用Pregel和GraphLab抽象的一個融合。

在GraphX中，更高階層的Pregel運算子是一個限制到圖拓僕的批量同步（bulk-synchronous）。Pregel運算子執行一系列的super-steps，再這些步驟中，頂點從之前的super-steps中接收進入訊息的總和，為頂點的屬性計算一個新值，然後在下一個super-step中發送訊息到相鄰的頂點。不像Pregel而更像GraphLab，訊息被作為一個邊三元組的函數平行的運算，且訊息運算會存取來源和目標頂點的特徵。在super-step中，未收到訊息的頂點會被跳過。當沒有任何訊息遺留時，Pregel運算子會停止迭代且回傳最後的圖。

> 注意，不像標準的Pregel實作，GraphX中的頂點只能夠發送訊息給相鄰頂點，且利用使用者自訂的通知函數來平行完成訊息的建立。這些限制允許了GraphX進行額外的優化。

以下是[Pregel操作](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphOps@pregel[A](A,Int,EdgeDirection)((VertexId,VD,A)⇒VD,(EdgeTriplet[VD,ED])⇒Iterator[(VertexId,A)],(A,A)⇒A)(ClassTag[A]):Graph[VD,ED])的型別簽章（signature）以及實做的草圖（注意，graph.cache呼叫已經移除了）


```scala
class GraphOps[VD, ED] {
  def pregel[A]
      (initialMsg: A,
       maxIter: Int = Int.MaxValue,
       activeDir: EdgeDirection = EdgeDirection.Out)
      (vprog: (VertexId, VD, A) => VD,
       sendMsg: EdgeTriplet[VD, ED] => Iterator[(VertexId, A)],
       mergeMsg: (A, A) => A)
    : Graph[VD, ED] = {
    // Receive the initial message at each vertex
    var g = mapVertices( (vid, vdata) => vprog(vid, vdata, initialMsg) ).cache()
    // compute the messages
    var messages = g.mapReduceTriplets(sendMsg, mergeMsg)
    var activeMessages = messages.count()
    // Loop until no messages remain or maxIterations is achieved
    var i = 0
    while (activeMessages > 0 && i < maxIterations) {
      // Receive the messages: -----------------------------------------------------------------------
      // Run the vertex program on all vertices that receive messages
      val newVerts = g.vertices.innerJoin(messages)(vprog).cache()
      // Merge the new vertex values back into the graph
      g = g.outerJoinVertices(newVerts) { (vid, old, newOpt) => newOpt.getOrElse(old) }.cache()
      // Send Messages: ------------------------------------------------------------------------------
      // Vertices that didn't receive a message above don't appear in newVerts and therefore don't
      // get to send messages.  More precisely the map phase of mapReduceTriplets is only invoked
      // on edges in the activeDir of vertices in newVerts
      messages = g.mapReduceTriplets(sendMsg, mergeMsg, Some((newVerts, activeDir))).cache()
      activeMessages = messages.count()
      i += 1
    }
    g
  }
}
```

注意，Pregel接受兩個參數列表（graph.pregel(list1)(list2)）。第一個參數列表包含了配置參數，如初始訊息、最大的迭代數、訊息發送邊的方向（預設向外）。第二個參數列表包含了用來接收訊息（vprog）、計算訊息（sendMsg）、合併訊息（mergeMsg）。

以下範例是我們可以使用Pregel運算子來表示單源最短路徑（Single source shortest path）的運算。

```scala
import org.apache.spark.graphx._
// Import random graph generation library
import org.apache.spark.graphx.util.GraphGenerators
// A graph with edge attributes containing distances
val graph: Graph[Int, Double] =
  GraphGenerators.logNormalGraph(sc, numVertices = 100).mapEdges(e => e.attr.toDouble)
val sourceId: VertexId = 42 // The ultimate source
// Initialize the graph such that all vertices except the root have distance infinity.
val initialGraph = graph.mapVertices((id, _) => if (id == sourceId) 0.0 else Double.PositiveInfinity)
val sssp = initialGraph.pregel(Double.PositiveInfinity)(
  (id, dist, newDist) => math.min(dist, newDist), // Vertex Program
  triplet => {  // Send Message
    if (triplet.srcAttr + triplet.attr < triplet.dstAttr) {
      Iterator((triplet.dstId, triplet.srcAttr + triplet.attr))
    } else {
      Iterator.empty
    }
  },
  (a,b) => math.min(a,b) // Merge Message
  )
println(sssp.vertices.collect.mkString("\n"))
```
