# [JSON資料集](https://spark.apache.org/docs/latest/sql-programming-guide.html#json-datasets)

Spark SQL能夠自動推斷 JSON 資料集的模式，讀取為 SchemaRDD。這種轉換可以透過下面兩種方法来實現

- jsonFile ：從一個包含 JSON 文件的目錄中讀取。文件中的每一行是一個 JSON 物件
- jsonRDD ：從存在的 RDD 讀取資料，這些 RDD 的每个元素是一個包含 JSON 物件的字串

注意作為*jsonFile*的文件不是一個典型的 JSON 文件，每行必須是獨立的並且包含一個有效的JSON物件。一個多行的JSON物件經常會失敗．

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

// A JSON dataset is pointed to by path.
// The path can be either a single text file or a directory storing text files.
val path = "examples/src/main/resources/people.json"
// Create a SchemaRDD from the file(s) pointed to by path
val people = sqlContext.jsonFile(path)

// The inferred schema can be visualized using the printSchema() method.
people.printSchema()
// root
//  |-- age: integer (nullable = true)
//  |-- name: string (nullable = true)

// Register this SchemaRDD as a table.
people.registerTempTable("people")

// SQL statements can be run by using the sql methods provided by sqlContext.
val teenagers = sqlContext.sql("SELECT name FROM people WHERE age >= 13 AND age <= 19")

// Alternatively, a SchemaRDD can be created for a JSON dataset represented by
// an RDD[String] storing one JSON object per string.
val anotherPeopleRDD = sc.parallelize(
  """{"name":"Yin","address":{"city":"Columbus","state":"Ohio"}}""" :: Nil)
val anotherPeople = sqlContext.jsonRDD(anotherPeopleRDD)
```

