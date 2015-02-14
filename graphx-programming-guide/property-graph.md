# [屬性圖](http://spark.apache.org/docs/1.2.0/graphx-programming-guide.html#the-property-graph)

[屬性圖](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph)是一個有向多重圖，它帶有連接到每個節點和邊的用戶定義的對象。
有向多重圖中多個平行(parallel)的邊共享相同的來源和目的地節點。支持平行邊的能力簡化了建模場景，這個場景中，相同的節點存在多種關係(例如co-worker和friend)。每個節點由一个
唯一的64位元的辨識碼（VertexID）作為key。GraphX並沒有對節點辨識碼限制任何的排序。同樣，節點擁有相應的來源和目的節點辨識碼。

屬性圖通過vertex(VD)和edge(ED)類型参数化，這些類型是分别與每個節點和邊相關聯的物件類型。

在某些情况下，在相同的圖形中，可能希望節點擁有不同的屬性類型。這可以通過繼承完成。例如，將用戶和產品視為一個二分圖，我們可以用以下方式

```scala
class VertexProperty()
case class UserProperty(val name: String) extends VertexProperty
case class ProductProperty(val name: String, val price: Double) extends VertexProperty
// The graph might then have the type:
var graph: Graph[VertexProperty, String] = null
```
和RDD一樣，屬性圖是不可變的、分布式的、容错的。圖的值或者結構的改變需要按期望的生成一個新的圖来實現。注意，原始圖的實質的部分(例如:不受影響的結構，屬性和索引)都可以在新圖中重用，用來减少這種內在的功能數據結構的成本。
執行者使用一系列節點分區試探法來對圖進行分區。如RDD一樣，圖中的每個分區可以在發生故障的情況下被重新創建在不同的機器上。

邏輯上的屬性圖對應於一對類型化的集合(RDD),這個集合編碼了每一個節點和邊的屬性。因此，圖類別包含訪問圖中的節點和邊的成员。

```scala
class Graph[VD, ED] {
  val vertices: VertexRDD[VD]
  val edges: EdgeRDD[ED]
}
```

`VertexRDD[VD]`和`EdgeRDD[ED]`類別分别繼承和最佳化自`RDD[(VertexID, VD)]`和`RDD[Edge[ED]]`。`VertexRDD[VD]`和`EdgeRDD[ED]`都支持額外的功能來建立在圖計算和利用内部最佳化。

## 屬性圖的例子

在GraphX項目中，假設我們想建構以下包括不同合作者的屬性圖。節點屬性可能包含用戶名和職業．我們可以用字串在邊上標記個合作者間的關係。

![屬性圖](../img/property_graph.png)

所得的圖形將具有類型簽名

```scala
val userGraph: Graph[(String, String), String]
```
有很多方式從一個原始文件、RDDs建構一個屬性圖。最一般的方法是利用[Graph object](https://sp變rk.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph$)。
下面的程式從RDDs產生屬性圖。

```scala
// Assume the SparkContext has already been constructed
val sc: SparkContext
// Create an RDD for the vertices
val users: RDD[(VertexId, (String, String))] =
  sc.parallelize(Array((3L, ("rxin", "student")), (7L, ("jgonzal", "postdoc")),
                       (5L, ("franklin", "prof")), (2L, ("istoica", "prof"))))
// Create an RDD for edges
val relationships: RDD[Edge[String]] =
  sc.parallelize(Array(Edge(3L, 7L, "collab"),    Edge(5L, 3L, "advisor"),
                       Edge(2L, 5L, "colleague"), Edge(5L, 7L, "pi")))
// Define a default user in case there are relationship with missing user
val defaultUser = ("John Doe", "Missing")
// Build the initial Graph
val graph = Graph(users, relationships, defaultUser)
```
在上面的例子中，我們用到了以下[Edge](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Edge)案例類別(case class)。邊有一個`srcId`和`dstId`分別對應於來源和目標節點的辨識碼。另外， `Edge` 類別有一個 `attr` 成員用来存儲存邊的屬性。

我們可以透過 `graph.vertices` 和 `graph.edges` 成員將一圖圖解構為相應的節點和邊。

```scala
val graph: Graph[(String, String), String] // Constructed from above
// Count all users which are postdocs
graph.vertices.filter { case (id, (name, pos)) => pos == "postdoc" }.count
// Count all the edg變s where src > dst
graph.edges.filter(e => e.srcId > e.dstId).count
```

```
注意，graph.vertices 返回一個 VertexRDD[(String, String)]，它繼承於 RDD[(VertexID, (String, String))]。所以我們可以用以下scala的case案例類別解構這個元組(tuple)。另一方面，
graph.edges 返回一個包含 Edge[String] 物件的 EdgeRDD。我們也可以使用案例類別建構子，如下例所示。
```

```scala
graph.edges.filter { case Edge(src, dst, prop) => src > dst }.count
```


除了屬性圖的節點和邊的檢視表，GraphX也包含了一個三元組(triplet)的檢視表，三元檢視表邏輯上將節點和邊的屬性保存為一個 `RDD[EdgeTriplet[VD, ED]]` ，它包含[EdgeTriplet](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.EdgeTriplet)類別的實例。
可以通過下面的Sql表達式表示這個連接(join)。
```sql
SELECT src.id, dst.id, src.attr, e.attr, dst.attr
FROM edges AS e LEFT JOIN vertices AS src, vertices AS dst
ON e.srcId = src.Id AND e.dstId = dst.Id
```

或者通過下面的圖來表示。

![triplet](../img/triplet.png)

`EdgeTriplet` 類別繼承於 `Edge` 類別，並且加入了 `srcAttr` 和 `dstAttr` 成员，這兩個成員分別包含來源和目的的屬性。我們可以用以下三元組檢視表產生字串集合用來描述用戶之間的關係。

```scala
val graph: Graph[(String, String), String] // Constructed from above
// Use the triplets view to create an RDD of facts.
val facts: RDD[String] =
  graph.triplets.map(triplet =>
    triplet.srcAttr._1 + " is the " + triplet.attr + " of " + triplet.dstAttr._1)
facts.collect.foreach(println(_)變
```


