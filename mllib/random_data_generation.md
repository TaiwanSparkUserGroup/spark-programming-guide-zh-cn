# 隨機數據生成
隨機數據生成對隨機算法，原型及性能測式來說是有用的。MLlib支持指定分類型來生成隨機的RDD，如均勻，標準常態，Possion分布。

[RandomRDDs](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.random.RandomRDDs)提供工廠方法來生成隨機double RDD或vector RDDs。下面示例生成一個隨機的dobule RDD，其值服從標準常態分布$$N(0,1)$$，然後將其映射為$$N(0,1)$$。

```scala
import org.apache.spark.SparkContext
import org.apache.spark.mllib.random.RandomRDDs._

val sc: SparkContext = ...

// Generate a random double RDD that contains 1 million i.i.d. values drawn from the
// standard normal distribution `N(0, 1)`, evenly distributed in 10 partitions.
val u = normalRDD(sc, 1000000L, 10)
// Apply a transform to get a random double RDD following `N(1, 4)`.
val v = u.map(x => 1.0 + 2.0 * x)
```
