---
title: MyBatis.2.关联查询、延时加载、查询缓存、Spring整合与逆向工程
date: 2019-03-03 09:49:42
tags: Mybatis
categories: 
    - Java
    - 框架
---
## 关联查询
### 一对一
我们可以使用模型中加入另一个模型属性的方式，也可以使用扩展类的方式，这里我们使用模型嵌套模型的方式。
案例：查找某个订单包括地址姓名
1. 在orders.java 加入User属性以及get、set方法
2. 在OrdersMapper.java 接口中添加`findOrderById(int id)` 根据订单id查找订单信息以及用户信息的方法
3. 编写 OrderMapper.xml 中映射相关属性和映射sql代码
``` xml
<resultMap id="ordersReslMap" type="orders">
   <id column="id" property="id" />
   <id column="number" property="number" />
   <id column="createtime" property="createTime" />
   <id column="note" property="note" />
<association property="user" javaType="user">
    <id column="user_id" property="id"/>
    <id column="username" property="username" />
    <id column="address" property="address" />
</association>
</resultMap>
    <select id="findOrderById" parameterType="int" resultMap="ordersReslMap">
        select
            o.*,
            u.username,
            u.address
        from
            orders o,
            user u
        where
            o.user_id = u.id
            and o.id= #{id}
    </select>
```
4. 测试
``` java
    @Test
    public void test2() throws IOException {

        OrdersMapper ordersMapper = session.getMapper(OrdersMapper.class);
        Orders orders= ordersMapper.findOrderById(3);
        System.out.println(orders);

        session.commit();

    }
```


### 一对多
案例：根据订单Id查找订单的信息、用户信息和订单商品明细
1. 在orders模型中加入用于存放订单商品的属性` private List<OrderDetail> orderDetails;` 并生成get\set方法
2. OrdersMapper.java 模型映射接口中定义 `findOrderDetailById(int id) ` 方法
3. 在OrdersMapper.xml 配置映射信息
``` xml
   <resultMap id="ordersDetailReslMap" type="orders">
        <id column="id" property="id" />
        <result column="number" property="number" />
        <result column="createtime" property="createTime" />
        <result column="note" property="note" />
        <association property="user" javaType="user">
            <id column="user_id" property="id"/>
            <result column="username" property="username" />
            <result column="address" property="address" />
        </association>
        <collection property="orderDetails" ofType="orderDetail">
            <id column="detail_id" property="id"/>
            <result column="items_id" property="items_id"/>
            <result column="items_num" property="items_num"/>
        </collection>
    </resultMap>
    <select id="findOrderDetailById" parameterType="int" resultMap="ordersDetailReslMap">
        select
            o.*,
            u.username,
            u.address,
            od.id detail_id,
            od.items_id,
            od.items_num
        from
            orders o,
            user u ,
            orderdetail od
        where
            o.user_id =u.id
            and o.id =od.orders_id
            and o.id=#{id}
    </select>
```
### 多对多
多对多操作主要是在mapper映射配置文件中使用标签嵌套实现
案例：查找所有用户信息和用户所购买商品明细
1. User模型嵌套order模型数组、order模型嵌套orderDetails模型数组、orderDetails嵌套items模型
2. 在orders模型中加入用于存放订单商品的属性` private List<OrderDetail> orderDetails;` 并生成get\set方法
3. 在UserMapper.xml 中定义映射
``` xml
    <!-- ==============查询用户信息及用户购买的商品信息============-->
    <resultMap id="userRslMap" type="user">
        <!-- 1.匹配user属性 -->
        <id column="id" property="id"></id>
        <result column="username" property="username"/>
        <result column="password" property="password"/>

        <!--2.匹配user的orderList-->
        <collection property="ordersList" ofType="orders">
            <id column="order_id" property="id"></id>
            <result column="number" property="number"/>
            <result column="createTime" property="createTime"/>
            <result column="note" property="note"/>

            <!-- 3.匹配Orders里有orderDetails-->
            <collection property="orderDetails" ofType="orderDetail">
                <id column="detail_id" property="id"></id>
                <result column="items_id" property="itemsId"/>
                <result column="items_num" property="itemsNum"/>

                <!-- 4.配置定单详情的商品信息-->
                <association property="items" javaType="items">
                    <id column="items_id" property="id"/>
                    <result column="name" property="name"/>
                    <result column="price" property="price"/>
                    <result column="detail" property="detail"/>
                </association>
            </collection>
        </collection>
    </resultMap>

    <select id="findUserAndOrderInfo" resultMap="userRslMap">
        SELECT
            u.id,
            u.username,
            u.address,
            o.id order_id,
            o.number,
            o.createtime,
            o.note,
            od.id detail_id,
            od.items_id,
            od.items_num,
            it.name,
            it.price,
            it.detail
        FROM
            user u,
            orders o,
            orderdetail od,
          items it
        WHERE
            o.user_id = u.id
          AND o.id = od.orders_id
          AND od.items_id = it.id
    </select>
```
4. 测试
``` java
   @Test
    public void test4() throws IOException {
        UserMapper userMapper = session.getMapper(UserMapper.class);
        List<User> users = userMapper.findUserAndOrderInfo();
        for (User user:users) {
            System.out.println("用户信息:"+ user);
            for (Orders orders:user.getOrdersList()){
                System.out.println("订单信息："+orders);
                for (OrderDetail o:orders.getOrderDetails()){
                    System.out.println("商品信息："+o);
                }
            }
            System.out.println("-----------------------------------------------------");
        }
```
## 延时加载
延迟加载又叫懒加载，也叫按需加载。也就是说先加载主信息，在需要的时候，再去加载从信息。
在mybatis中，resultMap标签 的association标签和collection标签具有延迟加载的功能。  
关联的对象或集合在使用时候才去数据库中查询数据。

