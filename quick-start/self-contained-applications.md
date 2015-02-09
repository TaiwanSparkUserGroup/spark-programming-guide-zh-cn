# [獨立應用程式](https://spark.apache.org/docs/latest/quick-start.html#self-contained-applications)

現在假設我們想要使用 Spark API 寫一個獨立的應用程式。我们將通過使用 Scala(用 SBT)，Java(用 Maven) 和 Python 寫一個簡單的應用程式來學習。

我们用 Scala 創建一個非常簡單的 Spark 應用程式。如此簡單，他的名字是 `SimpleApp.scala`：

```scala
/* SimpleApp.scala */
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf

object SimpleApp {
  def main(args: Array[String]) {
    val logFile = "YOUR_SPARK_HOME/README.md" // 應該是你系統上的某個文件
    val conf = new SparkConf().setAppName("Simple Application")
    val sc = new SparkContext(conf)
    val logData = sc.textFile(logFile, 2).cache()
    val numAs = logData.filter(line => line.contains("a")).count()
    val numBs = logData.filter(line => line.contains("b")).count()
    println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
  }
}
```

這個程式僅僅是看 Spark README 中計算行裡面包含 'a' 和包含 'b' 的次數。你需要注意將 `YOUR_SPARK_HOME` 替換成你已經安裝 Spark 的路徑。不像之前的 Spark Shell 例子，這裡初始化了自己的 SparkContext，我們把 SparkContext 初始化做為程式的一部分。

我們通過 SparkContext 的建構函數傳入 [SparkConf](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf) 物件，這個物件包含了一些關於我們程式的訊息。

我們的程式依賴於 Spark API，所以我們需要包含一個 sbt 設定文件，`simple.sbt` ．這個文件解釋了 Spark 是一個依賴(dependency)。這個文件還需要補充 Spark 依賴於一個 repository：

```scala
name := "Simple Project"

version := "1.0"

scalaVersion := "2.10.4"

libraryDependencies += "org.apache.spark" %% "spark-core" % "1.2.0"
```

要讓 sbt 正确工作，我们需要把 `SimpleApp.scala` 和 `simple.sbt` 按照標準的文件目錄結構配置。做好之後，我們可以把程序的代碼創建成一個 JAR 套件。然后使用 `spark-submit` 來運行我們的程式。

```
# Your directory layout should look like this
$ find .
.
./simple.sbt
./src
./src/main
./src/main/scala
./src/main/scala/SimpleApp.scala

# Package a jar containing your application
$ sbt package
...
[info] Packaging {..}/{..}/target/scala-2.10/simple-project_2.10-1.0.jar

# Use spark-submit to run your application
$ YOUR_SPARK_HOME/bin/spark-submit \
  --class "SimpleApp" \
  --master local[4] \
  target/scala-2.10/simple-project_2.10-1.0.jar
...
Lines with a: 46, Lines with b: 23
```
