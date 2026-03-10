Hibernate是一个开源的对象映射关系（ORM）框架，适配Java程序，该框架使得开发者可以采用面向对象的方式操作数据库，且无需编写JDBC代码。本文将以构建一个Maven项目为例，介绍如何实现YashanDB与Hibernate的对接，从而实现利用Hibernate框架进行基于YashanDB的Java应用开发。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项：

1. 已安装Jdk8的Java应用环境
2. 已安装Springboot 1.x
3. 已安装Maven 3.8
4. 已安装Hibernate-core-5.x
5. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包
6. 已向我们的技术支持人员获取YashanDialect-for-hibernate5方言包
7. 已存在一个可正常访问的YashanDB服务端。

## 对接配置

请参照如下步骤进行YashanDB与Hibernate的对接配置：

1. 检查Maven核心配置文件pom.xml中是否指定了如下依赖项，没有则加上：

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    
        <!-- spring jpa，里面包含了hibernate核心类  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
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
4. 用上述相同的方式添加YashanDialect-for-hibernate5方言包。
5. 编辑application.properties文件进行参数配置（请将下例中的host_ip、port、username和password修改为实际值）：

```properties
spring.datasource.url=jdbc:yasdb://host_ip:port/yasdb
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=com.yashandb.jdbc.Driver

# 方言必须设置为崖山方言YasDialect
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.YasDialect

# 是否自动建表
spring.jpa.properties.hibernate.hbm2ddl.auto=create

# 字段名和数据库中对象名的映射关系:驼峰转下划线
spring.jpa.properties.hibernate.naming-strategy = org.hibernate.cfg.ImprovedNamingStrategy
```

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

2. 创建User实体类，并使其与YashanDB的user1表对应。

```java
import lombok.Data;
import org.hibernate.annotations.ColumnDefault;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.SequenceGenerator;
import javax.persistence.Table;
import java.time.LocalTime;

@Data
@Entity
@Table(name = "user1")
public class User {

    @Id // hibernate要求实体类中必须定义一个id属性，否则会报错。该属性被作为业务表的自增主键列
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "ID_GENERATOR")
    @SequenceGenerator(name = "ID_GENERATOR", sequenceName = "user_sequence") //user_sequence为序列名，可自定义命名
    @ColumnDefault("user_sequence.nextval") //序列名称需与@SequenceGenerator中的sequenceName一致
    private int id;
    
    @Column(name = "name")
    private String name;
    
    public User() {
    }

    public User(String name) {
        this.name = name;
    }
    
    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

3. 实现Repository接口，可自由选择继承JpaRepository，或者继承CrudRepository。

```java
import com.example.pojo.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User,Integer> {
// 注意JpaRepository<User,Integer>这里传入的泛型类型，第一个是上面定义的Entity，第二个是Entity的主键的类型
// JpaRepository中可以不实现任何接口，其默认接口已足够使用。
}
```

4. 创建单元测试类。

```java
package com.example;

import com.example.dao.UserRepository;
import com.example.pojo.User;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.junit.Assert.assertEquals;

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserTest {
    @Autowired
    private UserRepository repository;
    @Before
    public void deleteAllUser(){
        repository.deleteAll();
    }

    @Test
    public void testAddUser() {
        User user = new User("sasa");
        repository.save(user);
        List<User> list = repository.findAll();
        assertEquals(1, list.size());
    }
}
```

## 常见问题

### 序列器名称不一致导致启动时建表失败

在定义实体类的Id属性时，@SequenceGenerator和@ColumnDefault中均指定了序列器名称，但Hibernate存在如下机制：

1. @SequenceGenerator中的sequenceName指定的序列名是驼峰格式时，Hibernate会将其转为下划线格式。
2. @ColumnDefault注解的value是驼峰格式时，Hibernate不做处理。

因此，如开发者使用驼峰格式命名一个序列器时，上述机制会导致两个注解使用了不一致的序列器名称。

解决办法为，对序列器统一采用下划线风格命名；或者不指定@ColumnDefault和@SequenceGenerator，此时Hibernate使用默认序列hibernate_sequence进行Id自增，但同时需修改@GeneratedValue为如下value：

```java
@GeneratedValue(strategy= GenerationType.AUTO)
```

### 数据类型校验失败导致服务启动失败

在Hibernate的hbm2ddl.auto配置项被定义为validate的情况下（如下），启动服务时会进行数据库表结构数据类型与Java实体类数据类型的一致性校验；不一致时，会由于Schema-validation校验失败，从而导致容器初始化Hibernate SessionFactory时失败。

```properties
spring.jpa.properties.hibernate.hbm2ddl.auto=validate
```

出现这个问题时，解决方案如下：

- 方案一：需要修改数据库表结构或Java实体类数据类型，保持数据库表结构与Java实体类数据类型一致。

- 方案二：将hbm2ddl.auto配置设置为none，服务启动时不进行校验。

### YashanDB中不存在的数据类型报错

通常不同品牌的数据库在数据类型定义上存在差异，例如MySQL中存在TEXT数据类型而YashanDB无此类型。因此，当复制了其他数据库类型定义到YashanDB的映射中时，可能会因为YashanDB中不存在该数据类型而报错。

解决办法为，在YashanDB中找一个虽定义不同但规格相近的数据类型进行替代。

例如，对于MySQL中的TEXT类型，YashanDB中存在CLOB类型同样可用于保存大量字符数据，因此可使用CLOB类型代替MySQL中的TEXT数据类型，如下：

```java
//将columnDefinition = "TEXT"改为columnDefinition = "CLOB"
@Column(name = "file_types", columnDefinition = "CLOB")
private String fileTypes;
```

### 特殊字符乱码

在服务端的字符集被指定为GBK时，YashanDB通常会使用NCHAR、NVARCHAR或NCLOB类型来存储那些解析时可能出现乱码的特殊字符文本值，但在通过Hibernate插入数据时，由于绑定参数的String类型默认为setString，导致无法避免乱码现象。

此类场景下，可以按需选择以下解决方案：

- 方案一：在对应字段上加@Nationalized注解。

- 方案二：在对应字段上加@Type(type = "org.hibernate.type.StringNVarcharType")注解。