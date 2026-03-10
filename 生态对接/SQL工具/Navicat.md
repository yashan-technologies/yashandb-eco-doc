Navicat是一款强大的跨平台数据库管理工具，支持多种类型的数据库连接，选择该工具的MySQL服务模式进行配置，即可实现对YashanDB（mysql模式）服务端的连接和操作。本文将对此对接过程进行介绍。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装Navicat工具
2. 已存在一个可正常访问的YashanDB服务端，且该服务端运行于mysql模式。

## 对接配置

请参照如下步骤进行YashanDB与Navicat的对接配置：

1. 从Navicat的菜单中，选择【文件 > 新建连接】
2. 在弹出窗口中，连接类型选择MySQL
3. 点击【下一步】按钮，进入连接配置页面，输入如下信息：
   - 连接名称：自定义一个连接名称，例如`YashanDB`
   - 主机：输入您的YashanDB服务端IP，多实例形态如YAC，选择其中一个IP输入
   - 端口：输入您的YashanDB服务端的MySQL监听端口，默认安装情况下该端口号为1690
   - 用户名：YashanDB中已创建并拥有合适权限的用户名称，请注意不可输入sys用户
   - 密码：YashanDB用户的密码

4. 点击【测试连接】，出现连接成功信息表示YashanDB与Navicat对接成功。

## 使用简介

上述对接配置完成后，在Navicat的【我的连接】区域中可以看到新建的连接名称，开发者可开始使用Navicat功能操作YashanDB。Navicat的使用指导参见[Navicat官方文档](https://www.navicat.com/en/support/online-manual)。
