GORM 是一款开源的 Go 语言 ORM 框架，yashandb-gorm（亦称 gorm-yasdb）是适配 GORM 框架的 YashanDB 方言库，基于 GORM 的 Dialector 机制实现，底层依赖崖山数据库 Go 驱动（yashandb-go）。本文将介绍如何通过 yashandb-gorm 实现 YashanDB 与 GORM 的对接，从而利用 ORM 框架进行基于 YashanDB 的 Go 应用开发。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项（以下版本不做严格要求，可按照需要灵活选择）：

- 已安装 Go 1.21 及以上版本的 Go 应用环境。

- 已安装 GORM v1.31.1 及以上（模块路径为 `gorm.io/gorm`），推荐版本如下：

    ```shell
    go get gorm.io/gorm@v1.31.1
    ```

- 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载 YashanDB Go 驱动包，或通过 Go 模块方式引入 `github.com/yashan-technologies/yashandb-go`。

- 已向我们的技术支持人员获取 yashandb-gorm 方言包，或通过 Go 模块方式引入 `github.com/yashan-technologies/yashandb-gorm`。

- 已存在一个可正常访问的 YashanDB 服务端。

### 兼容性说明

| 驱动版本 | 版本发布时间 | 新特性                               |
| -------- | ------------ | ------------------------------------ |
| 1.0.2    | 2026.3.18    | 首版本支持，依赖 yashandb-go v1.4.3 |

> 当前 GitHub 仓库（`yashan-technologies/yashandb-gorm`）已发布的最新版本为 **v1.0.2**，请使用 Git Tag 对应的版本号引入依赖，勿使用未发布的版本号。

yashandb-gorm 依赖 `gorm.io/gorm v1.31.1` 与 `yashandb-go v1.4.3`；其中 Go 驱动最低兼容 YashanDB C 驱动版本为 **v23.4.1.100**，建议使用 YashanDB **23.4.x** 系列数据库服务端。

## 对接配置

请参照如下步骤进行 YashanDB 与 GORM 的对接配置：

1. 在您的 Go 应用项目中初始化模块（如尚未初始化）：

    ```shell
    go mod init your-project
    ```

2. 在 `go.mod` 文件中添加如下依赖项：

    ```go
    require (
        github.com/yashan-technologies/yashandb-gorm v1.0.2
        gorm.io/gorm v1.31.1
    )
    ```

    也可通过以下命令直接安装指定版本：

    ```shell
    go get github.com/yashan-technologies/yashandb-gorm@v1.0.2
    ```

    执行以下命令拉取依赖：

    ```shell
    go mod tidy
    ```

3. 配置数据库连接。yashandb-gorm 采用 DSN 格式连接 YashanDB，格式为：

    ```
    username/password@host:port
    ```

    请将 `your_username`、`your_password`、`your_host` 和 `your_port` 修改为实际值。

4. 在代码中通过 `gorm.Open` 打开数据库连接：

    ```go
    import (
        yasdb "github.com/yashan-technologies/yashandb-gorm"
        "gorm.io/gorm"
    )

    func main() {
        dsn := "your_username/your_password@your_host:your_port"
        db, err := gorm.Open(yasdb.Open(dsn), &gorm.Config{})
        if err != nil {
            panic(err)
        }
        // 后续可进行数据库操作
    }
    ```

5. （可选）如需在本地开发时引用源码版本的 yashandb-gorm，可在 `go.mod` 中添加 `replace` 指令：

    ```go
    replace github.com/yashan-technologies/yashandb-gorm => /path/to/yashandb-gorm
    ```

6. （可选）如需自定义连接池参数，可通过标准 `database/sql` 接口进行配置：

    ```go
    sqlDB, err := db.DB()
    if err != nil {
        panic(err)
    }
    sqlDB.SetMaxOpenConns(100)
    sqlDB.SetMaxIdleConns(10)
    ```

