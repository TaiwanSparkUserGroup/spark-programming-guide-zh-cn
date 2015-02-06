# 頂點和邊的RDDs

GraphX揭開了儲存在圖中的頂點和邊的RDD。因為GraphX將頂點和邊保存在優化過的資料結構中，這些資料結構提供了額外的功能，分別傳回`VertexRDD`和`EdgeRDD`。這一章節，我們將學習它們一些有用的功能。

## VertexRDDs

`VertexRDD[A]`繼承`RDD[(VertexID, A)]`並且增加了額外的限制條件，那就是每個`VertexID`只能出現一次。此外，`VertexRDD[A]`代表一組具有型別A特性的頂點。在內部情況下，能透過將頂點屬性儲存到一個可重複使用的hash-map的資料結構即可達成。所以，若兩個`VertexRDDs`是從相同的`VertexRDD`(如藉由`filter`或`mapValues`)基底產生的，它們就能夠在常數時間內完成合併，而不需要透過hash的計算。為了利用索引式的資料結構，`VertexRDD`提供了下列的附加功能：

```scala
class VertexRDD[VD] extends RDD[(VertexID, VD)] {
  // Filter the vertex set but preserves the internal index
  def filter(pred: Tuple2[VertexId, VD] => Boolean): VertexRDD[VD]
  // Transform the values without changing the ids (preserves the internal index)
  def mapValues[VD2](map: VD => VD2): VertexRDD[VD2]
  def mapValues[VD2](map: (VertexId, VD) => VD2): VertexRDD[VD2]
  // Remove vertices from this set that appear in the other set
  def diff(other: VertexRDD[VD]): VertexRDD[VD]
  // Join operators that take advantage of the internal indexing to accelerate joins (substantially)
  def leftJoin[VD2, VD3](other: RDD[(VertexId, VD2)])(f: (VertexId, VD, Option[VD2]) => VD3): VertexRDD[VD3]
  def innerJoin[U, VD2](other: RDD[(VertexId, U)])(f: (VertexId, VD, U) => VD2): VertexRDD[VD2]
  // Use the index on this RDD to accelerate a `reduceByKey` operation on the input RDD.
  def aggregateUsingIndex[VD2](other: RDD[(VertexId, VD2)], reduceFunc: (VD2, VD2) => VD2): VertexRDD[VD2]
}
```

举个例子，`filter`操作如何返回一个VertexRDD。过滤器实际使用一个`BitSet`实现，因此它能够重用索引以及保留和其它`VertexRDDs`做连接时速度快的能力。同样的，`mapValues`操作
不允许`map`函数改变`VertexID`，因此可以保证相同的`HashMap`数据结构能够重用。当连接两个从相同的`hashmap`获取的VertexRDDs和使用线性扫描而不是昂贵的点查找实现连接操作时，`leftJoin`
和`innerJoin`都能够使用。

从一个`RDD[(VertexID, A)]`高效地构建一个新的`VertexRDD`，`aggregateUsingIndex`操作是有用的。概念上，如果我通过一组顶点构造了一个`VertexRDD[B]`，而`VertexRDD[B]`是
一些`RDD[(VertexID, A)]`中顶点的超集，那么我们就可以在聚合以及随后索引`RDD[(VertexID, A)]`中重用索引。例如：

```scala
val setA: VertexRDD[Int] = VertexRDD(sc.parallelize(0L until 100L).map(id => (id, 1)))
val rddB: RDD[(VertexId, Double)] = sc.parallelize(0L until 100L).flatMap(id => List((id, 1.0), (id, 2.0)))
// There should be 200 entries in rddB
rddB.count
val setB: VertexRDD[Double] = setA.aggregateUsingIndex(rddB, _ + _)
// There should be 100 entries in setB
setB.count
// Joining A and B should now be fast!
val setC: VertexRDD[Double] = setA.innerJoin(setB)((id, a, b) => a + b)
```

## EdgeRDDs

`EdgeRDD[ED]`继承自`RDD[Edge[ED]]`，使用定义在[PartitionStrategy](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.PartitionStrategy)的
各种分区策略中的一个在块分区中组织边。在每个分区中，边属性和相邻结构被分别保存，当属性值改变时，它们可以最大化的重用。

`EdgeRDD`暴露了三个额外的函数

```scala
// Transform the edge attributes while preserving the structure
def mapValues[ED2](f: Edge[ED] => ED2): EdgeRDD[ED2]
// Revere the edges reusing both attributes and structure
def reverse: EdgeRDD[ED]
// Join two `EdgeRDD`s partitioned using the same partitioning strategy.
def innerJoin[ED2, ED3](other: EdgeRDD[ED2])(f: (VertexId, VertexId, ED, ED2) => ED3): EdgeRDD[ED3]
```

在大多数的应用中，我们发现，EdgeRDD操作可以通过图操作者(graph operators)或者定义在基本RDD中的操作来完成。
