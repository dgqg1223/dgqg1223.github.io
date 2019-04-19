---
title: Mybatis.1.基础
date: 2019-03-1 15:51:32
tags: Mybatis
categories: 
    - Java
    - 框架
---

## MyBatis介绍
DOC：http://www.mybatis.org/mybatis-3/zh/index.html

MyBatis前身iBatis，MyBatis是一个优秀的持久层框架，对jdbc操作的数据库过程进行封装，使开发者只需要关注sql本身。帮助我门处理注册驱动、创建connection、创建statement、手动设置参数、结果检索等JDBC繁杂的过程代码。


## MyBatis原理
通过xml 或注解的方式将要执行的各种statement（statement、preparedStatement、CallableStatement）配置起来，并通过java对象和statement中的sql进行映射生成最终的sql语句，最后由mybatis框架执行sql并将结果映射成java对象并返回。


## MyBatis核心
1. 配置文件
全局配置文件：配置了数据源、事务等信息  
映射文件：执行sql相关的信息  

2. SqlSessionFactory会话工厂
读取配置文件信息（全局和映射配置文件），构造出`SqlSessionFactory`会话工厂  

3. SqlSession会话
通过SqlSessionFactory,可以创建`SqlSession`会话。MyBatis是通过SqlSession操作数据库的  

4. SqlSession原理
SqlSession本身不能操作数据库，它通过底层的`Executor`执行接口来操作数据库，Executor接口有两个实现类，普通执行器和缓存执行器（默认）  

5. Executor原理
Executor执行器要处理的SQL信息是封装到底层对象`MappedStatement`中，该对象包括:SQL语句、输入参数映射信息、输出结果映射信息。输入参数和输入结果映射类型包括`java简单类型`、`HashMap集合对象`、`POJO对象类型`  


## MyBatis入门案例
### 环境搭建
创建数据库和表  
下载MyBatis（3.5.0）：https://github.com/mybatis/mybatis-3  
创建项目导入lib下的jar包、mybatis-3.5.0.jar、mysql驱动包    
在classpath下创建log4j.properties如下：（文件内容可以从mybatis-3.2.7.pdf中拷贝）
``` log4j.propertie
# Global logging configuration
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```


### 开发步骤
1. 根据需求创建创建PO（model）类User.class
``` java
package com.chen.model;
import java.io.Serializable;
import java.util.Date;

public class User implements Serializable {
    private int id;
    private String username;// 用户姓名

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

  
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + "]";
    }

}
```
2. 创建全局配置文件SqlMapConfig.xml（文件名可以自定义）
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置mybatis的环境信息 默认开发环境-->
    <environments default="development">
        <environment id="development">
            <!-- 配置JDBC事务控制，mybastis进行管理-->
            <transactionManager type="JDBC"/>
            <!-- 配置数据源，采用JDBC连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&amp;characterEncoding=utf8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/chen/sqlmap/User.xml" />
    </mappers>
</configuration>
```
3. 编写映射文件，并在全局配置文件中加载
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="user">
    <!--根据id查询 -->
    <select id="findUserById" parameterType="int" resultType="com.chen.model.User">
        select * from user where id = #{id}
    </select>
</mapper>
```
4. 编写测试程序连接并操作数据库
``` java
package com.chen.Test;


import com.chen.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;

public class Demo1 {
    @Test
    public void test1() throws IOException {
        //读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //通过SqlSessionFactoryBUidler创建SqlSessionFactory会话工厂
        SqlSessionFactory SessionFactory = new SqlSessionFactoryBuilder().build(is);
        //通过SqlSessionFactory创建Sqlsession
        SqlSession session = SessionFactory.openSession();
        //调用SqlSession的操作数据库的方法
        User user = session.selectOne("findUserById", 10);
        System.out.println(user);
        //关闭SqlSession
        session.commit();
    }
}
```
### 关于占位符  
1. 字符串拼接使用 `${value}`  
`${value}`在字符串拼接时候使用，如模糊查询，如果参数为简单类型时，参数名必须为value，使用这种占位符会引起sql注入问题，所以不推荐使用，但是有些场景必须使用比如 `order by ${value}`
2. 普通简单类型占位符  `#{abc}`  
普通占位符 和jdbc中的`?`同理,如果拼接是简单类型参数那么参数名和填任意字符

