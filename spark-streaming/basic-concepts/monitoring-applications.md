# 監控應用程式

除了Spark的監控功能，Spark Streaming增加了一些专有的功能。應用StreamingContext的時候，[Spark web UI](https://spark.apache.org/docs/latest/monitoring.html#web-interfaces)
顯示添加的`Streaming`菜單，用以顯示運行的receivers（receivers是否是存活狀態、接收的紀錄數、receiver錯誤等）和完成的批次的統計訊息（批次處理時間、佇列等待等等）。這可以用來監控
串流應用程式的處理過程。

在WEB UI中的`Processing Time`和`Scheduling Delay`兩個度量指標是非常重要的。第一個指標表示批次資料處理的時間，第二個指標表示前面的批次處理完畢之後，當前批在佇列中的等待時間。如果
批次處理時間比批次間隔時間持續更長或者佇列等待時間持續增加，這就預示系统無法以批次資料產生的速度處理這些資料，整個處理過程滯後了。在這種情況下，考慮減少批次處理時間。

Spark Streaming程式的處理過程也可以藉由[StreamingListener](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.scheduler.StreamingListener)介面來監控，這
個介面允許你獲得receiver狀態和處理時間。注意，這個介面是開發者API，它有可能在未來提供更多的訊息。