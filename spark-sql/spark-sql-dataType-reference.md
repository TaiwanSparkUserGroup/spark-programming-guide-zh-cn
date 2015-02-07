# [Spark SQL資料類型](https://spark.apache.org/docs/latest/sql-programming-guide.html#spark-sql-datatype-reference)

- 數字類型
    - ByteType：代表一個位元的整数。範圍是-128到127
    - ShortType：代表兩個位元的整数。範圍是-32768到32767
    - IntegerType：代表4個位元的整数。範圍是-2147483648到2147483647
    - LongType：代表8個位元的整数。範圍是-9223372036854775808到9223372036854775807
    - FloatType：代表4位元的單精度浮點數
    - DoubleType：代表8位元的雙精度浮點數
    - DecimalType：代表任意精度的10進位資料。透過内部的java.math.BigDecimal支援。BigDecimal由一個任意精度的不定長度(unscaled value)的整數和一個32位元整数组成
    - StringType：代表一個字串
    - BinaryType：代表個byte序列值
    - BooleanType：代表boolean值
    - Datetime類型
        - TimestampType：代表包含年，月，日，時，分，秒的值
        - DateType：代表包含年，月，日的值
    - 複雜類型
        - ArrayType(elementType, containsNull)：代表由elementType類型元素組成的序列值。`containsNull` 用來指明 `ArrayType` 中的值是否有null值
        - MapType(keyType, valueType, valueContainsNull)：表示包括一组鍵 - 值組合的值。透過 keyType 表示 key 資料的類型，透過 valueType 表示 value 資料的類型。`valueContainsNull` 用來指明 `MapType` 中的值是否有null值
        - StructType(fields):表示一個擁有 `StructFields (fields)` 序列結構的值
            - StructField(name, dataType, nullable):代表 `StructType` 中的域(field)，field的名字通过 `name` 指定， `dataType` 指定field的資料類型， `nullable` 表示filed的值是否有null值。

Spark的所有資料類型都定義在 `org.apache.spark.sql` 中，你可以透過 `import  org.apache.spark.sql._` 取用它們。

資料類型 | Scala中的值類型 | 取用或者創建資料類型的API
--- | --- | ---
ByteType | Byte | ByteType
ShortType | Short | ShortType
IntegerType | Int | IntegerType
LongType | Long | LongType
FloatType | Float | FloatType
DoubleType | Double | DoubleType
DecimalType | scala.math.BigDecimal | DecimalType
StringType | String | StringType
BinaryType | Array[Byte] | BinaryType
BooleanType | Boolean | BooleanType
TimestampType | java.sql.Timestamp | TimestampType
DateType | java.sql.Date | DateType
ArrayType | scala.collection.Seq | ArrayType(elementType, [containsNull]) 注意 containsNull 預設為 true
MapType | scala.collection.Map | MapType(keyType, valueType, [valueContainsNull]) 注意 valueContainsNull 預設為 true
StructType | org.apache.spark.sql.Row | StructType(fields) ，注意 fields 是一個 StructField 序列，不能使用兩個相同名字的 StructField
StructField | The value type in Scala of the data type of this field (For example, Int for a StructField with the data type IntegerType) | StructField(name, dataType, nullable)