### 延时加载案例
全局映射配置文件 定义允许延时加载
``` xml
    <!--配置允许懒加载-->
    <settings>
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
```
关联的对象或者集合 在mapper映射文件中使用` <collection property="user" select="com.chen.mapper.UserMapper.findUserById" column="user_id" />` 
``` xml
  </select>
    <resultMap id="orderByLazyloadingResultMap" type="orders">
        <id column="id" property="id"></id>
        <result column="note" property="note"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createTime"/>
        <collection property="user" select="com.chen.mapper.UserMapper.findUserById" column="user_id" />

    </resultMap>
    <select id="findOrderAndUserByLazyloading" resultMap="orderByLazyloadingResultMap">
        select * from orders;
    </select>

```

## 查询缓存
### MyBatis的缓存理解
Mybatis的缓存，包括一级缓存和二级缓存，一级缓存是默认使用的。二级缓存需要手动开启。
一级缓存和二级缓存是map集合存放在各自的缓存空间中key是由sql语句、条件、statement等信息组成一个唯一值。缓存中的value，就是查询出的结果对象
![1.png](1.png)

### 一级缓存 SqlSession
一级缓存是SqlSession级别 一级缓存是一个map集合 存放在SqlSession数据区域中（缓存区）一级缓存默认开启
同一个SqlSession中 当我们查询查询过的数据，会不再查询数据库，通过之前查询的缓存来得到数据  
当我们对一个数据进行删除、修改、新增时候会清除缓存区中的数据，这时我们再次查询数据就会重新到数据库中查询  

案例
``` java
    @Test
    public void test1() throws IOException {

        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");
        //创建会话工程
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
        //创建SqlSession 会话
        SqlSession session = sessionFactory.openSession();

        //通过会话获取dao接口
        UserMapper userMapper = session.getMapper(UserMapper.class);
        User user = userMapper.findUserById(14);

        User user2 =userMapper.findUserById(14);
        //默认开启一级缓存
        System.out.println(user);
        System.out.println(user2);
        //通过log4j日志输出可以看到之访问过一次数据库

        userMapper.saveUser(new User("黎明","1",new Date(),"北京"));
        session.commit();
        //通过log4j日志输出可以看到当我们对数据库进行插入操作后 重新执行了select语句，缓存被删除了
        User user3 =userMapper.findUserById(14);
        System.out.println(user3);
        session.close();
    }
```

