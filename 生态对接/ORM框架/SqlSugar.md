SqlSugar是一款开源的关系映射器（ORM）框架，适用于.NET的对象，该框架封装了对数据库的各项操作，使开发者可以直接用.NET而非SQL语句来操作数据库。本文将介绍如何实现YashanDB与SqlSugar的对接适配，从而让用户通过.NET语言连接和操作YashanDB。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装.NET core 2.1应用环境。

2. 已安装Visual Studio工具。

3. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载并安装YashanDB ODBC驱动，其中是否配置数据源将影响应用项目的数据库连接操作：

   - 如您的应用项目还需进行ODBC开发，则必须配置数据源，后续应用开发中按DSN格式连接YashanDB
   - 如您的应用项目无需进行ODBC开发，则可以不配置数据源，后续应用开发中按Driver格式连接YashanDB

   请根据实际需要选择。

4. 已向我们的技术支持人员获取SqlSugar-yashan适配包。

5. 已存在一个可正常访问的YashanDB服务端。

## 对接配置

请按如下步骤在您的.NET项目中进行YashanDB与SqlSugar的对接配置：

1. 解压SqlSugar-yashan适配包，将会看到目录下存在SqlSugar.5.1.4.197.nupkg和SqlSugar.YasdbCore.1.0.0.nupkg两个文件。
2. 从Visual Studio菜单中，选择【工具 > NuGet包管理器 > 管理解决方案的NuGet包】。
3. 在弹出窗口中选择【程序包源】，点击【+】新建一个NuGet包源，为其命名，例如SqlSugar，路径请选择SqlSugar.5.1.4.197.nupkg路径。
4. 点击Visual Studio的【解决方案资源管理器】，选中您的应用项目，右键弹出菜单中选择【管理Nuget程序包】，在【已安装的包】列表中选择SqlSugar。
5. 点击【安装】，成功安装后即完成应用项目对SqlSugar.5.1.4.197.nupkg的引入。
6. 使用相同方式引入SqlSugar.YasdbCore.1.0.0.nupkg。


## 连接YashanDB

完成对接配置后，即可使用在SqlSugar框架中连接YashanDB执行应用开发，以下为一段简单的连接示例代码：

::: tabs

== DSN格式连接

当配置了ODBC数据源时，以DSN格式连接YashanDB（请将YashanDB_Default_DSN、your_name和your_password修改为实际值；请注意**DbType的值必须为DbType.Yasdb**）：

```c
public readonly static string Connection = "DSN=YashanDB_Default_DSN;Uid=your_name;Pwd=your_password";
var db = new SqlSugarClient(new ConnectionConfig()
	{
		IsAutoCloseConnection = true,
		DbType = DbType.Yasdb,  
		ConnectionString = Connection,
		LanguageType = LanguageType.Default,
		MoreSetting = new ConnMoreSettings()
		{
		}
},
it => {
	it.Aop.OnLogExecuting = (sql, para) =>
	{
		Console.WriteLine(UtilMethods.GetNativeSql(sql, para));
	};
});
```

== Driver格式连接

当未配置ODBC数据源时，以Driver格式连接YashanDB（请将your_host、your_port、your_name和your_password修改为实际值；请注意**DbType的值必须为DbType.Yasdb**）：

```c
//Connection的另外一种写法为："Driver=YashanDB;Server=your_host;Port=your_port;Uid=your_name;Pwd=your_password"
public readonly static string Connection = "Driver=YashanDB;Url=your_host:your_port;Uid=your_name;Pwd=your_password";
var db = new SqlSugarClient(new ConnectionConfig()
	{
		IsAutoCloseConnection = true,
		DbType = DbType.Yasdb,  
		ConnectionString = Connection,
		LanguageType = LanguageType.Default,
		MoreSetting = new ConnMoreSettings()
		{
		}
},
it => {
	it.Aop.OnLogExecuting = (sql, para) =>
	{
		Console.WriteLine(UtilMethods.GetNativeSql(sql, para));
	};
});
```

:::

> **Note**:
>
> 1. YashanDB对接SqlSugar时不支持如下成员类型：
>
>    - 时间间隔类型TimeSpan
>
>    - 带时区的时间类型DateTimeOffset
>
>    - 地理信息类型Geometry
>
>    - 地理信息类型Geography
>
> 2. YashanDB对接SqlSugar时不支持如下功能接口：
>
>    - 大数据写入接口BulkCopy
>    - 大数据写入接口BulkCopyAsync
