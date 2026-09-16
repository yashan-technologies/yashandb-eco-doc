Druid 是 Alibaba 开源的 JDBC 连接池和 SQL 处理组件，提供连接池、SQL 解析、监控统计和 SQL 防火墙等能力。YashanDB 适配版本可根据 `jdbc:yasdb:` URL 自动选择 YashanDB 驱动、连接有效性检查器和 Oracle 兼容 SQL 方言。本文介绍如何在 Java 和 Spring Boot 应用中使用 Druid 连接 YashanDB，以及如何使用 SQL 解析与 WallFilter 能力。

## 对接前准备

在进行对接操作前，请准备以下环境：

- JDK 8 或更高版本；使用 Spring Boot 3.x 或 4.x 时需使用 JDK 17 或更高版本。
- Maven 3.6 或更高版本。
- Druid 1.2.29.1 或 1.2.7 版本，推荐使用 1.2.29.1。
- 已在 [YashanDB 官网下载中心](https://download.yashandb.com/download)获取 YashanDB JDBC 驱动，或使用 Maven 依赖引入驱动。
- 已存在一个可正常访问的 YashanDB 服务端，并已获取主机、端口、用户名和密码。

本文示例使用以下连接信息，请替换为实际值：

| 配置项 | 示例值 | 说明 |
| --- | --- | --- |
| JDBC URL | `jdbc:yasdb://127.0.0.1:1688/yasdb` | 格式为 `jdbc:yasdb://<host>:<port>/<database>` |
| 驱动类 | `com.yashandb.jdbc.Driver` | Druid 可根据 URL 自动识别，也可显式配置 |
| 用户名 | `your_username` | YashanDB 登录用户 |
| 密码 | `your_password` | YashanDB 登录密码 |

## 添加依赖

在 Maven 项目的 `pom.xml` 中添加 Druid 和 YashanDB JDBC 驱动：

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.29.1</version>
    </dependency>
    <dependency>
        <groupId>com.yashandb</groupId>
        <artifactId>yashandb-jdbc</artifactId>
        <version>1.10.7</version>
    </dependency>
</dependencies>
```

> 支持 YashanDB 的 Druid 版本为 1.2.29.1 和 1.2.7，本文以 1.2.29.1 为例。可从 [YashanDB Druid Releases](https://github.com/yashan-technologies/druid/releases) 下载对应版本的发布包，也可检出所需标签后自行打包构建。以下命令将 1.2.29.1 安装到本地 Maven 仓库：

```bash
./mvnw versions:set -DnewVersion=1.2.29.1 -DgenerateBackupPoms=false
./mvnw -pl core -am -DskipTests install
```

如果通过下载中心获取 JDBC 驱动，请将驱动 JAR 安装到本地 Maven 仓库或直接加入应用的 classpath。驱动版本应与 YashanDB 服务端版本兼容，具体要求请以对应版本的 YashanDB 产品文档为准。

## 使用 DruidDataSource

以下示例创建连接池、执行一条查询并释放资源：

```java
import com.alibaba.druid.pool.DruidDataSource;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

public class DruidYashanDbDemo {
    public static void main(String[] args) throws Exception {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl("jdbc:yasdb://127.0.0.1:1688/yasdb");
        dataSource.setUsername("your_username");
        dataSource.setPassword("your_password");
        dataSource.setDriverClassName("com.yashandb.jdbc.Driver");

        dataSource.setInitialSize(2);
        dataSource.setMinIdle(2);
        dataSource.setMaxActive(10);
        dataSource.setMaxWait(60000);

        dataSource.setValidationQuery("SELECT 'x' FROM DUAL");
        dataSource.setValidationQueryTimeout(3);
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(false);
        dataSource.setTestOnReturn(false);
        dataSource.setTimeBetweenEvictionRunsMillis(60000);

        try {
            dataSource.init();
            System.out.println("Druid dbType: " + dataSource.getDbType());

            try (Connection connection = dataSource.getConnection();
                    Statement statement = connection.createStatement();
                    ResultSet resultSet = statement.executeQuery(
                            "SELECT CURRENT_TIMESTAMP FROM DUAL")) {
                if (resultSet.next()) {
                    System.out.println("YashanDB time: " + resultSet.getObject(1));
                }
            }
        } finally {
            dataSource.close();
        }
    }
}
```

运行后若能输出 `Druid dbType: yashandb` 和数据库时间，即表示 Druid 已识别 YashanDB，且连接池可正常获取和归还连接。

`driverClassName` 和 `validationQuery` 均可省略：Druid 会根据 `jdbc:yasdb:` URL 自动识别 `com.yashandb.jdbc.Driver`，YashanDB 连接有效性检查器默认使用 `SELECT 'x' FROM DUAL`。为便于排查配置问题，首次对接时建议显式配置。

## 在 Spring Boot 中使用

根据 Spring Boot 版本选择对应的 Druid Starter：

| Spring Boot 版本 | Maven artifactId |
| --- | --- |
| 2.x | `druid-spring-boot-starter` |
| 3.x | `druid-spring-boot-3-starter` |
| 4.x | `druid-spring-boot-4-starter` |

以 Spring Boot 3.x 为例，在 `pom.xml` 中添加以下依赖；使用 Starter 后无需再单独添加 `com.alibaba:druid`：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-3-starter</artifactId>
        <version>1.2.29.1</version>
    </dependency>
    <dependency>
        <groupId>com.yashandb</groupId>
        <artifactId>yashandb-jdbc</artifactId>
        <version>1.10.7</version>
    </dependency>
</dependencies>
```

在 `application.yml` 中配置数据源：

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:yasdb://127.0.0.1:1688/yasdb
    username: your_username
    password: your_password
    driver-class-name: com.yashandb.jdbc.Driver
    druid:
      initial-size: 2
      min-idle: 2
      max-active: 10
      max-wait: 60000
      validation-query: SELECT 'x' FROM DUAL
      validation-query-timeout: 3
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      time-between-eviction-runs-millis: 60000
      filter:
        stat:
          enabled: true
          log-slow-sql: true
          slow-sql-millis: 2000
        wall:
          enabled: true
```

如果使用的 Starter 构件在 Maven 仓库中不可用，可下载 Druid `1.2.29.1` 或 `1.2.7` 源码并自行构建。以 1.2.29.1 的 Spring Boot 3 Starter 为例，将前文安装命令中的 `core` 替换为对应 Starter 模块：

```bash
./mvnw -pl druid-spring-boot-3-starter -am -DskipTests install
```

应用可通过 Spring 标准方式注入 `DataSource` 并执行验证查询：

```java
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

@Service
public class DatabaseService {
    private final JdbcTemplate jdbcTemplate;

    public DatabaseService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Object currentDatabaseTime() {
        return jdbcTemplate.queryForObject(
                "SELECT CURRENT_TIMESTAMP FROM DUAL", Object.class);
    }
}
```

## 使用 YashanDB SQL 方言

Druid 将 YashanDB 作为独立数据库类型 `DbType.yashandb` 处理，并复用 Oracle 兼容的 Lexer、Parser 和 Visitor 基础设施。需要脱离连接池进行 SQL 格式化、解析或表名统计时，应显式传入该数据库类型：

```java
import com.alibaba.druid.DbType;
import com.alibaba.druid.sql.SQLUtils;
import com.alibaba.druid.sql.ast.SQLStatement;
import com.alibaba.druid.sql.visitor.SchemaStatVisitor;

import java.util.List;

String sql = "SELECT id, name FROM user1 WHERE id = 1";

String formattedSql = SQLUtils.format(sql, DbType.yashandb);
List<SQLStatement> statements = SQLUtils.parseStatements(sql, DbType.yashandb);

SchemaStatVisitor visitor = SQLUtils.createSchemaStatVisitor(DbType.yashandb);
statements.get(0).accept(visitor);

System.out.println(formattedSql);
System.out.println(visitor.getTables());
```

使用 `WallFilter` 时，Druid 会为 `yashandb` 数据库类型加载 Oracle 兼容的 SQL 防火墙规则。启用前应根据应用实际 SQL 验证规则，避免业务所需的管理语句或批量语句被默认策略拦截。

## XA 事务

需要 XA 事务时，可使用 `com.alibaba.druid.pool.xa.DruidXADataSource`。Druid 识别到 `yashandb` 后，会通过 YashanDB JDBC 驱动的 `com.yashandb.xa.YasXAConnection` 创建 XA 连接。除将数据源类型替换为 `DruidXADataSource` 外，还需在事务管理器中按其要求注册 `XADataSource`；请同时确认所用 JDBC 驱动和 YashanDB 服务端版本支持 XA。

## 常见问题

### Druid 未识别为 yashandb

检查 JDBC URL 是否以 `jdbc:yasdb:` 开头，并确认使用的是支持 YashanDB 的 Druid 1.2.29.1 或 1.2.7 版本。也可以显式配置驱动类 `com.yashandb.jdbc.Driver`。初始化后通过 `dataSource.getDbType()` 检查结果，预期值为 `yashandb`。

### 提示找不到 YashanDB 驱动类

如果出现 `ClassNotFoundException: com.yashandb.jdbc.Driver`，说明 JDBC 驱动未进入运行时 classpath。请检查 Maven 依赖是否生效，或确认手工添加的驱动 JAR 已被应用打包和加载。Druid 自身不会内置数据库驱动。

### 连接有效性检查失败

确认当前用户可执行 `SELECT 'x' FROM DUAL`，并检查网络、端口和账号权限。若需要使用自定义检查语句，可通过 `validationQuery` 或 Spring Boot 的 `spring.datasource.druid.validation-query` 修改；超时时间可通过 `validationQueryTimeout` 设置。

### WallFilter 拦截业务 SQL

先关闭 `wall` 验证是否由防火墙规则引起，再根据被拦截的 SQL 调整 WallFilter 配置。不要在未评估风险的情况下整体放开危险语句；生产环境应只开放业务确实需要的规则。
