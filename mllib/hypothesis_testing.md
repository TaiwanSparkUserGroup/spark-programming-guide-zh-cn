# 假設檢定
在統計分析中，假設檢定是一個強大的工具，用來判斷結果的統計量是否充分，以及結果是否隨機。MLlib目前支持Pearson卡方檢定($$\chi^2$$) 來檢定適配度和獨立性。輸入數據類型決定了是否產生適配度或獨立性，適配度檢定需要Vector輸入類型，而獨立性檢定需要一個Matrix矩陣輸入。

MLlib也支持RDD[LabeledPoint]輸入類型，然後使用卡方獨立性檢定來進行特徵選擇。

[Statistics](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.stat.Statistics$)提供了進行Pearson卡方檢定的方法。下面示例演示了怎樣運行和解釋假設定。
```scala
import org.apache.spark.SparkContext
import org.apache.spark.mllib.linalg._
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.stat.Statistics._

val sc: SparkContext = ...

val vec: Vector = ... // a vector composed of the frequencies of events

// compute the goodness of fit. If a second vector to test against is not supplied as a parameter,
// the test runs against a uniform distribution.
val goodnessOfFitTestResult = Statistics.chiSqTest(vec)
println(goodnessOfFitTestResult) // summary of the test including the p-value, degrees of freedom,
                                 // test statistic, the method used, and the null hypothesis.

val mat: Matrix = ... // a contingency matrix

// conduct Pearson's independence test on the input contingency matrix
val independenceTestResult = Statistics.chiSqTest(mat)
println(independenceTestResult) // summary of the test including the p-value, degrees of freedom...

val obs: RDD[LabeledPoint] = ... // (feature, label) pairs.

// The contingency table is constructed from the raw (feature, label) pairs and used to conduct
// the independence test. Returns an array containing the ChiSquaredTestResult for every feature
// against the label.
val featureTestResults: Array[ChiSqTestResult] = Statistics.chiSqTest(obs)
var i = 1
featureTestResults.foreach { result =>
    println(s"Column $i:\n$result")
    i += 1
} // summary of the test
```