7. （可选）选择命名模式。yashandb-gorm 通过 `NamingCaseSensitive` 配置项控制表名、列名的大小写处理方式，须在 `gorm.Open` 时指定：

    - **默认模式**（推荐）：使用第 4 步中的 `yasdb.Open(dsn)` 即可。表名、列名默认统一转为大写，适用于大多数 YashanDB 场景。
    - **大小写敏感模式**：若库中表名、列名须严格保留原始大小写，使用 `yasdb.New` 并开启 `NamingCaseSensitive`：

    ```go
    db, err := gorm.Open(yasdb.New(yasdb.Config{
        DSN:                 "your_username/your_password@your_host:your_port",
        NamingCaseSensitive: true,
    }), &gorm.Config{})
    if err != nil {
        panic(err)
    }
    ```

上述配置完成后，即可开始您的 GORM 开发，连接和操作 YashanDB 服务端。

yashandb-gorm 支持 `db.AutoMigrate(&YourModel{})` 自动建表/补列；生产环境建议仍优先使用手工建表 SQL，便于控制序列、索引等细节。

## 简单使用示例

上述对接配置完成后，开发者即可开始定义 Go 结构体与 YashanDB 表对象进行映射，以下为一个简单的实现示例：

1. 在 YashanDB 中创建如下表对象：

    ```sql
    -- 建表语句
    CREATE TABLE user1
    (
        id   INT PRIMARY KEY,
        name VARCHAR(30) NULL
    );

    -- 预置5条数据
    INSERT INTO USER1 (id, name) VALUES
    (1, 'Jone'),
    (2, 'Jack'),
    (3, 'Tom'),
    (4, 'Sandy'),
    (5, 'Billie');
    ```

2. 定义 User 结构体，并使其与 YashanDB 的 user1 表对应：

    ```go
    type User struct {
        ID   int    `gorm:"primaryKey"`
        Name string `gorm:"size:30"`
    }

    func (User) TableName() string {
        return "user1"
    }
    ```

    > **Note**：
    >
    > - 结构体字段须与第 1 步建表语句中的列一一对应（`ID` 对应 `id`，`Name` 对应 `name`）。
    > - yashandb-gorm 默认将表名和列名转换为大写，上述结构体将映射为 `USER1` 表，列名为 `ID`、`NAME`。

3. 插入数据：

    ```go
    user := User{ID: 20, Name: "zhang"}
    if err := db.Create(&user).Error; err != nil {
        panic(err)
    }

    var users []User
    if err := db.Find(&users).Error; err != nil {
        panic(err)
    }
    fmt.Printf("当前共 %d 条记录\n", len(users)) // 建表时预置了5条，插入后为6条
    ```

4. 查询数据：

    ```go
    var found User
    if err := db.Where("name = ?", "Jone").First(&found).Error; err != nil {
        panic(err)
    }
    fmt.Printf("查询成功: ID=%d, Name=%s\n", found.ID, found.Name)
    ```

5. 更新与删除数据：

    ```go
    // 更新
    db.Model(&found).Update("name", "JoneUpdated")

    // 删除
    db.Delete(&found)
    ```

6. 事务操作示例：

    ```go
    err := db.Transaction(func(tx *gorm.DB) error {
        u := User{ID: 21, Name: "Alice"}
        if err := tx.Create(&u).Error; err != nil {
            return err
        }
        return nil
    })
    if err != nil {
        panic(err)
    }
    ```

## 常见问题

以下说明均基于上文 **user1** 表（仅含 `id`、`name` 两列）展开；若涉及其他表结构，文中会单独给出建表语句。

### 自增主键的使用问题

一般情况下，YashanDB 里自增主键通过 **Sequence（序列）** 实现。若需在 **user1** 表上使用自增主键，须先在库中创建序列并绑定到 `id` 列：

```sql
CREATE SEQUENCE sequence__USER1_ START WITH 10 INCREMENT BY 1;
ALTER TABLE user1 MODIFY id INT DEFAULT sequence__USER1_.nextval;
```

此后插入数据时可不指定 `id`，由数据库自动生成主键值（仍使用上文定义的 `User` 结构体）：

```go
user := User{Name: "zhang"}
if err := db.Create(&user).Error; err != nil {
    panic(err)
}
fmt.Println(user.ID) // 输出数据库自动生成的 ID
```

