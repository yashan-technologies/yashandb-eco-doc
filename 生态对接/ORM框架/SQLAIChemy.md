SQLAlchemy是一款开源的Python SQL工具包和对象关系映射器（ORM），yashandb_sqlalchemy是适配SQLAlchemy框架的YashanDB方言库，已通过SQLAlchemy社区用例集。本文将对介绍如何通过yashandb_sqlalchemy实现YashanDB与SQLAlchemy的对接。

## 对接准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装python3以上版本的Python应用环境

2. 已安装SQLAlchemy 1.4.x，安装命令如下：

```shell
pip3 install sqlalchemy==1.4.50
```

3. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载并安装YashanDB Python驱动包

4. 已向我们的技术支持人员获取yashandb_sqlalchemy方言包
5. 已存在一个可正常访问的YashanDB服务端

## 对接配置

请参照如下步骤进行YashanDB与Django的对接配置：

1. 使用以下命令安装django_yashandb方言包：

```shell
pip3 install --force-reinstall  yashandb_sqlalchemy-1.0.0-py3-none-any.whl
```

2. 使用python命令创建一个YashanDB数据库连接引擎（请修改your_host、your_port、your_name、your_password、your_dbname为实际值）：

```python
$ python3
>>> import yashandb_sqlalchemy
>>> import sqlalchemy as sa

>>> sa.create_engine('yashandb://your_name:your_password@your_host:your_port/your_dbname')
# 或
>>> sa.create_engine('yashandb+yasdb://your_name:your_password@your_host:your_port/your_dbname')
```

上述配置完成后，即可开始您的SQLAlchemy开发，连接和操作YashanDB服务端。