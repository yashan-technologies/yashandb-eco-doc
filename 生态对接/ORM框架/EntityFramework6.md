Entity Framework 6（EF6）是由微软公司提供的一款关系映射器（ORM），适用于 .NET 的对象，该框架封装了对数据库的各项操作，使开发者可以直接用.NET而非SQL语句来操作数据库。本文将介绍如何实现YashanDB与EF6的对接，从而让用户通过.NET语言连接和操作YashanDB。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装.NET应用环境，版本要求为.net framework 4.5、4.5.1、4.5.2、以及6.0中的某一项。
2. 已安装Visual Studio工具或.NET CLI工具。
3. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载并安装YashanDB ADO.NET驱动。
4. 已存在一个可正常访问的YashanDB服务端。

## 配置EF6

请按如下步骤在您的.NET项目中配置实体框架：

1. 在YashanDB ADO.NET驱动包解压后的文件夹中，找到Yashandb.Data.EntityFramework文件夹所在路径。
2. 在应用程序项目中引入Yashandb.Data.EntityFramework项目。

::: tabs

== Visual Studio

1. 在Visual Studio中新建或者打开一个.NET应用程序项目

2. 从Visual Studio菜单中，选择【文件 > 添加 > 现有项目】

3. 打开Yashandb.Data.EntityFramework文件夹所在路径，选择Yashandb.Data.EntityFramework.csproj文件，将Yashandb.Data.EntityFramework加入到应用所在解决方案内。

4. 右键单击应用程序项目，依次选择【添加 > 项目引用 > 项目】，勾选Yashandb.Data.EntityFramework选项。


== .NET CLI

1. 进入到应用程序项目所在文件夹

1. 使用dotnet命令指定Yashandb.Data.EntityFramework文件夹所在路径，将Yashandb.Data.EntityFramework项目添加到该应用程序项目内。

    ```bash
    dotnet add reference F:\YashanDB.ADO.NET\Yashandb.Data.EntityFramework\Yashandb.Data.EntityFramework.csproj
    ```

:::


## 连接YashanDB

完成EF6配置后，可使用如下两种方法在您的.NET应用项目中注册YashanDB的Provider：

- 方法一：在App.config中添加配置

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <entityFramework>
        <defaultConnectionFactory
                type="Yashandb.Data.EntityFramework.YasdbConnectionFactory, Yashandb.Data.EntityFramework"/>
        <providers>
            <provider invariantName="Yashandb.Data.EntityFramework"
                type="Yashandb.Data.EntityFramework.YasdbProviderServices, Yashandb.Data.EntityFramework"/>
        </providers>
    </entityFramework>
</configuration>
```

- 方法二：在定义DbContext时使用属性注入DbConfigurationType(typeof(YasdbEFConfiguration))

```c
[DbConfigurationType(typeof(YasdbEFConfiguration))]
public class DefaultContext : DbContext
{
    public DefaultContext (string connStr) : base(connStr)
    {
        Database.SetInitializer<DefaultContext>(null);
    }
    public DbSet<Comp> Comps { get; set; }
}
```

之后，即可使用EF6对YashanDB执行各项SQL操作。

```c
public class Program
{
    static void Main(string[] args)
    {
        using ( DefaultContext ctx = new DefaultContext("name=YasdbContext") )
        {
            ctx.Database.Delete();
            ctx.Database.CreateIfNotExists();
            Comp c = new Comp { Name = "aa", DateBegan = DateTime.Now, NumEmployees = 10 };
            ctx.Comps.Add(c);
            ctx.SaveChanges();
            foreach ( Comp val in ctx.Comps )
            {
                Console.WriteLine(val.Name);
            }
        }
    }
    
    public class Comp
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }
        public string Name { get; set; }
        public DateTime DateBegan { get; set; }
        public int NumEmployees { get; set; }
    }

    [DbConfigurationType(typeof(YasdbEFConfiguration))]
    public class DefaultContext : DbContext
    {
        public DefaultContext (string connStr) : base(connStr)
        {
            Database.SetInitializer<DefaultContext>(null);
        }

        public DbSet<Comp> Comps { get; set; }
    }
}
```

> **Note**:
>
> YashanDB对接EF6时不支持如下成员类型：
>
> - 时间间隔类型TimeSpan
> - 带时区的时间类型DateTimeOffset
> - 地理信息类型Geometry
> - 地理信息类型Geography