需注意的是，自增主键回填依赖单行 `INSERT ... RETURNING INTO`。对切片调用 `Create` 时，yashandb-gorm 会按行循环执行单行插入并回填主键。若回填异常（例如主键仍为 0），可改为显式循环单行插入，或插入后通过其他条件查询获取主键值。

### 表名与列名大小写问题

yashandb-gorm 提供两种命名模式，通过连接配置项 `NamingCaseSensitive` 选择（默认关闭）。

#### 默认模式（推荐）

使用 `yasdb.Open(dsn)` 连接时，`NamingCaseSensitive` 为 `false`（默认值）。在此模式下，yashandb-gorm 内置命名策略**默认**会将表名和列名统一转换为**大写**。例如结构体 `User` 通过 `TableName()` 映射为 `USER1` 表，字段 `Name` 映射为 `NAME` 列。

在编写 `Where` 等查询条件时，建议使用结构体字段名或小写形式，GORM 会自动完成映射：

```go
var user User
db.Where("name = ?", "Jone").First(&user)
```

若表列为下划线命名（如 `raw_data`），请在结构体上使用 `gorm:"column:raw_data"` 等标签显式映射；驱动会在运行时将列名统一转为大写，以匹配 YashanDB 返回的列名。

若需将结构体映射到其他表名，可实现 `TableName` 方法（须提前在库中创建对应表）：

```go
func (User) TableName() string {
    return "my_user1"
}
```

#### 大小写敏感模式

若数据库中的表名、列名需要**严格保留原始大小写**（如小写列名 `user_name`），可开启大小写敏感模式。此时生成的 SQL 会对标识符统一加双引号，不再自动转大写：

```go
db, err := gorm.Open(yasdb.New(yasdb.Config{
    DSN:                 "your_username/your_password@your_host:your_port",
    NamingCaseSensitive: true,
}), &gorm.Config{})
```

| 模式 | 连接方式 | 普通列名 `user_name` | 保留字 `ORDER` |
| ---- | -------- | -------------------- | -------------- |
| 默认（关闭） | `yasdb.Open(dsn)` | `USER_NAME`（转大写，不加引号） | `"ORDER"`（转大写加引号） |
| 大小写敏感（开启） | `yasdb.New(Config{NamingCaseSensitive: true})` | `"user_name"`（原样加引号） | `"ORDER"`（原样加引号） |

> **Note**：
>
> - 大小写敏感模式下，结构体中的表名、列名须与数据库中的实际大小写一致。
> - 若需在数据库中保留精确的小写列名，也可在默认模式下通过 `gorm:"column:\"name\""` 手动加引号，无需开启大小写敏感模式。

### 保留字字段名问题

YashanDB 存在大量 SQL 保留字（如 `USER`、`INDEX`、`ORDER` 等）。建议开发时尽量避免使用保留字作为表名或列名。如确需使用，可通过 `gorm` 标签显式指定列名。以下以 **task1** 表为例：

```sql
CREATE TABLE task1
(
    id         INT PRIMARY KEY,
    task_index VARCHAR(128) NULL
);
```

```go
type Task struct {
    ID        int    `gorm:"primaryKey"`
    TaskIndex string `gorm:"size:128"` // 映射到 TASK_INDEX 列，避免使用保留字 INDEX 作为字段名
}

func (Task) TableName() string {
    return "task1"
}
```

### 数据类型映射问题

不同数据库在数据类型上存在差异，使用 yashandb-gorm 时 Go 类型与 YashanDB 类型的默认映射关系如下：

| Go / GORM 类型 | YashanDB 类型 | 说明 |
| -------------- | ------------- | ---- |
| int、uint | BIGINT | 自增字段附加 Sequence 默认值 |
| float | DOUBLE | |
| bool | BOOLEAN | |
| string | VARCHAR(n) | 未指定 `size` 时默认 n=1024；大文本请使用 `gorm:"type:text"` / `gorm:"type:clob"` 映射为 CLOB（勿仅依赖极大的 `size` 值） |
| time.Time | TIMESTAMP | |
| []byte | BLOB | |
| 自定义 json 类型 | BLOB | 通过 `gorm:"type:json"` 标签标识 |

