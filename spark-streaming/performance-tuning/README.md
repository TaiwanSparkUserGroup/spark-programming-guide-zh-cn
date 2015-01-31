# 性能調教

集群中的Spark Streaming應用程式獲得最好的性能需要一些調整。這章將介紹幾個參數和配置，提高Spark Streaming應用程式的性能。你需要考慮兩件事情：

- 高效地利用集群资來源減少批次資料的處理時間
- 設定正確的批次大小（size），使資料的處理速度能夠赶上資料的接收速度

* [減少批次資料的執行時間](reducing-processing-time.md)
* [設定正確的批次大小](setting-right-batch-size.md)
* [記憶體調教](memory-tuning.md)