Entity Framework Core 6（EF Core 6）是微软在 .NET 6 平台上推出的轻量级、跨平台 ORM 框架。相比前代，它在性能、功能和开发体验上都有显著提升。本文将介绍如何实现YashanDB（yashan模式）与EF Core 6的对接，从而让用户通过.NET语言连接和操作YashanDB。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

- 已安装.NET应用环境，版本要求为.net 6.0+。
- 已安装Visual Studio工具或.NET CLI工具。
- 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载并安装YashanDB ADO.NET驱动。
- 已存在一个可正常访问的YashanDB服务端（yashan模式）。

## 配置EF Core6

请按如下步骤在您的.NET项目中配置实体框架：

1. 在YashanDB ADO.NET驱动包解压后的文件夹中，找到EFCore/YashanDBProvider.6.x.x.nupkg软件包，复制到你的一个本地路径下。
2. YashanDBProvider.6.x.x.nupkg软件包所在路径指定为nuget 源路径。
   ```bash
   dotnet nuget add source 【你的包存放路径】 -n 【源名称】
   ```
3. 安装YashanDBProvider软件包。
   进入项目目录下，执行：
   ```bash
   dotnet add package YashanDBProvider --source 【你的源名称】 -v 【6.x.x】
   ```
4. 验证是否安装成功（可选）。
   进入项目目录下，执行：
   ```bash
   dotnet list package
   ```
看到输出里包含“YashanDBProvider”就代表成功。

完整示例
   ```bash
   # 1. 添加本地源（假设包放在 D:\packages）
   C:\> dotnet nuget add source D:\packages -n yashan-nuget
   
   # 2. 进入项目目录
   C:\> cd C:\Projects\MyApp
   
   # 3. 安装包
   C:\Projects\MyApp> dotnet add package YashanDBProvider --source yashan-nuget -v 6.0.0
   
   # 4. 验证是否成功（可选）
   dotnet list package
   ```

## 连接YashanDB

完成EF Core6配置后，可使用如下方法在您的.NET应用项目中使用YashanDB的Provider：

在项目的DbContext实现类里面写如下配置代码：

```C#
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            var connectionString = "Addr=192.0.0.1;Port=1688;User=regress;Password=regress;";
            optionsBuilder.UseYashanDB(connectionString);
        }
```


之后，即可使用EF Core6对YashanDB执行各项SQL操作。

```c
public class Program
{
    static void Main(string[] args)
    {
        using ( DbContext ctx = new Context1() )
        {
            context.Database.EnsureDeleted();
            context.Database.EnsureCreated();

            var newStudent = new Student { Name = "张三", Age = 20 };
            context.Students.Add(newStudent);
            await context.SaveChangesAsync();
            Console.WriteLine($"已添加学生，ID: {newStudent.Id}");

            var students = await context.Students.ToListAsync();
            foreach (var s in students)
            {
                Console.WriteLine($"ID: {s.Id}, Name: {s.Name}, Age: {s.Age}");
            }

            var student = await context.Students.FirstOrDefaultAsync();
            if (student != null)
            {
                student.Age = 21;
                await context.SaveChangesAsync();
                Console.WriteLine("已更新年龄");
            }

            student = await context.Students.FirstOrDefaultAsync();
            if (student != null)
            {
                context.Students.Remove(student);
                await context.SaveChangesAsync();
                Console.WriteLine("已删除学生");
            }
            Console.WriteLine("完成\n");
        }
    }
    
    public class Student
    {
        public int Id { get; set; } 
        public string Name { get; set; }
        public int Age { get; set; }
    }
    public class Context1 : DbContext
    {
        public DbSet<Student> Students { get; set; }
        
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            var connectionString = "Addr=172.16.60.174;Port=1688;User=regress;Password=regress;";
            optionsBuilder.UseYashanDB(connectionString);
        }
    }
}
```

> **Note**:
>
> YashanDB对接EF6时不支持如下成员类型：
>
> - Json类型
> - 地理信息类型Geometry
> - 地理信息类型Geography
>
> 暂时不支持迁移相关能力。