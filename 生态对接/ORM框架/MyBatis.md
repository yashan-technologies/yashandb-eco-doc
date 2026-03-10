MyBatis是一款基于Java的持久层框架，它支持自定义SQL、存储过程以及高级映射，使得开发者可以采用面向对象的方式操作数据库，且无需编写JDBC代码。本文将以构建一个Maven项目为例，介绍如何实现YashanDB与MyBatis的对接，从而实现利用ORM框架进行基于YashanDB的Java应用开发。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项（以下版本不做严格要求，可按照需要灵活选择）：

1. 已安装Jdk8或Jdk11的Java应用环境
2. 已安装Springboot 2.6
3. 已安装Maven 3.8
4. 已安装Mybatis 3.2
5. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包
6. 已存在一个可正常访问的YashanDB服务端。

## 对接配置

请参照如下步骤进行YashanDB与MyBatis的对接配置：

1. 检查Maven核心配置文件pom.xml中是否指定了如下依赖项，没有则加上：

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

<!--        Mybatis-plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.2</version>
        </dependency>

    <!--        test-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- lombok  -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    </dependencies>
```

2. 以IDEA编辑器为例，从IDEA的菜单中，选择【 File > Project Structure > Libraries】
3. 点击【+】，并选择【Java】，从本地选择已准备的YashanDB JDBC驱动包完成库添加；如为多模块项目，只需要选择相应的模块执行本操作。
4. 本文采用Springboot最常用的方式导入数据源，即通过application.yml导入，并以Springboot自带的Hikari连接池为例进行数据源配置。

::: tabs

== 单数据源配置

请将下例中的your_host、your_port、your_username和your_password修改为实际值。

```yaml
spring:
  datasource:
    url: jdbc:yasdb://your_host:your_port/yasdb
    driver-class-name: com.yashandb.jdbc.Driver
    username: your_username
    password: your_password
```

== 多数据源配置

多数据源配置采用dynamic-datasource-spring-boot-starter，本文以YashanDB与MySQL两个数据源（可自行选择其他数据源）为例，进行两个数据源的切换。请将下例中的your_host、your_port、 dbname、your_username和your_password修改为实际值。

```yaml
spring:
  datasource:
    dynamic:
      primary: master
      strict: false
      datasource:
        master:
          url: jdbc:yasdb://your_host:your_port/yasdb
          driver-class-name: com.yashandb.jdbc.Driver
          username: your_username
          password: your_password
        slave1:
          url: jdbc:mysql://your_host:your_port/dbname
          driver-class-name: com.mysql.cj.jdbc.Driver
          username: your_username
          password: your_password
```

:::

## 简单使用示例

上述对接配置完成后，开发者即可开始定义Java对象与YashanDB对象进行映射，以下为一个简单的实现示例：

1. 在YashanDB中创建如下表对象：

```sql
-- 建表语句
CREATE TABLE user1 
(
	id INT PRIMARY KEY,
	name VARCHAR(30) NULL
);
-- 预置5条数据
INSERT INTO USER1 (id, name) VALUES
(1, 'Jone'),
(2, 'Jack',),
(3, 'Tom',),
(4, 'Sandy'),
(5, 'Billie');
```

2. 创建User实体类，并使其与YashanDB的user1表对应：

```java
import lombok.Data;

@Data
public class User {
    private Long id;
    private String name;
}
```

3. 创建User的Mapper文件：

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.dao.UserMapper">
    <select id="getAllUsers" resultType="com.example.pojo.User">
        select * from user1
    </select>
    <insert id="insertUser" parameterType="com.example.pojo.User">
        insert into user1(id,name) values(#{id},#{name})
    </insert>

</mapper>

```

4. 定义mapper对应的java接口文件：

```java
import java.util.List;

@Mapper
public interface UserMapper {
    List<User> getAllUsers();
    void insertUser(User user);
}
```

5. 创建单元测试类

```java
public class UserTest {

    @Autowired
    private UserMapper userMapper;
    
    @Test
    public void testAddUser() {
        assertEquals(5, userMapper.getAllUsers().size());// 建表时预置了5条数据
        userMapper.insertUser(new User(20,"zhang"));
        List<User> list = userMapper.getAllUsers();
        assertEquals(6, list.size());
    }
}
```

## 常见问题

#### 自增主键的使用问题

