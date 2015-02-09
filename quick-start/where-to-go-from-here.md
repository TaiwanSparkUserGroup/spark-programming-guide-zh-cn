# [下一步？](https://spark.apache.org/docs/latest/quick-start.html#where-to-go-from-here)

恭喜你成功運行你的第一個 Spark 應用程式!

- 要深入了解 API，可以從[Spark编程指南](https://spark.apache.org/docs/latest/programming-guide.html)開始，或者從其他的元件開始，例如：Spark Streaming。
- 要讓程式運行在集群(cluster)上，前往[部署概論](https://spark.apache.org/docs/latest/cluster-overview.html)。
- 最後，Spark 在 `examples` 文件目錄里包含了 [Scala](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples), [Java](https://github.com/apache/spark/tree/master/examples/src/main/java/org/apache/spark/examples) 和 [Python](https://github.com/apache/spark/tree/master/examples/src/main/python) 的幾個簡單的例子，你可以直接運行它们：

```
# For Scala and Java, use run-example:
./bin/run-example SparkPi

# For Python examples, use spark-submit directly:
./bin/spark-submit examples/src/main/python/pi.py
```