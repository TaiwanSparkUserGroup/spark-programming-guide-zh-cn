# 頂點和邊的RDDs

GraphX提供了儲存在圖中的頂點和邊的RDD。因為GraphX將頂點和邊保存在優化過的資料結構中，這些資料結構提供了額外的功能，分別傳回`VertexRDD`和`EdgeRDD`。這一章節，我們將學習它們一些有用的功能。

## VertexRDDs

`VertexRDD[A]`繼承了`RDD[(VertexID, A)]`並且新增了額外的限制條件，那就是每個`VertexID`只能出現一次。此外，`VertexRDD[A]`代表一組具有型別A特性的頂點。在程式內部，透過將頂點屬性儲存到一個可重複使用的hash-map的資料結構來達成。所以，若兩個`VertexRDDs`是從相同的`VertexRDD`(如藉由`filter`或`mapValues`)基底產生的，它們就能夠在常數時間內完成合併，而避免了hash的計算。為了利用索引式的資料結構，`VertexRDD`提供了下列的附加功能：

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

舉例，`filter`運算子是如何回一個`VertexRDD`。`filter`實際上是由`BitSet`實作，因此重複使用索引值以及保留快速與其他`VertexRDDs`合併的能力。相同的，`mapValues`運算子不允許`map`函數改變`VertexID`，確保相同的`hashMap`的資料結構被重複使用。當合併兩個從相同`hashMap`得到的`VertexRDDs`且利用線性搜尋（linear scan）而非花費時間較長的點查詢（point lookups）來實現合併時，`leftJoin`和`innerJoin`都能夠使用。

`aggregateUsingIndex`運算子能夠有效率地將一個`RDD[(VertexID, A)]`建造成一個新的`VertexRDD`。概念上，我透過一組為一些`VertexRDD[A]` 的`super-set`頂點建造了`VertexRDD[B]`，那麼我們就能夠在聚合（aggregate）和往後查詢`RDD[(VertexID, A)]`時重複使用索引。例如：

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

`EdgeRDD[ED]`繼承了`RDD[Edge[ED]]`，使用定義在[PartitionStrategy](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.PartitionStrategy)眾多分割方法其中一種，將邊作區塊性的分割。在每個分區中，邊的屬性和周遭結構會被個別的儲存，能夠在屬性改變時，最大化重用。

`EdgeRDD`揭示了三個額外的函數：

```scala
// Transform the edge attributes while preserving the structure
def mapValues[ED2](f: Edge[ED] => ED2): EdgeRDD[ED2]
// Revere the edges reusing both attributes and structure
def reverse: EdgeRDD[ED]
// Join two `EdgeRDD`s partitioned using the same partitioning strategy.
def innerJoin[ED2, ED3](other: EdgeRDD[ED2])(f: (VertexId, VertexId, ED, ED2) => ED3): EdgeRDD[ED3]
```

在多數的應用中，我們會發現`EdgeRDD`的操作可以透過圖形運算子（graph operators）或是定義在基本RDD中的操作來完成。

## Optimized Representation
