## YashanDB 生态对接文档

本仓库维护 YashanDB 与常用数据库工具、开发框架、数据集成平台及其他开源生态组件的对接文档，帮助开发者完成驱动、方言、Connector 或协议配置，并验证基本使用流程。

文档内容会定期与 [YashanDB 官网文档中心](https://doc.yashandb.com/) 同步。具体产品版本、驱动下载和兼容性要求，请以当前版本的产品说明和组件文档为准。

- [YashanDB 官网](https://www.yashandb.com/)
- [产品下载中心](https://download.yashandb.com/)
- [生态对接总览](生态对接/00生态对接.md)
- [贡献指南](CONTRIBUTING.md)

### 快速找到文档

根据你的使用场景选择入口：

| 使用场景 | 推荐入口 | 关键准备 |
| --- | --- | --- |
| 使用图形化或命令行 SQL 工具 | [SQL 工具](生态对接/00生态对接.md#sql-tools) | 根据工具选择 YashanDB JDBC 驱动或 MySQL 协议，并确认服务端模式和端口 |
| 开发 Java 应用或 ORM 项目 | [Java ORM 文档](生态对接/00生态对接.md#orm-frameworks) | 准备 YashanDB JDBC 驱动；使用 Hibernate 时还需要对应方言 |
| 开发 Python 应用或 ORM 项目 | [Python ORM 文档](生态对接/00生态对接.md#orm-frameworks) | 准备 YashanDB Python 驱动和对应 ORM 方言 |
| 开发 .NET 应用或 ORM 项目 | [Entity Framework 6](生态对接/ORM框架/EntityFramework6.md)、[Entity Framework Core 6](生态对接/ORM框架/EntityFrameworkCore6.md)、[SqlSugar](生态对接/ORM框架/SqlSugar.md) | 根据组件要求准备 ADO.NET、ODBC 驱动或适配包 |
| 开发 Go 应用或 ORM 项目 | [GORM](生态对接/ORM框架/GORM.md) | 准备 YashanDB Go 驱动和 GORM 方言包 |
| 批量导入导出、流式处理或 CDC | [数据集成文档](生态对接/00生态对接.md#data-integration) | 重点确认 Connector、YStream、Kafka、Flink 或 Spark 的版本兼容关系 |
| 发布地理空间数据 | [GeoServer](生态对接/其他/GeoServer.md) | 准备 YashanDB JDBC 驱动和匹配的 GeoServer 方言包 |

### 组件索引

#### SQL 工具

| 组件 | 接入方式或场景 | 文档 |
| --- | --- | --- |
| DataGrip | 使用 YashanDB JDBC 驱动 | [查看文档](生态对接/SQL工具/DataGrip.md) |
| DBeaver | 使用 YashanDB JDBC 驱动 | [查看文档](生态对接/SQL工具/DBeaver.md) |
| MySQL Client | 通过 MySQL 协议连接 mysql 模式 | [查看文档](生态对接/SQL工具/MySQL Client.md) |
| Navicat | 通过 MySQL 协议连接 mysql 模式 | [查看文档](生态对接/SQL工具/Navicat.md) |

#### ORM 框架

| 组件 | 接入方式或场景 | 文档 |
| --- | --- | --- |
| Django | Python 驱动和 Django 方言 | [查看文档](生态对接/ORM框架/Django.md) |
| Entity Framework 6 | ADO.NET 驱动和 EF6 适配 | [查看文档](生态对接/ORM框架/EntityFramework6.md) |
| Entity Framework Core 6 | ADO.NET 驱动和 EF Core 6 适配 | [查看文档](生态对接/ORM框架/EntityFrameworkCore6.md) |
| GORM | Go 驱动和 GORM 方言 | [查看文档](生态对接/ORM框架/GORM.md) |
| Hibernate | JDBC 驱动和 Hibernate 方言 | [查看文档](生态对接/ORM框架/Hibernate.md) |
| MyBatis | YashanDB JDBC 驱动 | [查看文档](生态对接/ORM框架/MyBatis.md) |
| MyBatis-Plus | YashanDB JDBC 驱动 | [查看文档](生态对接/ORM框架/MyBatis-Plus.md) |
| SQLAlchemy | Python 驱动和 SQLAlchemy 方言 | [查看文档](生态对接/ORM框架/SQLAIChemy.md) |
| SqlSugar | ODBC 驱动和 SqlSugar 适配包 | [查看文档](生态对接/ORM框架/SqlSugar.md) |

#### 数据集成

| 组件 | 接入方式或场景 | 文档 |
| --- | --- | --- |
| Debezium | YStream 和 Debezium Connector | [查看文档](生态对接/数据集成/Debezium.md) |
| Flink | JDBC、Flink Connector 和 YStream CDC | [查看文档](生态对接/数据集成/Flink.md) |
| Kafka | Kafka Connect JDBC | [查看文档](生态对接/数据集成/Kafka.md) |
| mydumper | mysql 模式单表导出 | [查看文档](生态对接/数据集成/mydumper.md) |
| mysqldump | mysql 模式整库导出 | [查看文档](生态对接/数据集成/mysqldump.md) |
| Spark | JDBC 或 Spark Connector | [查看文档](生态对接/数据集成/Spark.md) |

#### 其他生态

| 组件 | 接入方式或场景 | 文档 |
| --- | --- | --- |
| GeoServer | JDBC 驱动和 GeoServer 方言 | [查看文档](生态对接/其他/GeoServer.md) |

### 文档约定

- 每级目录可包含必选的 `00` 前言文件、可选的 `_index.md`、可选的 `image/` 目录及文档文件。
- `_index.md` 控制官网导航顺序；`filename` 使用逗号分隔子目录或文件名，`enName` 与其逐项对应。
- 图片统一放在当前目录的 `image/` 目录下，使用 PNG 格式，并通过相对路径引用，例如 `./image/example.png`。
- 新增、删除或重命名组件时，同步更新组件文档、README 组件索引和相关 `_index.md`。
- 保留官网扩展语法（例如 `::: tabs`），不要改写为普通 Markdown。
- 提交前确认本地图片、Markdown 链接和示例均可访问或可执行。

### 参与贡献

欢迎通过 Issue 和 Pull Request 改进文档。开始修改前请阅读[贡献指南](CONTRIBUTING.md)，其中包含：

- Issue、分支、Commit 和 PR 的命名要求
- 本地 `opendoccheck` 检查命令
- 文档、索引、图片和官网导航的同步规则
- 评审、合入和发布流程

提交前至少完成以下检查：

1. 为变更关联当前仓库中的有效 Issue。
2. 同步更新受影响的文档、索引、图片和 README 入口。
3. 按贡献指南运行本地文档检查，并修复所有非零结果。
4. 确认 PR 标题、Commit Message 和 PR 正文符合规范。

### 反馈与问题

如果发现文档缺失、步骤无法执行或内容与当前产品版本不一致，请提交 Issue，并附上产品版本、组件版本、操作系统、复现步骤和实际结果，便于维护者定位问题。
