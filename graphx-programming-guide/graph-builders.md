# 圖形建構式

GraphX提供了幾種方式從RDD或是硬碟上的頂點和邊建立圖。在預設情況下，圖形建構式不會為圖的邊重新分割，而是把邊保留在預設的區塊中（例如HDFS的原始區塊）。
[Graph.groupEdges](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@groupEdges((ED,ED)⇒ED):Graph[VD,ED])要求重新分割圖形，因為它假定相同的邊會被分配到同一個區塊，所以你必須在使用`groupEdges`前使用[Graph.partitionBy](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@partitionBy(PartitionStrategy):Graph[VD,ED])

```scala
object GraphLoader {
  def edgeListFile(
      sc: SparkContext,
      path: String,
      canonicalOrientation: Boolean = false,
      minEdgePartitions: Int = 1)
    : Graph[Int, Int]
}
```

[GraphLoader.edgeListFile](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphLoader$@edgeListFile(SparkContext,String,Boolean,Int):Graph[Int,Int])提供了一個從硬碟上邊的清單讀取一個圖形的方式。格式如下面範例（起始頂點ID，目標頂點ID），`#`表示註解行。

```scala
# This is a comment
2 1
4 1
1 2
```

從指定的邊建立一個圖，自動建立所有邊提及的所有頂點。所有的頂點和邊的屬性預設都是1。`canonicalOrientation`參數允許重新導向正向（srcId < dstId）的邊。這在[connected components](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.ConnectedComponents$)演算法中需要用到。`minEdgePartitions`參數用來規定邊的分區生成的最小數量。邊分區可能比指定的分區還要多。例如，一個HDFS檔案有更多的區塊。

```scala
object Graph {
  def apply[VD, ED](
      vertices: RDD[(VertexId, VD)],
      edges: RDD[Edge[ED]],
      defaultVertexAttr: VD = null)
    : Graph[VD, ED]
  def fromEdges[VD, ED](
      edges: RDD[Edge[ED]],
      defaultValue: VD): Graph[VD, ED]
  def fromEdgeTuples[VD](
      rawEdges: RDD[(VertexId, VertexId)],
      defaultValue: VD,
      uniqueEdges: Option[PartitionStrategy] = None): Graph[VD, Int]
}
```
[Graph.apply](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph$@apply[VD,ED](RDD[(VertexId,VD)],RDD[Edge[ED]],VD)(ClassTag[VD],ClassTag[ED]):Graph[VD,ED])允許從頂點和邊的RDD上建立一個圖。重複的頂點會被任意地挑出，而只有從邊RDD出來的頂點才會有預設屬性，頂點RDD並不會有。

[Graph.fromEdges](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph$@fromEdges[VD,ED](RDD[Edge[ED]],VD)(ClassTag[VD],ClassTag[ED]):Graph[VD,ED])只允許從一個邊的RDD上建立一個圖，且自動地建立邊提及的頂點，並給予這些頂點預設值。

[Graph.fromEdgeTuples](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph$@fromEdgeTuples[VD](RDD[(VertexId,VertexId)],VD,Option[PartitionStrategy])(ClassTag[VD]):Graph[VD,Int])只允許一個edge tuple組成的RDD上建立一個圖，並給予邊的值為1。自動地建立邊所提及的頂點，並給予這些頂點預設值。它還支援刪除邊，為了刪除邊，需要傳遞一個[PartitionStrategy](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.PartitionStrategy)值為`Some`作為參數`uniqueEdges`的值（如uniqueEdges = some(PartitionStrategy.RandomVertexCut)），要刪除同一分區相同的邊，一個分割策略是必須的。
