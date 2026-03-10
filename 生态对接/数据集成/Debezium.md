Debezium是一款开源的基于变更数据捕获（CDC）的分布式平台，通过该平台可实现实时捕获数据库的数据变更，以事件流形式发布到Kafka中。YashanDB提供 Debezium Connector组件，实现与Debezium的对接，可用于同步全量快照数据，捕获并记录YashanDB中发生的行级更改（包括CDC运行过程中的新增的表）；并可通过配置组件，使Debezium捕获指定的schema和表，将更改事件同步到Kafka。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装Jdk11及以上的Java应用环境
2. 已安装Kafka 2.x/3.x，以及对应版本的Apache Zookeeper和Kafka Connect
3. 已下载debezium-connector-oracle-2.4.2.Final-plugin依赖包，参考[Maven中央仓库](https://repo1.maven.org/maven2/io/debezium/debezium-connector-oracle/2.4.2.Final/debezium-connector-oracle-2.4.2.Final-plugin.tar.gz)
5. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包和YStream组件包
6. 已向我们的技术支持人员获取YashanDB Debezium Connector组件包
7. 已存在一个可正常访问的YashanDB服务端。

## 对接配置

请参照如下步骤进行YashanDB与Debezium的对接配置（以Linux平台为例）：

1. 找到Kafka软件的安装目录，在其下创建plugins目录：

```bash
$ cd /.../kafka
$ mkdir plugins
```

2. 进入plugins目录，创建debezium-connector-yashandb目录：

```bash
$ cd plugins
$ mkdir debezium-connector-yashandb
```

3. 解压debezium-connector-oracle-2.4.2.Final-plugin依赖包内容至plugins/debezium-connector-yashandb目录：


```bash
$ cd debezium-connector-yashandb
$ tar zxf /.../debezium-connector-oracle-2.4.2.Final-plugin.tar.gz -C ./
```

4. 删除目录下debezium-connector-oracle-2.4.2.Final.jar文件：

```bash
$ rm debezium-connector-oracle-2.4.2.Final.jar
```

5. 将YashanDB JDBC驱动包、YStream组件包和YashanDB Debezium Connector组件包（均为jar包）放至plugins/debezium-connector-yashandb目录下

6. 返回到kafka目录层级，进入config目录，编辑目录下的connect-distributed.properties配置文件，定义plugin.path配置项的值为plugins目录路径：

```bash
$ cd ../../config
$ vi connect-distributed.properties

plugin.path=/.../kafka/plugins
```

7. 重启Kafka Connect进程：

```plaintext
$ cd ..
$ ./bin/connect-distributed.sh config/connect-distributed.properties
```

8. 浏览器访问`http://kafka_hostname:8083/connector-plugins`，如果有如下YashanDBConnector，即YashanDB connector部署成功。

```json
[
  {
    "class": "io.debezium.connector.yashandb.YashanDBConnector",
    "type": "source",
    "version": "2.4.2.4.3"
  }
]
```

## 使用简介

在完成上述配置后，您还需要分别启动YashanDB服务端的YStream服务，以及Debezium端的Connector任务，来开始对YashanDB的CDC。

### 启动YashanDB YStream

请参照如下步骤启动YashanDB服务端的YStream服务（如某一步骤中的内容在YashanDB中已实现，则可略过）：

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
GRANT SELECT ON SYS.V_$DATABASE TO connect_user;
GRANT SELECT ON SYS.V_$TRANSACTION TO connect_user;
GRANT SELECT ON SYS.V_$YSTREAM_SERVER TO connect_user;
GRANT FLASHBACK ANY TABLE TO connect_user; 
GRANT SELECT ANY TABLE TO connect_user;
GRANT YSTREAM_CAPTURE TO connect_user;
GRANT ALTER SESSION TO connect_user;
GRANT SELECT ON SYS.DBA_LOBS TO connect_user;
GRANT SELECT ON SYS.V_$DATAFILE TO connect_user;
GRANT SELECT ON SYS.DBA_SEGMENTS TO connect_user;
GRANT FLASHBACK ON SYS.DBA_SEGMENTS TO connect_user;
GRANT FLASHBACK ON SYS.DBA_LOBS TO connect_user;
GRANT SELECT ON SYS.DBA_TABLES TO connect_user;
GRANT FLASHBACK ON SYS.DBA_TABLES TO connect_user;
GRANT SELECT ON SYS.DBA_TAB_PARTITIONS TO connect_user;
GRANT FLASHBACK ON SYS.DBA_TAB_PARTITIONS TO connect_user;
GRANT SELECT ON SYS.DBA_TAB_SUBPARTITIONS TO connect_user;
GRANT FLASHBACK ON SYS.DBA_TAB_SUBPARTITIONS TO connect_user; 
```

4. 调用DBMS_YSTREAM_ADM高级包的CREATE函数创建YStream服务（请将serverName、connect_user和start_scn修改为实际值）：

```sql
--start_scn可通过查询select CURRENT_SCN from V$DATABASE获取
EXEC DBMS_YSTREAM_ADM.CREATE('serverName', 'connect_user', start_scn)
```

5. 调用DBMS_YSTREAM_ADM高级包的ADD_TABLES函数为YStream服务新增解析表名和模式，Debezium将捕获此函数定义的表名和模式，不执行此函数，或函数参数传入NULL表示捕获所有（请将serverName、table和schema修改为实际值）：

```sql
--可同时指定多个表或多个模式
EXEC DBMS_YSTREAM_ADM.ADD_TABLES('serverName', 'table1,table2', 'schema1,schema2');
```

6. 调用DBMS_YSTREAM_ADM高级包的SET_PARAMETER函数为YStream服务设置相关参数，不执行此函数则使用默认值（请将serverName修改为实际值）：

```sql
--可同时指定多个表或多个模式
EXEC DBMS_YSTREAM_ADM.SET_PARAMETER('serverName', 'PARALLELISM', '3');
```

7. 调用DBMS_YSTREAM_ADM高级包的START函数启动YStream服务（请将serverName修改为实际值）：

```sql
EXEC DBMS_YSTREAM_ADM.START('serverName');
```

### 启动Debezium Connector

以Kafka Connect的 REST API为例，配置Debezium Connector连接器并启动任务（请将your_host、your_port、your_dbname、your_name、your_password、your_schema、your_table和serverName修改为实际值）：

```bash
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{
  "name": "yashandb-connector", 
  "config": {
         "connector.class" : "io.debezium.connector.yashandb.YashanDBConnector", 
         "database.hostname" : "your_host", 
         "database.port" : "your_port", 
         "database.user" : "your_name", 
         "database.password" : "your_password",  
         "database.dbname" : "your_dbname", 
         "topic.prefix" : "my_topic", 
         "database.url" : "jdbc:yasdb://your_host:your_port/your_dbname", 
         "table.include.list" : "your_schema.your_table",
         "database.ystream.server.name" : "serverName",  
         "lob.enabled" : "true",
         "schema.history.internal.kafka.bootstrap.servers" : "kafka:9092",
         "schema.history.internal.kafka.topic": "schema-changes.inventory" 
  }
}'
```

至此，Debezium将开始对YashanDB的变更数据进行实时捕获，并发送到Kafka中，用于您的数据分析和处理。使用以下命令可查询过程中任务状态等信息：

```bash
curl -s -X GET http://localhost:8083/connectors/yashandb-connector/status
```


### 连接器参数

上面连接器的配置示例仅列出了部分参数，以下将列出连接器的一些关键参数（但包括YashanDB Debezium Connector组件定制的全部参数），完整的参数信息请参考[Debezium官方文档](https://debezium.io/documentation/reference/2.4/connectors/oracle.html)。

| 参数名                            | 默认值                                         | 参数说明      |
| ------------------------------- | ---------------------------------------------- |-------------------------|
| name                            | (none)                                     | 连接器的唯一名称，必选参数，要求全局唯一                |
| connector.class                 | (none)                                     | 连接器的Java类的名称，固定为`io.debezium.connector.oracle`      |
| database.hostname               | (none)                                     | YashanDB数据库服务的IP地址或hostname   |
| database.port                   | (none)                                     | YashanDB数据库服务的端口 |
| database.user                   | (none)                                     | 连接YashanDB数据库的用户名   |
| database.password               | (none)                                     | 连接YashanDB数据库的用户名的密码   |
| database.dbname                 | (none)                                     | YashanDB数据库的名称     |
| database.url                    | (none)                                     | YashanDB数据库的JDBC URL   |
| database.ystream.server.name    | (none)                                     | Ystream服务名称，要求全局唯一，连接器会根据该名称自动创建相应的Ystream服务          |
| ystream.blocking.queue.size     | 128                                            | YStream客户端内置阻塞队列的长度，获取增量逻辑日志时直接从该队列获取          |
| ystream.poll.timeout            | 10                                             | 从阻塞队列中获取下一个结果的超时时间（单位：秒） |
| ystream.client.response.timeout | 60                                             | YStream服务端等待YStream客户端响应的最长时间（单位：秒）         |
| topic.prefix                    | (none)                                     | 主题前缀，用于为连接器从中捕获更改的Oracle数据库服务器提供命名空间。该参数值将用作连接器发出的所有Kafka主题名称的前缀，要求全局唯一，由字母、数字、连字符、点和下划线组成。<br/>连接器无法恢复其数据库架构的历史主题，一旦更改该值并重新启动，连接器将会向新主题发出后续事件，**请不要轻易更改该参数值**   |
| snapshot.mode                   | initial                                        | 连接器对捕获的表进行快照的模式<br/>* always：连接器每次启动时始终执行快照（表结构和数据），快照完成后连接器开始捕获并记录目标表发生的表结构和数据更改<br/>* initial：连接器首次启动时执行快照（表结构和数据），快照完成后连接器开始捕获并记录目标表发生的表结构和数据更改，后续启动时不会再次执行快照<br/>* initial_only：连接器首次启动时执行快照（表结构和数据），在目标表发生连接器启动后的首次更改时中止快照，且连接器不处理目标表发生的任何后续更改<br/>* schema_only：连接器每次启动时始终执行快照（仅含表结构），快照完成后连接器开始捕获并记录目标表发生的表结构更改<br/>* schema_only_recovery：基于schema_only模式的恢复模式，可用于连接器意外断连后再次重启时，连接器启动后会执行快照恢复损坏或丢失的历史主题，照完成后，连接器的表现同schema_only模式。**仅在连接器上一次意外断连时间点至快照时间点期间未发生表结构更改的情况下，才能安全使用此模式**。您也可以按需定期设置该值清理因意外断连而增长的历史主题<br/>更多详情请查阅[debezium官方文档](https://debezium.io/documentation/reference/2.4/connectors/oracle.html)  |
| schema.include.list             | (none)                                     | 需捕获变更的schema清单，可选参数，清单采用正则表达式，多个schema名称间用逗号`,`分隔，若配置该参数，连接器将只捕获清单中包含的schema相关变更 <br/>通常该参数与schema.exclude.list参数择一配置即可，且schema.include.list优先级更高（即配置了schema.include.list后schema.exclude.list将失效），若二者均不配置，则默认捕获所有非系统schema的更改|
| schema.exclude.list             | (none)                                     | 无需捕获变更的schema清单，可选参数，清单采用正则表达式，多个schema名称间用逗号`,`分隔，若配置该参数，连接器将捕获除清单中包含的schema外的其他非系统schema的相关变更 <br/>通常该参数与schema.include.list参数择一配置即可，且schema.include.list优先级更高（即配置了schema.include.list后schema.exclude.list将失效），若二者均不配置，则默认捕获所有非系统schema的更改   |
| table.include.list              | (none)                                     | 需捕获变更的表清单，可选参数，清单采用正则表达式，多个表名间用逗号`,`分隔，若配置该参数，连接器将只捕获清单中包含的表相关变更 <br/>通常该参数与table.exclude.list参数择一配置即可，且table.include.list优先级更高（即配置了table.include.list后table.exclude.list将失效），若二者均不配置，则默认捕获所有非系统表的更改          |
| table.exclude.list               | (none)                                     | 无需捕获变更的表清单，可选参数，清单采用正则表达式，多个表名间用逗号`,`分隔，若配置该参数，连接器将捕获除清单中包含的表外的其他非系统表的相关变更 <br/>通常该参数与table.include.list参数择一配置即可，且table.include.list优先级更高（即配置了table.include.list后table.exclude.list将失效），若二者均不配置，则默认捕获所有非系统表的更改  |
| max.batch.size                  | 2048                                           | 连接器每次迭代期间要处理的单批事件的最大大小，正整数值 |
| max.queue.size                  | 9182                                           | 阻塞队列可以容纳的最大记录数，正整数值。当Debezium读取数据库中的事件流时，它会在将事件写入Kafka之前将其放置在阻塞队列中。在连接器接收消息的速度快于将消息写入Kafka的速度的情况下，或当Kafka不可用时，阻塞队列可以为从数据库读取更改事件提供背压。当连接器定期记录偏移量时，队列中保存的事件将被忽略<br/>需将max.queue.size的值设置为大于max.batch.size的数值   |
| max.queue.size.in.bytes         | 0（disabled）                                  | 阻塞队列的最大容量（单位：字节），长整数值。默认情况下，不会为阻塞队列指定卷限制。如需指定队列可以消耗的字节数，请将此属性设置为正长值<br/>若同时设置了max.queue.size，则当队列大小达到任一属性指定的限制时将阻止对队列的写入。例如max.queue.size=1000且max.queue.size.in.bytes=5000，在队列包含1000条记录或队列中的记录量达到5000字节后，将阻止向队列写入      |
| poll.interval.ms                | 500 （0.5 second）                             | 连接器在每次迭代期间应等待新更改事件出现的时间（单位：毫秒），正整数值  |
| snapshot.fetch.size             | 10000                                          | 在snapshot快照时从每个表一次读取的最大行数，连接器将以该参数指定大小分批多次读取表内容        |
| query.fetch.size                | 10000                                          | JDBC查询的fetch size   |
| lob.enabled                     | false                                          | 控制是否在更改事件中发送大对象（CLOB或BLOB等）列值。默认情况下，更改事件中大对象列不包含列值。如需捕获大型对象值并在更改事件中对其进行序列化（会有一定的开销），请将该参数设置为true      |
| decimal.handling.mode           | precise                                        | 连接器处理NUMBER、DECIMAL和NUMERIC列的浮点值的模式<br/>* precise：使用java.math精确表示值，以二进制形式在更改事件中表示的BigDecimal值。该模式对负scale的浮点值支持程度有限，建议使用string模式<br/>* double：使用双精度值表示值，该模式可能会导致精度损失<br/>* string：将值编码为格式化字符串，该模式可能会导致有关真实类型的语义信息丢失           |
| unavailable.value.placeholder   | `__debezium_unavailable_value`                 | 连接器用于标识未从数据库中获取到真实数据值但该值未发生更改的常数代值，例如虽未从数据库获取到某一lob值但明确该lob值未发生更改就使用unavailable.value.placeholder参数值代替该lob值                 |
| skipped.operations              | t                                              | 连接器在流式传输过程中需跳过的操作清单，多个操作名称间用逗号`,`分隔<br/>可以配置的操作名包括：<br/>* c：表示插入/创建操作<br/>* u：表示更新操作<br/>* d：表示删除操作<br/>* t：表示截断操作，默认情况下，只跳过截断操作                 |
| signal.data.collection          | (none) value                               | 用于向连接器发送信号的数据采集的完全限定名称，格式为`<databaseName>.<schemaName>.<tableName>`       |
| signal.enabled.channels         | source                                         | 连接器可用的信号通道名称清单。默认情况下，可用通道包括source、kafka、file、jmx        |
| notification.enabled.channels   | (none)                                     | 连接器可用的通知通道名称清单。默认情况下，可用通道包括sink、log、jmx   |
| incremental.snapshot.chunk.size | 1024                                           | 在增量快照块期间，连接器获取并读入内存的最大行数   |
| topic.naming.strategy           | `io.debezium.schema.SchemaTopicNamingStrategy` | 用于确定数据更改、模式更改、事务、心跳事件等的主题名称的类名，默认为`SchemaTopicNamingStrategy`    |
| topic.delimiter                 | .                                              | 主题名称的分隔符，默认为`.`    |
| snapshot.max.threads            | 1                                              | 连接器在执行初始快照时使用的线程数。如需启用并行初始快照，请将属性设置为大于1的值        |
| schema.history.internal.store.only.captured.tables.ddl | false | 一个布尔值，用于指定连接器是记录架构或数据库中所有表中的架构结构，还是仅记录指定用于捕获的表中的架构结构。<br>指定以下值之一：<br>`false`*（默认）*<br>在数据库快照期间，连接器会记录数据库中所有非系统表的架构数据，包括未指定用于捕获的表。最好保留默认设置。如果稍后决定从最初未指定用于捕获的表中捕获更改，则连接器可以轻松开始从这些表中捕获数据，因为它们的架构结构已存储在架构历史记录主题中。Debezium 需要表的模式历史记录，以便它可以识别发生更改事件时存在的结构。<br>`true`<br>在数据库快照期间，连接器仅记录 Debezium 从中捕获更改事件的表的表模式。如果更改默认值，并且稍后将连接器配置为从数据库中的其他表捕获数据，则连接器缺少从表中捕获更改事件所需的架构信息。 |
| converters | No default | 配置Debezium的自定义转换器 |
| \<converter_name>.type | No default | 配置Debezium的自定义转换器的类名 |
| \<converter_name>.\<param_name> | No default | 自定义转换器的配置，配置信息根据转换器的使用方式来设置 |
| ddl.parse.fail.retry.read.table  | false                                          | 增量DDL解析失败后，处理DML事件时全量读取源表结构分析。schema.history.internal.skip.unparseable.ddl与ddl.parse.fail.retry.read.table均设置为true生效。|

### 数据类型映射

当Debezium检测到YashanDB的表的某个值发生更改时，会发出表示该更改的更改事件。每个更改事件记录的结构与原始表的结构相同，事件记录包含每个列值的字段。表列的数据类型决定了连接器如何在更改事件字段中表示列的值，最终Debezium需要将这些数据类型转换为Kafka所支持的数据类型。YashanDB Debezium Connector组件内置了这套映射关系供Debezium进行转换：

> **Note**:
>
> YashanDB中的XMLTYPE、JSON以及UDT数据类型未做映射，因此包含这些数据类型的表不支持对接Debezium。

| YashanDB data type     | Kafka data type                      |
| ---------------------- | ------------------------------------ |
| CHAR[(M)]              | STRING                               |
| NCHAR[(M)]             | STRING                               |
| VARCHAR[(M)]           | STRING                               |
| NVARCHAR[(M)]          | STRING                               |
| BLOB                   | BYTES                                |
| CLOB                   | STRING                               |
| NCLOB                  | STRING                               |
| RAW                    | BYTES                                |
| TINYINT                | INT8                                 |
| SMALLINT               | INT16                                |
| INT                    | INT32                                |
| BIGINT                 | INT64                                |
| FLOAT                  | FLOAT32                              |
| DOUBLE                 | FLOAT64                              |
| NUMBER                 | BYTES / INT8 / INT16 / INT32 / INT64 |
| BIT(1)                 | BOOLEAN                              |
| BIT(n)                 | BYTES                                |
| BOOLEAN                | BOOLEAN                              |
| DATE                   | INT64                                |
| TIME                   | INT64                                |
| TIMESTAMP              | INT64                                |
| INTERVAL YEAR TO MOUTH | FLOAT64                              |
| INTERVAL DAY TO SECOND | FLOAT64                              |
| ROWID                  | STRING                               |
| UROWID                 | STRING                               |

### 断点续传

为了应对异常场景下的数据一致性保证，本连接器提供了断点续传的功能，任务运行到增量阶段，基于kafka的两阶段提交机制，连接器将数据库日志点位提交到kafka，重启或者恢复任务时，连接器会从kafka中拉取最新的数据库日志点位，并从该数据库日志点位开始读取数据并发送到kafka，保证了连接器发送数据到kafka达到精确一次语义。

### 数据定制化转换

数据类型映射中，DATE、TIME、TIMESTAMP映射INT64，如果想定制成'yyyy-MM-dd HH:mm:ss.SSSSSS'的形式。connector提供了三种转换器来定制化转换这三种类型的数据。

| CustomConverter转换器                                        | 参数            | 说明                                            |
| ------------------------------------------------------------ | --------------- | ----------------------------------------------- |
| io.debezium.connector.yashandb.converters.TimestampToStringConverter | format:数据格式 | 将Timestamp类型数据转换成定制化格式的字符串数据 |
| io.debezium.connector.yashandb.converters.DateToStringConverter | format:数据格式 | 将Date类型数据转换成定制化格式的字符串数据      |
| io.debezium.connector.yashandb.converters.TimeToStringConverter | format:数据格式 | 将Time类型数据转换成定制化格式的字符串数据      |

使用样例，在配置里填写以下：

```plaintext
# 命名两个转换器，yashandb_timestamp_formatter转换器用于将TIMESTAMP类型数据转换，yashandb_date_formatter转换器用于将DATE类型数据转换
”converters“: "yashandb_timestamp_formatter,yashandb_date_formatter"
# yashandb_timestamp_formatter绑定成io.debezium.connector.yashandb.converters.TimestampToStringConverter类名
”yashandb_timestamp_formatter.type“：”io.debezium.connector.yashandb.converters.TimestampToStringConverter“
# timestamp数据格式化成yyyy-MM-dd HH:mm:ss.SSSSSS格式
”yashandb_timestamp_formatter.format.datetime“：”yyyy-MM-dd HH:mm:ss.SSSSSS“
# yashandb_date_formatter绑定成io.debezium.connector.yashandb.converters.DateToStringConverter类名
”yashandb_date_formatter.type“：”io.debezium.connector.yashandb.converters.DateToStringConverter“
# DATE数据格式化成yyyy-MM-dd格式
”yashandb_date_formatter.format.date“：”yyyy-MM-dd“
```

#### 如何定制化编写一个转换器？

参考连接[Debezium-custom-converters](https://debezium.io/documentation/reference/2.4/development/converters.html#custom-converters)。

下面的示例显示了实现该接口的 Java 类的转换器实现io.debezium.spi.converter.CustomConverter：

```java
public interface CustomConverter<S, F extends ConvertedField> {

    @FunctionalInterface
    interface Converter {  
        Object convert(Object input);
    }

    interface ConverterRegistration<S> { 
        void register(S fieldSchema, Converter converter); 
    }

    void configure(Properties props);

    void converterFor(F field, ConverterRegistration<S> registration); 
}

```

- interface Converter 接口：将数据从一种类型转换为另一种类型的函数。
- interface ConverterRegistration 接口：注册转换器的回调。
- register(S fieldSchema, Converter converter)：为当前字段注册给定的架构和转换器。不应针对同一字段多次调用。
- converterFor(F field, ConverterRegistration registration)：注册自定义值和模式转换器以供特定字段使用。

##### 自定义转换器方法

接口的实现CustomConverter必须包括以下方法：

1. configure()
   1. 将连接器配置中指定的属性传递给转换器实例。该configure方法在连接器初始化时运行。您可以将转换器与多个连接器一起使用，并根据连接器的属性设置修改其行为。
   2. 该configure方法接受以下参数：
      1. props
         包含要传递给转换器实例的属性。每个属性指定用于转换特定类型列的值的格式。
2. converterFor()
   1. 注册转换器以处理数据源中的特定列或字段。Debezium 调用该converterFor()方法以提示转换器调用转换registration。该converterFor方法对每一列运行一次。
   2. 该方法接受以下参数：
      1. field
         传递有关所处理字段或列的元数据的对象。列元数据可以包括列或字段的名称、表或集合的名称、数据类型、大小等。
      2. registration
         io.debezium.spi.converter.CustomConverter.ConverterRegistration提供目标架构定义和用于转换列数据的代码的类型的对象。registration当源列与转换器应处理的类型匹配时，转换器将调用该参数。调用该register方法为架构中的每个列定义转换器。架构使用 Kafka Connect API 表示SchemaBuilder。将来，将添加独立的架构定义 API。

##### Debezium 自定义转换器示例

下面的示例实现了一个简单的转换器，它执行以下操作：

- 运行该configure方法，该方法根据schema.name连接器配置中指定的属性值配置转换器。转换器配置特定于每个实例。

- 运行该converterFor方法，该方法注册转换器来处理数据类型设置为的源列中的值isbn。
  - STRING根据为属性指定的值识别目标架构schema.name。
  - 将源列中的 ISBN 数据转换为String值。

示例 1. 一个简单的自定义转换器

```java
public static class IsbnConverter implements CustomConverter<SchemaBuilder, RelationalColumn> {

    private SchemaBuilder isbnSchema;

    @Override
    public void configure(Properties props) {
        isbnSchema = SchemaBuilder.string().name(props.getProperty("schema.name"));
    }

    @Override
    public void converterFor(RelationalColumn column,
            ConverterRegistration<SchemaBuilder> registration) {

        if ("isbn".equals(column.typeName())) {
            registration.register(isbnSchema, x -> x.toString());
        }
    }
}
```

##### Debezium 和 Kafka Connect API 模块依赖关系

自定义转换器 Java 项目对 Debezium API 和 Kafka Connect API 库模块具有编译依赖项。这些编译依赖项必须包含在您的项目中pom.xml，如以下示例所示：

```xml
<dependency>
    <groupId>io.debezium</groupId>
    <artifactId>debezium-api</artifactId>
    <version>${version.debezium}</version> 
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>connect-api</artifactId>
    <version>${version.kafka}</version> 
</dependency>
```

- ${version.debezium}表示 Debezium 连接器的版本，根据connector支持的版本，这里应该是2.4.2.Final。
- ${version.kafka}代表您环境中的 Apache Kafka 版本。

## kafka 日志文件变更事件格式说明
Debezium 和 Kafka Connect 是围绕连续的事件消息流设计的。为了方便处理可变事件结构，Kafka Connect 中的每个事件都是自包含的。 每个消息键和值都有两部分：schema和payload。 schema描述了payload的结构，而payload包含实际数据。
默认情况下，每个数据更改事件采用JSON格式进行描述，每个数据更改事件都有一个键和一个值。

### 表数据变更事件格式说明
如下以test001例表分两种情况进行展示说明。
```sql
CREATE TABLE connect_user.test001 (
   COL001 NUMBER(6,0) PRIMARY KEY,
   COL002 VARCHAR2(100 BYTE),
   COL003 VARCHAR2(100 BYTE) NOT NULL,
   COL004 VARCHAR2(100 BYTE)
);
```

(1).当debezium从YaShanDB 快照数据迁移时，迁移connect_user.test001表数据时在Kafka 的topic生成的日志变更事件格式如下：

```json
{
  "schema": {
    "type": "struct",
    "fields": [
      {
        "type": "struct",
        "fields": [
          {
            "type": "int32",
            "optional": false,
            "field": "COL001"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL002"
          },
          {
            "type": "string",
            "optional": false,
            "field": "COL003"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL004"
          }
        ],
        "optional": true,
        "name": "my_topic11040003.CONNECT_USER.TEST001.Value",
        "field": "before"
      },
      {
        "type": "struct",
        "fields": [
          {
            "type": "int32",
            "optional": false,
            "field": "COL001"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL002"
          },
          {
            "type": "string",
            "optional": false,
            "field": "COL003"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL004"
          }
        ],
        "optional": true,
        "name": "my_topic11040003.CONNECT_USER.TEST001.Value",
        "field": "after"
      },
      {
        "type": "struct",
        "fields": [
          {
            "type": "string",
            "optional": false,
            "field": "version"
          },
          {
            "type": "string",
            "optional": false,
            "field": "connector"
          },
          {
            "type": "string",
            "optional": false,
            "field": "name"
          },
          {
            "type": "int64",
            "optional": false,
            "field": "ts_ms"
          },
          {
            "type": "string",
            "optional": true,
            "name": "io.debezium.data.Enum",
            "version": 1,
            "parameters": {
              "allowed": "true,last,false,incremental"
            },
            "default": "false",
            "field": "snapshot"
          },
          {
            "type": "string",
            "optional": false,
            "field": "db"
          },
          {
            "type": "string",
            "optional": true,
            "field": "sequence"
          },
          {
            "type": "string",
            "optional": false,
            "field": "schema"
          },
          {
            "type": "string",
            "optional": false,
            "field": "table"
          },
          {
            "type": "string",
            "optional": true,
            "field": "txId"
          },
          {
            "type": "string",
            "optional": true,
            "field": "scn"
          },
          {
            "type": "string",
            "optional": true,
            "field": "commit_scn"
          },
          {
            "type": "int32",
            "optional": true,
            "field": "batch_row_id"
          },
          {
            "type": "int64",
            "optional": true,
            "field": "position_scn"
          },
          {
            "type": "int64",
            "optional": true,
            "field": "group_lsn"
          },
          {
            "type": "int32",
            "optional": true,
            "field": "group_offset"
          },
          {
            "type": "bytes",
            "optional": true,
            "field": "instance_id"
          },
          {
            "type": "string",
            "optional": true,
            "field": "rs_id"
          },
          {
            "type": "int64",
            "optional": true,
            "field": "ssn"
          },
          {
            "type": "int32",
            "optional": true,
            "field": "redo_thread"
          },
          {
            "type": "string",
            "optional": true,
            "field": "user_name"
          }
        ],
        "optional": false,
        "name": "io.debezium.connector.yashandb.Source",
        "field": "source"
      },
      {
        "type": "string",
        "optional": false,
        "field": "op"
      },
      {
        "type": "int64",
        "optional": true,
        "field": "ts_ms"
      },
      {
        "type": "struct",
        "fields": [
          {
            "type": "string",
            "optional": false,
            "field": "id"
          },
          {
            "type": "int64",
            "optional": false,
            "field": "total_order"
          },
          {
            "type": "int64",
            "optional": false,
            "field": "data_collection_order"
          }
        ],
        "optional": true,
        "name": "event.block",
        "version": 1,
        "field": "transaction"
      }
    ],
    "optional": false,
    "name": "my_topic11040003.CONNECT_USER.TEST001.Envelope",
    "version": 1
  },
  "payload": {
    "before": null,
    "after": {
      "COL001": 100001,
      "COL002": "xxxxxxxxxxxxxxxxxx001",
      "COL003": "yyyyyyyyyyyyyyyyyyy001",
      "COL004": null
    },
    "source": {
      "version": "2.4.2.4.3",
      "connector": "yashandb",
      "name": "my_topic11040003",
      "ts_ms": 1762241220117,
      "snapshot": "last",
      "db": "",
      "sequence": null,
      "schema": "CONNECT_USER",
      "table": "TEST001",
      "txId": null,
      "scn": "755320504801107968",
      "commit_scn": null,
      "batch_row_id": 0,
      "position_scn": 755320504801107969,
      "group_lsn": 0,
      "group_offset": 0,
      "instance_id": "AAAAAAAAAAA=",
      "rs_id": null,
      "ssn": 0,
      "redo_thread": null,
      "user_name": null
    },
    "op": "r",
    "ts_ms": 1762241282908,
    "transaction": null
  }
}
```
(2).当debezium从YaShanDB 增量数据同步时，同步connect_user.test001表数据时在Kafka 的topic生成的日志变更事件格式如下：
```json
{
  "schema": {
    "type": "struct",
    "fields": [
      {
        "type": "struct",
        "fields": [
          {
            "type": "int32",
            "optional": false,
            "field": "COL001"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL002"
          },
          {
            "type": "string",
            "optional": false,
            "field": "COL003"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL004"
          }
        ],
        "optional": true,
        "name": "my_topic11040003.CONNECT_USER.TEST001.Value",
        "field": "before"
      },
      {
        "type": "struct",
        "fields": [
          {
            "type": "int32",
            "optional": false,
            "field": "COL001"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL002"
          },
          {
            "type": "string",
            "optional": false,
            "field": "COL003"
          },
          {
            "type": "string",
            "optional": true,
            "field": "COL004"
          }
        ],
        "optional": true,
        "name": "my_topic11040003.CONNECT_USER.TEST001.Value",
        "field": "after"
      },
      {
        "type": "struct",
        "fields": [
          {
            "type": "string",
            "optional": false,
            "field": "version"
          },
          {
            "type": "string",
            "optional": false,
            "field": "connector"
          },
          {
            "type": "string",
            "optional": false,
            "field": "name"
          },
          {
            "type": "int64",
            "optional": false,
            "field": "ts_ms"
          },
          {
            "type": "string",
            "optional": true,
            "name": "io.debezium.data.Enum",
            "version": 1,
            "parameters": {
              "allowed": "true,last,false,incremental"
            },
            "default": "false",
            "field": "snapshot"
          },
          {
            "type": "string",
            "optional": false,
            "field": "db"
          },
          {
            "type": "string",
            "optional": true,
            "field": "sequence"
          },
          {
            "type": "string",
            "optional": false,
            "field": "schema"
          },
          {
            "type": "string",
            "optional": false,
            "field": "table"
          },
          {
            "type": "string",
            "optional": true,
            "field": "txId"
          },
          {
            "type": "string",
            "optional": true,
            "field": "scn"
          },
          {
            "type": "string",
            "optional": true,
            "field": "commit_scn"
          },
          {
            "type": "int32",
            "optional": true,
            "field": "batch_row_id"
          },
          {
            "type": "int64",
            "optional": true,
            "field": "position_scn"
          },
          {
            "type": "int64",
            "optional": true,
            "field": "group_lsn"
          },
          {
            "type": "int32",
            "optional": true,
            "field": "group_offset"
          },
          {
            "type": "bytes",
            "optional": true,
            "field": "instance_id"
          },
          {
            "type": "string",
            "optional": true,
            "field": "rs_id"
          },
          {
            "type": "int64",
            "optional": true,
            "field": "ssn"
          },
          {
            "type": "int32",
            "optional": true,
            "field": "redo_thread"
          },
          {
            "type": "string",
            "optional": true,
            "field": "user_name"
          }
        ],
        "optional": false,
        "name": "io.debezium.connector.yashandb.Source",
        "field": "source"
      },
      {
        "type": "string",
        "optional": false,
        "field": "op"
      },
      {
        "type": "int64",
        "optional": true,
        "field": "ts_ms"
      },
      {
        "type": "struct",
        "fields": [
          {
            "type": "string",
            "optional": false,
            "field": "id"
          },
          {
            "type": "int64",
            "optional": false,
            "field": "total_order"
          },
          {
            "type": "int64",
            "optional": false,
            "field": "data_collection_order"
          }
        ],
        "optional": true,
        "name": "event.block",
        "version": 1,
        "field": "transaction"
      }
    ],
    "optional": false,
    "name": "my_topic11040003.CONNECT_USER.TEST001.Envelope",
    "version": 1
  },
  "payload": {
    "before": null,
    "after": {
      "COL001": 100002,
      "COL002": "xxxxxxxxxxxxxxxxxx002",
      "COL003": "yyyyyyyyyyyyyyyyyyy002",
      "COL004": null
    },
    "source": {
      "version": "2.4.2.4.3",
      "connector": "yashandb",
      "name": "my_topic11040003",
      "ts_ms": 1762217944460,
      "snapshot": "false",
      "db": "",
      "sequence": null,
      "schema": "CONNECT_USER",
      "table": "TEST001",
      "txId": "390922265",
      "scn": "755343132508905472",
      "commit_scn": null,
      "batch_row_id": 0,
      "position_scn": 755343132508905472,
      "group_lsn": 943349,
      "group_offset": 240,
      "instance_id": "AAAAAAAAAAA=",
      "rs_id": null,
      "ssn": 0,
      "redo_thread": null,
      "user_name": null
    },
    "op": "c",
    "ts_ms": 1762246744808,
    "transaction": null
  }
}
```
上述两者JSON数据的schema结构是相同的，差异主要体现在payload部分，反映了不同的操作类型和上下文。从两种数据格式我们可知变更事件总体上划分为schema、payload两部分，下面针对这两部分分别进行说明。

#### schema部分
Debezium 消息的 schema 部分采用分层结构设计，将数据变更事件拆解为多个逻辑模块。其核心包含三个层次：
* 数据层（通过 before 和 after 两个对称结构分别定义变更前/后的完整行数据）、
* 元数据层（source 结构描述数据库来源、表名、时间戳、SCN 等追踪信息）
* 操作控制层（op 字段标识操作类型，ts_ms 记录处理时间，transaction 管理事务边界）。

这种模块化设计通过明确的字段职责划分（如 before/after 的可空性对应不同操作场景）和统一的类型定义，既保证了数据的一致性描述，又为消费端提供了按需读取的灵活性，实现了变更数据的自描述性传输。


| 序号 | 关键词    | 功能作用                           | 值说明                |
|----|--------|--------------------------------|--------------------|
| 1  | fields | 定义一个结构体中所包含的字段列表               | 每个字段都有自己的类型、名称和可选项 |
| 2  | name  | 标识该 schema 的唯一名称，通常包含连接器、库、表信息 | 如："my_topic.CONNECT_USER.TEST001.Value"           |
| 3  | field  | 在父级结构中，标识当前嵌套结构体的字段名 | "before", "after", "source"  |
| 4  | optional  | 声明该字段是否允许为 null |true或 false。例如，插入操作时 before为 null            |
| 5  | type  | 定义字段的数据类型  | 如："struct", "int32", "string", "int64" |
| 6  | version  | Schema 的版本号   | 用于 schema 演化管理   |


#### payload部分
payload 部分的功能是承载数据变更事件的具体内容。它是根据 schema 定义所填充的实际数据值，是动态的、与具体操作相关的。其结构直接映射 schema 的定义，包含了数据变更的核心信息：变更前后的数据镜像（before 和 after）、操作类型（op）、变更的源数据库元信息（source）以及时间戳等。payload 是消费者业务逻辑处理的直接对象，回答了"发生了什么变更"、"变更的内容是什么"以及"在何处何时发生"等关键问题。

| 序号 | 关键词    | 功能作用                           | 值说明                                         |
|----|--------|--------------------------------|---------------------------------------------|
| 1  | before | 数据变更前的完整状态。用于更新和删除操作               | -插入(c)：null<br/>-更新(u)：变更前的行数据<br/>-删除(d)：被删除的行数据 |
| 2  | after  | 数据变更后的完整状态。用于插入和更新操作 | - 插入(c)：新增的行数据<br/>- 更新(u)：变更后的行数据 <br/>- 删除(d)：null        |
| 3  | source  | 描述变更事件的元数据，如数据来源、时间等 | 必选字段。包含 db、table、ts_ms（数据库时间）、scn 等关键信息                 |
| 4  | op  | 表示数据变更的操作类型 | - c：创建/插入 <br/>- u：更新 <br/>- d：删除 <br/>- r：快照（初始读取） |
| 5  | ts_ms  | 连接器处理该事件的时间戳（毫秒）  | 用于计算复制延迟      |
| 6  | transaction  | （可选）包含事务相关信息   | 如事务ID，用于将同一事务内的多个变更关联起来                              |


#### 特别说明


在生产环境的部分场景中，为降低网络传输与存储开销并提升处理效率，可配置 Debezium 在消息中省略 schema 信息。具体配置如下：

在 Debezium conncetor的配置文件config/connect-distributed.properties中，需将以下参数设置为false（原描述中配置为true）：
```properties
key.converter.schemas.enable=false
value.converter.schemas.enable=false
```
* schemas.enable 设为 false 时，Kafka 消息的 key 和 value 将仅包含原始数据，不附带 schema 元信息（字段结构、类型等），即不包含 schema 部分仅保留 payload 部分事件数据，从而减少单条消息的数据量，降低传输和存储开销，提升迁移效率。
* 上述实例中transaction项为null，是因为上述只捕获了数据的插入操作，如果需要事务操作，在begin...end进行事务操作时，begin操作会带上transaction项信息。
* 适用场景：适用于数据源与目标端表结构固定、双方对字段含义和类型已达成共识的场景（无需依赖 schema 进行字段映射）。

### DDL事件格式说明
除了表数据行的变更，Debezium 还能捕获并记录数据库表结构的变更（DDL），这对于保证数据定义的同步至关重要。Debezium 支持记录 Schema 元数据的历史变更信息，并将其存储在由配置项 schema.history.internal.kafka.topic 指定的 Kafka 主题中。其核心作用是追踪数据库表结构的全量变更历史，包括表的创建、修改等 DDL 操作。当 Debezium 连接器重启时，会通过读取这些历史记录重建表结构，确保能正确解析后续的数据变更事件。
这类 DDL事件数据采用分层结构设计，包含：
* 元数据层：记录变更相关的上下文信息；
* 位置信息层：精确定位变更在数据库日志中的位置；
* 结构变更层：详细描述表结构的具体修改内容。

这种分层设计既保证了 Schema 变更的可追溯性，也确保了连接器重启后的数据一致性。


在 Debezium source连接器任务的配置中，若设置了如下参数，则会启用 DDL 事件捕获功能，并将这些变更事件写入指定的 Kafka 主题（示例中为 schema-changes.inventory11040003）：
```json
"schema.history.internal.kafka.topic" : "schema-changes.inventory11040003"
```

如下以test001例表分两种情况进行展示说明：
```sql
CREATE TABLE connect_user.test001 (
   COL001 NUMBER(6,0) PRIMARY KEY,
   COL002 VARCHAR2(100 BYTE),
   COL003 VARCHAR2(100 BYTE) NOT NULL,
   COL004 VARCHAR2(100 BYTE)
);
```
（1）当迁移任务启动时，会把源库中已存在的connect_user.test001表元数据迁移到对应的Kafka topic中（schema-changes.inventory11040003），事件格式如下：
```json
{
  "source" : {
    "server" : "my_topic11040003"
  },
  "position" : {
    "instance_id" : "AAAAAAAAAAA=",
    "position_scn" : 755320504801107969,
    "ystream_start_scn" : "0",
    "group_lsn" : 0,
    "batch_row_id" : 0,
    "group_offset" : 0,
    "snapshot_scn" : "755320504801107968",
    "snapshot" : true,
    "scn" : "755320504801107968",
    "snapshot_completed" : false,
    "ystream_server_create" : false
  },
  "ts_ms" : 1762241280384,
  "databaseName" : "YASHANDB",
  "schemaName" : "CONNECT_USER",
  "ddl" : "CREATE TABLE \"CONNECT_USER\".\"TEST001\"\n(\"COL001\" NUMBER(6, 0),\n\"COL002\" VARCHAR(100),\n\"COL003\" VARCHAR(100) NOT NULL ENABLE,\n\"COL004\" VARCHAR(100),\nPRIMARY KEY (\"COL001\")\nUSING INDEX\nPCTFREE 8 INITRANS 2 MAXTRANS 255\nTABLESPACE \"USERS\" ENABLE\n) PCTFREE 8 INITRANS 2 MAXTRANS 255\nLOGGING\nTABLESPACE \"USERS\"\nSEGMENT CREATION DEFERRED\nORGANIZATION HEAP",
  "tableChanges" : [ {
    "type" : "CREATE",
    "id" : "\"CONNECT_USER\".\"TEST001\"",
    "table" : {
      "defaultCharsetName" : null,
      "primaryKeyColumnNames" : [ "COL001" ],
      "columns" : [ {
        "name" : "COL001",
        "jdbcType" : 2,
        "typeName" : "NUMBER",
        "typeExpression" : "NUMBER",
        "charsetName" : null,
        "length" : 6,
        "scale" : 0,
        "position" : 1,
        "optional" : false,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : false,
        "enumValues" : [ ]
      }, {
        "name" : "COL002",
        "jdbcType" : 12,
        "typeName" : "VARCHAR",
        "typeExpression" : "VARCHAR",
        "charsetName" : null,
        "length" : 100,
        "position" : 2,
        "optional" : true,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : true,
        "enumValues" : [ ]
      }, {
        "name" : "COL003",
        "jdbcType" : 12,
        "typeName" : "VARCHAR",
        "typeExpression" : "VARCHAR",
        "charsetName" : null,
        "length" : 100,
        "position" : 3,
        "optional" : false,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : false,
        "enumValues" : [ ]
      }, {
        "name" : "COL004",
        "jdbcType" : 12,
        "typeName" : "VARCHAR",
        "typeExpression" : "VARCHAR",
        "charsetName" : null,
        "length" : 100,
        "position" : 4,
        "optional" : true,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : true,
        "enumValues" : [ ]
      } ],
      "attributes" : [ ]
    },
    "comment" : null
  } ]
}
```
（2）当任务启动完成之后，用户对源库表connect_user.test001元数据做变更（如ALTER TABLE）：

```sql
ALTER TABLE connect_user.test001 ADD (
    COL005 VARCHAR2(100 BYTE)
);
```
在Kafka 的topic生成的日志变更事件格式如下：
```json
{
  "source" : {
    "server" : "my_topic11040003"
  },
  "position" : {
    "transaction_id" : null,
    "instance_id" : "AAAAAAAAAAA=",
    "position_scn" : 755373352680075264,
    "ystream_start_scn" : "0",
    "group_lsn" : 943455,
    "batch_row_id" : 0,
    "group_offset" : 1658,
    "snapshot_scn" : "755320504801107968",
    "ystream_server_create" : false
  },
  "ts_ms" : 1762254123528,
  "databaseName" : "",
  "schemaName" : "CONNECT_USER",
  "ddl" : "ALTER TABLE connect_user.test001 ADD (\r\n    COL005 VARCHAR2(100 BYTE)\r\n)",
  "tableChanges" : [ {
    "type" : "ALTER",
    "id" : "\"CONNECT_USER\".\"TEST001\"",
    "previousId" : "\"CONNECT_USER\".\"TEST001\"",
    "table" : {
      "defaultCharsetName" : null,
      "primaryKeyColumnNames" : [ "COL001" ],
      "columns" : [ {
        "name" : "COL001",
        "jdbcType" : 2,
        "typeName" : "NUMBER",
        "typeExpression" : "NUMBER",
        "charsetName" : null,
        "length" : 6,
        "scale" : 0,
        "position" : 1,
        "optional" : false,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : false,
        "enumValues" : [ ]
      }, {
        "name" : "COL002",
        "jdbcType" : 12,
        "typeName" : "VARCHAR",
        "typeExpression" : "VARCHAR",
        "charsetName" : null,
        "length" : 100,
        "position" : 2,
        "optional" : true,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : true,
        "enumValues" : [ ]
      }, {
        "name" : "COL003",
        "jdbcType" : 12,
        "typeName" : "VARCHAR",
        "typeExpression" : "VARCHAR",
        "charsetName" : null,
        "length" : 100,
        "position" : 3,
        "optional" : false,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : false,
        "enumValues" : [ ]
      }, {
        "name" : "COL004",
        "jdbcType" : 12,
        "typeName" : "VARCHAR",
        "typeExpression" : "VARCHAR",
        "charsetName" : null,
        "length" : 100,
        "position" : 4,
        "optional" : true,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : true,
        "enumValues" : [ ]
      }, {
        "name" : "COL005",
        "jdbcType" : 12,
        "typeName" : "VARCHAR2",
        "typeExpression" : "VARCHAR2",
        "charsetName" : null,
        "length" : 100,
        "position" : 5,
        "optional" : true,
        "autoIncremented" : false,
        "generated" : false,
        "comment" : null,
        "hasDefaultValue" : true,
        "enumValues" : [ ]
      } ],
      "attributes" : [ ]
    },
    "comment" : null
  } ]
} 
```
部分关键字描述如下：

| 序号 | 关键词 | 功能作用             | 取值范围/说明 |
|----|------|------------------|---------------|
| 1  | source.server | 标识产生此模式变更的连接器名称  | 字符串，如："my_topic11040003" |
| 2  | position.transaction_id | 事务id             | 字符串值 |
| 3  | position.instance_id | 数据库实例的唯一标识       | Base64编码的字节数组 |
| 4  | position.position_scn | 当前模式变更的系统变更号     | 长整型数字，用于精确定位 |
| 5  | position.ystream_start_scn | YStream服务启动时的SCN | 字符串格式的SCN值 |
| 6  | position.snapshot_scn | 快照开始的SCN         | 字符串格式的SCN值 |
| 7  | position.snapshot | 标识是否为快照模式        | 布尔值，true表示快照阶段 |
| 8  | position.scn | 模式变更发生时的SCN      | 字符串格式的SCN值 |
| 9  | position.snapshot_completed | 快照是否完成           | 布尔值，false表示快照进行中 |
| 10 | position.ystream_server_create | 是否创建了YStream服务   | 布尔值 |
| 11 | ts_ms | 模式变更记录的时间戳       | 长整型，毫秒时间戳 |
| 12 | databaseName | 数据库名称            | 字符串，如："YASHANDB" |
| 13 | schemaName | 模式/用户名           | 字符串，如："CONNECT_USER" |
| 14 | ddl | 实际的DDL语句         | 完整的SQL创建语句 |
| 15 | tableChanges.type | 表变更类型            | "CREATE"（创建表）、"ALTER"（修改表）等 |
| 16 | tableChanges.id | 表的完整标识           | 格式："schema.table" |
| 17 | tableChanges.table.primaryKeyColumnNames | 主键列名列表           | 字符串数组 |
| 18 | tableChanges.table.columns | 表的列定义数组          | 包含每个列的详细属性 |
| 19 | columns.name | 列名               | 字符串 |
| 20 | columns.jdbcType | JDBC数据类型代码       | 数字，如：2=NUMBER，12=VARCHAR |
| 21 | columns.typeName | 数据库类型名称          | 字符串，如："NUMBER"、"VARCHAR" |
| 22 | columns.length | 数据类型长度           | 数字 |
| 23 | columns.scale | 数字类型的小数位数        | 数字（仅数值类型有效） |
| 24 | columns.position | 列在表中的位置          | 数字，从1开始 |
| 25 | columns.optional | 列是否可为空           | 布尔值，false表示NOT NULL |
| 26 | columns.autoIncremented | 是否自增列            | 布尔值 |
| 27 | columns.hasDefaultValue | 是否有默认值           | 布尔值 |

## 常见问题

#### Q1. 报错“YashanDB does not yet have the YStream server ‘serverxx’ or check option 'database.ystream.server.name' if the parameters are filled in correctly. Please create and configure the YStream server, refer to the link 'xxx'.”该如何处理？

该报错表示`database.ystream.server.name`参数值对应的YStream服务不存在，请先调用DBMS_YSTREAM_ADM高级包对应函数创建YStream服务并完成相关配置。

#### Q2. 报错：YashanDB YStream server status is xxx. Please execute 'DBMS\_YSTREAM\_ADM.START( server\_name IN VARCHAR(64) );' start YStream server。

该报错表示`database.ystream.server.name`参数值对应的YStream服务处于非运行状态或者非启动状态，请先在数据库中调用DBMS_YSTREAM_ADM.START函数启动该YStream服务：

```plaintext
exec DBMS_YSTREAM_ADM.START('YStream服务名');
```

#### Q3. Decimal数值同步到Kafka后，为什么序列化出来数据出错？

debezium会将负scale的Decimal进行特殊处理，建议使用参数`decimal.handling.mode`=string来规避。

#### Q4. DATE\TIME\TIMESTAMP数值同步到Kafka后，为什么是时间戳的形式，而不是‘yyyy-MM-dd HH:mm:ss.SSSSSS’的形式？

debezium的默认处理方式是将时间类型映射到INT64，如果需要映射到固定格式数据，可参考上面的《数据定制化转换》。

#### Q5. 为什么元数据快照阶段快照了同步范围之外的表，比如日志中打印”Capturing structure of table xx.xx“？

debezium默认情况下会快照数据库中所有表的表结构，可以通过设置`schema.history.internal.store.only.captured.tables.ddl`为true，只快照同步范围内的表，具体解释可以查看连接器参数说明。

#### Q6. 如何才能配置监听到`truncate table xx`事件呢？

设置skipped.operations为none，具体解释可以查看连接器参数说明。