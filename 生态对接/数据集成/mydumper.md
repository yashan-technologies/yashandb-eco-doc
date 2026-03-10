mydumper是一款针对MySQL数据库的高性能多线程备份工具，YashanDB（mysql模式）在语法上兼容MySQL数据库，并支持使用mydumper对YashanDB进行单表导出。本文将对此对接过程进行介绍。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装mydumper v0.16.9-1
2. 已存在一个可正常访问的YashanDB服务端，且服务端运行于mysql模式。

## 对接配置

使用mydumper命令可直接操作YashanDB，无需任何配置。

## 使用简介

以下列出一个简单的导出YashanDB的指定表的命令示例和解释：

```shell
$ mydumper -u sales -p sales -h 192.168.1.2 -P 1690 -T sales.products,sales1.area -t 3 -o /yashandb/backup
```

- -u：YashanDB中已创建并拥有合适权限的用户名称
- -p：YashanDB用户的密码
- -h：YashanDB服务端IP（多实例形态如YAC，选择其中一个IP）
- -T：要导出的表，必须为database.table的格式
- -t：并发导出线程数量
- -o：输出文件的目录路径

请执行`mydumper --help`命令查看完整的mydumper使用指导。