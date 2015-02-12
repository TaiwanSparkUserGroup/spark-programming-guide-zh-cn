# 共享變數

一般情況，當傳遞一個操作函數( 例如 map 或 reduce) 給 Spark時，Spark 實際上是操作這個函數變數的副本。這些變數被複製到每台機器上，而且這些變數在遠端機器上的更新都不會傳送回驅動程式。一般來說，跨任務的讀寫操作變數效率不高，但 Spark 還是為了兩種常用模式提供共享變數：廣播變數 ( broadcast variable) 與累加器 ( accumulator)

## 廣播變數

廣播變數允許程式將一個可讀變數存在每台機器的記憶體裡，而不是每個任務都存有一份副本。例如，利用廣播變數，我們能將一個大資料量輸入的集合副本發送到每個節點上。( Broadcast variables allow the programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks.They can be used, for example,
to give every node a copy of a large input dataset in an efficient manner.) Spark 也會試著利用有效率的廣播算法來分配廣播變數，以減少傳遞之間的資源成本。

一個廣播變數可以利用`SparkContext.broadcast(v)` 方法從一個初始化變數v 中產生。廣播變數是v的一個包裝變數，它的值可以透過`value` 方法取得，可參考下列程式：

```scala
 scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
 broadcastVar: spark.Broadcast[Array[Int]] = spark.Broadcast(b5c40191-a864-4c7d-b9bf-d87e1a4e787c)
 scala> broadcastVar.value
 res0: Array[Int] = Array(1, 2, 3)
```
广播变量创建以后，我们就能够在集群的任何函数中使用它来代替变量v，这样我们就不需要再次传递变量v到每个节点上。另外，为了保证所有的节点得到广播变量具有相同的值，对象v不能在广播之后被修改。

廣播變數建好以後，就能夠在集群的任何函數中使用它，來代替變數v，這樣一來，就不用再次傳送變數v 到其他節點上。此外，為了保障所有節點得到的廣播變數值相同，廣播之後就不能對物件v再做修改。

## 累加器

顧名思義，累家氣是一種只能利用關連操作做“加” 操作的變數，因此他能夠快速的執行併行操作。而且他們能夠操作`counters`和`sums`。Spark 原本支援數值類型的累加器，開發人員可以自行增加可被支援的類型。

如果建立一個具名的累加器，它可在 Spark UI 上顯示。這對理解運作階段 ( running stages ) 的過程很有幫助。( 注意：python 中已支援 [AccumulatorPython](http://spark.apache.org/docs/1.0.2/api/python/))

一个累加器可以通过调用`SparkContext.accumulator(v)`方法从一个初始变量v中创建。运行在集群上的任务可以通过`add`方法或者使用`+=`操作来给它加值。然而，它们无法读取这个值。只有驱动程序可以使用`value`方法来读取累加器的值。
如下的代码，展示了如何利用累加器将一个数组里面的所有元素相加：

```scala
scala> val accum = sc.accumulator(0, "My Accumulator")
accum: spark.Accumulator[Int] = 0
scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum += x)
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s
scala> accum.value
res2: Int = 10
```

```python
>>> accum = sc.accumulator(0)
Accumulator<id=0, value=0>
>>> sc.parallelize([1, 2, 3, 4]).foreach(lambda x: accum.add(x))
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s
scala> accum.value
10
```

這個例子利用內建的整數類型累加器，開發者還可以利用[AccumulatorParam](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.AccumulatorParam) 建立自己的累加器。

AccumulatorParam 接口有兩個方法：`zero`方法替你的資料類型提供一個“0 值” ( zero value )；`addInPlace`方法計算兩個值的和。例如，我们有一個`Vector`物件代表數學向量，累加器可以透過下列方式進行定義：

```scala
object VectorAccumulatorParam extends AccumulatorParam[Vector] {
  def zero(initialValue: Vector): Vector = {
    Vector.zeros(initialValue.size)
  }
  def addInPlace(v1: Vector, v2: Vector): Vector = {
    v1 += v2
  }
}
// Then, create an Accumulator of this type:
val vecAccum = sc.accumulator(new Vector(...))(VectorAccumulatorParam)
```

在scala 中，Spark 支援一般[Accumulable](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.Accumulable) 接口來累計數值-結果類型和用於累加的元素類型不同( 例如收集的元素建立列表)。Spark 支援`SparkContext.accumulableCollection` 方法累加一般的scala 集合。
