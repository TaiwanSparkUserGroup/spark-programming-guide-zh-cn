# 提交應用程式

在Spark bin 目錄下的`spark-submit` 讓你在集群上啟動應用程式。它可以通過統一接口使用Spark 支援的所有[ 集群管理器](https://spark.apache.org/docs/latest/cluster-overview.html#cluster-manager-types)，所有你不必為每個管理器作相對應的設定。

## 用spark-submit 啟動應用程式

`bin/spark-submit` 指令負責建立包含Spark 以及他所相依的類別路徑(classpath)，他支援不同的集群管理器以及Spark 支持的加載模式。

```shell
./bin/spark-submit \
  --class <main-class>
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  ... # other options
  <application-jar> \
  [application-arguments]
```

常用選項如下：

- `--class`：你的應用程式入口點(如org.apache.spark.examples.SparkPi)
- `--master`：集群的master URL(如spark://23.195.26.187:7077)
- `--deploy-mode`：在worker 節點部署你的driver(cluster) 或者本地作為外部客戶端(client)。預設是client。
- `--conf`：自定的Spark 配置屬性，格式是key=value。
- `application-jar`：包含應用程式以及其相依的jar 包路徑。這個URL 必須在集群中全域可用，例如，存放在所有節點的`hdfs://` 路徑或是`file://` 路徑
- `application-arguments`：傳遞給主類別的參數

常見的部署策略是從網路提交你的應用程式。這種設定之下，適合`client` 模式。在`client` 模式，driver直接在`spark-submit` 中啟動。這樣的方式直接作為集群客戶端。由於應用程式的輸入與輸出都與控制台相連，所以也適合與 REPL 的應用程式。

另一種選擇，如果你的應用程式從一個與worker 機器距離很遠的機器上提交，一般情況下，用`cluster` 模式可減少drivers和executors 的網路延遲。注意，`cluster` 模式目前不支援獨立集群、mesos集群以及python應用程式。

有幾個我們使用的集群管理特有的選項。例如，在Spark讀立即群的`cluster`模式下，你也可以指定`--supervise` 以確保driver 自動重新啟動(如果它因為發生錯誤而退出失敗)。
為了列出spark-submit 所有可用參數，用`--help` 執行。

```shell
# Run application locally on 8 cores
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar \
  100

# Run on a Spark Standalone cluster in client deploy mode
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# Run on a Spark Standalone cluster in cluster deploy mode with supervise
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --deploy-mode cluster
  --supervise
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# Run on a YARN cluster
export HADOOP_CONF_DIR=XXX
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn-cluster \  # can also be `yarn-client` for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000

# Run a Python application on a Spark Standalone cluster
./bin/spark-submit \
  --master spark://207.184.161.138:7077 \
  examples/src/main/python/pi.py \
  1000
```

## Master URLs

傳送給Spark 的url 可用下列模式

Master URL | Meaning
--- | ---
local | 用一個worker 本地執行Spark
local[K] | 用k 個worker 本地執行Spark (理想情況下，數值為機器CPU 的數量)
local[*] | 有多少worker 就用多少，以本地執行Spark
spark://HOST:PORT | 連結到指定Spark 獨立集群master。端口必須是master 配置的端口，預設是7077
mesos://HOST:PORT | 連結到指定的mesos 集群
yarn-client | 以`client` 模式連結到Yarn 集群。集群位置設定在變數HADOOP_CONF_DIR
yarn-cluster | 以`cluster`模式連結到Yarn 集群。集群位置設定在變數HADOOP_CONF_DIR
