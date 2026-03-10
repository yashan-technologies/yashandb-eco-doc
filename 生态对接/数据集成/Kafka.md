Apache Kafka是一款开源的分布式流处理平台，主要应用于大数据实时数据处理。YashanDB支持通过Kafka-Connect-Jdbc连接器将数据导入到Kafka中，本文将对这一对接过程进行介绍。

本文提供该功能的验证过程介绍，所使用测试环境为Centos7.9.2009。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项（以下版本不做严格要求，可按照需要灵活选择）：

1. 已下载Kafka安装包，例如：

   ```shell
   wget https://archive.apache.org/dist/kafka/3.2.0/kafka_2.13-3.2.0.tgz
   ```

2. 已下载Kafka-Connect-Jdbc安装包，例如：

   ```shell
   wget https://d1i4a15mxbxib1.cloudfront.net/api/plugins/confluentinc/kafka-connect-jdbc/versions/10.5.1/confluentinc-kafka-connect-jdbc-10.5.1.zip
   ```

3. 已存在一个可正常访问的YashanDB服务端。

## 对接配置

请参照如下步骤进行YashanDB与Kafka的对接配置（以单节点启动Kafka为例）：

1. 进入Kafka安装包所在目录，解压安装包，并更改安装目录名称：

    ```shell
    $ tar -zxvf kafka_2.13-3.2.0.tgz 
    $ mv kafka_2.13-3.2.0  kafka
    ```

3. 启动Zookeeper：

    ```shell
    $ ./kafka/bin/zookeeper-server-start.sh ./kafka/config/zookeeper.properties
    ```

4. 启动Kafka：

    ```shell
    $ ./kafka/bin/kafka-server-start.sh ./kafka/config/server.properties 
    ```

5. 将下载的Kafka-Connect-Jdbc安装包移至Kafka安装包所在目录（如已在相同目录则跳过）：

	```shell
	$ mv /.../confluentinc-kafka-connect-jdbc-10.5.1.zip ./
	```

6. 创建plugins目录，并将Kafka-Connect-Jdbc安装包解压到plugins目录下：

    ```shell
    $ mkdir plugins
    $ unzip confluentinc-kafka-connect-jdbc-10.5.1.zip  -d  ./plugins/
    ```
    
7. 将Kafka-Connect-Jdbc安装目录下的etc/source-quickstart-sqlite.properties复制一份到kafka/config并改名为source-quickstart-yasdb.properties：

    ```shell
    $ cp ./plugins/confluentinc-kafka-connect-jdbc-10.5.1/etc/source-quickstart-sqlite.properties ./kafka/config/
    $ cd kafka/config
    $ mv source-quickstart-sqlite.properties source-quickstart-yasdb.properties
    ```

8. 编辑source-quickstart-yasdb.properties文件，提供JDBC连接信息（请将your_host、your_port、your_dbname、your_username和your_password修改为实际值）：

    ```properties
    name=test-source-yasdb-jdbc-connect
    connector.class=io.confluent.connect.jdbc.JdbcSourceConnector
    tasks.max=1
    # The remaining configs are specific to the JDBC source connector. In this example, we connect to a
    # SQLite database stored in the file test.db, use and auto-incrementing column called 'id' to
    # detect new rows as they are added, and output to topics prefixed with 'test-sqlite-jdbc-', e.g.
    # a table called 'users' will be written to the topic 'test-sqlite-jdbc-users'.
    connection.url=jdbc:yasdb://your_host:your_port/your_dbname
    connection.user=your_username
    connection.password=your_password
    query=select * from user1 limit 100
    mode=bulk
    incrementing.column.name=id
    topic.prefix=test-yasdb-jdbc-
    ```
    
9. 在同目录下新建connect-standalone.properties文件，配置pulgins目录信息（请替换为实际的plugins路径）：

    ```properties
    $ vi connect-standalone.properties
    
    plugin.path=/home/kkadmin/kafka/plugins
    ```
    
10. 回到上级目录，即Kafka安装包所在目录，启动Kafka到YashanDB的连接：

    ```shell
    $ cd ..
    $ ./bin/connect-standalone.sh ./config/connect-standalone.properties ./config/source-quickstart-yasdb.properties
    ```

上述对接配置完成后，开发者即可开始使用Kafka，执行YashanDB的数据导入。
