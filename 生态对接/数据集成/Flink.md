Apache Flink是一款开源的分布式数据流处理引擎，适用于大规模数据处理，可简化大数据应用开发。YashanDB提供了两种和Flink工具的对接方式，如下：

1. 通过载入YashanDB JDBC驱动以及YashanDB Flink Connector组件，可实现Flink连接YashanDB并读取/写入数据。
2. 通过载入YashanDB JDBC驱动、YStream组件以及YashanDB CDC连接器，可实现Flink对YashanDB的变更数据捕获（CDC）。

## YashanDB读取/写入

### 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装Jdk11及以上的Java应用环境
2. 已安装Flink 1.15，或Flink 1.16~1.19
3. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包
4. 已向我们的技术支持人员获取YashanDB Flink Connector组件包，其中1.1.0版本适配Flink 1.15，其他适配Flink 1.16~1.19
5. 已存在一个可正常访问的YashanDB服务端。

### 对接配置

请参照如下步骤进行YashanDB与Flink的对接配置：

1. 找到Flink软件的安装目录，在目录下找到lib文件夹。
2. 将YashanDB JDBC驱动包和YashanDB Flink Connector组件包（均为jar包）放至lib文件夹中。
3. 启动Flink集群：

```bash
$ ./bin/start_cluster.sh
```

### 使用简介

完成上述配置后，即可开始Flink对YashanDB的数据读取和写入，Flink为这一过程提供了SQL语法格式工具，该工具可以CLI方式供开发者直接操作数据流，或可被集成到应用程序中。本文以CLI方式介绍YashanDB Flink Connector连接器的配置和使用：

1. 进入Flink软件的安装目录，启动Flink SQL Client：

```bash
$ ./bin/sql_client.sh embedded
```

2. 创建源表连接器（请将your_host、your_port、your_dbname、your_name、your_password和your_table修改为实际值）：

```sql
-- register an YashanDB table 'products' in Flink SQL
Flink SQL> CREATE TABLE products (
     ID INT NOT NULL,
     NAME STRING,
     DESCRIPTION STRING,
     WEIGHT DECIMAL(10, 3),
     PRIMARY KEY(id) NOT ENFORCED
     ) WITH (
     'connector' = 'yashandb',
     'url' = 'jdbc:yasdb://your_host:your_port/your_dbname'
     'username' = 'your_name',
     'password' = 'your_password',
     'schema-name' = 'sales',
     'table-name' = 'products');
```

3. 创建成功后即可在Flink SQL中查询YashanDB中products表：

```sql
Flink SQL> SELECT * FROM products;
```

#### 连接器参数

上面连接器的配置示例仅列出了部分参数，以下为全部可使用的参数说明：

