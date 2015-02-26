# 在YARN上執行Spark

## 配置

大部分是`Spark on YARN`模式提供的配置與其它部署模式提供的配置相同。下面這些是`Spark on YARN`模式提供的配置選擇。

### Spark 屬性

Property Name | Default | Meaning
--- | --- | ---
spark.yarn.applicationMaster.waitTries | 10 | ApplicationMaster等待Spark master的次數以及SparkContext初始化嘗試的次數
spark.yarn.submit.file.replication | HDFS 預設的複製次數（3） | 上傳到HDFS的文件的HDFS複製水準。這些文件包括Spark jar、app jar以及任何分布式記憶體文件/檔案
spark.yarn.preserve.staging.files | false | 預設為true，在作業結束時保留階段性文件（Spark jar、app jar以及任何分布式記憶體文件）而非刪除他們
spark.yarn.scheduler.heartbeat.interval-ms | 5000 | Spark application master給YARN ResourceManager傳送心跳的時間間隔（ms）
spark.yarn.max.executor.failures | numExecutors * 2,最小為3 | 失敗應用程式之前最大的執行失敗次數
spark.yarn.historyServer.address | (none) | Spark歷史服務器（如host.com:18080）的位置。這位置不應該包含一個模式（http://）。預設情況下下沒有設定值，是因為該選項是一個可選選項。當Spark應用程式完成以ResourceManager UI到Spark歷史服務器UI的連結時，這個位址可從YARN ResourceManager得到
spark.yarn.dist.archives | (none) | 抓取逗號分隔的檔案列表到每一個執行器的工作目錄
spark.yarn.dist.files | (none) | 放置逗號分隔的文件列表到每個執行器的工作目錄
spark.yarn.executor.memoryOverhead | executorMemory * 0.07,最小384 | 配給每個執行器的記憶體大小（以MB為單位）。它屬VM 消耗、interned字符串或者其他本地消耗用的記憶體。這往往隨著執行器大小而增加。（典型情況下是6%-10%）
spark.yarn.driver.memoryOverhead | driverMemory * 0.07,最小384 | 分配给每個driver的記憶體大小（以MB為單位）。它屬VM 消耗、interned字符串或者其他本地消耗用的記憶體。這會隨著執行器大小而增加。（典型情況下是6%-10%）
spark.yarn.queue | default | 應用程式傳送到的YARN 隊列的名稱
spark.yarn.jar | (none) | Spark jar文件的位置，會覆蓋預設的位置。預設情況下，Spark on YARN將會用到本地安裝的Spark jar。但是Spark jar也可以是HDFS中的一個公共位置。這讓YARN記憶體它到節點上，而不用在每次運作應用程式時都需要分配。指向HDFS中的jar包，可以這個參數為"hdfs:///some/path"
spark.yarn.access.namenodes | (none) | 你的Spark應用程式訪問的HDFS namenode列表。例如，`spark.yarn.access.namenodes=hdfs://nn1.com:8032,hdfs://nn2.com:8032`，Spark應用程式必須訪問namenode列表，Kerberos必須正確被配置以訪問他們。Spark獲得namenode的安全通過，這樣Spark應用程式就能訪問這些遠端的HDFS集群。
spark.yarn.containerLauncherMaxThreads | 25 | 為了啟動執行者容器，應用程式master用到的最大線程數量
spark.yarn.appMasterEnv.[EnvironmentVariableName] | (none) | 增加通過`EnvironmentVariableName`指定的環境變量到Application Master處理YARN上的啟動。用户可以指定多個設定，從而設定多個環境變數。在yarn-cluster模式下，這控制著Spark driver的環境。在yarn-client模式下，這僅僅控制執行器啟動者的環境。

## 在YARN上啟動Spark

確保`HADOOP_CONF_DIR`或`YARN_CONF_DIR`指向的目錄包含Hadoop集群的（客戶端）配置文件。這些配置用於寫資料到dfs與YARN ResourceManager。

有兩種佈署模式可以用在YARN上啟動Spark應用程式。在yarn-cluster模式下，Spark driver執行在application master中，這個程序被集群中的YARN所管理，客戶端會在初始化應用程式
之后关闭。在yarn-client模式下，driver运行在客户端进程中，application master仅仅用来向YARN请求资源。

和Spark獨立模式以及Mesos模式不同，這些模式中，master的位置由"master"參數指定，而在YARN模式下，ResourceManager的位置從Hadoop配置取得。因此master參數是簡單的`yarn-client`和`yarn-cluster`。

在yarn-cluster模式下啟動Spark應用程式。

```shell
./bin/spark-submit --class path.to.your.Class --master yarn-cluster [options] <app jar> [app options]
```

例如：
```shell
$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn-cluster \
    --num-executors 3 \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    --queue thequeue \
    lib/spark-examples*.jar \
    10
```

以上啟動了一個YARN客戶端程式用來啟動預設的 Application Master，然後SparkPi會作為Application Master的子線程運作。客戶端會定期訪問Application Master作狀態更新並且江更新顯示在控制台上。一旦應用程式執行完畢，客戶端就會退出。

在yarn-client模式下啟動Spark應用程式，執行下面的shell脚本

```shell
$ ./bin/spark-shell --master yarn-client
```

### 增加其他的jar

在yarn-cluster模式下，driver執行在不同機器上，所以離開了保存在本地客戶端的文件，`SparkContext.addJar`將不會作事。為了讓`SparkContext.addJar`用到保存在客戶端的文件，
可以在啟動命令列上加`--jars`選項。
```shell
$ ./bin/spark-submit --class my.main.Class \
    --master yarn-cluster \
    --jars my-other-jar.jar,my-other-other-jar.jar
    my-main-jar.jar
    app_arg1 app_arg2
```

## 注意事項

- 在Hadoop 2.2之前，YARN不支持容器核的資源請求。因此，當執行早期版本時，通過命令行參數指定CPU的數量無法傳遞到YARN。在調度決策中，CPU需求是否成功兌現取決於用哪個調度器以及如何配置他
- Spark executors使用的本地目錄將會YARN配置（yarn.nodemanager.local-dirs）的本地目錄。如果用戶指定了`spark.local.dir`，他將被忽略。
- `--files`和`--archives`選項支援指定帶有 * # * 符號的文件名稱。例如，你能夠指定`--files localtest.txt#appSees.txt`，它上傳你在本地命名為`localtest.txt`的文件到HDFS，但是將會連結道明稱為`appSees.txt`。當你的應用程式執行在YARN上時，你應該用`appSees.txt`去引用該文件。
- 如果你在yarn-cluster模式下執行`SparkContext.addJar`，並且用到了本地文件， `--jars`選項允許`SparkContext.addJar`函數能夠工作。如果你正在使用 HDFS, HTTP, HTTPS或FTP，你不需要用到該選項

