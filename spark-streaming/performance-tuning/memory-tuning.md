# 記憶體調優

調整記憶體的使用以及Spark應用程式的垃圾回收行為已經在[Spark優化指南](../../other/tuning-spark.md)中詳細介紹。在這一節，我們重點介紹幾個强烈推薦的自定義選項，它們可以
減少Spark Streaming應用程式垃圾回收的相關暫停，獲得更穩定的批次處理時間。

- Default persistence level of DStreams：和RDDs不同的是，預設的持續化級别是序列化資料到記憶體中（DStream是`StorageLevel.MEMORY_ONLY_SER`，RDD是` StorageLevel.MEMORY_ONLY`）。
即使保存資料為序列化形态會增加序列化/反序列化的開销，但是可以明顯的減少垃圾回收的暫停。
- Clearing persistent RDDs：預設情況下，藉由Spark内置策略（LUR），Spark Streaming生成的持續化RDD將會從記憶體中清理掉。如果spark.cleaner.ttl已經設定了，比這個時間存在更老的持續化
RDD將會被定時的清理掉。正如前面提到的那樣，這個值需要根據Spark Streaming應用程式的操作小心設定。然而，可以設定配置選項`spark.streaming.unpersist`為true來更智能的去持續化（unpersist）RDD。這個
配置使系统找出那些不需要经常保有的RDD，然後去持續化它們。這可以減少Spark RDD的記憶體使用，也可能改善垃圾回收的行為。
- Concurrent garbage collector：使用並發的標記-清除垃圾回收可以進一步減少垃圾回收的暫停時間。盡管並發的垃圾回收會減少系统的整體吞吐量，但是仍然推薦使用它以獲得更穩定的批次處理時間。