| 参数名                     | 是否必选 | 默认值 | 数据类型     | 参数描述                                                  |
| -------------------------- | -------- | ------- | -------- | ------------------------------------------------------------ |
| connector                  | 必选 | (none)  | String   | 指定需要使用的连接器，必须固定为`yashandb`                 |
| username                   | 必选 | (none)  | String   | YashanDB数据库的连接用户名                                   |
| password                   | 必选 | (none)  | String   | YashanDB数据库的连接用户的登录密码                           |
| schema-name                | 必选 | (none)  | String   | YashanDB数据库的schema名称                                   |
| table-name                 | 必选 | (none)  | String   | YashanDB数据库的table名称                                    |
| url                        | 可选 | (none)  | String   | YashanDB数据库的JDBC URL                                   |
| sink.buffer-flush.max-rows | 可选 | 100     | Integer  | flush前缓存记录的最大值，设置为`'0'`表示禁用        |
| sink.buffer-flush.interval | 可选 | 1s      | Duration | flush间隔时间，超过该时间后异步线程将flush数据，设置为 `'0'`表示禁用<br/>为了完全异步地处理缓存的flush事件，可以将`'sink.buffer-flush.max-rows'`设置为`'0'`并配置适当的flush时间间隔 |
| sink.max-retries           | 可选 | 3       | Integer  | 写入记录到数据库失败后的最大重试次数                       |
| sink.parallelism           | 可选 | (none)  | Integer  | 用于定义JDBC sink算子的并行度，默认情况下并行度由框架决定（即使用与上游链式算子相同的并行度） |
| bulk-load-enable           | 可选 | false   | boolean  | 是否开启bulkload插入模式，该模式仅对YashanDB的LSC表生效<br/>若设置为true，原普通插入将变成bulkInsert、原upsert插入将变成bulkupsert，使用BULKLOAD模式的详细介绍请查阅[hint](https://doc.yashandb.com/yashandb/23.2/zh/Development-Guide/SQL-Reference-Manual/General-SQL-Syntax/hint.html) |
| invert-bit-type-data       | 可选 | true    | boolean  | 是否开启反转BIT类型的BYTE数组，默认开启（兼容MySQL2YashanDB因BIT数据类型的大小端的差异），设置为false则不反转<br/>如果其他数据库作为源，且BIT数据类型的大小端跟YashanDB相同可使用该参数 |
| scan.partition.column      | 可选 | (none)  | String   | 用于将输入进行分区的列名，分区扫描相关介绍请查阅[Flink官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.17/zh/docs/connectors/table/jdbc/#分区扫描) |
| scan.partition.num         | 可选 | (none)  | Integer  | 分区数                                                     |
| scan.partition.lower-bound | 可选 | (none)  | Integer  | 第一个分区的最小值                                         |
| scan.partition.upper-bound | 可选 | (none)  | Integer  | 最后一个分区的最大值                                       |

#### 部分功能说明

##### 键处理

当写入数据到YashanDB时，Flink会判断源表定义是否有主键。如果定义了主键，则连接器将以upsert模式工作，否则连接器将以append模式工作。

- 在upsert模式下，连接器将根据主键判断插入新行或更新已存在的行，该模式可确保幂等性。建议为源表定义主键，并确保该主键为YashanDB中对应表的唯一键或主键。

  > **Note**:
  >
  > 在存算一体分布式部署中，为保证对同一主键的同一操作只生成一条记录，需在Flink SQL中执行`SET 'table.exec.sink.upsert-materialize' = 'FORCE';`将table.exec.sink.upsert-materialize参数设置为FORCE。

- 在append模式下，连接器会把所有记录解释为INSERT消息，如果违反了YashanDB中主键或者唯一约束，INSERT插入可能会失败。

主键语法的更多详情，请查阅[Flink官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.17/zh/docs/dev/table/sql/create/#create-table)。

##### 分区扫描

分区扫描相关介绍，请查阅[Flink官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.17/zh/docs/connectors/table/jdbc/#分区扫描)。

##### 幂等写入

如果在源表定义中存在主键，JDBC sink将使用upsert语义（而非普通的INSERT语句），如果YashanDB中存在违反唯一性约束，则原子地添加新行或更新现有行，从而确保幂等性。

如果出现故障，Flink作业会从上次成功的checkpoint恢复并重新处理，基于该机制可能会在恢复过程中重复处理消息。如果使用upsert模式，在需要重复处理记录的场景中可有效避免违反数据库主键约束和产生重复数据。

除了故障恢复场景外，数据源（kafka topic）也可能随着时间推移自然地包含多个具有相同主键的记录，因此upsert模式将更符合预期。

##### BulkLoad模式

为提高YashanDB的LSC表的写入性能，本连接器提供bulkload模式，可通过`bulk-load-enable`设置，若开启bulkload模式（即设置为true），原普通插入将变成bulkInsert、原upsert插入将变成bulkupsert。

#### 数据类型映射

YashanDB Flink Connector组件内置了一套数据映射，用于YashanDB和Flink SQL之间的数据类型转换，见下表。

YashanDB Flink Connector组件不对时间类型的时区进行处理，如遇时区问题导致时间差异，可参考[Flink官网文档相关介绍](https://nightlies.apache.org/flink/flink-cdc-docs-master/zh/docs/faq/faq/#q2-使用-mysql-cdc增量阶段读取出来的-timestamp-字段时区相差8小时怎么回事呢)。

| YashanDB type          | Flink SQL type |
| ---------------------- | -------------- |
| TINYINT                | TINYINT        |
| SMALLINT               | SMALLINT       |
| INT                    | INT            |
| BIGINT                 | BIGINT         |
| FLOAT                  | FLOAT          |
| DOUBLE                 | DOUBLE         |
| NUMBER                 | DECIMAL        |
| BIT                    | bytes          |
| BOOLEAN                | BOOLEAN        |
| DATE                   | TIMESTAMP      |
| TIME                   | TIME           |
| TIMESTAMP              | TIMESTAMP      |
| INTERVAL YEAR TO MONTH | BIGINT         |
| INTERVAL DAY TO SECOND | BIGINT         |
| CHAR                   | STRING         |
| VARCHAR                | STRING         |
| NCHAR                  | STRING         |
| NVARCHAR               | STRING         |
| BLOB                   | bytes          |
| CLOB                   | STRING         |
| NCLOB                  | STRING         |
| RAW                    | bytes          |
| ROWID                  | STRING         |
| UROWID                 | bytes          |

## YashanDB CDC

在执行对接前，请您先了解YashanDB CDC组件的使用限制：

- 用于CDC的表上的数据类型不能为JSON，XMLTYPE，UDT，ST_GEOMETRY或BOX2D。
- 用于CDC的表上含LOB数据类型时，该表必须存在主键，否则，在对该表进行update/delete变更捕获时，LOB数据会丢失。
- 如使用Flink Stream API，每个任务的表数量不能超过1万。
- Ystream服务最多为32个，因此Flink的任务最多同时只能运行32个。

### 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装Jdk11及以上的Java应用环境
2. 已安装Flink 1.15，或Flink 1.16~1.19
3. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包和YStream组件包
4. 已向我们的技术支持人员获取YashanDB CDC组件包，其中1.1.0版本适配Flink 1.15，其他适配Flink 1.16~1.19
5. 已存在一个可正常访问的YashanDB服务端。

### 对接配置

请参照如下步骤进行YashanDB与Flink的对接配置：

1. 找到Flink软件的安装目录，在目录下找到lib文件夹。
2. 将YashanDB JDBC驱动包、YStream组件包和YashanDB CDC组件包（均为jar包）放至lib文件夹中。
3. 启动Flink集群：

```bash
$ ./bin/start_cluster.sh
```

### 使用简介

完成上述配置后，您还需要配置YashanDB服务端的YStream服务，以及创建连接器，来开始Flink对YashanDB的CDC。

#### 配置YashanDB YStream

请参照如下步骤配置YashanDB服务端的YStream服务（如某一步骤中的内容在YashanDB中已实现，则可略过）：

1. 配置Ystream内存池（请将streamPoolSize修改为实际值）：

```sql
ALTER SYSTEM SET STREAM_POOL_SIZE = streamPoolSize;
```


2. 按需开启库级或表级附加日志（请将tablename修改为实际值）：

```sql
--当您需要监听库下全部对象时（包含新增对象），可开全库附加日志
ALTER DATABASE ADD SUPPLEMENTAL LOG TABLE TYPE (HEAP);
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA ( ALL) COLUMNS;

--当您仅需要监听某些表时，可开启表级附加日志
ALTER TABLE tablename ADD SUPPLEMENTAL LOG DATA ( ALL ) COLUMNS;
```

> **Caution**：
>
> 不开启附加日志或开启附加日志的对象不正确会导致数据丢失甚至任务失败。

3. 为YStream服务的连接用户授权（请将connect_user修改为实际值）：

```sql
GRANT CREATE SESSION TO connect_user;
GRANT SELECT ON V_$DATABASE TO connect_user;
GRANT SELECT ON V_$TRANSACTION TO connect_user;
GRANT SELECT ON V_$YSTREAM_SERVER TO connect_user;
GRANT FLASHBACK ANY TABLE TO connect_user; 
GRANT SELECT ANY TABLE TO connect_user;
GRANT YSTREAM_CAPTURE TO connect_user;
```

#### 创建连接器

Flink SQL为Flink提供的SQL语法格式工具，该工具可以CLI方式供开发者直接操作数据流，或可被集成到应用程序中。本文以CLI方式介绍YashanDB CDC连接器的配置和使用：

1. 进入Flink软件的安装目录，启动Flink SQL Client：

```bash
$ ./bin/sql_client.sh embedded
```

2. 创建源表连接器（请将your_host、your_port、your_dbname、your_name、your_password和your_table修改为实际值）：

```sql
-- register an YashanDB table 'products' in Flink SQL
Flink SQL> CREATE TABLE products (
     ID INT NOT NULL,
     NAME STRING,
     DESCRIPTION STRING,
     WEIGHT DECIMAL(10, 3),
     PRIMARY KEY(id) NOT ENFORCED
     ) WITH (
     'connector' = 'yashandb-cdc',
     'hostname' = 'your_host'
     'port' = 'your_port',
     'username' = 'your_name',  --对应YStream中的connect_user
     'password' = 'your_password',
     'ystream.serverName'='ystream_server', 
     'schema-name' = 'sales',
     'table-name' = 'products');
```

3. 创建成功后即可在Flink SQL中查询YashanDB中products表：

```sql
Flink SQL> SELECT * FROM products;
```

#### 连接器参数

上面连接器的配置示例仅列出了部分参数，以下为全部可使用的参数说明：

| 参数名                                     | 是否可选 | 默认值  | 数据类型 | 参数描述                                                     |
| ------------------------------------------ | -------- | ------- | -------- | ------------------------------------------------------------ |
| connector                                  | 必选     | (none)  | String   | 指定需要使用的连接器，必须固定为`yashandb-cdc`。             |
| hostname                                   | 必选     | (none)  | String   | YashanDB数据库服务的IP地址或hostname                         |
| port                                       | 必选     | (none)  | Integer  | YashanDB数据库服务的端口                                     |
| username                                   | 必选     | (none)  | String   | YashanDB数据库的连接用户名                                   |
| password                                   | 必选     | (none)  | String   | YashanDB数据库的连接用户的登录密码                           |
| schema-name                                | 必选     | (none)  | String   | YashanDB数据库的schema名称                                   |
| table-name                                 | 必选     | (none)  | String   | YashanDB数据库的table名称                                    |
| url                                        | 可选     | (none)  | String   | YashanDB数据库的JDBC URL，如果不设置此选项，系统会根据hostname和port自动生成JDBC URL |
| scan.startup.mode                          | 可选     | initial | String   | YashanDB CDC的启动模式，可选值为`initial`或`latest-offset`   |
| scan.incremental.snapshot.chunk.size       | 可选     | 8096    | Integer  | 表快照的块大小（行数），在读取表的快照时将根据该配置将捕获的表拆分为多个块 |
| scan.snapshot.fetch.size                   | 可选     | 1024    | Integer  | 读取表快照时每次轮询的最大获取大小                           |
| connect.max-retries                        | 可选     | 3       | Integer  | 连接器应重试构建YashanDB数据库服务器连接的最大重试次数       |
| connection.pool.size                       | 可选     | 20      | Integer  | 连接池大小                                                   |
| scan.incremental.snapshot.chunk.key-column | 可选     | (none)  | String   | 表快照的块键，在读取表快照时，捕获的表被块键拆分为多个块，默认情况下，块键为“ROWID”，此列必须是主键的列 |
| ystream.serverName                         | 必选     | (none)  | String   | Ystream服务名称，要求全局唯一，连接器会根据该名称自动创建相应的Ystream服务进行增量数据读取 |
| ystream.parallel                           | 可选     | 4       | Integer  | Redo解析线程并发数，提高解析线程并发数可以提升性能但会消耗更多资源，请合理配置该值 |
| ystream.txnAgeSpillThreshold               | 可选     | 600     | Integer  | LCR溢出触发的时间阈值（单位：秒）<br/>解析过程中，若某个事务长时间不提交，等待时间超过该值时该事务所有LCR会溢出到系统表进行持久化并释放内存，此类LCR所对应的日志无需再被重复解析，用户可自行按需清理附加日志 |
| ystream.txnLcrSpillThreshold               | 可选     | 128M    | String   | LCR溢出触发的内存占用阈值（单位：字节）<br/>解析过程中，若某个事务所占内存超过该值，该事务所有LCR会溢出到系统表进行持久化并释放内存，此类LCR所对应的日志无需再被重复解析，用户可自行按需清理附加日志 |
| ystream.checkpointInterval                 | 可选     | 3       | Integer  | Checkpoint执行的间隔（单位：秒）<br/>每次Checkpoint会在系统表持久化最新的重启恢复点。客户端设置最新的applied position后，需要等待一个Checkpoint间隔才能被数据库感知 |
| ystream.queueSize                          | 可选     | 128     | Integer  | 异步队列容纳LCR的大小，YStream客户端启用2条异步队列，以分级处理数据提高吞吐 |
| ystream.pollTimeout                        | 可选     | 10      | Integer  | YStream客户端从队列中获取数据的最长阻塞时间（单位：秒）      |
| ystream.clientResponseTimeout              | 可选     | 60      | Integer  | YStream客户端等待服务端响应的最长超时时间（单位：秒）        |

#### 部分功能说明

##### Exactly-Once Processing

YashanDB CDC连接器首先读取快照阶段，然后再精确一致性读取变更数据事件，即使中途中发生任务失败，依赖Flink的checkpoint或savepoint也会从失败点位或指定点位进行重新拉取增量任务。

##### 启动读取位置

通过配置选项`scan.startup.mode`可设置连接器的启动模式：

- initial（default）: 先启动全量快照读取，再启动增量读取redo log捕获更改事件。

- latest-offset：不启动快照读取，直接从当前日志点位增量读取redo log捕获更改事件。

##### 单线程增量读取

YashanDB CDC增量源无法并行读取，因为只能一个任务接收更改事件。

##### DataStream Source

YashanDB CDC连接器也可以配置DataStream源，示例如下：

```java
import com.sics.flink.connector.yashandb.config.StartupOptions;
import com.sics.flink.connector.yashandb.internal.options.YstreamOptions;
import com.sics.flink.connector.yashandb.source.YashanDBIncrementalSource;
import com.sics.flink.connector.yashandb.source.cdc.deserialization.JsonYstreamDeserializationSchema;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.configuration.ReadableConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TestYashanDBCdc {
  public static void main(String[] args) throws Exception {
    ReadableConfig config = new Configuration();

    YashanDBIncrementalSource<String> source =
        YashanDBIncrementalSource.<String>newBuilder()
            .hostname("your_host")
            .port(your_port)
            .fetchSize(1024)
            .schemaList("sales")
            .startupOptions(StartupOptions.initial())
            .deserializer(new JsonYstreamDeserializationSchema())
            .username("your_name")
            .password("your_password")
            .tableList("products")
            .ystreamOptions(YstreamOptions.defaultOption) // ystream option
            .ystreamServerName("ystream_server")
            .build();
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

    env.fromSource(source, WatermarkStrategy.noWatermarks(), "yashandb source")
        .setParallelism(4)
        .print()
        .setParallelism(1);
    env.execute("Print yashadnb Snapshot + RedoLog");
  }
}
```

#### 数据类型映射

YashanDB CDC组件内置了一套数据映射，用于YashanDB和Flink SQL之间的数据类型转换，见下表。

| YashanDB type          | Flink SQL type |
| ---------------------- | -------------- |
| TINYINT                | TINYINT        |
| SMALLINT               | SMALLINT       |
| INT                    | INT            |
| BIGINT                 | BIGINT         |
| FLOAT                  | FLOAT          |
| DOUBLE                 | DOUBLE         |
| NUMBER                 | DECIMAL        |
| BIT                    | bytes          |
| BOOLEAN                | BOOLEAN        |
| DATE                   | TIMESTAMP      |
| TIME                   | TIME           |
| TIMESTAMP              | TIMESTAMP      |
| INTERVAL YEAR TO MONTH | BIGINT         |
| INTERVAL DAY TO SECOND | BIGINT         |
| CHAR                   | STRING         |
| VARCHAR                | STRING         |
| NCHAR                  | STRING         |
| NVARCHAR               | STRING         |
| BLOB                   | bytes          |
| CLOB                   | STRING         |
| NCLOB                  | STRING         |
| RAW                    | bytes          |
| ROWID                  | STRING         |
| UROWID                 | bytes          |

#### 常见问题

##### Q1. 为什么运行YashanDB CDC需要YStream依赖？

YashanDB CDC内部使用了YashanDB的YStream来捕获增量数据信息，YashanDB CDC组件包不包含YStream组件，需要用户自行或取YStream组件包放入Flink的lib目录下。

##### Q2. YashanDB CDC报错“YashanDB YStream serverName 'server'  has existed in database,  Please enter a non-existent option 'ystream.serverName' in V\_$YSTREAM\_SERVER.”该怎么处理？

- 首次启动YashanDB CDC，会根据用户填写的选项`ystream.serverName`自动创建YashanDB数据库的Ystream服务，如果数据库中已存在同名YStream服务会创建失败并报此错，需修改`ystream.serverName`值并确保全局唯一。

- 非首次启动YashanDB CDC：

  - 如果使用savepoint启动，YashanDB CDC会复用上次创建的YStream服务重新从上一个数据点位拉取任务，不涉及新建YStream服务，不会出现此报错。

  - 若用户重新开启新的YashanDB CDC任务，需重新配置新的唯一`ystream.serverName`值。

为降低重名报错的复现率，可在确认YStream服务已无需再使用后自行删除（在YashanDB中执行），删除语句参考如下：

```sql
EXEC DBMS_YSTREAM_ADM.STOP('ystream_server');
EXEC DBMS_YSTREAM_ADM.DROP('ystream_server');
```

##### Q3. 运行YashanDB CDC报错“The number of YStream server in the database has reached 32, and YashanDB's YStream server supports a maximum of 32. Please manually execute 'EXEC DBMS\_YSTREAM\_ADM.DROP('ystream\_server')' to delete the YStream server in' select * from V\_$YSTREAM\_SERVER'”该如何处理？

该错误信息表示YashanDB数据库中的YStream服务数量已达最大值32个，可查看V_$YSTREAM_SERVER视图获取所有YStream服务信息，并结合实际需求手动清理未使用/无需再用的YStream服务。