### 查询与模糊查询
映射文件中定义user.xml
``` xml
    <!-- 根据id查询 使用#{} -->
    <select id="findUserById" parameterType="int" resultType="com.chen.model.User">
        select * from user where id = #{id}
    </select>
    <!-- 迷糊查询 使用${} -->
    <select id="findUserByLikeName" parameterType="String" resultType="com.chen.model.User">
        select * from user where username like '%${value}%'
    </select>
```
建立测试类
``` java
public class Demo2 {
    SqlSession session;

    @Before
    public void before() throws IOException {
        //读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //通过SqlSessionFactoryBUidler创建SqlSessionFactory会话工厂
        SqlSessionFactory SessionFactory = new SqlSessionFactoryBuilder().build(is);
        //通过SqlSessionFactory创建Sqlsession
        session = SessionFactory.openSession();
    }
    @After
    public void after() {
        //关闭SqlSession
        session.commit();
    }
}
```
测试类调用方式
``` java
    //查询一条结果使用session.selectOne()
    User user = session.selectOne("findUserById", 10);
    System.out.println(user);
    //返回为结果集使用session.selectList()
    List<User> users = session.selectList("findUserByLikeName", "明");
    System.out.println(users);
```

### 删除、更新、修改
映射文件user.xml
``` xml
    <!--插入数据 这里的占位符写模型的属性 -->
    <insert id="insertUser" parameterType="com.chen.model.User">
        insert into user (username,sex,birthday,address)
        value(#{username},#{sex},#{birthday},#{address});
    </insert>
    <!-- 删除数据 -->
    <delete id="deleteUser" parameterType="int">
        delete from user where id=#{id};
    </delete>
    <!-- 修改数据 -->
    <update id="updateUser" parameterType="com.chen.model.User">
        update user set sex=#{sex},address=#{address}
        where id= #{id};
    </update>
```
测试类
``` java
    //插入用户
    @Test
    public void test1() throws IOException {
        User user = new User("dgqg1232", "2", new Date(), "哈尔滨");
        int affectRow = session.insert("insertUser", user);
        session.commit();//事务提交
        System.out.println("受影响行数" + affectRow);
    }

    //删除用户
    @Test
    public void test2() throws IOException {
        int affectRow = session.delete("deleteUser", 10);
        session.commit();//事务提交
        System.out.println("受影响行数" + affectRow);
    }

    //修改用户
    @Test
    public void test3() throws IOException {
        User user = new User();
        user.setId(11);
        user.setSex("1");
        user.setAddress("深圳");
        int affectRow = session.update("updateUser", user);
        session.commit();//事务提交
    }
```
### 插入数据实现主键自增
映射文件
``` xml
    <!--
        [selectKey标签]：通过select查询来生成主键
        [keyProperty]：指定存放生成主键的属性
        [resultType]：生成主键所对应的Java类型
        [order]：指定该查询主键SQL语句的执行顺序，相对于insert语句
        [last_insert_id]：MySQL的函数，要配合insert语句一起使用 -->
    <!-- 插入用户设置id实现id自增 -->
    <insert id="insertUser1" parameterType="com.chen.model.User">
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into user (username,sex,birthday,address)
        value(#{username},#{sex},#{birthday},#{address});
    </insert>
```
测试类
``` java
   //插入用户设置id实现id自增
    @Test
    public void test4() throws IOException {
        User user = new User("chenq", "2", new Date(), "哈尔滨");
        int affectRow =session.insert("insertUser1",user);
        session.commit();
        System.out.println(affectRow + " " + user.getId());
    }
```

### 关于主键为UUID
``` xml
<!-- 注意UUID主键定义 resultType="String" order="BEFORE" -->
<insert id="insertUser" parameterType="com.gyf.domain.User">
		<selectKey keyProperty="id" resultType="String" order="BEFORE">
			SELECT UUID()
		</selectKey>
		INSERT INTO USER (username,sex,birthday,address) 
		VALUES(#{username},#{sex},#{birthday},#{address})
</insert>
```
### orcal主键 
``` sql
SELECT user_seq.nextval() FROM dual
```



