mysqldump是MySQL数据库管理系统内置的备份和迁移工具，YashanDB（mysql模式）在语法上兼容MySQL数据库，并支持使用mysqldump对YashanDB进行整库导出。本文将对此对接过程进行介绍。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装mysqldump 10.13
2. 已存在一个可正常访问的YashanDB服务端，且服务端运行于mysql模式。

## 对接配置

使用mysqldump命令可直接操作YashanDB，无需任何配置。

## 使用简介

使用mysqldump进行YashanDB指定数据库导出的操作命令格式如下（[]表示可选）：

```shell
mysqldump 连接选项 [options] --databases dbname [表选项] [options] > backup.sql
```

### 连接选项

运行mysqldump命令需要同时指定如下选项连接YashanDB：

- -u：指定数据库用户名
- -p：指定数据库用户密码，可直接在命令行显式输入参数（需注意信息安全），或者不输入参数采取交互式输入密码；请注意采取显式输入方式时，-p后应直接输入参数，不可有空格
- -h：指定数据库的IP地址，使用本机地址时可省略此选项
- -P：指定数据库的MySQL监听端口，默认安装的YashanDB产品对应MySQL监听端口为1690

### --databases

运行mysqldump命令需要同时指定database的名称，用于指定导出的数据源，请保证上述连接用户对该database的权限。

### 表选项：--tables

本选项用于指定待导出的表名称，可同时指定多张表导出。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --no-tablespaces=true --databases sales --tables area branches > backup.sql
```

### 表选项：--ignore-table

本选项用于指定导出时排除指定的表或视图。

必须使用数据库名称.对象名称的格式来指定要排除的对象，需排除多个时，请多次使用本选项。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --no-tablespaces=true --databases sales --ignore-table sales.area --ignore-table sales.branches> backup.sql
```

### --where

本选项用于指定WHERE条件。

指定的WHERE条件将被应用到所有要导出的表上，只有满足条件的记录行才会被导出，但所有的表结构仍会被导出。本选项使用于增量导出和条件导出场景。

如果指定的条件包含空格或其他命令解释器特有的字符，则必须用双引号包围，例如`--where="user='jimf'"`。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --no-tablespaces=true --databases sales --tables area branches --where="area_no='01'" > backup.sql
```

### --no-tablespaces

本选项用于指定是否排除日志文件组（CREATE LOGFILE GROUP）和表空间创建语句（CREATE TABLESPACE）的输出。

选项参数值包括TRUE和FALSE，默认值为FALSE。

由于YashanDB当前并不支持导出日志文件组和表空间创建语句，因此应恒指定为TRUE。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --no-tablespaces=true --databases sales > backup.sql
```

### --set-gtid-purged

本选项用于控制是否在导出的SQL文件中包含GTID（全局事务标识符）清除设置。

选项参数值包括AUTO、ON、OFF，默认值为AUTO。

由于YashanDB无GTID，因此该值应被设为OFF。设为AUTO和ON不会生效，仍按OFF执行；设为ON时会有错误消息提示，但不影响执行。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --set-gtid-purged=OFF --no-tablespaces=true --databases sales > backup.sql
```

### --skip-opt

本选项用于禁用默认启用的选项，即关闭--opt选项。

默认启用的选项包括--add-drop-table、--add-locks、--create-options、--disable-keys、--extended-insert、--lock-tables、--quick和--set-charset，命令中指定本选项则会禁用这些选项。

在将mysqldump用于如下场景时不建议指定本选项：

1. 大数据量导出，直接禁用上述选项而不进行任何优化，可能导致速度很慢。
2. 需要保证数据一致性的情况。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --skip-opt --no-tablespaces=true --databases sales > backup.sql
```

### --single-transaction

本选项用于控制是否以数据一致性状态导出。

选项参数值包括TRUE和FALSE，默认值为FALSE。

指定为TRUE时，mysqldump将在单个事务中执行转储操作，确保获得一致的数据快照，具体动作如下：