一般情况下YashanDB里自增主键都是通过序列实现的，而MyBatis使用自增主键主要分为以下情形：

* 情形一：插入语句中不带id，让服务端通过序列自动生成id。
* 情形二：插入语句中不带id，让服务端通过序列自动生成id，并且在语句执行结束后返回所生成的id。

情形一中无需做任何配置，只要数据库里定义表结构时指定自增序列，以及mapper.xml在定义SQL语句时不带主键直接插入即可。

```xml
    <insert id="insertUserWithoutId" parameterType="com.example.pojo.User">
        -- 插入时不指定id的值
        insert into user1(name) values(#{name})  
    </insert>
```

情形二则需要mapper.xml定义时加上如下参数：

| 参数             | 默认值 | 含义                                                         |
| ---------------- | ------ | ------------------------------------------------------------ |
| useGeneratedKeys | false  | 是否返回主键，需要返回主键时应设置为true                     |
| keyColumn        | 无     | 主键列的列名（表结构里定义的列名）                           |
| keyProperty      | 无     | 主键所对应的字段名（Entity里定义的字段名，如果与表结构里定义的相同，可省略） |

mapper.xml示例

```xml
    <insert id="insertUserReturnKey" parameterType="com.example.pojo.User" useGeneratedKeys="true" keyColumn="id" keyProperty="id">
        insert into user1(name) values(#{name})  -- 插入时不指定id的值
    </insert>
```

效果如下：

```java
    @Test
    public void testAddUser() {
        User user = new User("zhang"); // 创建user时不指定id的值
        userMapper.insertUserReturnKey(user);// 插入
        assertNotEquals(0,user.getId()); // 执行完插入后，数据库里自动生成的id值会自动回填到user对象中。
    }
```

需注意的是，YashanDB仅支持获取单行插入语句的返回值，在多行插入时，若使用了useGeneratedKeys等参数来返回主键值将会发生报错。解决办法为：

- 去掉useGeneratedKeys等参数。
- 采取别的办法实现多行插入并获取返回值，例如把多行插入改造成单行插入然后在Java代码里循环处理。

#### CLOB序列化成JSON时报错

实际应用中经常需要把从数据库查询出来的结果集直接返回到前端页面，在这个过程中Spring会自动把Entity对象序列化成JSON串。但由于CLOB和BLOB大对象数据不可被序列化，所以在Entity对象中含有CLOB和BLOB类型的字段时应用会报错，错误信息大致如下：

```java
Type definition error: [simple type, class com.yashandb.jdbc.YasLobInputStream]; nested exception is com.fasterxml.jackson.databind.exc.InvalidDefinitionException:No serializer found for class com.yashandb.jdbc.YasLobInputStream and no properties discovered to create BeanSerializer(to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS)(through reference chain: com.lead.common.core.domain.AjaxResult["data"]->java.util.ArrayList[0]->com.lead.common.core.domain.vo.ComboBoxVo["value"]->com.alibaba.druid.proxy.jdbc.ClobProxyImpl["asciiStream"])
```

解决办法为，在定义实体类时，将数据库中的CLOB类型字段直接定义成String，而将数据库中的BLOB类型字段直接定义成byte[]，如下所示：

```java
import lombok.Data;

@Data
public class User {
    private Long id;
    private String name;
    private String clobVal;
    private byte[] blobVal;
}
```

这种方法既避免了因为无法序列化而报错的问题，又省去了因为处理LOB类型带来的额外的处理逻辑。

另外，对于返回值为CLOB类型的数据库函数（例如GROUP_CONCAT），在使用时也应用String去承接其返回值。

#### useColumnLabel配置项不生效

在MyBatis中，配置项seColumnLabel默认为true，即使用ColumnLabel实现字段映射，如果配置为false则表示使用ColumnName映射。

而在YashanDB JDBC中，依据JDBC标准，在存在别名的情况下，resultsetmetaData的getColumnLabel和getColumnName接口返回的都是别名，无论useColumnLabel设置为true或false。

例如执行select id userId,name userName from table1;时，不论useColumnLabel设置如何，YashanDB JDBC永远使用userId和userName作为字段名去映射。

#### SQL语句结束符

正常情况下，MyBatis环境中SQL语句无需结束符。

在JDBC参数allowMultiStmt为true（即开启支持多SQL）的场景下，MyBatis环境中SQL语句可以采用分号（`;`）作为结束符。
