Django是一个免费开源的Python Web开发框架，内部包含了数据库ORM组件。django_yashandb是适配Django ORM框架的YashanDB方言库，且已通过Django社区用例集。本文将对介绍如何通过django_yashandb实现YashanDB与Django的对接。

## 对接准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装python3.8以上版本的Python应用环境

2. 已安装Django 3.2.x，安装命令如下：

```shell
pip3 install django==3.2.25
```

3. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载并安装YashanDB Python驱动包

4. 已向我们的技术支持人员获取django_yashandb方言包
5. 已存在一个可正常访问的YashanDB服务端

## 对接配置

请参照如下步骤进行YashanDB与Django的对接配置：

1. 使用以下命令安装django_yashandb方言包：

```shell
pip3 install --force-reinstall django_yashandb-1.0.1-py3-none-any.whl
```

2. 编辑Django框架的配置文件，即您的Python应用项目下的settings.py文件，增加或修改如下配置项（请修改your_host、your_port、your_name、your_password为实际值）：

```python
    DATABASES = {
       "default": {
           "ENGINE": "django_yashandb",
           "HOST": "your_host",
           "PORT": "your_port",
           "USER": "your_name",
           "PASSWORD": "your_password",
       },
    }
```

上述配置完成后，即可开始您的Django开发，连接和操作YashanDB服务端，但存在如下注意事项：

- 请勿手动为AutoField、BigAutoField插入与当前数据库中该字段最大值跨度过大的值，这可能会导致框架长时间阻塞。
- 以绑定参数查询数据库时，请显式指定output_field参数的值，以便YashanDB按指定的数据类型进行结果返回。

