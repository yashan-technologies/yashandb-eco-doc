MySQL Client是MySQL数据库的核心命令行工具，YashanDB（mysql模式）在语法上兼容MySQL数据库，并支持使用MySQL Client连接YashanDB服务端进行SQL操作。本文将对此对接过程进行介绍。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装MySQL 5.7.42，该版本内置的MySQL Client版本为14.14
2. 已存在一个可正常访问的YashanDB服务端，且服务端运行于mysql模式。

## 对接配置

在命令行输入如下语句即可实现YashanDB与MySQL Client的对接：

```bash
$ mysql -u your_name -p your_password -h your_host -P your_port
```

- your_name：YashanDB中已创建并拥有合适权限的用户名称，请注意不可使用sys用户
- your_password：YashanDB用户的密码
- your_host：YashanDB服务端IP（多实例形态如YAC，选择其中一个IP）
- your_port：YashanDB服务端上的MySQL监听端口（默认安装情况下为1690）

## 使用简介

上述对接配置完成后，即进入MySQL Client的交互式命令界面，开发者可输入命令或SQL语句操作YashanDB。

### SHOW命令

SHOW命令用于展示数据库的各项信息，以下列出YashanDB支持的命令选项：

#### SHOW {DATABASES |SCHEMAS}

仅用于兼容，查询结果恒为当前的database。

#### SHOW MASTER STATUS

仅用于兼容，查询结果恒为空集。

#### SHOW SLAVE STATUS [FOR CHANNEL channel]

仅用于兼容，查询结果恒为空集。

#### SHOW {GLOBAL|SESSION}STATUS

显示会话的状态。

#### SHOW {CHARSET |CHARACTER SET}

显示支持的字符集。

#### SHOW COLLATION

显示支持的字符序。

#### SHOW [FULL]PROCESSLIST

显示会话进程列表信息。

- 不指定FULL关键字时，最多显示100条信息。

- 指定FULL关键字时，显示所有信息，

#### SHOW [FULL]TABLES [{FROM|IN}database\_name]

显示指定database_name中所有的表。
FULL关键字用于显示表类型。

#### SHOW [FULL]{COLUMNS |FIELDS}{FROM |IN}table\_name[{FROM|IN}database\_name]

显示指定table_name表的列定义信息。
FULL关键字用于显示列的字符序与注释。

#### SHOW {GLOBAL |SESSION}VARIABLES

显示系统变量。

- GLOBAL:显示全局变量，等同于查询V_SMYSQL_GLOBAL_VARIABLES视图。

- SESSION:显示会话变量，等同于查询V_5MYSQL_VARIABLES视图。

#### SHOW TABLE STATUS [{FROM|IN}database\_name]

显示指定database_name库中所有表的详细信息（包含视图）。

#### SHOW PROCEDURE STATUS

显示所有存储过程的详细信息。

#### SHOW FUNCTION STATUS

显示所有函数的详细信息。

#### SHOW {INDEX|INDEXES|KEYS}{FROM |IN}table\_name [{FROMIIN}database\_name]WHERE expr

显示指定table_name表上的素引或键。

#### SHOW CREATE {DATABASE |SCHEMA}[IF NOT EXISTS]database\_name

仅用于兼容，无实际含义。

#### SHOW CREATE TABLE table\_name

显示指定table_name表的定义，必须指定为当前用户的表。

#### SHOW CREATE TRIGGER trigger\_name

显示指定trigger_name触发器的定义。

执行该语句，必须具备yashan模式下的DBA相应权限。

#### SHOW CREATE VIEW view\_name

显示指定view_name视图的定义，其中，CHARACTER_SET_CLIENT和COLLATION_CONNECTION字段仅语法支持，无实际含义。

#### SHOW CREATE PROCEDURE proc\_name

显示指定view_name存储过程的定义，必须指定为当前用户可访问的对象。

#### SHOW GRANTS [FOR user\_name]

显示用户权限。
目前该语句与MySQL官方语法存在如下差异：

- YashanDB的mysql模式会将`show grants for '';`中的''视为标识符进行处理，执行会报错。

- YashanDB的mysql模式会将`show grants for nu11;`中的null视为用户名进行处理。

- 执行结果会分行打印用户的所有权限。

#### SHOW ENGINES engine\_name {STATUS |MUTEX}

显示指定engine_name存储引擎的信息。

- STATUS关键字用于查询状态信息。

- MUTEX关键字用于查询互斥、读写锁统计信息。

#### SHOW PLUGINS

仅用于兼容，查询结果恒为空集。

### SET命令

SET命令用于在线修改系统变量，修改将立即生效，但不会写入my.ini文件。

**set::=**

```ebnf+diagram
syntax::=SET [("@@GLOBAL.")|("@@SESSION.")] system_var_name "=" value
```

#### @@GLOBAL I @@SE5SION

指定当前修改生效的作用域：

- @@GLOBAL:表示全局生效。
- @@SESSION：表示仅当前会话生效。

该选项可省略，则默认为仅当前会话生效。

#### system\_var\_name

需要修改的系统变量的名称。

#### value

系统变量的值，必须符合格式要求和值域范围。

**示例**

```sql
SET @@GLOBAL.WAIT_TIMEOUT=28860;
```

### SET NAMES命令

SET NAMES命令用于设置客户端、连接与结果的字符集及连接的字符序，即设置以下系统变量：

- 字符集相关：CHARACTER_SET_CLIENT、CHARACTER_SET_CONNECTION和CHARACTER_SET_RESULTS
- 字符序相关：COLLATION_CONNECTION

**set_names::=**

```ebnf+diagram
syntax::=SET NAMES {"charset_name" DEFAULT | [COLLATE "collation_name"]}
```

#### charset\_name

字符集名称，可选项包括[ASCII|GB18030|GBK|LATIN1|UTF8|UTF8MB3|UTF8MB4]。

#### DEFAULT | COLLATE collation\_name

指定连接的字符集排序规则，可以指定为：

- DEFAULT:COLLATION_CONNECTION沿用服务端默认字符序（即COLLATION_SERVER的值）。
- 使用关键字COLLATE指定具体的字符序名称。

**示例**

```sql
SET NAMES gbk COLLATE gbk_bin;
```

### SQL语句

可执行的SQL语句及语法介绍请点击本文档中心左导航上方的![](./image/docc.png)，选择【崖山数据库】，查看YashanDB（mysql模式）的文档介绍。

