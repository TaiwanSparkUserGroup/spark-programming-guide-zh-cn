# 分層抽樣
在 MLlib 中,不同於其他统計方法,分層抽样方法如 sampleByKey 和 sampleByKeyExact，運行在鍵值對格式的 RDD 上。對分層抽样来说，keys是一个標簽, 值是特定的屬性。比如，key 可以是男人或女人、文檔 ID，其相應的值可以是人口數據中 的年齡列表或者文檔中的詞列表。sampleByKey 方法對每一个觀測擲幣決定是否抽中它, 所以需要對数据进行一次遍歷,也需要輸入期望抽样的大小。而 sampleByKeyExact方法並不是簡單地在每一層中使用sampleByKey 方法随機抽样，它需要更多資源，但將提供信心度高達 99.99%的精確抽样大小。在 Python 中，目前不支持 sampleByKeyExact。

[sampleByKeyExact()](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions)充許使用者準確抽取 $$[f_k \cdot n_k]
 \quad\forall k\in\mathbb K $$個元素，這裡的$$f_k$$是從鍵k中期望抽取的比例，$$n_k$$是從鍵k中抽取的鋌值對數量，而$$\mathbb K$$是鍵的集合。為了確保抽樣大小，無放回抽樣對數據會多一次遍歷；然而，有放回的抽樣會多兩次遍歷。

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.rdd.PairRDDFunctions

val sc: SparkContext = ...

val data = ... // an RDD[(K, V)] of any key value pairs
val fractions: Map[K, Double] = ... // specify the exact fraction desired from each key

// Get an exact sample from each stratum
val approxSample = data.sampleByKey(withReplacement = false, fractions)
val exactSample = data.sampleByKeyExact(withReplacement = false, fractions)
```