例如，其他数据库中的 `TEXT` 类型在 YashanDB 中应使用 `CLOB` 代替。普通 `string` 字段在 AutoMigrate 时默认映射为 `VARCHAR`，不会自动建成 CLOB。若需大文本列，请使用 `gorm:"type:text"` / `gorm:"type:clob"`；也可先手工建表再映射，如下以 **article1** 表为例（须提前建表）：

```sql
CREATE TABLE article1
(
    id      INT PRIMARY KEY,
    content CLOB NULL
);
```

```go
type Article struct {
    ID      int    `gorm:"primaryKey"`
    Content string // 表列已是 CLOB 时，用 string 读写即可
}

func (Article) TableName() string {
    return "article1"
}
```

### CLOB 与 BLOB 字段的使用

在 Go 结构体中，建议将 CLOB 类型字段定义为 `string`，BLOB 类型字段定义为 `[]byte`，可直接完成读写，无需额外处理。以下以 **document1** 表为例（须提前建表）：

```sql
CREATE TABLE document1
(
    id       INT PRIMARY KEY,
    title    VARCHAR(128) NULL,
    content  CLOB NULL,
    raw_data BLOB NULL
);
```

```go
type Document struct {
    ID      int    `gorm:"primaryKey"`
    Title   string
    Content string // 表列为 CLOB 时，用 string 读写即可
    RawData []byte `gorm:"column:raw_data"` // 表列为 BLOB
}

func (Document) TableName() string {
    return "document1"
}
```

> **Note**：
>
> - 本节示例假定表已按上方 SQL 手工创建。若使用 AutoMigrate 且未指定大文本类型，`string` 默认映射为 `VARCHAR(1024)`，不会建成 CLOB。
> - 建表使用下划线列名（如 `raw_data`）时，须用 `gorm:"column:raw_data"` 显式指定；驱动会在运行时统一转为大写，以匹配 YashanDB 返回的列名。

### Upsert（冲突更新）的使用

GORM 提供的 `OnConflict` 子句在 yashandb-gorm 中会被转换为 YashanDB 支持的 `ON DUPLICATE KEY UPDATE` 语法。以下以 **user2** 表为例（含唯一约束列，须提前建表）：

```sql
CREATE TABLE user2
(
    id    INT PRIMARY KEY,
    name  VARCHAR(30) NULL,
    email VARCHAR(128) NULL
);
CREATE UNIQUE INDEX idx_user2_email ON user2 (email);
```

```go
import "gorm.io/gorm/clause"

type User2 struct {
    ID    int    `gorm:"primaryKey"`
    Name  string `gorm:"size:30"`
    Email string `gorm:"size:128"`
}

func (User2) TableName() string {
    return "user2"
}

user := User2{ID: 1, Name: "orig", Email: "test@example.com"}
db.Create(&user)

db.Clauses(clause.OnConflict{
    DoUpdates: clause.AssignmentColumns([]string{"name"}),
}).Create(&User2{ID: 1, Name: "updated", Email: "test@example.com"})
```

> **Note**：
>
> - 冲突判定依赖表上的主键或唯一约束（上例为 `email` 列的唯一索引），由数据库在执行 `ON DUPLICATE KEY UPDATE` 时判定；`OnConflict.Columns` 不会改变本方言生成的 SQL，可省略。
> - 执行 Upsert 时须为 `id` 等必填列赋值，不可为 NULL。
> - 未指定 `DoUpdates` 时，默认更新所有非主键列；`DoNothing: true` 表示冲突时不更新。

### 分页查询说明

使用 `Limit` 且未指定 `Order` 时，驱动会自动按主键补 `ORDER BY`，以保证分页语法合法。

### 外键 ON UPDATE 不支持

YashanDB 不支持外键的 `ON UPDATE` 子句。在手动编写建表 SQL 定义外键时，请勿添加 `ON UPDATE` 子句。

### 调试 SQL 语句

开发阶段可通过 GORM 的 `Debug` 方法打印实际执行的 SQL 语句，便于排查问题（以下示例基于 **user1** 表）：

```go
var user User
db.Debug().Where("name = ?", "Jone").First(&user)
```