## Mybatis DOA编写
### 演示案列
UserDaoImpl.class
``` java
public class UserDaoImpl implements UserDao {

    private SqlSessionFactory ssf;

    public UserDaoImpl(SqlSessionFactory ssf) {
        this.ssf = ssf;
    }

    @Override
    public void save(User user) {
        //获取seesion
        SqlSession session = ssf.openSession();
        session.insert("insertUser",user);
        session.commit();
        session.close();
        return;
    }

    @Override
    public User findUserByID(int id) {
        //获取session
        SqlSession session = ssf.openSession();
        User user = session.selectOne("findUserById",id);
        session.close();
        return user;
    }
}
```
测试类调用DAO
``` java
public class Demo3 {
    SqlSession session;
    SqlSessionFactory SessionFactory;

    @Before
    public void before() throws IOException {
        //读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //通过SqlSessionFactoryBUidler创建SqlSessionFactory会话工厂
        SessionFactory = new SqlSessionFactoryBuilder().build(is);
    }
    @Test
    public void test1() throws IOException {
        UserDao ud = new UserDaoImpl(SessionFactory);
        User user = ud.findUserByID(14);
        User user2 = new User("dgqg","1",new Date(),"北京");
        ud.save(user2);
        System.out.println(user);

    }
}
```
### 通过Mapper代理方式实现DAO （取代UserDao和UserImpl 开发中实现方式）
Mapper代理的开发方式，程序员只需要编写mapper接口（相当于dao接口）即可。  Mybatis会自动的为mapper接口生成动态代理实现类。  
不过要实现mapper代理的开发方式，需要遵循一些开发规范。 

#### 开发规范
1. mapper接口的全限定名要和mapper映射文件的namespace的值相同。
2. mapper接口的方法名称要和mapper映射文件中的statement的id相同；
3. mapper接口的方法参数只能有一个，且类型要和mapper映射文件中statement的parameterType的值保持一致。
4. mapper接口的返回值类型要和mapper映射文件中statement的resultType值或resultMap中的type值保持一致；

通过规范式的开发mapper接口，可以解决原始dao开发当中存在的问题：模板代码已经去掉；剩下去不掉的操作数据库的代码，其实就是一行代码。这行代码中硬编码的部分，通过第一和第二个规范就可以解决。

#### 需要记住的
通过session.getMappio

