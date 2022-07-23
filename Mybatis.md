#  Mybatis

官方文档：

[mybatis – MyBatis 3 | 简介](https://mybatis.org/mybatis-3/zh/index.html)

# 1 、简介



## 1.1 什么是mybatis

- MyBatis 是一款优秀的持久层框架; 它支持自定义 SQL、存储过程以及高级映射。

- MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。
- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。



## 1.2 持久化

数据持久化：一种动作

- 持久化就是将程序的数据在持久状态和瞬时状态转化的过程
  - 把数据（内存中的对象）保存到可永久保存的存储设备中
  - 主要应用就是把内存中的对象存储到数据库中，或者磁盘文件，或者xml数据文件中。

- 内存：**断电即失**
- 数据库（Jdbc）,io文件持久化。都是持久化机制

**为什么要持久化？**

- 有一些对象，不能让他丢掉
- 内存太贵，对象不能长期的存在内存中

## 1.3 持久层

**Dao层**、Service层、Controller层

- 完成持久化工作的代码块 Dao层 （data access object 数据访问对象）
- 持久化的实现过程大多通过各种关系数据库来完成
- 我们的系统架构中，需要一个相对独立的逻辑层面，专注于数据持久化的逻辑实现
- 层界限十分明显 说白了就是为了操作数据库存在的

## 1.4 为什么需要mybatis

- 帮助程序员将数据存入到数据库中


- 方便


- 传统的JDBC代码太复杂了，简化，框架，自动化


- 不用MyBatis也可以，技术没有高低之分


- 优点：
  - 简单易学
  - 灵活
  - sql和代码的分离，提高了可维护性。
  - 提供映射标签，支持对象与数据库的orm字段关系映射
  - 提供对象关系映射标签，支持对象关系组建维护
  - 提供xml标签，支持编写动态sql

  
  
  
  
  

## 1.5 mybatis原理

![image-20210929092556316](Mybatis.assets/image-20210929092556316.png)

# 2 第一个mybatis程序

思路： 搭建环境    导入mybatis   编写代码  测试



## 2.1 代码演示

1 搭建数据库

```mysql
// 创建数据库
create database `mybatis`; // 还是尽量用大写的 这里我为了写的更简洁的全部用了小写

use `mybatis`;
// 创建表加入数据
create table `user`(
`id` int(20) not null,
`name` varchar(30) default null,
`pwd` varchar(30) default null,
primary key (`id`)
)engine = innodb default character set=utf8;

insert into `user`(`id`, `name`, `pwd`) values
(1,'张三','123456'),
(2,'李四','123456'),
(3,'王五','123456');
```



2 新建项目

- 新建一个maven项目

记得更改maven的配置setting.xml 和repo的位置 

删除src 作为一个父项目 生成子项目不用每一次都导包了

- 导入maven依赖

```xml
<!-- 导入依赖-->
<dependencies>
<!--mysql驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
<!--mybatis-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>
<!--junit-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 2.2 创建一个模块

- 编写mybatis核心配置文件
  - 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 **XML 配置文件**或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。

  - 从 XML 文件中构建 SqlSessionFactory 的实例非常简单，**建议使用类路径下的资源文件**进行配置。 但也可以使用任意的输入流（InputStream）实例，比如用文件路径字符串或 file:// URL 构造的输入流。**MyBatis 包含一个名叫 Resources 的工具类**，它包含一些实用方法，使得从类路径或其它位置加载资源文件更加容易。

  - ```java
    String resource = "org/mybatis/example/mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    ```

  - **XML 配置文件**中包含了对 **MyBatis 系统的核心设置**，包括**获取数据库连接实例的数据源**（DataSource）以及决定事务作用域和控制方式的事务管理器（TransactionManager）。一个简单的xml配置文件如下

  - ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    // 核心配置文件 连接数据库
    <configuration>
      <environments default="development">
        <environment id="development">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
            <property name="driver" value="${driver}"/>
            <property name="url" value="${url}"/>
            <property name="username" value="${username}"/>
            <property name="password" value="${password}"/>
          </dataSource>
        </environment>
      </environments>
        // 每一个mapper.xml都需要在核心配置文件里配置 这里有个坑是要用/而不是.
      <mappers>
        <mapper resource="org/mybatis/example/BlogMapper.xml"/>
      </mappers>
    </configuration>
    ```
  
- 编写mybatis工具类

  ​      封装获得sqlsession的连接（可以理解为执行sql的对象）

  - 通过xml配置文件获得一个输入流inputStream
  -  通过一个SqlSessionFactoryBuilder来生成SqlSessionFactory 的实例
  - **从 SqlSessionFactory 中获取 SqlSession**，SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以**通过 SqlSession 实例来直接执行已映射的 SQL 语句**

```java
//sql工厂生成sql执行对象
public class MybatisUtils {
    // 提升作用域
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            // 获取SqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    // 获取SqlSession对象
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

## 2.3 编写代码

- 创建实体类

  ```java
  public class User {
      // 实体类
      private int id;
      private String name; // 数据库中varchar类型java中变成string
      private String pwd;
  
      public User() {
      }
  
      public User(int id, String name, String pwd) {
          this.id = id;
          this.name = name;
          this.pwd = pwd;
      }
  
      public int getId() {
          return id;
      }
  
      public void setId(int id) {
          this.id = id;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getPwd() {
          return pwd;
      }
  
      public void setPwd(String pwd) {
          this.pwd = pwd;
      }
  // 重写一些toString() 不然返回的是地址
      @Override
      public String toString() {
          return "User{" +
                  "id=" + id +
                  ", name='" + name + '\'' +
                  ", pwd='" + pwd + '\'' +
                  '}';
      }
  }
  ```

- Mapper接口

  ```java
  public interface UserMapper {
      List<User> getUserList();
  }
  ```

- 接口实现类（用Mapper.xml文件来实现） 不再使用原来的UserDaoImpl

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个对应的Mapper接口-->
<mapper namespace="com.cjt.dao.UserMapper">
<!--   select 查询语句-->
    id是实现的方法  resultType是返回的结果是实体类
    <select id="getUserList" resultType="com.cjt.pojo.User">
        select * from mybatis.user
    </select>
</mapper>
```

## 2.4 测试

Junit包来测试

```java
public class UserMapperTest {

    @Test
    public void test(){
        // 第一步： 获取sqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        // 执行Sql
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = mapper.getUserList();
        // 输出结果
        for (User user : userList) {
            System.out.println(user);
        }
        // 关闭连接
        sqlSession.close();

    }
}
```

**可能出现的问题：**

- **Maven静态资源过滤问题**（maven的约定大于配置）

出现找不到mapper.xml的异常

需要下边加到pom.xml中（maven的核心配置）

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

- 绑定接口错误
- 方法名不对
- 返回类型不对

## 2.5 总结步骤

1. 创建数据库
2. 新建maven，导入maven依赖（mysql mybatis junit）
3. resources中导入 mybatis核心配置xml文件 |用来连接数据库 配置jdbc的连接 写入mapper.xml |**每次新的mapper都要加入进去**
4. 写一个mybatis工具类 |来封装生成sqlSession的过程 （通过sqlSessionFactory来生成）|**不会再发生变动**
5. 一个数据库对应的实体类 |包括 数据库的所有行属性 set get方法 有参无参构造 重写toString()方法 |**不会再发生变动**
6. 一个**UserMapper接口** 
7. **Mapper接口实现类 用Mapper.xml来实现** |包括namespace（命名空间）id是实现的方法  resultType是返回的结果：实体类 然后是sql语句 
8. 测试

#  3 CRUD

三个步骤是始终要固定的

 ```  java
 // 通过写好的工具类获取sqlSession对象
 SqlSession sqlSession = MybatisUtils.getSqlSession();
 // 获取mapper来执行sql
 UserMapper mapper = sqlSession.getMapper(UserMapper.class);
 // 关闭sqlSession
 sqlSession.close();
 ```



## 1 namespace

namespqce中的包名要和mapper（接口）名字一致

## 2 select

选择查询语句

id：对应的namespace的方法名

parameterType：传入sql语句的参数类型

resultType ： sql语句返回值类型（完整的类名或者别名）



**需求： 根据id 查询用户**

1 在Mapper接口中增加这个方法

```java
//    根据id查询用户
    User getUserById(int id);
```

2 在mapper.xml 中实现这个方法

```xml
<select id="getUserById" resultType="com.cjt.pojo.User" parameterType="int">
    select * from mybatis.user where id = #{id}
</select>
```

3 test

```Java
@Test
    public void getUserByIdTest(){
        // 第一步： 获取sqlSession对象 不会变
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        // 执行Sql
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        // 之前的都是固定的
        User userById = mapper.getUserById(1);
        System.out.println(userById);
		// 关闭
        sqlSession.close();
```



## 3 insert

1 在Mapper接口中增加这个方法

```java
//    insert 一个用户 插入的是一个user对象
    int addUser(User user);
```

2 在mapper.xml 中实现这个方法

```xml
<insert id="addUser" parameterType="com.cjt.pojo.User" >
    insert into mybatis.user (id, name, pwd) VALUES (#{id},#{name},#{pwd});
</insert>
```

3 test

```java
@Test
public void addUser(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User user = new User(5, "吴亦凡", "654321");
    int result = mapper.addUser(user);
    if(result > 0){
        System.out.println("插入成功");
    }
    // 提交事务 插入数据需要提交事务 增删改必须提交事务
    sqlSession.commit();

    sqlSession.close();
}
```

**增删改 都涉及事务 都要提交事务 **sqlSession.commit();

## 4 update

1 在Mapper接口中增加这个方法

```java
//    更改一个用户
    int updateUser(User user);
```

2 在mapper.xml 中实现这个方法

```xml
<update id="updateUser" parameterType="com.cjt.pojo.User">
    update mybatis.user set name = #{name},pwd = #{pwd} where id = #{id};
</update>
```

3 test

```java
@Test
public void updateUSer(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    int i = mapper.updateUser(new User(5,"haha","123"));
    System.out.println(i);
    sqlSession.commit();
    sqlSession.close();
```

## 5 delete

1 在Mapper接口中增加这个方法

```java
// 删除一个用户
    int deleteUser(int id);
```

2 在mapper.xml 中实现这个方法

```xml
<delete id="deleteUser" parameterType="int">
    delete from mybatis.user where id = #{id};
</delete>
```

3 test

```java
@Test
public void deleteUser(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int i = mapper.deleteUser(5);
    System.out.println(i);
    sqlSession.commit();
    sqlSession.close();
}
```

注意点：

- **增删改都要提交事务**

## 6 分析错误

- namespace 绑定接口用 .      核心配置文件中的mapper注册 用 / 即resource绑定mapper要用/
- 程序配置文件要符合规范
- 空指针异常问题 没有注册到资源
- 输出的xml文件中存在中文乱码问题
- maven资源没有导出

## 7 万能map

假设实体类参数过多，可以考虑使用map来实现 参数量少，直接传递参数即可



1 在Mapper接口中增加这个方法

```java
//    万能的map
    int addUser2(Map<String,Object> map);
```

2 在mapper.xml 中实现这个方法

```xml
<insert id="addUser2" parameterType="map" >
    insert into mybatis.user (id, name, pwd) VALUES (#{id},#{name},#{pwd});
</insert>
```

3 test

```java
public void addUser2(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    Map<String,Object> map = new HashMap<>();
    map.put("id",6);
    map.put("name","长春");
    int result = mapper.addUser2(map);
    if(result > 0){
        System.out.println("插入成功");
    }
    // 提交事务 插入数据需要提交事务 增删改必须提交事务
    sqlSession.commit();
    sqlSession.close();
}
```

Map传递参数，直接在sql中取出key即可！ 【parameter=“map”】

对象传递参数，直接在sql中取出对象的属性即可！ 【parameter=“Object”】

只有一个基本类型参数的情况下，可以直接在sql中取到

多个参数用Map , **或者注解！**

## 8 模糊查询

1 Java代码执行的时候，传递通配符% %

```java
List<User> userList = mapper.getUserLike("%李%");
```

2 在sql拼接中使用通配符(这样可能发生sql注入)

```xml
select * from user where name like "%"#{value}"%"
```

# 4 配置解析

## 4.1 核心配置文件

mybatis-config.xml 系统核心配置文件

能配置的文件

```xml
    properties（属性）
    settings（设置）
    typeAliases（类型别名）
    typeHandlers（类型处理器）
    objectFactory（对象工厂）
    plugins（插件）
    environments（环境配置）
    	environment（环境变量）
    		transactionManager（事务管理器）
    		dataSource（数据源）
    databaseIdProvider（数据库厂商标识）
    mappers（映射器）
```

## 4.2 环境配置 encironments

MyBatis 可以配置成适应多种环境 **通过id进行切换 id保证唯一**

**事务管理器：transactionManager**

```xml
<transactionManager type="[JDBC|MANAGED"/>
```

**数据源 dataSource**

```xml
<dataSource type="[POOLED|UNPOOLED|JNDI">
```

- unpooled 这个数据源每次被请求时打开和关闭连接
- pooled 利用“池” 的概念将jdbc连接对象组织起来，使得并发的web应用快速响应请求的处理方式
- jndi 这个数据源的实现是为了 能在spring或应用服务器这类容器中使用，容易可以集中或者在外部配置数据源，然后放置一个jndi上下文的引用

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境**

学会使用配置多套运行环境！

MyBatis默认的事务管理器transactionManager就是JDBC ，数据源类型dataSource type：连接池：POOLED

## 4.3 属性 properties

我们可以通过properties属性来实现引用配置文件

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。【db.poperties】如下

1 编写一个db.poperties文件

```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?userSSL=false&useUnicode=true&characterEncoding=UTF-8
username=root
password=root

```

2 在核心配置文件中引入 （注意核心配置文件各种配置的放置顺序是有规定的）poperties就应该放在开头

```xml
<!--引入外部配置文件-->
    <properties resource="db.properties">
        <property name="username" value="dev_user"/>
  		<property name="password" value="F2Fa3!33TYyg"/>
    </properties>
```

- 可以直接引入外部文件

- 可以在其中增加一些属性配置

- 如果两个文件有同一个字段，优先使用外部配置文件的

- 也可以在 SqlSessionFactoryBuilder.build() 方法中传入属性值

  ```java
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, props);
  // ... 或者 ...
  SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, props);
  ```

优先级顺序：通过**方法参数**传递的属性具有**最高**优先级，resource/url 属性中指定的**配置文件**次之，最低优先级的则是 **properties 元素中**指定的属性。

从 MyBatis 3.4.2 开始，你可以为占位符指定一个默认值

详见官方文档[mybatis – MyBatis 3 | 配置](https://mybatis.org/mybatis-3/zh/configuration.html)

## 4.4 别名 typeAliases

- 类型别名可为 Java 类型设置一个缩写名字。 它**仅用于 XML 配置.**
- 意在**降低冗余**的全限定类名书写。

```xml
<!--    别名-->
    <typeAliases>
        <typeAlias type="com.cjt.pojo.User" alias="User"></typeAlias>
    </typeAliases>
```

也可以指定一个包，每一个在包 `domain.blog` 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 `domain.blog.Author` 的别名为 `author`,；若有注解，则别名为其注解值。见下面的例子：

```xml
<typeAliases>
    <package name="com.cjt.pojo"/>
</typeAliases>
```

在实体类比较少的时候，使用第一种方式。

如果实体类十分多，建议用第二种扫描包的方式。

第一种可以DIY别名，第二种不行，如果非要改，需要**在实体类上增加注解**。

```java
@Alias("author")
public class Author {
    ...
}

```

## 4.5 设置 setting

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。

## 4.6 其他配置

- [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
- [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
- plugins 插件（maven仓库）
  - mybatis-generator-core
  - mybatis-plus
  - 通用mapper



## 4.7 映射器 （mappers）

MapperRegistry：注册绑定我们的Mapper文件；

**方式一：推荐使用**

```xml
<!--每一个Mapper.xml都需要在MyBatis核心配置文件中注册-->
<mappers>
    <mapper resource="com/kuang/dao/UserMapper.xml"/>
</mappers>
```

方法二：使用class文件绑定注册

```xml
<mapper class="com.cjt.dao.UserMapper"></mapper>
```

**注意点：**

- 接口和他的Mapper配置文件必须同名
- 接口和他的Mapper配置文件必须在同一个包下

方式三：使用包扫描进行注入

```xml
<package name="com.cjt.dao"/>
```

## 4.8 生命周期



![mybatis执行流程](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)

声明周期和作用域是至关重要的，因为错误的使用会导致非常严重的**并发**问题。

**SqlSessionFactoryBuilder:**

- 一旦创建了SqlSessionFactory，就不再需要它了
- 局部变量



**sqlSessionFactory：**

- 说白了就可以想象为：**数据库连接池**
- SqlSessionFactory一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建一个实例。
- 因此SqlSessionFactory的最佳作用域是应用作用域（ApplocationContext）。
- 最简单的就是**使用单例模式或静态单例模式**。保证全局只有一个变量

SqlSession：

- 连接到连接池的一个请求
- SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的**最佳的作用域是请求或方法作用域。**
- **用完之后需要赶紧关闭**，否则资源被占用！

![SqlSession](Mybatis.assets/image-20210711155714515.png)

**每一个Mapper 都代表一个业务！**

# 5 resultMap 解决属性名和字段名不一致的问题

## 5.1 问题

数据库中的字段

![image-20210711155857944](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/image-20210711155857944.png)

实体类中的字段

![image-20210711160714192](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/image-20210711160714192.png)

测试出现问题 password = null

![image-20210711160752680](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/image-20210711160752680.png)

解决方法：

- 起别名：

```xml
<select id="getUserById" resultType="com.cjt.pojo.User">
        select id, name, pwd as password from mybatis.user where id = #{id}
    </select>
```

## 5.2 resultMap

结果集映射

```
id name pwd

id name password
```

```xml
<resultMap id="UserMap" type="com.cjt.pojo.User">
<!--column数据库的字段, property实体类中的字段-->
        <result column="id" property="id"></result>
        <result column="name" property="name"></result>
        <result column="pwd" property="password"></result>
    </resultMap>
```

- resultMap 元素是 MyBatis 中最重要最强大的元素。


- ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。


- ResultMap 的优秀之处——你完全可以不用显式地配置它们。


如果这个世界总是这么简单就好了。

# 6 日志

## 6.1 日志工厂

![日志](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/image-20210711172431044.png)

- SLF4J
- LOG4J 【掌握】
- LOG4J2
- JDK_LOGGING
- COMMONS_LOGGING
- STDOUT_LOGGING 【掌握】
- NO_LOGGING

在MyBatis中具体使用哪一个日志实现，在设置中设定。它会使用最先找到的（按上文列举顺序查找）如果一个都未找到，日志功能就会被禁用

在mybatis核心配置文件中，配置我们的日志

```xml
<!--setting-->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```



![image-20210711192712726](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/image-20210711192712726.png)

## 6.2 LOG4J

什么是LOG4J：

- Log4j是Apache的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件；


- 我们也可以控制每一条日志的输出格式；
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程；
- 最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码

使用步骤：

1 导入Log4j的包(pom.xml)

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

2 配置文件编写 log4j.properties

```properties
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/cjt.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sq1.PreparedStatement=DEBUG
```

3 setting设置日志实现

```xml
<!--setting-->
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
```

4 直接输出

![image-20210711195107274](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/image-20210711195107274.png)

5 在程序中输出

1. 在要使用Log4j的类中，导入包 **import org.apache.log4j.Logger;** 包不要导错了

2. 日志对象，参数为当前测试类的class对象 放在方法外

   ```java
   static Logger logger = Logger.getLogger(UserMapperTest.class);
   ```

3. 日志级别

   ```java
   logger.info("info:进入了testLog4j方法");
   logger.debug("debug:进入了testLog4j方法");
   logger.error("error:进入了testLog4j方法");
   ```



# 7 分页

**为什么要分页**？

- 处理大量数据时，使用分页来减少数据库的压力

## 7.1 使用Limit进行分页

```mysql
SELECT * from user limit startIndex,pageSize startIndex从0开始
```

接口

```java
List<User> getUserByLimit(Map<String,Integer> map);
```

Mapper.xml

```xml
<select id="getUserByLimit" parameterType="map" resultType="com.cjt.pojo.User">
    select * from mybatis.user limit #{startIndex}, #{pageSize};
</select>
```

测试

```java
@Test
    public void getUserListByLimit(){
        logger.info("info:进入了getUserListByLimit测试方法");
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map<String,Integer> map = new HashMap<>();
        map.put("startIndex",2);
        map.put("pageSize",2);
        List<User> user = mapper.getUserByLimit(map);
        for (User user1 : user) {
            System.out.println(user1);
        }
        sqlSession.close();
    }
```

## 7.2 使用RowBouds类来实现分页

在java测试代码层面实现分页，不推荐使用

1 接口

```java
List<User> getUserByRowBounds();
```



2 Mapper.xml

```xml
<select id="getUserByRowBounds" resultType="com.cjt.pojo.User">
        select * from mybatis.user
    </select>
```



3 测试

```java
@Test
    public void getUserListByRowBounds(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        RowBounds rowBounds = new RowBounds(2, 2);
        List<Object> objects = sqlSession.selectList("com.cjt.dao.UserMapper.getUserByRowBounds", null, rowBounds);
        for (Object object : objects) {
            System.out.println(object);
        }
        sqlSession.close();
```



## 7.3 分页插件

PageHelper





# 8 使用注解开发

## 8.1 面向接口编程

- 大家之前都学过面向对象编程，也学习过接口，但在真正的开发中，很多时候我们会选择面向接口编程
- 根本原因 :  **解耦 , 可拓展** , **提高复用** , 分层开发中 , 上层不用管具体的实现 , 大家都遵守共同的标准, 使得开发变得容易 , 规范性更好
- 在一个面向对象的系统中，系统的各种功能是由许许多多的不同对象协作完成的。在这种情况下，各个对象内部是如何实现自己的,对系统设计人员来讲就不那么重要了；
- 而各个对象之间的协作关系则成为系统设计的关键。小到不同类之间的通信，大到各模块之间的交互，在系统设计之初都是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是指按照这种思想来编程。

**关于接口的理解**

- 接口从更深层次的理解，应是定义（规范，约束）与实现（名实分离的原则）的分离。

接口的本身反映了系统设计人员对系统的抽象理解。

- 接口应有两类：
  - **第一类**是对一个个体的抽象，它可对应为一个抽象体(abstract class)；
  - **第二类**是对一个个体某一方面的抽象，即形成一个抽象面（interface);
  - 一个个体有可能有多个抽象面。抽象体与抽象面是有区别的

## 8.2 利用注解开发

1 注解在接口上实现

```java
@Select("select * from mybatis.user")
List<User> getUser();
```

2 在核心配置文件中绑定接口

```xml
<!--使用class绑定接口-->
<mappers>
    <mapper class="com.cjt.dao.UserMapper"></mapper>
</mappers>
```

3 测试

```java
@Test
public void test(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> users = mapper.getUser();

    for (User user : users) {
        System.out.println(user);
    }
}
```



**本质：利用反射机制实现**

**底层：动态代理**

## 8.3 mybatis详细流程

![image-20210928224544026](Mybatis.assets/image-20210928224544026.png)

## 8.4 注解增删改

改造工具类中的 getSqlSession()可以实现事务自动提交

```java
// 获取SqlSession对象
public static SqlSession getSqlSession(){
    return sqlSessionFactory.openSession(true); // 事务自动提交
}
// 重载
    public static SqlSession getSqlSession(boolean flag){
        return sqlSessionFactory.openSession(flag);
    }
```

### select

1 接口注解

```java
@Select("select * from mybatis.user where id = #{id2}")
User getUserById(@Param("id2") int id);
```

2 测试

```java
@Test
public void testById(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User userById = mapper.getUserById(2);
    System.out.println(userById);
    sqlSession.close();
}
```

### @Param 注解

- 基本类型的参数或者String类型，需要加上
- 引用类型不需要加
- 如果只有一个基本类型的话，可以忽略，但是建议大家都加上
- 我们**在SQL中引用的就是我们这里的@Param()中设定的属性名**（都是id2）
- 如果参数是JavaBean，则不能使用@Param
- 不使用@Param注解时，参数只能有一个，并且是JavaBean

### insert

1 接口注解

```java
@Insert("insert into user (id,name,pwd) values (#{id},#{name},#{pwd})")
int insertUser(User user);
```

2 测试

```java
@Test
public void testInsert(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);// 设置事务自动提交
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int i = mapper.insertUser(new User(10, "李石", "12345"));
    System.out.println(i);
    sqlSession.close();
}
```

### delete

1 接口注解

```java
@Delete("delete from user where id = #{id} and name = #{name}")
int deleteUser(@Param("id") int id,@Param("name") String name);
```

2 TEST

```java
@Test
public void testDelete(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int i = mapper.deleteUser(10,"李石");
    System.out.println(i > 0 ? "删除成功" : "删除失败");
    sqlSession.close();
}
```

### update

1 接口注解

```java
@Update("update user set name = #{name} , pwd = #{pwd} where id = #{id}")
int updateUser(User user);
```

2 测试

```java
@Test
public void testUpdate(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    int i = mapper.updateUser(new User(6, "王六", "666"));
    System.out.println(i > 0 ? "更新成功" : "更新失败");
    sqlSession.close();
}
```

## 8.5 # 与 $ 的区别

- `${}`是 Properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于静态文本替换，比如${driver}会被静态替换为`com.mysql.jdbc.Driver`。
- `#{}`是 sql 的参数占位符，MyBatis 会将 sql 中的`#{}`替换为?号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的?号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。
- \#{} 的作用主要是替换预编译语句(PrepareStatement)中的占位符? 【推荐使用】
- ${} 的作用是直接进行字符串替换

## 8.6 Lombok

介绍：Lombok项目是一个**Java库**，它会自动插入编辑器和构建工具中，Lombok**提供了一组有用的注释**，用来消除Java类中的大量样板代码。仅五个字符(@Data)就可以替换数百行代码从而产生干净，简洁且易于维护的Java类。

使用步骤：

1 在idea中安装Lombok

2 在项目中导入jar包

```xml
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>
```

3 能加的注解

```java
@Getter and @Setter
@FieldNameConstants // 字段属性常量
@ToString
@EqualsAndHashCode
@AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor // 构造器
@Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog // 日志
@Data // 各种方法
@Builder
@SuperBuilder
@Singular
@Delegate
@Value
@Accessors
@Wither
@With
@SneakyThrows
@val
@var
experimental @var
@UtilityClass
Lombok config system
Code inspections
Refactoring actions (lombok and delombok)
```



```
@Data  get set tostring hashcode equals 
@AllArgsConstructor 有参构造
@NoArgsConstructor 无参构造
```

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    // 实体类
    private int id;
    private String name;
    private String pwd;
}
```

![使用lombok插件之后生成的方法](C:\Users\楚江涛\Desktop\视频md文件\Mybatis.assets/image-20210712131727453.png)

缺点：

- 不支持多种参数构造器的重载  可以自己加
- 降低了源代码的可读性和完整性



# 9 多对一处理

- 多个学生对应一个老师
- 对于学生而言，多个学生**关联**一个老师【多对一】
- 对于老师而言，**集合**一个老师教多个学生【一对多】
- 按照结果嵌套查询：对应mysql联表查询
- 按照查找嵌套查询：对应mysql子查询

## 9.1 设计

1.  数据库表设计

```sql
CREATE TABLE `teacher` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`, `name`) VALUES (1, '秦老师'); 

CREATE TABLE `student` (
  `id` INT(10) NOT NULL,
  `name` VARCHAR(30) DEFAULT NULL,
  `tid` INT(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fktid` (`tid`),
  CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`) -- 外键
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO `student` (`id`, `name`, `tid`) VALUES (1, '小明', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (2, '小红', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (3, '小张', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (4, '小李', 1); 
INSERT INTO `student` (`id`, `name`, `tid`) VALUES (5, '小王', 1);
```

2. 引入lombok依赖(pom.xml)

   ```xml
   <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <version>1.18.20</version>
   </dependency>
   ```

3.  生成实体类 添加注释

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Student {
       private int id;
       private String name;
       // 学生需要关联一个老师
       private Teacher teacher;
   }
   
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Teacher {
       private int id;
       private String name;
   }
   
   ```

4. 编写mapper接口【无论用不用都要有】

   ```java
   public interface StudentMapper {  
   }
   ```

   ```java
   public interface TeacherMapper {
   }
   ```

5. 编写Mapper接口对应的Mapper.xml【无论用不用都要有】

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper  namespace = "com.cjt.mapper.TeacherMapper">
   
   </mapper>
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper  namespace = "com.cjt.mapper.StudentMapper">
   
   </mapper>
   ```

6. mapper绑定到xml核心配置文件中    mybatis-config.xml 

   ```xml
   <mappers>
       <package name="com.cjt.mapper"/>  // 这里使用包名进行绑定
   </mappers>
   ```

7. 测试

   ```java
   @Test
   public void TestById(){
       SqlSession sqlSession = MybatisUtils.getSqlSession();
       TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
       Teacher teacher = mapper.getTeacher(1);
       System.out.println(teacher);
       sqlSession.close();
   }
   ```

## 9.2 按查询嵌套处理

1 写接口方法

```java
// 查询所有的学生信息，以及对应的老师的信息
public List<Student> getStudent();
```

2 mapper.xml实现

```xml
<select id="getStudent" resultMap="StudentTeacher">
    select * from mybatis.student
</select>

<resultMap id="StudentTeacher" type="student">
    // property 实现类中的名字  column 数据库表中的名字
    <result property="id" column="id"></result>
    <result property="name" column="name"></result>
    <!--复杂的属性，我们需要单独处理 对象：association  集合： collection-->
    <!--association关联属性  property属性名 javaType属性类型 column在多的一方的表中的列名-->
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"></association>
</resultMap>

<select id="getTeacher" resultType="teacher">
    select * from mybatis.teacher where id = #{id}
</select>
```

多参数时mapper.xml实现

```xml
<resultMap id="StudentTeacher"type="Student">
    <!--association关联属性  property属性名 javaType属性类型 column在多的一方的表中的列名-->
    <association property="teacher" column="{id=tid,name=tid}" javaType="Teacher" select="getTeacher"/></resultMap>

<!--这里传递过来的id，只有一个属性的时候，下面可以写任何值
association中column多参数配置：column="{key=value,key=value}"其实就是键值对的形式，
key是传给下个sql的取值名称，value是片段一中sql查询的字段名。-->

<select id="getTeacher" resultType="teacher">    
    select * from teacher where id = #{id} and name = #{name}
</select
```



3 测试

```java
@Test
public void testGetStudent(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    List<Student> list = mapper.getStudent();
    for (Student student : list) {
        System.out.println(student);
    }
    for (Student student : list) {
        System.out.println("学生名：" + student.getName() + " \t老师名:" + student.getTeacher().getName());
    }
    sqlSession.close();
}
```

## 9.3 按照结果嵌套查询

1 接口方法

```java
public List<Student> getStudent2();
```

2 mapper.xml实现

```xml
<!-- 按照结果嵌套查询-->
<select id="getStudent2" resultMap="StudentTeacher2">
    select s.id as sid,s.name as sname,t.name as tname,t.id as tid
    from mybatis.teacher as t,mybatis.student as s
    where s.tid = t.id
</select>

<resultMap id="StudentTeacher2" type="student">
    <result property="id" column="sid"></result>
    <result property="name" column="sname"></result>
    <association property="teacher" javaType="Teacher">
        <result property="id" column="tid"></result>
        <result property="name" column="tname"></result>
    </association>
</resultMap>
```

3 测试

```java
@Test
public void testGetStudent2(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
    List<Student> list = mapper.getStudent2();
    for (Student student : list) {
        System.out.println(student);
    }
    for (Student student : list) {
        System.out.println("学生名：" + student.getName() + "\t老师名:" + student.getTeacher().getName());
    }
    sqlSession.close();
}
```

回顾mysql多对一查询方式

- 子查询 按照查询嵌套
- 联表查询 按照结果嵌套查询



# 10 一对多的处理

比如：一个老师拥有多个学生 

对于老师而言就是一对多

1 实体类修改

```java
public class Teacher {
    private int id;
    private String name;

    // 一个老师拥有多个学生
    private List<Student> students;
}
```

```java
public class Student {
    private int id;
    private String name;
    private int tid;
}
```

## 10.1 按照结果嵌套查询

1.接口方法

```java
// 获取指定老师下的所有学生和老师的信息
Teacher getTeacher(@Param("tid") int id);
```

2 xml实现

```xml
<select id="getTeacher" resultMap="TeacherStudent">
    select t.id tid, t.name tname, s.name sname, s.id sid
    from mybatis.teacher as t ,mybatis.student as s
    where t.id = s.tid and t.id = #{tid}
</select>
<resultMap id="TeacherStudent" type="cjt.pojo.Teacher">
    <result property="id" column="tid"></result>
    <result property="name" column="tname"></result>
<!-- 集合使用collection JavaType是用来指定pojo中属性的类型  ofType指定的是映射到list集合属性中pojo的类型-->
    <collection property="students" ofType="student">
        <result property="id" column="sid"></result>
        <result property="name" column="sname"></result>
        <result property="tid" column="tid"></result>
    </collection>
</resultMap>
```

3 测试

```java
@Test
public void test(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
    Teacher teacher = mapper.getTeacher(1);
    System.out.println(teacher);
    sqlSession.close();
}
```

## 10.2 按照查找嵌套查询

1 接口方法

```java
Teacher getTeacher2(@Param("tid") int id);
```

2 xml实现

```xml
<!-- 按照查找嵌套查询-->
        <select id="getTeacher2" resultMap="TeacherStudent2">
            select id, name from mybatis.teacher as t  where id = #{tid}
        </select>

        <resultMap id="TeacherStudent2" type="teacher">
            <result property="id" column="id"></result>
            <result property="name" column="name"></result>
            <collection property="students" column="id"  ofType="student" javaType="ArrayList" select="getStudent"></collection>
        </resultMap>

        <select id="getStudent" resultType="student">
            select * from mybatis.student where tid = #{tid}
        </select>
```

3 测试

```java
@Test
public void test2(){
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
    Teacher teacher = mapper.getTeacher2(1);
    System.out.println(teacher);
    sqlSession.close();
}
```

## 10.3 小结

- 关联 - association 【多对一】
- 集合 - collection 【一对多】
- javaType & ofType
  - JavaType用来指定实体类中的类型
  - ofType用来指定映射到List或者集合中的pojo类型，泛型中的约束类型

注意点：

- 保证SQL的可读性，尽量保证通俗易懂
- 注意一对多和多对一，属性名和字段的问题
- 如果问题不好排查错误，可以使用日志，建议使用Log4j



# 11 动态SQL

什么是动态sql

官方文档

```markdown
动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

if
choose (when, otherwise)
trim (where, set)
foreach
```

## 11.1 搭建环境

创建一个基础工厂

- 新建一个maven项目
- 写入依赖
- 编写配置文件
- 写实体类
- 写接口和实现方法的xml

1 新建一个maven项目

![动态sql-maven项目](Mybatis.assets/image-20210713114943421.png)

2 写入依赖（父项目写过的就不需要写了）

```xml
<parent>
    <artifactId>Mybatis-Study</artifactId>
    <groupId>com.cjt</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>
。。。
<dependencies>
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
        </dependency>
       junit
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>RELEASE</version>
        </dependency>
    </dependencies>
```

3 实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date createTime; // 属性名和字段名不一致，数据库中是create_time 需要开启驼峰命名转换来解决
    private int views;
}
```

4 编写配置文件 （setting 别名 mappers 环境 et.al)

```xml
<!--setting-->
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
    <setting name="mapUnderscoreToCamelCase" value="true"/> // 下划线驼峰命名自动转换
</settings>
...
<mappers>
        <mapper resource="com/cjt/mapper/BlogMapper.xml"></mapper>
        <mapper class="com.cjt.mapper.BlogMapper"></mapper> // 用这个可以使用注解
    </mappers>
```

5 接口和xml（加入了插入数据的方法）

```java
public interface BlogMapper {
    int addBlog(Blog blog);
}
```

```xml
<mapper namespace="com.cjt.mapper.BlogMapper">

    <insert id="addBlog" parameterType="blog">
        insert into mybatis.blog (id, title, author, create_time, views)
        VALUES (#{id},#{title},#{author},#{createTime},#{views});
    </insert>

</mapper>
```

初始化博客

```java
@Test
public void addBlog(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);

    Blog blog = new Blog();
    blog.setId(IDUtil.getId());
    blog.setTitle("Mybatis");
    blog.setAuthor("cjt");
    blog.setCreateTime(new Date());
    blog.setViews(9999);

    mapper.addBlog(blog);

    blog.setId(IDUtil.getId());
    blog.setTitle("Java不简单");
    mapper.addBlog(blog);

    blog.setId(IDUtil.getId());
    blog.setTitle("Spring不简单");
    mapper.addBlog(blog);

    blog.setId(IDUtil.getId());
    blog.setTitle("微服务不简单");
    mapper.addBlog(blog);

    sqlSession.close();

}
```

手动改了views的初始化结果

![初始化结果](Mybatis.assets/image-20210713120411768.png)

## 11.2 If

1 接口方法

```java
// 查询满足某些条件的博客
List<Blog> queryBlogIf(Map map);
```

2 xml实现

```xml
<!--需求1: 根据作者的名字和博客名字来查询博客，如果作者名字为空，则只根据博客名字，反之亦然
select * from blog where title = #{title} and author = #{author}-->

<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from mybatis.blog where 1=1 // 这里这是为了跑的顺畅
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</select>
```

3 测试

```java
@Test
public void queryBlogByIf(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    Map<String,String> map = new HashMap<>();
    map.put("title","Mybatis");
    map.put("author","cjt");
    List<Blog> blogs = mapper.queryBlogIf(map);
    for (Blog blog : blogs) {
        System.out.println(blog);
    }
    sqlSession.close();
}
```

## 11.3 where

- “where”标签会知道如果它包含的标签中有返回值的话，它就插入一个‘where’。此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。【这是我们使用的最多的案例】

- *where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

xml修改

```xml
<!--需求1: 根据作者的名字和博客名字来查询博客，如果作者名字为空，则只根据博客名字，反之亦然
select * from blog where title = #{title} and author = #{author}-->

<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <if test="title != null">
            and title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </where>
</select>
```

## 11.4 choose

不想用到所有的查询条件，查询条件有一个满足就可以 类似于switch语句 有一个满足以后就不会继续执行接下来的

```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
    select * from mybatis.blog
    <where>
        <choose>
            <when test="title != null">
                title = #{title}
            </when>
            <when test="author != null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```

## 11.5 set

用于动态更新语句的类似解决方案叫做 *set*。*set* 元素可以用于动态包含需要更新的列，忽略其它不更新的列

*set* 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号（这些逗号是在使用条件语句给列赋值时引入的）

1 接口

```java
// 更新博客
int updateBlog(Map map);
```

2 xml实现

```xml
<update id="updateBlog" parameterType="map">
    update mybatis.blog
    <set>
        <if test="title != null">
            title = #{title}, // 如果放在where之前，会自动去除','
        </if>
        <if test="author != null">
            author = #{author},
        </if>
    </set>
    where id = #{id}
</update>
```

3 测试

```java
@Test
public void updateBlog(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    Map<String,String> map = new HashMap<>();
    map.put("title","Mybatis");
    map.put("author","cjtTT");
    map.put("id","4f47567609b845e5bf2f46adcc1953c5");

    int i = mapper.updateBlog(map);
    System.out.println(i);
    sqlSession.close();
}
```

## 11.6 trim

```xml
<trim prefix="" prefixOverrides="" suffix="" suffixOverrides="">

</trim>
```

自定义去除或者加入

和 *where* 元素等价的自定义 trim 元素为：

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```

和 *set* 元素等价的自定义 trim 元素为：

```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

## 11.7 SQL片段

某个sql语句用的特别多，为了增加代码的复用性 简化代码，需要将这些代码提取出来，然后使用时直接调用

提出sql片段 <sql>

```xml
<sql id="if-title-author" >
    <if test="title != null">
        title = #{title},
    </if>
    <if test="author != null">
        author = #{author},
    </if>
</sql>
```

引用sql片段 <include>

```xml
<update id="updateBlog" parameterType="map">
    update mybatis.blog
    <set>
        <include refid="if-title-author"></include>  // 引用sql片段
    </set>
    where id = #{id}
```

注意：

- 最好基于单表来定义sql片段，提高片段的可重用性
- 在sql片段中不要包括where set

## 11.8 Foreach

*oreach* 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符。这个元素也不会错误地添加多余的分隔符，看它多智能！

**提示** 你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给 *foreach*。当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

1 接口

```java
// 查询1 2 3 号博客的信息
List<Blog> queryForeach(Map<String, List<Integer>> map);
```

2 xml实现

```xml
<select id="queryForeach" parameterType="map" resultType="blog">
    <!--select * from blog where 1 = 1 and (id =1 or id =2 or id =3)-->
    select * from mybatis.blog
    <where>
        <!--collection 指定输入对象中的集合属性
 			item 每次遍历的对象
			open 开始遍历时的拼接字符串
			close 结束时的拼接字符串
			separator 遍历对象之间需要拼接的字符串-->
        <foreach collection="ids" item="id" open="(" separator="or" close=")">
            id = #{id}
        </foreach>
    </where>
</select>
```

3 测试

```java
@Test
public void queryBlogForeach(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    // 使用万能map作为参数输入 key是Stirng value是遍历的集合
    Map<String,List<Integer>> map = new HashMap<>();
    //遍历的集合 存放拼接的对象
    List<Integer> list = new LinkedList<>();
    list.add(1);
    list.add(2);
    list.add(3);
    // 放入map中
    map.put("ids",list);
    
    List<Blog> blogs = mapper.queryForeach(map);
    for (Blog blog : blogs) {
        System.out.println(blog);
    }
    sqlSession.close();
}
```

## 11.9 总结

**动态SQL就是在拼接SQL语句，我们只要保证SQL的正确性，按照SQL的格式，去排列组合就可以了**

建议：

- 先在Mysql中写出完整的SQL，再对应的去修改成我们的动态SQL实现通用即可



# 12 缓存

## 12.1 简介

```markdown
查询 ：连接数据库，耗资源
一次查询的结果，给他暂存一个可以直接取到的地方 --> 内存：缓存
我们再次查询的相同数据的时候，直接走缓存，不走数据库了
```

**什么是缓存[Cache]？**

- 存在内存中的临时数据
- 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库文件）查询，从缓存中查询，从而提高查询效率，解决了**高并发系统的性能**问题

**为什么使用缓存？**

- 减少和数据库的交互次数，减少系统开销，提高系统效率

**什么样的数据可以使用缓存？**

- 经常查询并且不经常改变的数据 【可以使用缓存】

## 12.2 mybatis 缓存

- MyBatis包含一个非常强大的查询缓存特性，它可以非常方便的定制和配置缓存，缓存可以极大的提高查询效率。
- MyBatis系统中默认定义了**两级缓存：一级缓存和二级缓存**
- **默认情况下，只有一级缓存开启**（**SqlSession级别**的缓存，也称为**本地缓存**）close() 之后就无效了
- **二级缓存**需要**手动开启和配置**，他是**基于namespace级别**的缓存。(接口级别)
- 为了提高可扩展性，MyBatis定义了**缓存接口Cache**。我们可以通过实现Cache接口来定义二级缓存。

## 12.3 一级缓存

- 一级缓存也叫本地缓存：SqlSession
  - 与数据库同一次会话期间查询到的数据会放在本地缓存中
  - 以后如果需要获取相同的数据，直接从缓存中拿，没必要再去查询数据库

测试步骤：

1 开启日志

2 测试在一个Session中查询两次记录

```java
@Test
public void selectById(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User user = mapper.selectUserById(1);
    System.out.println(user);
    System.out.println("-------------------------------");
    User user2 = mapper.selectUserById(1);
    System.out.println(user2);
    sqlSession.close();
}
```

3 查看日志

![日志](Mybatis.assets/image-20210713180020097.png)

缓存失效的情况

1. 查询不同的东西

2. 增删改操作，可能会改变原来的数据，所以必定会刷新缓存

3. 查询不同的Mapper.xml

4. 手动清理缓存

   ```java
   sqlSession.clearCache();
   ```

小结： 一级缓存默认开启，只在一次sqlSession中有效

一级缓存就是一个Map

## 12.3 二级缓存

- 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存
- 工作机制
  - 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中
  - 如果会话关闭了，这个会员对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中
  - 新的会话查询信息，就可以从二级缓存中获取内容
  - 不同的mapper查询出的数据会放在自己对应的缓存（map）中

一级缓存开启（SqlSession级别的缓存，也称为本地缓存）

- 二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
- 为了提高可扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来定义二级缓存。

步骤

1 开启全局缓存【mybatis-config.xml】

```xml
<!--显示的开启全局缓存-->
<setting name="cacheEnabled" value="true"/>
```

2 在当前Mapper.xml中使用缓存

```xml
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"></cache>
FIFO 缓存失效方式 flushInterval 隔多长时间刷新 size 最多可以存储的对象或者列表的引用数 
readOnly 返回的对象是只读的 因为进行修改可能会在不同线程中的调用者产生冲突
```

3 测试

```java
@Test
public void selectById(){
    SqlSession sqlSession = MybatisUtils.getSqlSession(true);
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    SqlSession sqlSession2 = MybatisUtils.getSqlSession(true);

    User user = mapper.selectUserById(1);
    System.out.println(user);
    sqlSession.close();

    System.out.println("-------------------------------");
    UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
    User user2 = mapper2.selectUserById(1);
    System.out.println(user2);
    
    System.out.println(user == user2); 
    // readOnly = true 这里是同一个 返回true readOnly = false 是一个拷贝，就返回false
    sqlSession2.close();
}
```

问题

- 所有的实体类都要先实现序列化接口

```java
public class User implements Serializable {
    private int id;
    private String name;
    private String pwd;
}
```

小结：

- 只要开启了二级缓存，在同一个Mapper下就有效
- 所有的数据都会放在一级缓存中
- 只有当前会话提交，或者关闭的时候，才会提交到二级缓存中

## 12.4 缓存原理

![在这里插入图片描述](Mybatis.assets/20200623165404113.png)

缓存顺序：

- 第一次查询走数据库，放在一级缓存中
- 查询先看二级缓存中有没有
- 再看一级缓存中有没有
- 最后再看数据库

## 12.5 自定义缓存

第三方缓存实现 EhCache

1 导依赖

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.2.1</version>
</dependency>
```

2 mapper.xml中使用这个缓存

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
```