### MyBatis自带的二级缓存
二级缓存是SessionFactory级别(namespace) 同样也是一个map集合，存放在二级缓存区中，二级缓存需要手动开启  
多个SqlSession 可以共享所查询的对象所生成的缓存数据  
同样我们对数据库进行了删除、修改、新增操作时候会清除二级缓存中的数据  
二级缓存需要session关闭后才会写入二级缓存区  
1. 需要在全局配置文件中定义二级缓存开启
``` xml
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
```
2. 在mapper映射文件中使用`<cache></cache>` 标签设置使用哪种缓存机制
``` xml
  <!--默认使用MyBatis的perpetaulCache缓存机制（永久缓存） type属性可以定义需要使用的缓存机制-->
    <cache></cache>
```
3. 案例
``` java
   @Test
    public void test2() throws IOException {

        InputStream is = Resources.getResourceAsStream("MyBatisConfig.xml");
        //创建会话工程
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
        //创建SqlSession 会话
        SqlSession session1 = sessionFactory.openSession();
        SqlSession session2 = sessionFactory.openSession();
        SqlSession session3 = sessionFactory.openSession();

        //通过会话获取dao接口
        UserMapper userMapper1 = session1.getMapper(UserMapper.class);
        User user = userMapper1.findUserById(14);
        System.out.println(user);
        //注意 需要关闭session后才提交到二级缓存中
        session1.close();

        UserMapper userMapper2 = session2.getMapper(UserMapper.class);
        User user2 = userMapper2.findUserById(14);
        //没有查询数据库操作，使用缓存获得
        System.out.println(user2);

        UserMapper userMapper3 = session3.getMapper(UserMapper.class);
        userMapper3.saveUser(new User("刘德华", "1", new Date(), "香港"));
        session3.commit();
        //当我们使用session3 进行新增用户操作后 缓存失效了
        User user3 = userMapper2.findUserById(14);

        System.out.println(user3);
        session2.close();
        session3.close();
    }
```

### 整合第三方缓存框架
Mybatis本身是一个持久层框架，它不是专门的缓存框架，所以它对缓存的实现不够好，不能支持分布式。  
我们以整合Ehcache为例 Ehcache是一个分布式的缓存框架。
1. 什么是分布式  
系统为了提高性能，通常会对系统采用分布式部署（集群部署方式） 
集群部署方式对多台服务器中的缓存数据进行集中管理，所以我们使用支持分布式缓存的框架例如（**Redis**、Memcached、Ehcache）

2. 整合思路
Cache是一个接口，缓存框架都会根据这个接口来实现相关方法  
Mybatis的Cache默认实现类是PerpetualCache  
如果想整合mybatis的二级缓存，那么实现Cache接口即可  

### 整合Ehcache
1. 导入MyBatis所对应的Ehcachejar包
`mybatis-ehcache-1.0.2.jar` 和`mybatis-ehcache-1.0.2.jar`

2. 在mapper映射文件中使用`<cache></cache>` 标签设置使用ehcache
``` xml
  <!--默认使用MyBatis的perpetaulCache缓存机制（永久缓存） type属性可以定义需要使用的缓存机制-->
      <cache type="org.mybaits.caches.ehcache.EhcacheCache"></cache>
```
3. Ehcache压缩包中找到核心配置文件拷入到项目src下(更名为ehcache.xml)
`ehcache2.6.5.zip\ehcache2.6.5\ehcache-core-2.6.5\ehcache-failsafe.xml `
``` xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
    <diskStore path="java.io.tmpdir"/>
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
</ehcache>
```
4. 配置文件参数  

