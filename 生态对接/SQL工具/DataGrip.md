DataGrip是一款强大的跨平台数据库管理工具，支持多种类型的数据库连接，在该工具中载入YashanDB提供的JDBC驱动包后，可以实现对YashanDB各部署形态，及各兼容模式形态的服务端连接和操作。本文将对此对接过程进行介绍。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装DataGrip工具
2. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包
3. 已存在一个可正常访问的YashanDB服务端。

## 对接配置

请参照如下步骤进行YashanDB与DataGrip的对接配置：

1. 从DataGrip的菜单中，选择【文件 > 新建 > 驱动程序】
2. 在弹出窗口中，选中【用户驱动程序】下的【User Driver】，可修改其名称，例如`YashanDB`
3. 点击【驱动程序文件】下的【+】，选择【自定义JAR...】，将YashanDB JDBC驱动包路径加载进来
4. 此时下拉【类：】右边的下拉框，将会出现YashanDB JDBC提供的类，选择`com.yashandb.jdbc.Driver`
5. 点击【应用】按钮保存新建的驱动信息
6. 点击【创建数据源】，进入数据源配置页面，输入如下信息：
   - 名称：可修改数据源名称，例如`YashanDB Data Source`
   - 用户：YashanDB中已创建并拥有合适权限的用户名称，请注意当连接的YashanDB处于mysql模式时，除sys外的用户名称前后需加上双引号
   - 密码：YashanDB用户的密码
   - URL：格式为'jdbc:yasdb://your_host:your_port/yasdb'，请将your_host和your_port替换为您的YashanDB服务端IP（多实例形态如YAC，选择其中一个IP），和该IP上的DB监听端口（默认安装情况下为1688）
7. 点击【确定】即完成DataGrip数据源配置。

## 使用简介

上述对接配置完成后，在DataGrip的【数据库资源管理器】区域中可以看到YashanDB数据源信息，开发者可在【console】区域输入SQL操作YashanDB。DataGrip的使用指导参见[DataGrip官方文档](https://www.jetbrains.com/help/datagrip)。

