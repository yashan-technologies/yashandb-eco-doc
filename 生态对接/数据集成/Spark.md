Apache Spark是一款开源的分布式计算引擎，适用于大规模数据处理。YashanDB提供了两种和Spark工具的对接方式，如下：

1. 通过载入YashanDB JDBC驱动，可实现Spark连接YashanDB并读取数据。
2. 通过载入YashanDB JDBC驱动以及YashanDB Spark Connector组件，可实现Spark连接YashanDB并读取/写入数据。

## Spark读取YashanDB数据的对接

本章节将以创建一个Maven项目为例，介绍这一对接实现过程。

### 对接前准备

在进行对接操作前，您需要先准备好如下事项（以下版本不做严格要求，可按照需要灵活选择）：

1. 已安装Jdk11的Java应用环境
2. 已安装Scala 2.12
3. 已安装SparkSql 3.0.2
4. 已安装Maven 3.8
5. 如您的应用所在操作系统为Windows，则需从[Github开源仓](https://github.com/cdarlint/winutils)下载winutils 3.0.2
6. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包
7. 已存在一个可正常访问的YashanDB服务端。

### 对接配置

请参照如下步骤进行YashanDB与Spark的对接配置：

1. 检查Maven核心配置文件pom.xml中是否指定了如下依赖项，没有则加上：

```xml
 <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>3.0.2</version>
            <scope>provided</scope>
        </dependency>
   </dependencies>
```

2. 以IDEA编辑器为例，从IDEA的菜单中，选择【 File > Project Structure > Libraries】
3. 点击【+】，并选择【Java】，从本地选择已准备的YashanDB JDBC驱动包完成库添加；如为多模块项目，只需要选择相应的模块执行本操作。

### 简单使用示例

上述对接配置完成后，开发者即可开始在Spark中连接YashanDB并读取数据，以下为简单的使用示例（请将your_host、your_port、your_dbname、your_name、your_password和your_table修改为实际值）：

```java
import org.apache.spark.SparkConf
import org.apache.spark.sql.SparkSession

object SparkTest {

  def main(args: Array[String]): Unit = {
    //Windows平台必须指定hadoop.home.dir，否则可能出现Failed to locate the winutils binary in the hadoop binary path错误；请更换为实际路径
    System.setProperty("hadoop.home.dir", "F:\\hadoop\\winutils-master\\hadoop-3.0.2")

    val sparkConf = new SparkConf()
    sparkConf.setAppName("test0001")
    sparkConf.setMaster("local[2]")

    val sparkSession = SparkSession.builder()
      .config("spark.sql.warehouse.dir", "spark-warehouse")
      .config(sparkConf)
      .getOrCreate()

    val jdbcDF = sparkSession.read
      .format("jdbc")
      .option("url", "jdbc:yasdb://your_host:your_port/your_dbname")
      .option("driver", "com.yashandb.jdbc.Driver")
      .option("user", "your_name")
      .option("password", "your_password")
      .option("dbtable", "your_table")
      .load()

    jdbcDF.show()

  }

}
```

## Spark读取/写入YashanDB数据的对接

### 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装Java 1.8应用环境
2. 已安装Scala 2.11或2.12
3. 已安装Spark 3.2~3.5
4. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包
5. 已向我们的技术支持人员获取YashanDB Spark Connector组件包
6. 已存在一个可正常访问的YashanDB服务端。

### 对接配置

请参照如下步骤进行YashanDB与Spark的对接配置：

1. 找到Spark软件所在安装目录下的jars目录，将YashanDB JDBC驱动包和YashanDB Spark Connector组件包（均为.jar文件）放至该目录下。
2. 重启Spark。

### 简单使用示例

上述对接配置完成后，开发者即可开始在Spark中连接YashanDB并读取/写入数据，以下为一个简单的连接示例（请将your_host、your_port、your_dbname、your_name、your_password和your_table修改为实际值）：

::: tabs

== SQL

```java
create TEMPORARY VIEW spark_yashandb USING yashandb OPTIONS(
  "url"="jdbc:yasdb://your_host:your_port/your_dbname",
  "dbtable"="your_table",
  "user"="your_name",
  "password"="your_password"
);

select * from spark_yashandb;
```

== DataFrame

```java
val yasdata = spark.read.format("yashandb")
.option("url","jdbc:yasdb://your_host:your_port/your_dbname")
.option("user","your_name")
.option("password","your_password")
.option("dbtable","your_table")
.option("fetchsize",10240)
.load();

yasdata.show(5)
```

:::

### 参数说明

Spark连接YashanDB执行读取/写入操作时，有一系列参数可定义，以下列出它们的名称和含义等信息：


| 参数名                                | 默认值 | 参数说明             | 参数生效范围      |
| --------------------------------------- | ------ |--------------------------------------------------------| ---------- |
| url                                     | (none) | YashanDB数据库的JDBC URL   |  读取数据、写入数据 |
| ipPort | (none) | IP和端口，如”192.168.126.100:1688“ | 写入数据 |
| dbtable                                 | (none) | 待读取或写入的JDBC表。在读取路径中使用该参数时，可以配置为SQL查询的FROM子句中有效的任何内容，例如使用括号中的子查询来代替完整的表<br/>不允许同时指定数据库表和查询选项      |  读取数据、写入数据 |
| user                                    | (none) | 连接用户名   |  读取数据、写入数据 |
| password                                | (none) | 连接密码                                  |  读取数据、写入数据 |
| batchsize                               | 1000   | JDBC批处理大小，该参数值也将决定每次往返写入数据的行数                                                     |  写入数据      |
| query                                   | (none) | 用于将数据读取到Spark中的查询语句，该参数指定的查询语句将用括号包围并用作FROM子句中的子查询，Spark将为该子查询分配一个别名。<br/>* 不允许同时指定数据库表和查询选项<br/>* 不允许同时指定query和partitionColumn参数。如仍需指定partitionColumn参数，可以使用dbtable参数指定子查询并用子查询别名限定相应分区列 |  读取数据、写入数据 |
| writeMode                               | INSERT | YashanDB的写入模式，支持INSERT、BULKINSERT或BULKUPSERT。INSERT为普通写入模式，BULKINSERT和BULKUPSERT为bulkload模式                       |  写入数据      |
| partitionColumn, lowerBound, upperBound | (none) | 若需并行查询分区列，则必须指定partitionColumn（指定分区列）、lowerBound（分区下限值）、upperBound（分区上限值）以及numPartitions（可并行的最大分区数）<br/>* partitionColumn必须指定为相关表中的数字、日期或时间戳列<br/>* lowerBound和upperBound用于决定分区步长，并非筛选表中的行（表中的所有行都将被分区并返回）   | 读取数据       |
| numPartitions                           | (none) | 在表读取和写入过程中可用于并行的最大分区数，该参数值也将决定并发JDBC连接的最大数量。若待写入的分区数超过该参数值，会在写入前调用coalize（numPartitions）减少待写入分区数 | 读取数据       |
| queryTimeout                            | 0      | 驱动程序等待Statement对象执行的时间（单位：秒），0表示没限等待                                                                 |  读取数据、写入数据 |
| fetchsize                               | 1000   | JDBC获取大小，该参数值也将决定每次往返读取数据的行数                                                                | 读取数据       |
| truncate                                | false  | 是否使用truncate table代替drop table操作      |  写入数据      |
| enablePartitionQuery                    | true   | 是否开启分区查询，开启后连接器会将数据库中分区表的每个子分区视作一个分区进行分区查询，分区查询只对分区表生效。单并行模式下，建议关闭分区查询       | 读取数据       |
| enableRouteQuery                        | false  | 是否开启route排序查询，开启后连接器会根据route排序分区。如需开启，请确保YashanDB为分布式部署且连接用户具备访问route$系统表的权限      | 读取数据       |
| readerBufferSize                        | 1024   | connector读的缓存队列的大小                                                                                    | 读取数据      |
| read.fields                             | (none) | 需读取数据的表的字段清单，多个字段名间用逗号`,`分隔      | 读取数据      |
| write.fields                            | (none) | 需写入数据的表的字段或字段顺序，多个字段名间用逗号`,`分隔。默认写入时需按照表字段顺序写入全部字段                                                        |  写入数据      |
| filterQuery                             | (none) | 过滤读取数据的表达式，此表达式拼接在查询语句的where子句后，查询时使用此表达式完成源端数据过滤                                                    | 读取数据      |
| pushDownPredicate                       | true   | 是否开启谓词下推到JDBC数据源。开启时Spark将尽可能向下推送YashanDB数据源的过滤器，否则不会向YashanDB数据源下推任何筛选器（所有筛选器都将由Spark处理）。当Spark执行谓词筛选的速度高于YashanDB数据源时，建议关闭谓词下推   | 读取数据      |
| enableJdbcWrite                         | true  | 是否开启JDBC写入模式，关闭时使用FastLoad写入模式                                                                  |  写入数据      |
| parallel.binder | 3      | JDBC写入模式下，binder线程的线程数，如果Binder线程成为瓶颈，可以适当调大此参数。 |  写入数据 |
| batchesPerTxn   | 3      | JDBC写入模式下，每个事务中的数据批数，导入为可变数据时，不建议调大此值；导入为稳态数据时，建议将此值调大至百级或千级。 |  写入数据 |
| fastload.sendCountAtOnce  | 10000                   | FastLoad写入模式下，FastLoad每次发送给服务端的行数                       |  写入数据 |
| fastload.maxWaitLineCount | 10000 \*（CPU核心数*2+1） | FastLoad写入模式下，FastLoad缓冲队列中最大行数                           |  写入数据 |
| fastload.commitCount      | 20000                   | FastLoad写入模式下，提交行数，为0则表示不中途提交                      |  写入数据 |
| fastload.closeTimeout     | 3000ms                  | FastLoad写入模式下，FastLoad关闭的超时等待时间                           |  写入数据 |
| fastload.senderCount      | 所在机器的CPU核心数     | FastLoad写入模式下，FastLoad的并行sender的数量                           |  写入数据 |
| fastload.maxReaderCount   | 所在机器的CPU核心数/2   | FastLoad写入模式下，FastLoad的并行reader的数量                           |  写入数据 |
| fastload.putDataWaitTime  | 10ms                    | FastLoad写入模式下，writer线程向FastLoad的缓冲队列发送数据的超时等待时间 |  写入数据 |

### 数据类型映射

Spark和YashanDB均有各自的一套数据类型体系，以下列出YashanDB Spark Connector组件对它们的映射定义：

| YashanDB Type | Spark Type                                      |
| ------------- | ----------------------------------------------- |
| Boolean       | BooleanType                                     |
| TINYINT       | IntegerType                                     |
| SMALLINT      | IntegerType                                     |
| INTEGER       | IntegerType                                     |
| BIGINT        | LongType                                        |
| DOUBLE        | DoubleType                                      |
| FLOAT         | FloatType                                       |
| REAL          | DoubleType                                      |
| NUMBER        | createDecimalType() or scale == null StringType |
| NUMERIC       | createDecimalType() or scale == null StringType |
| DECIMAL       | createDecimalType() or scale == null StringType |
| DATE          | DateType                                        |
| TIME          | TimestampType                                   |
| TIMESTAMP     | TimestampType                                   |
| YM_INTERVAL   | createYearMonthIntervalType()                   |
| DS_INTERVAL   | createDayTimeIntervalType()                     |
| CHAR          | StringType                                      |
| NCHAR         | StringType                                      |
| VARCHAR       | StringType                                      |
| NVARCHAR      | StringType                                      |
| CLOB          | StringType                                      |
| NCLOB         | StringType                                      |
| RAW           | StringType                                      |
| JSON          | StringType                                      |
| BLOB          | BinaryType                                      |
| BIT           | LongType                                        |