参数|描述
---|---
maxElementsInMemory |设置基于内存的缓存中可存放的对象最大数目 
eternal|设置对象是否为永久的,true表示永不过期,此时将忽略
timeToIdleSeconds|默认值是false 设置对象空闲最长时间,以秒为单位, 超过这个时间,对象过期。当对象过期时,EHCache会把它从缓存中清除。如果此值为0,表示对象可以无限期地处于空闲状态。 
timeToLiveSeconds|默认值是false 设置对象生存最长时间,超过这个时间,对象过期。如果此值为0,表示对象可以无限期地存在于缓存中. 该属性值必须大于或等于 timeToIdleSeconds 属性值 
overflowToDisk|设置基于内在的缓存中的对象数目达到上限后,是否把溢出的对象写到基于硬盘的缓存中 
diskPersistent|当jvm结束时是否持久化对象 true false 默认是false
diskExpiryThreadIntervalSeconds|指定专门用于清除过期对象的监听线程的轮询时间
memoryStoreEvictionPolicy|当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）


### 二级缓存应用场景
使用场景：对于访问响应速度要求高，但是实时性不高的查询，可以采用二级缓存技术。  
注意：在使用二级缓存的时候，要设置一下刷新间隔（cache标签中有一个flashInterval属性）来定时刷新二级缓存，这个刷新间隔根据具体需求来设置，比如设置30分钟、60分钟等，单位为毫秒。  
### 局限性
Mybatis二级缓存对细粒度的数据，缓存实现不好。  
场景：  
对商品信息进行缓存，由于商品信息查询访问量大，但是要求用户每次查询都是最新的商品信息，此时如果使用二级缓存，就无法实现当一个商品发生变化只刷新该商品的缓存信息而不刷新其他商品缓存信息，因为二级缓存是mapper级别的，当一个商品的信息发送更  新，所有的商品信息缓存数据都会清空。  
解决此类问题，需要在业务层根据需要对数据有针对性的缓存。  

比如可以对经常变化的 数据操作单独放到另一个namespace的mapper中。  

### 禁用指定方法的二级缓存
在xml映射文件中使用`useCache="false"`属性来禁用方法使用二级缓存
``` xml
<select id = "findUserbById" parameterType"int" resultType="user" useCache="false">
    select * form User where id=#{?}
</select>
```

### 指定方法执行后不刷新缓存
在xml映射文件中使用` flashCache="true"`属性使方法执行后刷新缓存
``` xml
<insert id = "save" parameterType"User" flashCache="true">
    insert into User(username,sex,birthday,address)
        value(#{username}，#{sex},#{birthday},#{address})
</insert>
```

## 逆向工程 生成model、配置文件和java类
Mybatis 官方提供了逆向工程
可以针对数据库中的表自动生成mybatis代码（mapper.java、mapper.xml、po类）
下载地址：https://github.com/mybatis/generator/releases/tag/mybatis-generator-1.3.2

### 使用方法
1. 创建java工程，拷入下载的jar包和数据库驱动包
2. 创建generator配置文件
创建generator.xml 配置文件，配置文件内容可以从文档中获得  http://www.mybatis.org/generator/index.html  
![2.png](2.png)
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--数据库配置-->
    <context id="MySqlTables" targetRuntime="MyBatis3">
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://123.56.25.134:3306/mybastisday1"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!--java类型解析-->
        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!--模型生成包名-->
        <javaModelGenerator targetPackage="com.chen.model" targetProject=".\src">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!--MyBatis映射.xml-->
        <sqlMapGenerator targetPackage="com.chen.mapper"  targetProject=".\src">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!--Mybatis 映射接口-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.chen.mapper"  targetProject=".\src">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!--配置生成的模型表,生成的模型首字母会自动大写-->
        <table tableName="items"></table>
        <table tableName="user"></table>
        <table tableName="orders"></table>
        <table tableName="orderdetail" domainObjectName="OrderDetail"></table>

    </context>
</generatorConfiguration>

```
3. 使用java类来执行逆向工程
``` java
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) throws Exception {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("generator.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
                callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```