#### 编写步骤
1. 创建`UserMapper.java` 接口
``` java
package com.chen.mapper;

import com.chen.model.User;

public interface UserMapper {
    public int save(User user);
    public User findUserByID(int id);

}
``` 
2. 创建 `UserMapper.xml` 映射文件 (xml头可参考user.xml)
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.chen.mapper.UserMapper">
    <insert id="save" parameterType="com.chen.model.User">
        insert into user (username,sex,birthday,address)
        value(#{username},#{sex},#{birthday},#{address});
    </insert>
    <select id="findUserByID" parameterType="int" resultType="com.chen.model.User">
        select * from user where id=#{id};
    </select>
</mapper>
```
3. `SqlMapConfig.xml` 全局映射文件中加入UserMapper.xml
``` xml
 <mappers>
        <!--<mapper resource="com/chen/sqlmap/User.xml" />-->
        <mapper resource="com/chen/mapper/UserMapper.xml"/>
    </mappers>
```
4. 编写测试类测试
package com.chen.test;

``` java
import com.chen.dao.UserDao;
import com.chen.dao.UserDaoImpl;
import com.chen.mapper.UserMapper;
import com.chen.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;

public class Demo4 {
    SqlSession session;

    @Before
    public void before() throws IOException {
        //读取配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //通过SqlSessionFactoryBUidler创建SqlSessionFactory会话工厂
        SqlSessionFactory SessionFactory = new SqlSessionFactoryBuilder().build(is);
        session = SessionFactory.openSession();
    }

    @After
    public void after() throws IOException {
        session.close();
    }

    @Test
    public void test1() throws IOException {
        UserMapper um = session.getMapper(UserMapper.class);
        User user = new User("mapper","1",new Date(),"上海");
        um.save(user);
        session.commit();
        um.findUserByID(14);


    }
}
```

## 全局配置文件的相关配置
### 配置数据库信息
src下创建db.properties 用于存放数据库配置（注意属性后面不要有空格）
``` properties
driverClass=com.mysql.jdbc.Driver
url=jdbc:mysql://123.56.25.134:3306/mybastisday1?useUnicode=true&amp;characterEncoding=utf8 
username=root 
password=123456 
```
在全局配置文件中加入 `<properties resource="db.properties" />`并使用`${}`引用属性
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="db.properties"/>

    <!-- 配置mybatis的环境信息 默认开发环境-->
    <environments default="development">
        <environment id="development">
            <!-- 配置JDBC事务控制，mybastis进行管理-->
            <transactionManager type="JDBC"/>
            <!-- 配置数据源，采用JDBC连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="${driverClass}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/chen/sqlmap/User.xml" />
        <mapper resource="com/chen/mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```
### setting(了解)
这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。下表描述了设置中各项的意图、默认值等。
全局映射文件中，可以使用setting标签对mybatis相关的设置
```
<configuration>
    <settings>
        <setting name="" value="" />
    </settings>
</configuration>
```
设置参数|描述|有效值|默认值
---|---|---|---
cacheEnabled|全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。|true / false|TRUE
lazyLoadingEnabled|延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态。|true / false|FALSE
aggressiveLazyLoading|当开启时，任何方法的调用都会加载该对象的所有属性。否则，每个属性会按需加载（参考lazyLoadTriggerMethods).|true / false|false (true in ≤3.4.1)
multipleResultSetsEnabled|是否允许单一语句返回多结果集（需要兼容驱动）。|true / false|TRUE
useColumnLabel|使用列标签代替列名。不同的驱动在这方面会有不同的表现， 具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。|true / false|TRUE
useGeneratedKeys|允许 JDBC 支持自动生成主键，需要驱动兼容。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby）。|true / false|FALSE
autoMappingBehavior|指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。|NONE, PARTIAL, FULL|PARTIAL
autoMappingUnknownColumnBehavior|指定发现自动映射目标未知列（或者未知属性类型）的行为。NONE: 不做任何反应 WARNING: 输出提醒日志 ('org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN)FAILING: 映射失败 (抛出 SqlSessionException)|NONE, WARNING, FAILING|NONE
defaultExecutorType|配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。|SIMPLE REUSE BATCH|SIMPLE
defaultStatementTimeout|设置超时时间，它决定驱动等待数据库响应的秒数。|任意正整数|Not Set (null)
defaultFetchSize|为驱动的结果集获取数量（fetchSize）设置一个提示值。此参数只可以在查询设置中被覆盖。|任意正整数|Not Set (null)
safeRowBoundsEnabled|允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为false。|true / false|FALSE
safeResultHandlerEnabled|允许在嵌套语句中使用分页（ResultHandler）。如果允许使用则设置为false。|true / false|TRUE
mapUnderscoreToCamelCase|是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。|true / false|FALSE
localCacheScope|MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。|SESSION / STATEMENT|SESSION
jdbcTypeForNull|当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。|"JdbcType 常量. 大多都为: NULL| VARCHAR and OTHER"|OTHER
lazyLoadTriggerMethods|指定哪个对象的方法触发一次延迟加载。|用逗号分隔的方法列表。|"equals|clone|hashCode|toString"
defaultScriptingLanguage|指定动态 SQL 生成的默认语言。|一个类型别名或完全限定类名。|org.apache.ibatis.scripting.xmltags.XMLLanguageDriver
defaultEnumTypeHandler|指定 Enum 使用的默认?TypeHandler?。 (从3.4.5开始)|一个类型别名或完全限定类名。|org.apache.ibatis.type.EnumTypeHandler
callSettersOnNulls|指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这对于有 Map.keySet() 依赖或 null 值初始化的时候是有用的。注意基本类型（int、boolean等）是不能设置成 null 的。|true / false|FALSE
returnInstanceForEmptyRow|当返回行的所有列都是空时，MyBatis默认返回null。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集 (i.e. collectioin and association)。（从3.4.2开始）|true / false|FALSE
logPrefix|指定 MyBatis 增加到日志名称的前缀。|任何字符串|Not set
logImpl|指定 MyBatis 所用日志的具体实现，未指定时将自动查找。|SLF4J / LOG4J / LOG4J2 / JDK_LOGGING / COMMONS_LOGGING / STDOUT_LOGGING / NO_LOGGING|Not set
proxyFactory|指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具。|CGLIB / JAVASSIST|JAVASSIST (MyBatis 3.3 or above)
vfsImpl|指定VFS的实现|自定义VFS的实现的类全限定名，以逗号分隔。|Not set
useActualParamName|允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的工程必须采用Java 8编译，并且加上-parameters选项。（从3.4.1开始）|true / false|TRUE
configurationFactory|指定一个提供Configuration实例的类。 这个被返回的Configuration实例用来加载被反序列化对象的懒加载属性值。 这个类必须包含一个签名方法static Configuration getConfiguration(). (从 3.2.3 版本开始)|类型别名或者全类名.|Not set

### typeAliases别名
#### MyBatis 为我们定义的别名
别名|映射文件的类型|
---|---
_byte|byte
_long|long
_short|short
_int|int
_integer|int
_double|double
_float|float
_boolean|boolean
string|String
byte|Byte
long|Long
short|Short
int|Integer
integer|Integer
double|Double
float|Float
boolean|Boolean
date|Date
decimal|BigDecimal
bigdecimal|BigDecimal

#### 自定义别名
使用mapper接口的全限定名和包的方式mapper接口和mapper映射文件要名称相同，且放到同一个目录下
在全局映射配置文件中定义别名
``` xml
    <typeAliases>
        <方式一>
        <typeAlias type="com.chen.model.User" alias="user" />
        <方式二 定义整个包 别名是类名首字母小写>
        <package name="com.chen.model" />
    </typeAliases>
```
在mapper.xml中使用别名
``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.chen.mapper.UserMapper">
    <select id="findUserByID" parameterType="int" resultType="user">
        select * from user where id=#{id};
    </select>
</mapper>
```
### mapper 加载映射文件
加载映射文件的几种方式
 ``` xml
<mppers>
    <!--第一种-->
    <mppers resource="com/chen/sqlmap/UserMapper.xml" />
    <!--第二种 mapper接口的全限定名-->
    <!--这种方式可以不写后缀.xml 但是要使用mapper接口注解开发-->
    <mapper class="com.chen.sqlmap.userMapper.xml" />
    <!--第三种 不使用-->
    <mapper url="file:///D:\workspace_spingmvc\mybatis_01\com\chen\sqlmap\User.xml" />
    <!--第四种 推荐这种,注册包下所有映射文件，别名使用映射文件名首字母小写 -->
    <mapper package="com.chen.sqlmap" />
</mppers>
```

### 使用注解开发mapper接口（不需要mapper.xml映射文件）
![20190228090651.png](20190228090651.png)

### Mybatis 的映射文件
#### 输入映射prameterType 
ParameterType参数可以为简单类型、POJO对象、HashMap。
1. 传递简单类型
``` xml
<select id="findUserById" parameterType="int" resultType="user">
</select>
```
2. 传递POJO对象
``` xml
<select id="findUserById" parameterType="user" resultType="user">
</select>
```
3. 传递POJO包装对象
由于我们只能传递一个简单类型或一个对象，所以当我们需要传入多个参数时候就需要建立个包装类，将需要传入多个参数包装到一个类中。
``` java
public class UserQueryVO {
    private User user;
    private Orders orders;
    puglic void getUser(){
        return user;
    }
    public void setUser(User user){
        tihs.user =user;
    }
    puglic void getorders(){
        return orders;
    }
    public void setUser(Orders orders){
        tihs.orders =orders;
    }
}
```
在mapper.java接口加入使用包装对象的方法
``` java
public List<User> fingUserList（UserQueryVO userQueryVO);
```
在mapper.xml映射文件使用包装对象
``` xml
<select id="findUserById" parameterType="UserQueryVO" resultType="user">
select * from user u,order o where u.id=o.userid and u.sex=#{user.sex} and o.id = #{orders.id}
</select>
```
4. 传递map对象
在mapper.java接口加入使用map的方法
``` java
public List<User> fingUserListByMap（Map<String,Object> map);
```
在mapper.xml映射文件使用map,注意parameterType值为`hashmap`
``` xml
<select id="findUserById" parameterType="hashmap" resultType="user">
select * from user where sec=#{sex} and username =#{username}
</select>
```
测试
``` java
@Test
public void test1() throws Exception {
    SqlSession session = ssf.openSession();
    UserMapper userMapper = session.getMapper(UserMapper.class);
    Map<String,Objext> map = new HashMap<String,object>();
    map.put("sex","女")
    map.put("username","chen");
    list<User> list =userMapper.findUserListByMap(map);
    session.close();
}
```
相关词解释:  
vo:键值对对象，相对于key values  
po:persist object 持久化对象(模型对象) 
pojo:简单的java对象
entity:实体(Hibernate叫法) 也是指模型对象

#### 输出映射resultType/resultMap
1. resultType 
当查询结果字段与pojo模型对象属性一致时候使用resultTpye,适用简单类型,pojo对象,pojo对象集合
2. resultMap
当查询结果的字段与pojo模型对象不一致时候,使用resultMap,
定义mapper接口
```
    public List<User> selectAllUserByResultMap();