1. 在转储开始时启动一个新事务。
2. 设置事务隔离级别为REPEATABLE READ。
3. 获得数据的一致性快照。
4. 在整个转储过程中保持该事务打开状态。
5. 转储完成后提交事务。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --single-transaction=true --no-tablespaces=true --databases sales > backup.sql
```

### --no-create-db

本选项用于控制是否导出CREATE DATABASE语句。

选项参数值包括TRUE和FALSE，默认值为FALSE，即命令会导出CREATE DATABASE语句。

禁止导出CREATE DATABASE语句不会影响到表结构和数据的备份，适用于已知目标数据库已存在的情况。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --no-create-db --no-tablespaces=true --databases sales > backup.sql
```

### --default-character-set

本选项用于指定导出时使用的字符集，确保正确处理数据库中的多字节字符和特殊字符。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --default-character-set=utf8mb4  --no-tablespaces=true --databases sales > backup.sql
```

### --extended-insert

本选项用于控制INSERT文件的生成方式。

选项参数值包括TRUE和FALSE，默认值为TRUE，即将多行数据合并到单个INSERT语句中，加速后续导入流程。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --extended-insert --no-tablespaces=true --databases sales > backup.sql
```

### --insert-ignore

本选项用于指定在导出的SQL文件中为所有的INSERT语句添加IGNORE关键字。

使用本选项后，在执行导入时可以在遇到重复键值时忽略错误继续执行，也不会因为主键或唯一键冲突而中断导入过程。

需注意的是，YashanDB并不支持INSERT IGNORE INTO语法，因此如导出的文件要用于YashanDB的导入，不可以指定本选项。

### --no-create-info

本选项用于控制是否导出CREATE TABLE语句。

选项参数值包括TRUE和FALSE，默认值为FALSE，即导出CREATE TABLE语句。

值为TRUE表示禁止导出CREATE TABLE语句，但仍会导出INSERT语句，适用于已知目标数据库表结构已存在的情况。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --no-create-info --no-tablespaces=true --databases sales > backup.sql
```

### --no-data

本选项用于控制是否只导出CREATE TABLE语句，而不导出数据。

选项参数值包括TRUE和FALSE，默认值为FALSE，即同时导出CREATE TABLE语句和数据。

值为TRUE表示只导出表结构，例如，通过加载转储文件创建表的空副本时可以设置该选项为TRUE，可生成体量小的文件，加快导出速度。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --no-data --no-tablespaces=true --databases sales > backup.sql
```

### --replace

本选项用于指定在生成的SQL文件中将所有的INSERT语句替换为REPLACE语句。

使用REPLACE INTO语句执行的导入可以在遇到重复键值时替换旧纪录，而不是报错，适用于需要强制更新数据的场景。

需注意的是，YashanDB并不支持REPLACE  INTO语法，因此如导出的文件要用于YashanDB的导入，不可以指定本选项。

示例

```shell
$ mysqldump -h192.168.1.2 -P1690 -usales -p******** --replace --no-tablespaces=true --databases sales > backup.sql
```

### --socket

本选项用于指定服务器的Unix套接字文件路径，套接字文件可用于UDS本地连接数据库。

指定本选项后，不需要再指定-h和-P选项，适用于mysqldump与数据库位于同一台服务器的场景。

需注意的是，YashanBD配置套接字文件需要在$YASDB_DATA/config/service.ini中配置IPC属性。

示例

```shell
# 配置service.ini
$ vi /data/yashan/yasdb_data/db-1-1/config/service.ini

SERVICE1 = {library = yas_my, name = /,
args = "URL=0.0.0.0:1690,IPC=/data/yashan/exp_imp//mysql.ipc,
	RSA_PRIVATE_FILE=/data/yashan/exp_imp/private_key.pem,
	RSA_PUBLIC_FILE=/data/yashan/exp_imp/public_key.pem"}

# 执行导出
$ mysqldump -usales -p******** --socket=/data/yashan/exp_imp//mysql.ipc --no-tablespaces=true --databases sales > backup.sql
```
