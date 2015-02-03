# 暫存或持續化

和RDD相似，DStreams也允許開發者持續化串流資料到記憶體中。在DStream上使用`persist()`函數可以自動地持續化DStream中的RDD到記憶體中。如果DStream中的資料需要計算多次，這是非常有用的。像`reduceByWindow`和`reduceByKeyAndWindow`這種窗口操作、`updateStateByKey`這種基於狀態的操作，持續化是預設的，不需要開發者調用`persist()`函數。

例如藉由網路（如kafka，flume等）獲取的輸入資料串流，預設的持續化策略是複製資料到兩個不同的節點以容錯。

注意，與RDD不同的是，DStreams預設持續化級别是儲存序列化資料到記憶體中，這將在[性能調教](../performance-tuning/README.md)章節介紹。更多的訊息請看[rdd持續化](../../programming-guide/rdds/rdd-persistences.md)