GeoServer是一款采用Java编写的，允许用户分享与编辑地理空间数据的开源软件，其支持使用开放标准对多数主要空间数据源进行发布。

YashanDB提供了gt-jdbc-yashandb方言包用于实现与GeoServer的对接，对应版本信息如下：

| GeoServer版本      | gt-jdbc-yashandb版本      |
|------------------|-------------------------|
| GeoServer 2.21.x | gt-jdbc-yashandb-21.0.1 |
| GeoServer 2.23.x | gt-jdbc-yashandb-23.3.1 |
| GeoServer 2.25.x | gt-jdbc-yashandb-25.5.1 |

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已从GeoServer官网下载GeoServer软件。
2. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载JDBC驱动包。
3. 已向我们的技术支持人员获取gt-jdbc-yashandb方言包。
4. 已存在一个可正常访问的YashanDB服务端。

## 配置步骤

依据GeoServer软件不同的安装方式，您可采取不同的操作步骤进行对接配置。

::: tabs

== exe安装

1. 运行exec程序安装并启动GeoServer
2. 进入GeoServer软件的安装目录

2. 将YashanDB JDBC驱动包和gt-jdbc-yashandb方言包复制到安装目录的lib目录下

3. 重启GeoServer

== 源码安装

以IDEA编辑器为例：

1. 在源码项目根路径下，创建libs目录

2. 将YashanDB JDBC驱动包和gt-jdbc-yashandb方言包复制到libs目录


3. 从IDEA菜单中，选择【FIle > Project Structure > Project Settings > Modules】
4. 在列出的Modules列表中，选择【gs-web-app】
5. 单击右边视图上的【+】，在弹出列表中选择【1 JARs or Directories】，之后选择libs目录导入

6. 从IDEA菜单中，选择【Run > Edit Configurations】
7. 在弹出的【Run/Debug Configuration】对话框中，点击+，按如下配置创建一个新的项目启动项：
   - Name: Start
   - Build and run: 依次为 java 11、-cp gs-web-app、org.geoserver.web.Start
   - Working directory: $MODULE_DIR$

4. 在窗口标题的运行小部件中选择【Start】，点击右边的运行图标，执行源码运行安装GeoServer。

:::