```
编写mapper映射文件
``` xml
    <resultMap id="userResultMap" type="user">
        <id property="id" column="uid"/>
        <result property="username" column="name"/>
        <result property="birthday" column="birday"/>
        <result property="sex" column="s"/>
        <result property="address" column="addr"/>
    </resultMap>
    <select id="selectAllUserByResultMap" resultMap="userResultMap">
        select id uid,username name,birthday birday,sex s,address addr from user;
    </select>
```
测试
``` java
  @Test
    public void test() throws IOException {
        UserMapper um = session.getMapper(UserMapper.class);
        List<User> list= um.selectAllUserByResultMap();
        System.out.println(list);
    }
```

## 动态sql
### if和where使用
``` xml
    <select id="findActiveUserNameLike" parameterType="user"
            resultType="user">
        SELECT * FROM user
        <where>
            <if test="username !=null and username!=''">
                USERNAME = #{username}
            </if>
            <if test="sex !=null and sex!=''">
                and sex = #{sex}
            </if>
        </where>
    </select>
```

### sql片段
定义提取where标签中的内容 定义为sql片段，需要时候引用
``` xml
    <sql id="select_user_where">
        <if test="username !=null and username!=''">
            USERNAME = #{username}
        </if>
        <if test="sex !=null and sex!=''">
            and sex = #{sex}
        </if>
    </sql>
    <select id="findActiveUserNameLike" parameterType="user"
            resultType="user">
        SELECT * FROM user
        <where>
            <include refid="select_user_where" />
        </where>
    </select>
