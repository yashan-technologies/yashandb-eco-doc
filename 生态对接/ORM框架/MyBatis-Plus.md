MyBatis-Plus是Mybatis的增强版，且比Mybatis更易使用。本文将以构建一个Maven项目为例，介绍如何实现YashanDB与MyBatis-Plus的对接，从而实现利用ORM框架进行基于YashanDB的Java应用开发。

## 对接前准备

在进行对接操作前，您需要先准备好如下事项（以下版本不做严格要求，可按照需要灵活选择）：

1. 已安装Jdk8或Jdk11的Java应用环境
2. 已安装Springboot 2.6
3. 已安装Maven 3.8
4. 已安装MyBatis-Plus 3.8
5. 已在[YashanDB官网下载中心](https://download.yashandb.com/download)下载YashanDB JDBC驱动包
6. 已存在一个可正常访问的YashanDB服务端。

## 对接配置

请参照如下步骤进行YashanDB与MyBatis-Plus的对接配置：

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
     (2, 'Jack'),
     (3, 'Tom'),
     (4, 'Sandy'),
     (5, 'Billie');
```

2. 创建User实体类，并使其与YashanDB的user1表对应：

```java
import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;

@Data
@TableName("user1")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
}
```

3. 定义mapper对应的java接口文件：

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.samples.quickstart.entity.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

@Mapper
public interface UserMapper extends BaseMapper<User> {
    List<User> selectUsers();
}
```

4. 创建单元测试类：

```java
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.samples.quickstart.entity.User;
import com.baomidou.mybatisplus.samples.quickstart.mapper.UserMapper;
import com.zaxxer.hikari.HikariDataSource;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;
import java.sql.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

@SpringBootTest
public class QuickStartTest {
    @Resource
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        userList.forEach(System.out::println);
    }

    @Test
    public void testSelect2(){
        User user = userMapper.selectById(1L);
        System.out.println(user);
    }
    
    @Test
    public void testInsertUser() {
        userMapper.insert(new User(1001L,"aaaa"));
        User user = userMapper.selectById(1001L);
        System.out.println(user);
    }

    @Test
    public void testSelect4(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> allEq = wrapper.allEq(new HashMap<String, Object>() {{
            put("id", 1);
            put("name", "Jone");
        }}, true);

        List<User> users = userMapper.selectList(allEq);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect5(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> eq = wrapper.eq("name", "Jone");
        List<User> users = userMapper.selectList(eq);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect6(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> ne = wrapper.ne("id", 1L);
        List<User> users = userMapper.selectList(ne);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect7(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> gt = wrapper.gt("id", 2L);
        List<User> users = userMapper.selectList(gt);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect8(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> ge = wrapper.ge("id", 2L);
        List<User> users = userMapper.selectList(ge);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect9(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> between = wrapper.between("id", 2L, 4L);
        List<User> users = userMapper.selectList(between);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect10(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> between = wrapper.notBetween("id", 3L, 4L);
        List<User> users = userMapper.selectList(between);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect11(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> like = wrapper.like("name", "J");
        List<User> users = userMapper.selectList(like);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect12(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> like = wrapper.notLike("name", "J");
        List<User> users = userMapper.selectList(like);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect13(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> like = wrapper.likeLeft("name", "J");
        List<User> users = userMapper.selectList(like);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect14(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> like = wrapper.likeRight("name", "J");
        List<User> users = userMapper.selectList(like);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect15(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> id = wrapper.isNotNull("id");
        List<User> users = userMapper.selectList(id);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect16(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> id = wrapper.isNull("id");
        List<User> users = userMapper.selectList(id);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect17(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> in = wrapper.in("id", new ArrayList<Long>(){{
            add(1L);
            add(2L);
        }});
        List<User> users = userMapper.selectList(in);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect18(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        QueryWrapper<User> inSql = wrapper.inSql("id", "select id from user1 where id < 4");
        List<User> users = userMapper.selectList(inSql);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect21(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        wrapper.func(i -> {
            if (false) {
                i.eq("id", 1);
            }else {
                i.eq("id", 2);
            }
        });

        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect22(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        wrapper.eq("id", 1).or().likeRight("name", "J");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelect23(){
        QueryWrapper<User>  wrapper = new QueryWrapper<>();
        wrapper.and(i -> i.eq("id", 1).likeRight("name", "J"));
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
    }
}
```

## 常见问题

#### 自增主键的使用

MyBatis-Plus要求Entity必须有主键，用@TableId注解来标识主键，其中id的type字段有五种取值（不同MyBatis-Plus版本略有差异，具体请参考MyBatis-Plus的官方文档，本文只作简单介绍）。

| 参数        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| AUTO        | 自动主键，在没给具体值的情况下使用数据库的自增               |
| NONE        | 无                                                           |
| INPUT       | 必须手动设置值，否则不能执行插入，更新等操作                 |
| ASSIGN_ID   | 不设置情况下默认为此类型，使用雪花算法自动生成long或者String类型的主键 |
| ASSIGN_UUID | 生成不带中划线的UUID                                         |

AUTO是最推荐的写法，比较简单灵活，只要表结构里设置了自增都可以用，而且使用数据库的自增不容易产生主键冲突。

```java
@Data
@TableName("user1")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
}
```

> **Note**:
>
> * 如需使用自增主键，不建议使用INPUT，然后自定义主键生成器，再从数据库中查自增序列给id赋值，这样的写法麻烦且效率低，而且还容易出错。
> * ASSIGN_ID和ASSIGN_UUID都是Java里自动生成主键，和数据库的自增主键无关。

#### 批量插入

MyBatis-Plus提供了ServiceImpl类，只要应用程序的Service继承ServiceImpl就可以使用里面预置的常用方法，例如saveBatch等。

UserService的定义：

```java
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.dao.UserMapper;
import com.example.pojo.User;

import org.springframework.stereotype.Service;

@Service
@EnableCaching(proxyTargetClass = true)
public class UserServiceImpl extends ServiceImpl<UserMapper,User> {

}

```

test写法：

```java
    @Autowired
    private UserService userService;
    
    @Test
    public void testAddUserList() {
        List<User> list = new ArrayList<>();
        User user1 = new User();
        user1.setName("aaa");
        list.add(user1);
        User user2 = new User();
        user2.setName("aaa1");
        list.add(user2);
        userService.saveBatch(list); // 不需要在UserServiceImpl里面定义saveBatch方法就可以使用
        List<User> list1 = userService.getUserList();
        list1.forEach(System.out::println);
    }
```
