# 自定義receiver指南

Spark Streaming可以從包括内置資料來源在内的任意資料來源獲取資料（其他資料來源包括flume，kafka，kinesis，文件，socket等等）。這需要開發者去實作一個定製`receiver`從具體的資料來源接收
資料。本指南介紹了實作自定義`receiver`的過程，以及怎樣將`receiver`用到Spark Streaming應用程式中。

## 實作一個自定義的Receiver

這一節開始實作一個[Receiver](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.receiver.Receiver)。一個自定義的receiver必須繼承
這個抽象類別,實作它的兩個函數`onStart()`（開始接收資料）以及`onStop()`（停止接收資料）。

需要注意的是`onStart()`和`onStop()`不能夠無限期的阻塞。通常情況下，`onStart()`啟動執行緒負責資料的接收，`onStop()`確保這個接收過程停止。接收執行緒也能夠調用`receiver`的`isStopped`
函數去檢查是否已經停止接收資料。

一旦接收了資料，這些資料就能夠藉由調用`store(data)`函數存到Spark中，`store(data)`是[Receiver]中的函數。有幾個overload的`store()`函數允許你儲存接收到的資料（record-at-a-time or as whole collection of objects/serialized bytes）

在接收執行緒中出現的任何例外都應該被捕捉或者妥善處理從而避免`receiver`在没有提示的情況下失敗。`restart(<exception>)`函數將會重新啟動`receiver`，它藉由非同步的方式首先調用`onStop()`函數，
然後在一段延遲之後調用`onStart()`函數。`stop(<exception>)`將會調用`onStop()`函數终止`receiver`。`reportError(<error>)`函數在不停止或者重啟`receiver`的情況下印出錯誤訊息到
驅動程式(driver)。

如下所示，是一個自定義的`receiver`，它藉由socket接收文本資料串流。它用分界符'\n'把文本流分割為行紀錄，然後將它們儲存到Spark中。如果接收執行緒碰到任何連接或者接收錯誤，`receiver`將會
重新啟動以嘗試再一次連接。

```scala

class CustomReceiver(host: String, port: Int)
  extends Receiver[String](StorageLevel.MEMORY_AND_DISK_2) with Logging {

  def onStart() {
    // Start the thread that receives data over a connection
    new Thread("Socket Receiver") {
      override def run() { receive() }
    }.start()
  }

  def onStop() {
   // There is nothing much to do as the thread calling receive()
   // is designed to stop by itself isStopped() returns false
  }

  //Create a socket connection and receive data until receiver is stopped
  private def receive() {
    var socket: Socket = null
    var userInput: String = null
    try {
     // Connect to host:port
     socket = new Socket(host, port)
     // Until stopped or connection broken continue reading
     val reader = new BufferedReader(new InputStreamReader(socket.getInputStream(), "UTF-8"))
     userInput = reader.readLine()
     while(!isStopped && userInput != null) {
       store(userInput)
       userInput = reader.readLine()
     }
     reader.close()
     socket.close()
     // Restart in an attempt to connect again when server is active again
     restart("Trying to connect again")
    } catch {
     case e: java.net.ConnectException =>
       // restart if could not connect to server
       restart("Error connecting to " + host + ":" + port, e)
     case t: Throwable =>
       // restart if there is any other error
       restart("Error receiving data", t)
    }
  }
}

```

## 在Spark串流應用程式中使用自定義的receiver

在Spark串流應用程式中，用`streamingContext.receiverStream(<instance of custom receiver>)`函數，可以使用自動用`receiver`。程式碼如下所示：

```scala
// Assuming ssc is the StreamingContext
val customReceiverStream = ssc.receiverStream(new CustomReceiver(host, port))
val words = lines.flatMap(_.split(" "))
...
```

完整的程式碼見例子[CustomReceiver.scala](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/CustomReceiver.scala)

## Receiver可靠性

基於Receiver的穩定性以及容錯語意，Receiver分為兩種類型

- Reliable Receiver：可靠的來源允許發送的資料被確認。一個可靠的receiver正確的響應一個可靠的來源，資料已經收到並且被正確地複製到了Spark中（指正確完成複製）。實作這個receiver並
仔細考慮來源確認的語意。
- Unreliable Receiver ：這些receivers不支援響應。即使對於一個可靠的來源，開發者可能實作一個非可靠的receiver，這個receiver不會正確響應。

為了實作可靠receiver，你必須使用`store(multiple-records)`去保存資料。保存的類型是阻塞訪問，即所有给定的紀錄全部保存到Spark中後才返回。如果receiver的配置儲存級别利用複製
(預設情況是複製)，則會在複製结束之後返回。因此，它確保資料被可靠地儲存，receiver恰當的響應给來源。這保證在複製的過程中，没有資料造成的receiver失敗。因為緩衝區資料不會響應，從而
可以從來源中重新獲取資料。

一個不可控的receiver不必實作任何這種邏輯。它簡單地從來源中接收資料，然後用`store(single-record)`一次一個地保存它們。雖然它不能用`store(multiple-records)`獲得可靠的保證，
它有下面一些優勢：

- 系统注重分區塊，將資料分為適當大小的區塊。
- 如果指定了速率的限制，系统注重控制接收速率。
- 因為以上兩點，不可靠receiver比可靠receiver更容易實作。

下面是兩類receiver的特徵
Receiver Type | Characteristics
--- | ---
Unreliable Receivers | 實作簡單；系统更關心區塊的生成和速率的控制；没有容錯的保證，在receiver失敗時會丢失資料
Reliable Receivers | 高容錯保證，零資料丢失；區塊的生成和速率的控制需要手動實作；實作的複雜性依賴來源的確認機制

## 實作和使用自定義的基於actor的receiver

自定義的Akka actor也能夠擁有接收資料。[ActorHelper](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.receiver.ActorHelper)trait可以
應用於任何Akka actor中。它運行接收的資料藉由調用`store()`函數儲存於Spark中。可以配置這個actor的監控(supervisor)策略處理錯誤。

```scala
class CustomActor extends Actor with ActorHelper {
  def receive = {
   case data: String => store(data)
  }
}
```

利用這個actor，一個新的輸入資料串流就能夠被創建。

```scala
// Assuming ssc is the StreamingContext
val lines = ssc.actorStream[String](Props(new CustomActor()), "CustomReceiver")
```

完整的程式碼例子[ActorWordCount.scala](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/ActorWordCount.scala)