```
### foreach遍历
案例：查询多个指定id的用户
`select * from user where id in (18,20,14 );`
映射接口
``` java
    public List<User> findUserByIds(List<Integer> ids);
```
映射文件
``` xml
    <select id="findUserByIds" parameterType="list" resultType="user">
        select * from user
        <where>
            <!--
            collection:集合属性
            item：遍历接受的遍历 for(Integer id : ids)
            open:遍历开始
            close：遍历结束
            separator：拼接
            -->
            <if test="list !=null and list.size > 0">
                <foreach collection="list" item="id" open="id in(" close=")" separator=",">
                    #{id}
                </foreach>
            </if>
        </where>
    </select>
```
测试文件
``` java
    @Test
    public void test3() throws IOException {
        UserMapper userMapper = session.getMapper(UserMapper.class);
        List<Integer> ids = new ArrayList<Integer>();
        ids.add(18);
        ids.add(20);
        ids.add(14);
        List<User> list = userMapper.findUserByIds2(ids);
        System.out.println(list);
    }
```

## MyBatis与Hibernate区别【面试题】
### Mybatis技术特点：
#### 优点
1. 通过直接编写SQL语句，可以直接对SQL进行性能的优化；
2. 学习门槛低，学习成本低。只要有SQL基础，就可以学习mybatis，而且很容易上手；
3. 由于直接编写SQL语句，所以灵活多变，代码维护性更好。
#### 缺点
1. 不能支持数据库无关性，即数据库发生变更，要写多套代码进行支持，移植性不好。
### Hibernate技术特点：
#### 优点  
1. 标准的orm框架，程序员不需要编写SQL语句。
2. 具有良好的数据库无关性，即数据库发生变化的话，代码无需再次编写。以后,mysql数据迁移到oracle，只需要改方言配置
#### 缺点
1. 学习门槛高，需要对数据关系模型有良好的基础，而且在设置OR映射的时候，需要考虑好性能和对象模型的权衡。
2. 程序员不能自主的去进行SQL性能优化。

### Mybatis应用场景：
需求多变的互联网项目，例如电商项目。
### Hibernate应用场景
需求明确、业务固定的项目，例如OA项目、ERP项目等。



