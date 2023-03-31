[TOC]

## JDBC

### 概念

- JDBC是使用Java语言操作关系型数据库的一套API

- Java DataBase Connectivity

  

### 步骤

0. 创建工程，导入jar包

1. 注册驱动

   ```java
   Class.forName("com.mysql.jdbc.Driver");
   ```

2. 获取连接

   ```java
   String url = "jdbc:mysql://127.0.0.1:3306/db1";
   String username = "root";
   String password = "password";
   Connection conn = DriverManager.getConnection(url,username,password);
   ```

3. 定义SQL语句

   ```java
   String sql = "update account set money = 2000 where id = 1";
   ```

4. 获取执行SQL对象

   ```java
   Statement statement = conn.createStatement();
   ```

5. 执行SQL

   ```java
   int count = statement.executeUpdate(sql);
   ```

6. 处理返回结果

   ```java
   System.out.println(count);
   ```

7. 释放资源

   ```java
   statement.close();
   conn.close();
   ```



------




### JDBC API

- DriverManager
- Connection
- Statement
- ResultSet
- PreparedStatement



#### DriverManager

##### 	作用：

​	1. 注册驱动

​			Class.forName("com.mysql.jdbc.**Driver**");

​			**Driver**类调用DriverManager

​	2. 获取数据库连接

​			getConnection(String url,String user,String password);



#### Connection

##### 作用：

  1. 获取执行SQL的对象

     - 普遍执行SQL对象

       ```
       Statement createStatement()
       ```

     - ​	预编译SQL的执行SQL对象：防止SQL注入

       ```
       PreparedStatement prepareStatement(sql)
       ```

  2. 管理事务 (事务内的子事件要成功都成功，要失败都失败)

     <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224094327214.png" alt="image-20230224094327214" style="zoom:80%;" /> 

     <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224094243372.png" alt="image-20230224094243372" style="zoom:80%;" />  

​		

#### Statement

##### 作用：

​	1.执行SQL语句

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224095007038.png" alt="image-20230224095007038" style="zoom:80%;" /> 



#### ResultSet

##### 作用：

​	1. 封装了DQL查询语句的结果

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224100319947.png" alt="image-20230224100319947" style="zoom:80%;" /> 

​	2. 获取查询结果  

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224100534565.png" alt="image-20230224100534565" style="zoom:80%;" /> 

##### 使用步骤： 

 1. 游标向下移动一行，判断改行是否有数据：next()

 2. 获取数据: getXxx(参数)

    ```java
    //循环判断游标是否是最后一行末尾
    while(rs.next()){
    	//获取数据
    	rs.getXxx(参数);
    }
    ```

##### 案例：

将account表中的数据，以account对象封装到List中。

```java
@Test
public void testResulSet() throws Exception {
    //1.注册驱动
    //Class.forName("com.mysql.jdbc.Driver");

    //2.获取连接：如果连接的是本机mysql,且端口号是3306，则可以简写
    String url = "jdbc:mysql:///db1";
    String username = "root";
    String password = "password";
    Connection conn = DriverManager.getConnection(url,username,password);

    //3.定义sql
    String sql = "select * from account";

    //4.获取执行的对象
    Statement stat = conn.createStatement();

    //5.执行sql
    ResultSet rs = stat.executeQuery(sql);

    //创建集合
    List<Account> list = new ArrayList<>();

    //6.处理结果，遍历rs的全部数据
    //6.1 光标向下移动一行，并且判断当前行是否有数据
    while(rs.next()){
        //6.2 获取数据 getXxx()
        int id = rs.getInt("id");
        String name = rs.getString("name");
        Double money = rs.getDouble("money");

        Account account = new Account(id,name,money);
        list.add(account);
    }

    //打印集合
    System.out.println(list);

    //7.释放资源
    rs.close();
    stat.close();
    conn.close();

}
```



#### PreparedStatement

（继承自Statement）

##### 作用：

 	1. 预编译SQL语句并执行：预防SQL注入问题

##### SQL注入案例：

```sql
select * from tb_user where username = 
'zhangsan' and password = '' or '1' = '1'
```

```java
@Test
public void testResulSet() throws Exception {
    //1.注册驱动
    //Class.forName("com.mysql.jdbc.Driver");

    //2.获取连接：如果连接的是本机mysql,且端口号是3306，则可以简写
    String url = "jdbc:mysql:///db1";
    String username = "root";
    String password = "password";
    Connection conn = DriverManager.getConnection(url,username,password);

    //接收用户输入 用户名和密码
    String name = "zhangsan";
    String pwd = "' or '1' = '1";

    String sql = "select * from tb_user where username = '"+name+"'" +
            " and password = '"+pwd+"'";

    System.out.println(sql);
    //获取stat对象
    Statement stat = conn.createStatement();

    //执行sql
    ResultSet rs = stat.executeQuery(sql);

    if (rs.next()){
        System.out.println("登录成功");
    }else {
        System.out.println("登录失败");
    }

    //7.释放资源
    rs.close();
    stat.close();
    conn.close();

}
```

##### 解决SQL注入：

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224115145254.png" alt="image-20230224115145254" style="zoom:80%;" />  



##### 优化后的案例：

```java
@Test
public void testResulSet() throws Exception {
    //1.注册驱动
    //Class.forName("com.mysql.jdbc.Driver");

    //2.获取连接：如果连接的是本机mysql,且端口号是3306，则可以简写
    String url = "jdbc:mysql:///db1";
    String username = "root";
    String password = "password";
    Connection conn = DriverManager.getConnection(url,username,password);

    //接收用户输入 用户名和密码
    String name = "zhangsan";
    String pwd = "' or '1' = '1";

    String sql = "select * from tb_user where username = ? and password = ?";

    //获取pstmt对象
    PreparedStatement pstmt = conn.prepareStatement(sql);

    // 设置参数
    pstmt.setString(1,name);
    pstmt.setString(2,pwd);

    //执行sql
    ResultSet rs = pstmt.executeQuery();

    if (rs.next()){
        System.out.println("登录成功");
    }else {
        System.out.println("登录失败");
    }

    //7.释放资源
    rs.close();
    pstmt.close();
    conn.close();

}
```



##### 好处：

1. ​	预编译SQL，性能更高

   （需要将userServerPrepStmts = true）

   ![image-20230224160326527](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224160326527.png) 

2. ​    防止SQL注入：将敏感字符转义





------



### 数据库连接池

- 数据库连接池简介
- Druid数据库连接池



#### 数据库连接池简介

- **数据库连接池**是个容器，负责分配、管理数据库连接

- 它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个

#### 数据库连接池实现

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224185458912.png" alt="image-20230224185458912" style="zoom:80%;" /> 

#### Druid数据库连接池

##### 	使用步骤

>  	1. 导入jar包
>  	2. 定义配置文件
>  	3. 加载配置文件
>  	4. 获取数据库连接池对象
>  	5. 获取连接



##### 配置文件：druid.properties

```java
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql:///db1?useSSL=false&useServerPrepStmts=true
username=root
password=password
# 初始化连接数量
initialSize=5
# 最大连接数
maxActive=10
# 最大等待时间
maxWait=3000
```

##### 案例：

```java
public class DruidDemo {
    public static void main(String[] args) throws Exception {
        //1.导入jar包

        //2.定义配置文件

        //3.加载配置文件
        Properties prop = new Properties();
        prop.load(new FileInputStream("jdbc-demo/src/druid.properties"));

        //4.获取连接池对象
        DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);

        //5.获取数据库连接
        Connection connection = dataSource.getConnection();

        System.out.println(connection);

    }
}
```





------



### JDBC练习

> 完成商品品牌数据的增删改查操作

> * 查询：查询所有数据
> * 添加：添加品牌
> * 修改：根据id修改
> * 删除：根据id删除

> 准备工作：创建tb_brand表，创建Brand类，二者相互对应。

##### 	查询

```java
@Test
public void testSelectAll() throws Exception {
    //1.获取Connection
    //1.1 加载配置文件
    Properties prop = new Properties();
    prop.load(new FileInputStream("src/druid.properties"));

    //1.2 获取连接池对象
    DataSource dataSource = DruidDataSourceFactory.createDataSource(prop);
    //1.3 获取数据库连接 Connection
    Connection conn = dataSource.getConnection();

    //2. 定义SQL
    String sql = "select * from tb_brand";

    //3. 获取pstmt对象
    PreparedStatement pstmt = conn.prepareStatement(sql);
    //4. 设置参数
    //5. 执行SQL
    ResultSet rs = pstmt.executeQuery();

    Brand brand = null;
    List<Brand> brands = new ArrayList<>();

    //6.处理结果 List<Brand> 封装Brand对象
    while(rs.next()){
        //获取数据
        int id = rs.getInt("id");
        String brandName = rs.getString("brand_name");
        String companyName = rs.getString("company_Name");
        int ordered = rs.getInt("ordered");
        String description = rs.getString("description");
        int status = rs.getInt("status");

        //封装Brand对象
        brand = new Brand();
        brand.setId(id);
        brand.setBrandName(brandName);
        brand.setCompanyName(companyName);
        brand.setOrdered(ordered);
        brand.setDescription(description);
        brand.setStatus(status);

        //装载集合
        brands.add(brand);
    }
    System.out.println(brands);

    //7.释放资源
    rs.close();
    pstmt.close();
    conn.close();
}
```



##### 	添加、修改、删除

> 与上述查询功能实现类似，只需更改SQL语句，和设置SQL语句中的占位符?即可。







------

## Maven

#### 1，Maven概述               

> Maven 是专门用于管理和构建Java项目的工具，他的主要功能有：
>
> | 提供了：                                    |                                                              |
> | ------------------------------------------- | ------------------------------------------------------------ |
> | 标准化的项目结构                            | <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224220607073.png" alt="image-20230224220607073" style="zoom: 50%;" /> |
> | 标准化的构建流程 （编译，测试，打包，发布） | <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224220638619.png" alt="image-20230224220638619" style="zoom: 50%;" /> |
> | 依赖管理机制（jar包、插件）                 | <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224221336067.png" alt="image-20230224221336067" style="zoom:50%;" /><br /><br /><img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230224221611418.png" alt="image-20230224221611418" style="zoom:50%;" /> |



#### 2，简介

##### 2.1 Maven模型

> 基于项目对象模型（POM）的概念，通过一小段描述信息来管理项目的构建、报告和文档。
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230225142845409.png" alt="image-20230225142845409" style="zoom: 80%;" /> 

##### 2.2 Maven仓库

> 分类：
>
> - 本地仓库：自己计算机上的一个目录
>
> - 中央仓库：由Maven团队维护的全球唯一仓库
>
>   - 地址：https://repo1.maven.org/maven2/
>
> - 远程仓库（私服）：一般由公司团队搭建的私有仓库
>
>   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230225150603919.png" alt="image-20230225150603919" style="zoom:50%;" />     

  

#### 3，安装配置

1. 解压maven压缩包

2. 配置环境变量MAVEN_HOME 为bin目录所在路径

3. 配置本地仓库：修改conf/settings.xml中的<localRepository>为一个指定目录

4. 配置阿里云私服：修改 conf/settings.xml 中的 <mirrors>标签，为其添加如下子标签：

   ```java
   <mirror>  
       <id>alimaven</id>  
       <name>aliyun maven</name>  
       <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
       <mirrorOf>central</mirrorOf>          
   </mirror>
   ```

   

#### 4，基本使用

##### 4.1 Maven常用命令

> - compile ：编译
> - clean：清理
> - test：测试
> - package：打包
> - install：安装

##### 4.2 Maven生命周期

> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230225155003801.png" alt="image-20230225155003801" style="zoom:67%;" /> 



#### 5，Idea配置Maven

##### 5.1 配置环境

Settings  >  Build,Execution,Deployment  >  Build Tools  >  Maven

> Maven home path:
>
> User settings file:

##### 5.2 Maven坐标详解

> 坐标：
>
> - 资源的唯一标识
> - 使用坐标来定义项目或引入项目中需要的依赖

>坐标组成：
>
>- groupId: 定义当前Maven项目隶属组织名称
>
>  ​	（通常是域名反写，例如：com.itheima）
>
>- artifacId: 定义当前Maven项目名称
>
>  ​	（通常是模块名称，例如 order-service、goods-service）
>
>- version：定义当前项目版本号
>
>  ![image-20230225162656249](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230225162656249.png) 



#### 6，依赖

##### 6.1 依赖管理

> 使用坐标导入jar包
>
> > - 在pom.xml文件中编写<dependecies><dependecy>标签，引入坐标的groupId，artifactId，version（可通过https://mvnrepository.com/获取）
> >
> > - 点击刷新按钮，即可生效  /  或点击 settings > Build > buildtools > 勾选any changes 即可自动生效。
> >
>  
>使用idea ALT + INSERT 选择Dependency，搜索本地仓库的 jar 包，即可自动导入



##### 6.2 依赖范围

> 通过设置坐标的依赖范围(scope)，可以设置对应jar包的作用范围：
>
> ​										编译环境、		测试环境、	运行环境（即compile后的包）
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230225170709645.png" alt="image-20230225170709645" style="zoom:80%;" /> 
>
> 默认值为compile







------

## MyBatis

#### 1，MyBatis简介

> 一款优秀的***持久层***  ***框架***，用于<u>简化 JDBC 开发</u>
>
> > 持久层：负责将数据保存到数据库的 那一层代码
> >
> > （J2EE 三层架构：表现层、业务层、持久层）
>
> > 框架：即一个半成品软件，是一套可重用的、通用的、软件基础代码模型

##### 1.1 MyBatis简化

![image-20230315191733442](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315191733442.png) 



#### 2，MyBatis快速入门

[mybatis – MyBatis 3 | 入门](https://mybatis.org/mybatis-3/zh/getting-started.html)

##### 2.1 案例——查询user表：

> 0. 创建user表，添加数据
>
>    <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301094155556.png" alt="image-20230301094155556" style="zoom:67%;" />
>
> 1. 创建Maven模块， （pom.xml中）导入坐标
>
>    <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301100438973.png" alt="image-20230301100438973" style="zoom: 67%;" /> 
>
> 2. 编写MyBatis核心配置文件 --> 替换连接信息，解决硬编码问题
>
>    （由官网复制来，更换部分连接信息）
>
>    resources -> mybatis-config.xml
>
>    <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301110651734.png" alt="image-20230301110651734" style="zoom: 67%;" /> 
>
> 3. 编写SQL映射文件 --> 统一管理sql语句，解决硬编码问题
>
>    （由官网复制来，更换 id 和 返回类型）
>
>    resources -> UserMapper.xml
>
>    <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301110859984.png" alt="image-20230301110859984" style="zoom:67%;" />
>
> 4. 编码
>    1. 定义pojo类 （并添加setter，getter，toString方法）
>
>       <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301110944933.png" alt="image-20230301110944933" style="zoom: 50%;" />
>
>    2. MyBatisDemo.class: 加载核心配置文件，获取SqlSessionFactory对象
>
>       <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301113053469.png" alt="image-20230301113053469" style="zoom:67%;" />
>
>    3. 获取SqlSession对象，执行SQL语句
>
>       <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301113114766.png" alt="image-20230301113114766" style="zoom:67%;" /> 
>
>       <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301113253531.png" alt="image-20230301113253531" style="zoom:67%;" />
>
>    4. 释放资源
>
>       <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301113311059.png" alt="image-20230301113311059" style="zoom:67%;" /> 

##### 2.2 解决SQL映射文件的 *警告* 提示

- 产生原因：ldea和数据库没有建立连接，不识别表信息
- 解决方式：在ldea中配置MySQL数据库连接

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301163144271.png" alt="image-20230301163144271" style="zoom:80%;" />

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301163240168.png" alt="image-20230301163240168" style="zoom: 50%;" /> 

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301163257384.png" alt="image-20230301163257384" style="zoom: 50%;" />

至此，在idea中也可以像在navicat中一样对数据库进行可视化操作。

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230301163434594.png" alt="image-20230301163434594" style="zoom:50%;" />



#### 3，Mapper代理开发

##### 3.1 Mapper代理入门案例

> 1.定义与SQL映射文件同名的Mapper接口，并且将Mapper接口和SQL映射文件放置在同一（同名）目录下 
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230302143521892.png" alt="image-20230302143521892" style="zoom:80%;" />
>
> 2.设置SQL映射文件的namespace属性为Mapper接口全限定名
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230302143721426.png" alt="image-20230302143721426" style="zoom: 67%;" />
>
> 3.在 Mapper 接口中定义方法，方法名就是SQL映射文件中sql语句的id，并保持参数类型和返回值类型一致
>
> ![image-20230302145757705](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230302145757705.png)
>
> 4.编码:		![image-20230302152913465](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230302152913465.png)
>
> ​		1.通过 SqlSession 的 getMapper方法获取 Mapper接口的代理对象
>
> ​		2.调用对应方法完成sql的执行
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230302152941248.png" style="zoom: 67%;" />



#### 4，MyBatis核心配置文件

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315194512165.png" alt="image-20230315194512165" style="zoom:80%;" />

```
1. typeAliases：给 pojo 中的类取别名 ，之后不用再以包名开头，直接用类名即可，且类名不区分大小写
2. environment：配置数据库连接信息，可以配置多个environment，通过default属性切换不同的environment
```

#### 5，配置文件完成增删改查

![image-20230315204751375](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315204751375.png)





#### 6，注解完成增删改查





#### 7，动态SQL









------

## WEB基础

> W3C标准：网页主要由三部分组成
>
> - 结构：HTML
> - 表现：CSS
> - 行为：JS



### 1，HTML

#### 1.1 简介

> - HyperText Markup Language ：超文本标记语言
> - HTML标签由浏览器来解析

#### 1.2 快速入门

```html
<html>
	<head>
		<title>
			html快速入门
		</title>
	</head>
	<body>
		乾坤未定，你我皆是黑马~
	</body>
</html>
```

#### 1.3 基础标签

[HTML 标签参考手册 (w3school.com.cn)](https://www.w3school.com.cn/tags/index.asp)

| 标签        | 描述 |
| ----------- | ---- |
| <h1> - <h6> | 定义标题 |
| <font>      | 定义文本字体、尺寸、颜色     |
| <b>、<i>、<u> | 定义粗体、斜体、下划线 |
| <center>    | 定义文本居中 |
| <p>         | 定义段落 |
| <br>        | 定义折行 |
| <hr>        | 定义水平线 |

#### 1.4 图片，音频，视频标签

```html
<!--图片			尺寸单位：px像素 / 百分比 -->
<img src = "url" width = "300" height="400">
<!--音频			controls为显示播放控件-->
<audio src = "url" controls></audio>
<!--视频-->
<video src = "url" controls width = "300" height="400"></video>
```

> src:
>
> - 相对路径： ./ 	：当前目录（省略不写默认为当前目录）
>
>   ​					../	：上一级
>
> - 绝对路径

#### 1.5 超链接标签

```html
<a href="url" target="_self">百度</a>
```

> target属性：
>
> ​	_self：默认值，当前页面打开
>
> ​	_blank:空白页打开

#### 1.6 列表标签

```html
<!--有序-->
<ol>
	<li>咖啡</li>
    <li>茶</li>
</ol>
<!--无序-->
<ul>
	<li>咖啡</li>
    <li>茶</li>
</ul>
```

#### 1.7 表格标签

```html
<table border="1" cellspacing = "0" width = "500">
    <tr>
        <th>名称</th>
        <th>描述</th>
    </tr>
    <tr>
        <td>xxx</td>
        <td>xxx</td>
    </tr>
    <tr>
        <td>xxx</td>
        <td>xxx</td>
    </tr>    
</table>
```

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227115320790.png" alt="image-20230227115320790" style="zoom:50%;" /> 

> 合并单元格，使用td(th)的属性：
>
> ​	colspan: 列
>
> ​	rowspan:行

#### 1.8 布局标签

> - div：定义HTML文档中的一个区域部分，经常与CSS一起使用，用来布局网页
> - span：用于组合行内元素

```html
<div></div>
<span></span>
```

#### 1.9 表单标签

> - 表单：在网页中主要负责数据收集功能，用<form>标签定义表单
>
> - 表单项（元素）：表单中的元素，包括下拉列表、文本域等。
>
> - form：定义表单
>
>   - action：指定表单数据提交的URL
>
>   - method：指定表单数据提交的方式
>
>     - get（默认）：请求参数会拼接在URL 之后。
>
>       ​						大小有限制
>
>     - post：请求参数会在http请求协议的请求体中。
>
>       ​			大小无限制

```html
<!-- #处应填写URL，填写#指发送至当前页面 -->
<form action="#" method="get">
    <!--表单数据要想被提交，必须指定其name属性-->
    <input type="text" name="username">
    <input type="submit">
</form>
```

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227143943715.png" alt="image-20230227143943715" style="zoom: 67%;" /><img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227143955651.png" style="zoom:67%;" />



#### 1.10 表单项标签

 - input：表单项，通过type属性控制输入形式<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227151028027.png" alt="image-20230227151028027" style="zoom: 80%;" />  
 - select：定义下拉列表，<option>定义列表项
 - textarea ：文本域



案例：

```html
<form action="#" method="post">
    <input type="hidden" name="id" value="123">

    <label for="username">用户名：</label>
    <input type="text" name="username" id="username"><br>
    
    <label for="password">密码：</label>
    <input type="password" name="password" id="password"><br>
    
    性别：
    <input type="radio" name="gender" value="1" id="male"><label for="male">男</label>
    <input type="radio" name="gender" value="2" id="female"><label for="female">女</label>

    <br>
    爱好：
    <input type="checkbox" name="hobby" value="1">旅游
    <input type="checkbox" name="hobby" value="2">电影
    <input type="checkbox" name="hobby" value="3">游戏

    <br>
    头像：
    <input type="file" value="选择文件">
    <br>
    
    城市：
    <select>
        <option value="beijing">北京</option>
        <option value="shanghai">上海</option>
        <option value="guangzhou">广州</option>
    </select>
    <br>
    
    个人描述：
    <textarea cols="20" rows="5" name="xxx"></textarea>
    <br>
    
    <input type="submit" value="注册">
    <input type="reset" value="重置">
    <input type="button" value="按钮">

</form>
```

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227161506992.png" alt="image-20230227161506992" style="zoom: 67%;" />  





### 2，CSS

#### 2.1 简介

> - Cascading Style Sheet：层叠样式表
> - 用于控制网页表现的语言

#### 2.2 CSS导入方式

 1. 内联样式

    ```html
    <div style="color:red">Hello CSS~</div>
    ```

 2. 内部样式

    ```html
    <style type="text/css">
        div{
            color:blue;
        }
    </style>
    ```

    <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227163354812.png" alt="image-20230227163354812" style="zoom: 80%;" /> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227163425596.png" alt="image-20230227163425596"  />

 3. 外部样式

    ```html
    <link rel="stylesheet" href="demo.css">
    ```

    <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227163539405.png" alt="image-20230227163539405" style="zoom:80%;" /> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227163606839.png" alt="image-20230227163606839" style="zoom:80%;" />

    <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227163151357.png" alt="image-20230227163151357" style="zoom: 67%;" /> 

#### 2.3 CSS选择器

> 概念：选择器是选取需设置样式的元素（标签）

> 分类：
>
> 1. 元素选择器                   格式：元素名称{color:red;}
>
> ```css
> 	div{color:red;}
> ```
>
> 2. id选择器(选择单个)     格式：#id属性值{color:red;}
>
>    ```css
>    #name{color:red;}
>    <div id="name">hello css2</div>
>    ```
>
> 3. 类选择器(选择多个)      格式：.class属性值{color:red;}
>
>    ```css
>    .cls{color:red;}
>    <div class="cls">hello css3</div>
>    ```

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227165449581.png" alt="image-20230227165449581" style="zoom:67%;" /> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227165507493.png" alt="image-20230227165507493" style="zoom: 80%;" /><img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230227165534062.png" alt="image-20230227165534062" style="zoom:80%;" />

#### 2.4 CSS属性

[CSS 参考手册 (w3school.com.cn)](https://www.w3school.com.cn/cssref/index.asp)





### 3，JavaScript

#### 3.1 简介

> JavaScript是一门跨平台、面向对象的脚本语言，用来控制网页行为，使网页可交互。

#### 3.2 JS引入方式

1. 内部脚本

   > 将JS代码定义在html页面中
   >
   > ```javascript
   > <script>
   > 	alert("hello JS ~");
   > </script>
   > <!-- 可以放在html文档的任何地方，一般把脚本置于<body>元素的底部，可以改善显示的速度 -->
   > ```

2. 外部脚本

   > 将JS代码定义在外部JS文件中，然后引入到html页面中
   >
   > - 外部文件 demo.js 
   >
   > ```javascript
   > alert("hello JS ~");
   > ```
   >
   > - 引入.js文件
   >
   > ```javascript
   > <script src="../js/demo.js"></script>
   > ```



#### 3.3 JS基础语法

> 基本语法与Java基本相同

##### 3.3.1 输出语句

 ```javascript
 window.alert("Hello JS~");//写入警告框
 
 document.alert("Hello JS2~");//写入html页面
 
 console.alert("Hello JS3~");//写入浏览器的控制台
 ```

##### 3.3.2 变量

使用var关键字（variable）声明变量  / let / const

```javascript
var test = 20;
test = "张三";
```

##### 3.3.3 数据类型

- 原始类型：

  <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230228094759507.png" alt="image-20230228094759507" style="zoom: 67%;" />  

  使用 alert(typeof(变量)); 查看变量类型

- 引用类型

##### 3.3.4 运算符

与java相同。唯一不同的是 

==：先判断类型是否一样，如果不一样，则进行类型转换，再去比较

===：先判断类型是否一样，如果不一样，则返回false，再去比较

##### 3.3.5 类型转换

- 其他类型转换为number：
  1. string：按照字符串的字面值，转换为数字，如果字面值不是数字，则转为NAN
  2. boolean：true转为1，false转为0
- 其它类型转为boolean：
  1. number：0和NAN为false，其他的数字为true
  2. string：空字符串为false，其他的字符串为true
  3. null：false
  4. undefined：false

#####  3.3.6 流程控制语句

> 与 java 相同

##### 3.3.7 函数

> 被设计为执行特定任务的代码块
>
> *js 是弱类型语言，形式参数和返回值不需要指定类型*
>
> - 定义：
>
>   ```javascript
>   function add(a，b){
>   	return a + b;
>   }
>   //或者
>   var add = function add(a,b){
>       return a + b;
>   }
>   ```
>
>   js 中，函数调用可以调用任意个参数，但只匹配对应的。
>
> - 调用：
>
>   ```javascript
>   let result = add(1,2);
>   let result = add(1,2,3);
>   ```



#### 3.4 JS常用对象

##### 3.4.1 Array

> - 定义：
>
>   ```javascript
>   var arr = new Array(1,2,3);
>   var arr = [1,2,3];
>   //java中这里用的是int arr = {1,2,3}
>   ```
>
> - 访问：与java相同  arr[index];
>
> - 特点：相当于JAVA中的集合，变长变类型
>
> - 属性：length
>
>   ```javascript
>   var arr = [1,2,3];
>   for(let i = 0;i < arr.length; i++){
>   	alert(arr[i]);
>   }
>   ```
>
> - 方法：
>
>   push：添加方法
>
>   splice( a , b )：从a开始删，删b个元素

##### 3.4.2 String

> 与java基本相同

> - 定义：
>
>   ```javascript
>   var str = new String("hello");
>   var str = "hello";
>   var str = 'hello';
>   ```
>
> - 属性：length()
>
> - 方法：charAt()
>
>   ​			indexOf()
>
>   ​			trim()

##### 3.4.3 自定义对象

```javascript
var person = {
	name:"zhangsan",
	age:23,
	eat.function(){
		alert("吃饭");
	}
}
//访问：
alert(person.name);
person.eat;
```

#### 3.5 BOM

> Browser Object Model 浏览器对象模型，JavaScript将浏览器的各个组成部分封装为对象。
>
> 组成：
>
> - Window：浏览器窗口对象
> - Navigator：浏览器对象
> - Screen：屏幕对象
> - History：历史记录对象
> - Location：地址栏对象

##### 3.5.1 Window

> 获取：window.function();	//window. 可以省略
>
> ```javascript
> window.alert("abc");
> ```
>
> 属性：获取其他BOM对象
>
> 方法：
>
> - alert()
>
> - confirm()：现时代有一段消息以及确认和取消按钮的对话框。
>
>   （确定返回true，取消返回false）
>
>   ```javascript
>   var flag = confirm("确认删除？");
>   if(flag){
>       //删除逻辑
>   }
>   ```
>
> - setInterval()：按照制定的周期（以毫秒计）来调用函数或计算表达式。（执行多次）
>
>   ```javascript
>   setInterval(function(){
>   	alert("hehe")
>   },2000);
>   ```
>
> - setTimeout()：在指定毫秒数后调用函数或计算表达式。（执行一次）
>
>   ```javascript
>   setTimeout(function(){
>   	alert("hehe")
>   },2000);
>   ```

##### 3.5.2 History

> - Hiistory：历史记录
>
> - 方法：
>
>   ```javascript
>   History.back();		//加载 history 列表的前一个URL
>   History.forward();	//加载 history 列表的下一个URL
>   ```

##### 3.5.3 Location

> - Location：地址栏对象
>
> - 属性：
>
>   ```javascript
>   doucment.write("3秒后跳转到首页");
>   setTimeout(function(){
>       location.href = "https://www.baidu.com";
>   },3000)
>   ```

#### 3.6 DOM

>  - Document Object Model 文档对象模型
>
>  - 将标记语言的各个组成部分封装为对象
>
>    - Document：整个文档对象
>
>    - Element：元素对象
>
>    - Attribute：属性对象
>
>    - Text：文本对象
>
>    - Comment：注释对象
>
>  - JavaScript 通过 DOM， 就能够对 HTML进行操作了
>
>    - 改变 HTML 元素的内容
>
>    - 改变 HTML 元素的样式（CSS）
>
>    - 对 HTML DOM 事件作出反应
>
>    - 添加和删除 HTML 元素

##### 3.6.1 获取Element对象

获取：使用Document对象的方法来获取

```javascript
//根据id属性值获取，返回一个Element对象
document.getElementById("id");	

//根据标签名称获取，返回Element对象数组
doucment.getElementsByTagName("div");

//根据name属性值获取，返回Element对象数组
doucment.getElementsByTagName("name");

//根据class属性值获取，返回Element对象数组
doucment.getElementsByClassName("class");
```

##### 3.6.2 常见HTML Element对象的使用

需求：

1、点亮灯泡

```javascript
//1，根据 id='light' 获取 img 元素对象
var img = document.getElementById(”light“);
//2，修改 img 对象的 src 属性来改变图片
img.src = "../imgs/on.gif";
```

2、将所有的div标签的标签体内容替换为 呵呵

```js
//1，获取所有的 div 元素对象
var divs = document.getElementsByTagName("div");
/*
        style:设置元素css样式
        innerHTML：设置元素内容
    */
//2，遍历数组，获取到每一个 div 元素对象，并修改元素内容
for (let i = 0; i < divs.length; i++) {
    //divs[i].style.color = 'red';
    divs[i].innerHTML = "呵呵";
}
```

3、使所有的复选框呈现被选中的状态

```js
//1，获取所有的 复选框 元素对象
var hobbys = document.getElementsByName("hobby");
//2，遍历数组，通过将 复选框 元素对象的 checked 属性值设置为 true 来改变复选框的选中状态
for (let i = 0; i < hobbys.length; i++) {
    hobbys[i].checked = true;
}
```



#### 3.7 事件监听

> - 事件：HTML事件是发生在HTML元素上的”事情“。比如：
>   - 按钮被点击
>   - 鼠标移动到元素之上
>   - 按下键盘按键
> - 事件监听：JavaScript可以在时间被侦测到时执行代码

##### 3.7.1 事件绑定

> - 方式一：通过HTML标签中的时间属性进行绑定
>
>   如下面代码，有一个按钮元素，我们是在该标签上定义 `事件属性`，在事件属性中绑定函数。`onclick` 就是 `单击事件` 的事件属性。
>
>   `onclick='on（）'` 表示该点击事件绑定了一个名为 `on()` 的函数
>
>     ```html
>     <input type="button" onclick='on()’>
>     ```
>
>     下面是点击事件绑定的 `on()` 函数
>
>     ```js
>     function on(){
>     	alert("我被点了");
>     }
>     ```
>
> - 方式二：通过 DOM 元素属性绑定
>
>   如下面代码是按钮标签，在该标签上我们并没有使用 `事件属性`，绑定事件的操作需要在 js 代码中实现
>
>   ```html
>   <input type="button" id="btn">
>   ```
>
>   下面 js 代码是获取了 `id='btn'` 的元素对象，然后将 `onclick` 作为该对象的属性，并且绑定匿名函数。该函数是在事件触发后自动执行
>
>   ```js
>   document.getElementById("btn").onclick = function (){
>       alert("我被点了");
>   }
>   ```

##### 3.7.2 常见事件

| 事件属性名  | 说明                     |
| ----------- | ------------------------ |
| onclick     | 鼠标单击事件             |
| onblur      | 元素失去焦点             |
| onfocus     | 元素获得焦点             |
| onload      | 某个页面或图像被完成加载 |
| onsubmit    | 当表单提交时触发该事件   |
| onmouseover | 鼠标被移到某元素之上     |
| onmouseout  | 鼠标从某元素移开         |
| onkeydown   | 某个按键被按下           |

##### 3.7.3 案例

表单验证

1、当输入框失去焦点时，验证输入内容是否符合要求。

```js
    //1.验证用户名是否符合规则
    //1.1 获取用户名的输入框
    var usernameInput = document.getElementById("username");
    //1.2 绑定onblur事件 失去焦点
    usernameInput.onblur = checkUsername;
    function checkUsername(){
    //1.3 获取用户输入的用户名
    var username = usernameInput.value.trim();

    //1.4 判断用户名是否符合规则
    var flag = username.length >=6 && username.length <= 12
    if(flag){
        //符合规则
        document.getElementById("username_err").style.display = "none";
    }else{
        //不符合规则
        document.getElementById("username_err").style.display = "";
    }
    return flag;
}
```

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230228171121043.png" alt="image-20230228171121043" style="zoom: 67%;" /> 

display为none时，不显示span；为空时，显示span。

2、当点击注册按钮时，判断所有输入框的内容是否都符合要求，如果不符合则阻止表单提交。

```js
//1.获取表单对象
var regForm = document.getElementById("reg-form");

//2.绑定onsubmit事件
regForm.onsubmit = function () {
    var flag = checkUsername() && checkPassword() && checkTel();
    return flag;
}
```

#### 3.8 正则表达式

##### 3.8.1 正则对象使用

> **创建对象**：（两种创建方式）
>
> - 直接量
>
>   ```js
>   var reg = /正则表达式/;
>   ```
>
> - 创建 RegExp 对象
>
>   ```js
>   var reg = new RegExp("正则表达式");
>   ```
>
> **函数：**
>
> `test(str)` ：判断指定字符串是否符合规则，返回 true或 false

##### 3.8.2 语法规则

> 正则表达式常用的规则如下：
>
> * ^：表示开始
>
> * $：表示结束
>
> * [ ]：代表某个范围内的单个字符，比如： [0-9] 单个数字字符
>
> * .：代表任意单个字符，除了换行和行结束符
>
> * \w：代表单词字符：字母、数字、下划线(_)，相当于 [A-Za-z0-9_]
>
> * \d：代表数字字符： 相当于 [0-9]
>
> 量词：
>
> * +：至少一个
>
> * *：零个或多个
>
> * ？：零个或一个
>
> * {x}：x个
>
> * {m,}：至少m个
>
> * {m,n}：至少m个，最多n个

##### 3.8.2 改进表单校验案例

改进前：

```js
var flag = username.length >=6 && username.length <= 12;
var flag = password.length >=6 && password.length <= 12;
var flag = tel.length == 11;
```

改进后：

```js
var reg = /^\w{6,12}$/;
var flag = reg.test(username);

var reg = /^\w{6,12}$/;
var flag = reg.test(password);

var reg = /^[1]\d{10}$/;
var falg = reg.test(tel);
```







------

## Web核心

### 1，Web核心介绍

> - B/S 架构：Browser/Server，浏览器/服务器 架构模式，
>
>   特点是，客户端只需要浏览器，应用程序的逻辑和数据都存储在服务器端。
>
>   浏览器只需要请求服务器，获取Web资源，服务器把Web资源发送给浏览器即可
>
> 
>
> - 好处：易于维护升级：服务器端升级后，客户端无需任何部署就可以使用到新的版本

> - 静态资源：HTML、CSS、JavaScript、图片等。负责页面展现
>
> - 动态资源：Servlet、JSP 等。负责逻辑处理
>
> - 数据库：负责存储数据
>
> - HTTP协议：定义通信规则
>
> - Web服务器：负责解析 HTTP 协议，解析请求数据，并发送响应数据

### 2，Http协议

#### 2.1 Http介绍

> Hyper Text Transfer Protocol，超文本传输协议，规定了浏览器和服务器之间数据传输的规则。
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230303103051045.png" alt="image-20230303103051045" style="zoom:67%;" />

> 特点：
>
> 1、基于TCP协议，面向连接，安全
>
> 2、基于请求—响应模型的：一次请求对应一次响应
>
> 3、HTTP协议是无状态的协议：对于事务处理没有记忆能力。每次请求—相应都是独立的。
>
> - ​	缺点：多次请求间不能共享数据。
> - ​	优点：速度快。

#### 2.2 Http-请求数据格式

##### 2.2.1 请求格式

> **请求数据分为 3 部分：**
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230303103806153.png" alt="image-20230303103806153" style="zoom: 67%;" /><img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230303104156450.png" alt="image-20230303104156450" style="zoom:67%;" />
>
> ​	1、请求行：请求数据的第一行。其中GET表示请求方式，/表示请求资源路径，HTTP/1.1表示协议版本
>
> ​	2、请求头：第二行开始，格式为key：value形式。
>
> ​	3、请求体：POST请求的最后一部分，存放请求参数

> **常见的HTTP 请求头：**
>
> - Host: 表示请求的主机名
> - User-Agent: 浏览器版本，例如Chrome浏览器的标识类似Mozilla/5.0 ... Chrome/79，IE浏览器的标识类似Mozilla/5.0 (Windows NT ...) like Gecko；
> - Accept：表示浏览器能接收的资源类型，如text/*，image/*或者*/*表示所有；
> - Accept-Language：表示浏览器偏好的语言，服务器可以据此返回不同语言的网页；
> - Accept-Encoding：表示浏览器可以支持的压缩类型，例如gzip, deflate等。

> **GET 请求和 POST 请求区别：**
>
> ​	1、GET请求请求参数在请求行中，没有请求体。
>
> ​		  POST请求请求参数在请求体中。
>
> ​	2、GET请求请求参数大小有限制，POST没有。



#### 2.3 Http-响应数据格式

##### 2.3.1 格式介绍

>**响应数据分为3部分：**
>
><img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230303105308240.png" style="zoom:67%;" />
>
>- 响应行：响应数据的第一行。
>
>  ​				其中HTTP/1.1表示协议版本，200表示响应状态码，OK表示状态码描述
>
>- 响应头：第二行开始，格式为key：value形式
>
>- 响应体： 最后一部分。存放响应数据

> **常见的HTTP 响应头：**
>
> - Content-Type：表示该响应内容的类型，例如text/html，image/jpeg；
> - Content-Length：表示该响应内容的长度（字节数）；
> - Content-Encoding：表示该响应压缩算法，例如gzip；
> - Cache-Control：指示客户端应如何缓存，例如max-age=300表示可以最多缓存300秒

##### 2.3.2 响应状态码

状态码大全：https://cloud.tencent.com/developer/chapter/13553 

| 状态码分类 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 1xx        | **响应中**——临时状态码，表示请求已经接受，告诉客户端应该继续请求或者如果它已经完成则忽略它 |
| 2xx        | **成功**——表示请求已经被成功接收，处理已完成                 |
| 3xx        | **重定向**——重定向到其它地方：它让客户端再发起一个请求以完成整个处理。 |
| 4xx        | **客户端错误**——处理发生错误，责任在客户端，如：客户端的请求一个不存在的资源，客户端未被授权，禁止访问等 |
| 5xx        | **服务器端错误**——处理发生错误，责任在服务端，如：服务端抛出异常，路由出错，HTTP版本不支持等 |

常见状态码：

* 200  ok 客户端请求成功
* 404  Not Found 请求资源不存在
* 500 Internal Server Error 服务端发生不可预期的错误



### 3，Tomcat

#### 3.1 简介

##### 3.1.1什么是Web服务器

- Web服务器是一个应该程序（软件），封装HTTP协议操作，简化开发。

- 可以将web项目部署到服务器中，主要功能是"提供网上信息浏览服务"。

![image-20230303113531832](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230303113531832.png)

##### 3.1.2 Tomcat

> Tomcat是一个开源免费的轻量级Web服务器，支持Servlet/JSP少量JavaEE规范。



#### 3.2 基本使用

##### 3.2.1 安装、启动和关闭

> 安装：tomcat官网下载解压即可
>
> 启动：点击 bin/startup.bat
>
> 关闭：
>
> ​	1.直接×掉运行窗口：强制关闭
>
> ​	2.bin\shutdown.bat：正常关闭
>
> ​	3.Ctrl+C：正常关闭

##### 3.2.2 解决控制台乱码

修改conf/logging.properties

```properties
java.util.logging.ConsoleHandler.encoding = UTF-8
```

##### 3.2.3 配置

1、修改启动端口号：conf/server.xml

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

注：HTTP协议默认端口号为80，如果将Tomcat端口号改为80，则将来访问Tomcat时，将不用输入端口号。

2、启动时可能出现的问题：

- 端口号冲突：找到对应程序，将其关闭掉

  <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306093223794.png" alt="image-20230306093223794" style="zoom:67%;" />

- 启动窗口一闪而过：检查JAVA_HOME环境变量是否正确配置





#### 3.3 Maven创建Web项目

##### 3.3.1 Web项目结构

* Maven Web项目结构: 开发中的项目

  <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/1627202865978.png" style="zoom:67%;" />

* 开发完成部署的Web项目

  <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306094326454.png" alt="image-20230306094326454" style="zoom: 67%;" />

  * 开发项目通过执行Maven打包命令==package==,可以获取到部署的Web项目目录
  * 编译后的Java字节码文件和resources的资源文件，会被放到WEB-INF下的classes目录下
  * pom.xml中依赖坐标对应的jar包，会被放入WEB-INF下的lib目录下

##### 3.3.2 创建Maven Web项目

> 使用骨架：
>
> ​	1.创建Maven项目
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100401948.png" alt="image-20230306100401948" style="zoom: 80%;" />
>
> ​	2.选择使用Web项目骨架
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100435904.png" alt="image-20230306100435904" style="zoom:80%;" />
>
> ​	3.输入Maven项目坐标创建项目
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100456972.png" alt="image-20230306100456972" style="zoom:80%;" />
>
> ​	4.确认Maven相关的配置信息后，完成项目创建
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100513055.png" alt="image-20230306100513055" style="zoom:80%;" />
>
> ​	5.删除pom.xml中多余内容
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100525548.png" alt="image-20230306100525548" style="zoom:80%;" />
>
> ​	6.补齐Maven Web项目缺失的目录结构
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100536223.png" alt="image-20230306100536223" style="zoom:80%;" />



> 不使用骨架：
>
> ​	1.创建Maven项目
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100600386.png" alt="image-20230306100600386" style="zoom:80%;" />
>
> ​	2.选择不使用Web项目骨架
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100629596.png" alt="image-20230306100629596" style="zoom:80%;" />
>
> ​	3.输入Maven项目坐标创建项目
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100640768.png" alt="image-20230306100640768" style="zoom:80%;" />
>
> ​	4.在pom.xml设置打包方式为war
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100651916.png" alt="image-20230306100651916" style="zoom:80%;" />
>
> ​	5.补齐Maven Web项目缺失webapp的目录结构
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100718270.png" alt="image-20230306100718270" style="zoom:80%;" />
>
> ​	6.补齐Maven Web项目缺失WEB-INF/web.xml的目录结构
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230306100731613.png" alt="image-20230306100731613" style="zoom:80%;" />



#### 3.4 IDEA使用Tomcat

> * Maven Web项目创建成功后，通过Maven的package命令可以将项目打包成war包，将war文件拷贝到Tomcat的webapps目录下，启动Tomcat就可以将项目部署成功，然后通过浏览器进行访问即可。
> * 然而我们在开发的过程中，项目中的内容会经常发生变化，如果按照上面这种方式来部署测试，是非常不方便的

##### 3.4.1 集成本地Tomcat

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307171310532.png" alt="image-20230307171310532" style="zoom:150%;" />



##### 3.4.2 Tomcat Maven插件

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307171243525.png" alt="image-20230307171243525" style="zoom:150%;" />

```xml
<build>
    <plugins>
        <!-- tomcat 插件 -->
        <plugin>
            <groupId>org.apache.tomcat.maven</groupId>
            <artifactId>tomcat7-maven-plugin</artifactId>
            <version>2.2</version>
            <configuration>
                <port>8081</port>
                <path>/</path>
            </configuration>
        </plugin>
    </plugins>
</build>
```



### 4，Servlet

#### 4.1 简介

> - Servlet 是 Java提供的一门动态web资源开发技术
>
> - Servlet 是 JavaEE规范之一，其实就是一个接口，将来我们需要定义Servlet类实现Servlet接口，并由web服务器运行Servlet



#### 4.2 快速入门

1. 创建 web项目，导入 Servlet依赖坐标

   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307193136027.png" alt="image-20230307193136027" style="zoom:67%;" />

2. 创建：定义一个类，实现 Servlet接口，并重写接口中所有方法，并在 service方法中输入一句话

   ALT + INSERT:

   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307191555289.png" alt="image-20230307191555289" style="zoom: 67%;" />

   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307191728803.png" alt="image-20230307191728803" style="zoom:67%;" />

3. 配置：在类上使用@WebServlet 注解，配置该 Servlet的访问路径

   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307191844971.png" alt="image-20230307191844971" style="zoom:67%;" />

4. 访问：启动 Tomcat，浏览器输入URL 访问该Servlet

   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307193229097.png" alt="image-20230307193229097" style="zoom: 80%;" />



#### 4.3 Servlet 执行流程

Servlet程序已经能正常运行，但是我们需要思考个问题: 我们并没有创建ServletDemo1类的对象，也没有调用对象中的service方法，为什么在控制台就打印了`servlet hello world~`这句话呢?

要想回答上述问题，我们就需要对Servlet的执行流程进行一个学习。

![](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307193906885.png)

* 浏览器发出`http://localhost:8080/web-demo/demo1`请求，从请求中可以解析出三部分内容，分别是`localhost:8080`、`web-demo`、`demo1`
  * 根据`localhost:8080`可以找到要访问的Tomcat Web服务器
  * 根据`web-demo`可以找到部署在Tomcat服务器上的web-demo项目
  * 根据`demo1`可以找到要访问的是项目中的哪个Servlet类，根据@WebServlet后面的值进行匹配
* 找到ServletDemo1这个类后，Tomcat Web服务器就会为ServletDemo1这个类创建一个对象，然后调用对象中的service方法
  * ServletDemo1实现了Servlet接口，所以类中必然会重写service方法供Tomcat Web服务器进行调用
  * service方法中有ServletRequest和ServletResponse两个参数，ServletRequest封装的是请求数据，ServletResponse封装的是响应数据，后期我们可以通过这两个参数实现前后端的数据交互

**小结**

介绍完Servlet的执行流程，需要大家掌握两个问题：

1. Servlet由谁创建?Servlet方法由谁调用?

> Servlet由web服务器（Tomcat）创建，Servlet方法由web服务器调用

2. 服务器怎么知道Servlet中一定有service方法?

> 因为我们自定义的Servlet,必须实现Servlet接口并复写其方法，而Servlet接口中有service方法



#### 4.4 Servlet 生命周期
- 对象的生命周期指一个对象 从被创建到被销毁的整个过程。

 - Servlet 运行在 Servlet 容器中（web服务器）中，其生命周期由容器来管理，分为4阶段：
   
   ```xml
   1、 加载和实例化:默认情况下，当Servlet第一次被访问时，由容器创建Servlet对象
   
   @WebServlet(urlPatterns = "/demo1",loadOnStartup = 1)
   loadOnstartup的取值有两类情况
   	（1）负整数:第一次访问时创建Servlet对象
   	（2）0或正整数:服务器启动时创建Servlet对象，数字越小优先级越高
   
   2、初始化：在Servlet实例化之后，容器将调用Servlet的init()方法初始化这个对象，完成一些如加载配置文件、创建连接等初始化的工作。该方法只调用一次
   
   3、请求处理：每次请求Servlet时，Servlet容器都会调用Servlet的service()方法对请求进行处理
   
   4、服务终止：当需要释放内存或者容器关闭时，容器就会调用Servlet实例的destroy()方法完成资源的释放。在destroy()方法调用之后，容器会释放这个Servlet实例，该实例随后会被Java的垃圾收集器所回收。该方法只调用一次
   ```

小结

1. Servlet对象在什么时候被创建的?

> 默认是第一次访问的时候被创建，可以使用@WebServlet(urlPatterns = "/demo2",loadOnStartup = 1)的loadOnStartup 修改成在服务器启动的时候创建。

2. Servlet生命周期中涉及到的三个方法，这三个方法是什么?什么时候被调用?调用几次?

>涉及到三个方法，分别是 init()、service()、destroy()
>
>init方法在Servlet对象被创建的时候执行，只执行1次
>
>service方法在Servlet被访问的时候调用，每访问1次就调用1次
>
>destroy方法在Servlet对象被销毁的时候调用，只执行1次



#### 4.5 Servlet 方法介绍

除上述的 init()、service()、destroy() 以外，还有：

- 获取Servlet信息

  ```java
  String getServletInfo() 
  //该方法用来返回Servlet的相关信息，没有什么太大的用处，一般我们返回一个空字符串即可
  public String getServletInfo() {
      return "";
  }
  ```

- 获取ServletConfig对象

  ```java
  ServletConfig getServletConfig()
  ```



#### 4.6 Servlet 体系结构

![image-20230307203058786](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230307203058786.png)

**小结**

1. HttpServlet的使用步骤

> 继承HttpServlet
>
> 重写doGet和doPost方法

2. HttpServlet原理

> 获取请求方式，并根据不同的请求方式，调用不同的doXxx方法



#### 4.7 Servlet urlPattern配置

- 一个Servlet,可以配置多个urlPattern

  <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308093825652.png" alt="image-20230308093825652" style="zoom:80%;" />

- urlPattern配置规则<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308093633297.png" alt="image-20230308093633297" style="zoom:80%;" />



#### 4.8 XML配置方式编写Servlet

前面对应Servlet的配置，我们都使用的是@WebServlet,这个是Servlet从3.0版本后开始支持注解配置，3.0版本前只支持XML配置文件的配置方法。

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308095435272.png" alt="image-20230308095435272" style="zoom:80%;" />





### 5，Request & Response

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308105839253.png" alt="image-20230308105839253" style="zoom:80%;" />

#### 5.1 Request

##### 5.1.1 继承体系

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308110052872.png" alt="image-20230308110052872" style="zoom:80%;" />

##### 5.1.2 获取请求数据

> 请求数据分为3部分：
>
> ​	1、请求行
>
> ​	2、请求头
>
> ​	3、请求体

- 获取**请求行**数据：

  <img src="C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230308164627460.png" alt="image-20230308164627460" style="zoom:80%;" />
  
  ```java
  String getMethod()：获取请求方式： GET
  String getContextPath()：获取虚拟目录(项目访问路径)： /request-demo
  StringBuffer getRequestURL(): 获取URL(统一资源定位符)： http://localhost:8080/request-demo/req1
  String getRequestURI()：获取URI(统一资源标识符)： /request-demo/req1
  String getQueryString()：获取请求参数（GET方式）：  username=zhangsan&password=123
  ```

- 获取**请求头**数据：

  <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308165621035.png" alt="image-20230308165621035" style="zoom:80%;" />

  ```java
  String getHeader(String name)：根据请求名称，获取值
  ```

- 获取**请求体**数据：

  <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308165740889.png" alt="image-20230308165740889" style="zoom:80%;" />

  ```java
  ServletInputStream  getInputStream()：获取字节输入流
  BufferedReader getReader()：获取字符输入流
  ```



```java
/**
 * request 获取请求数据
 */
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取请求头: user-agent: 浏览器的版本信息
        String agent = req.getHeader("user-agent");
		System.out.println(agent);
    }
```



##### 5.1.3获取请求参数

> - 请求参数是请求数据的一部分，在用户登录时，输入的用户名和密码就是请求参数。
>
> - GET请求的请求参数在请求行中，
>
>   POST请求的请求参数在请求体中。



> 普通的获取请求参数的方式：
>
> GET方式：
>
> ```java
> String getQueryString()
> ```
>
> POST方式：
>
> ```java
> BufferedReader getReader();
> ```
>



> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308201741098.png" style="zoom:80%;" />
>
> 上述获取参数方式**有如下问题**，doGet 和 doPost 中除了获取请求参数不一样，其余的处理业务的代码几乎是相同的，为了避免这部分代码的重复，
>
> 就使用以下通用方法。

> 解决方案一:
>
> ```java
> @WebServlet("/req1")
> public class RequestDemo1 extends HttpServlet {
>     @Override
>     protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
>         //获取请求方式
>         String method = req.getMethod();
>         //获取请求参数
>         String params = "";
>         if("GET".equals(method)){
>             params = req.getQueryString();
>         }else if("POST".equals(method)){
>             BufferedReader reader = req.getReader();
>             params = reader.readLine();
>         }
>         //将请求参数进行打印控制台
>         System.out.println(params);
>       
>     }
>     @Override
>     protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
>         this.doGet(req,resp);
>     }
> }
> ```
>
> 使用request的getMethod()来获取请求方式，根据请求方式的不同分别获取请求参数值，这样就可以解决上述问题，
>
> 但是以后每个Servlet都需要这样写代码，实现起来比较麻烦，这种方案我们不采用



> 解决方案二：
>
> request对象已经将上述获取请求参数的方法进行了封装，并且request提供的方法实现的功能更强大，以后只需要调用request提供的方法即可。
>
> request 根据不同请求方式获取请求参数，并将请求参数的内容进行分割，存入一个Map集合中。
>
> ![image-20230308202946895](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230308202946895.png)

##### 5.1.4 IDEA快速创建Servlet

settings -> Editor -> File and Code Templates -> Other

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309100446739.png" alt="image-20230309100446739" style="zoom:80%;" />

右键新建servlet文件，即可按照修改好的模版文件创建。



##### 5.1.5 请求参数中文乱码问题

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309110553463.png" alt="image-20230309110553463" style="zoom:80%;" />



##### 5.1.6 request请求转发

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309110703225.png" alt="image-20230309110703225" style="zoom:80%;" />

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309112157126.png" alt="image-20230309112157126" style="zoom:80%;" />

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309112612689.png" alt="image-20230309112612689" style="zoom:80%;" />





#### 5.2 Response

> - Request：使用request对象来获取请求数据
> - Response：使用response对象来设置相应数据

##### 5.2.1 Response 设置响应数据功能介绍 

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309113521064.png" alt="image-20230309113521064" style="zoom: 67%;" />

##### 5.2.2 Response 完成重定向

![image-20230309113913246](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309113913246.png)



> 路径问题：
>
> - 明确路径给谁使用？
>
>   - 浏览器使用：需要加虚拟目录（项目访问路径）
>   - 服务端使用：不需要加虚拟目录
>
> - 练习：
>
>   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309142835869.png" alt="image-20230309142835869" style="zoom:150%;" />
>
>
> 

##### 5.2.3 Response 响应字符数据

- 使用

  1、通过Response对象获取字符输出流

  ```java
  PrintWriter writer = resp.getWriter();
  ```

  2、写数据

  ```java
  writer.write("aaa");
  ```

- 注意

  - 流不需要关闭，随着响应结束，response对象销毁，由服务器关闭

  - 中文数据会乱码（因为通过Response获取的字符输出流默认编码：ISO-8859-1）

    使用如下方法解决：

    ```java
    resp.setContentType("text/html;charset=utf-8");
    ```

##### 5.2.4 Response 响应字节数据

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309152303547.png" alt="image-20230309152303547" style="zoom:80%;" />





#### 5.3 案例

##### 5.3.1 用户登录

> 流程说明：
>
> 1.用户填写用户名密码，提交到 LoginServlet
>
> 2.在 LoginServlet中使用 MyBatis查询数据库，验证用户名密码是否正确
>
> 3.如果正确，响应“登录成功”，如果错误，响应“登录失败”

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309152928959.png" alt="image-20230309152928959" style="zoom:80%;" />



> 准备工作：
>
> ​	1.复制资料中的静态页面到项目的webapp目录下
>
> ​	2.创建 db1数据库，创建 tb_user表，创建 User实体类
>
> ​	3.导入MyBatis坐标，MySQL驱动坐标
>
> ​	4.创建mybatis-config.xml核心配置文件，UserMapper.xml映射文件，UserMapper接口

> 流程说明：
>
> ​	1.用户填写用户名密码，提交到 LoginServlet
>
> ​	2.在 LoginServlet中使用 MyBatis查询数据库，验证用户名密码是否正确
>
> ​	3.如果正确，响应“登录成功”，如果错误，响应“登录失败”



##### 5.3.2 用户注册

> 流程说明：
>
> ​	1.用户填写用户名、密码等信息，点击注册按钮，提交到 RegisterServlet
>
> ​	2.在 RegisterServlet 中使用 MyBatis 保存数据
>
> ​	3.保存前，需要判断用户名是否已经存在：根据用户名查询数据库

![image-20230309172332125](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230309172332125.png)

> 1、在UserMapper类中添加方法
>
> ```java
>  /*
>     * 根据用户名查询用户
>     * */
>     @Select("select * from tb_user where username = #{username}")
>     User selectByUsername(String username);
> 
>     /*
>     * 添加用户
>     * */
>     //id 为自增长，设置为 null 即可
>     @Select("insert into tb_user values(null,#{username},#{password})")
>     void add(User user);
> ```
>
> 
>
> 2、创建register.html
>
> ```html
> <!DOCTYPE html>
> <html lang="en">
> <head>
>     <meta charset="UTF-8">
>     <title>欢迎注册</title>
>     <link href="css/register.css" rel="stylesheet">
> </head>
> <body>
> 
> <div class="form-div">
>     <div class="reg-content">
>         <h1>欢迎注册</h1>
>         <span>已有帐号？</span> <a href="login.html">登录</a>
>     </div>
>     <form id="reg-form" action="/request-demo/registerServlet" method="post">
> 
>         <table>
> 
>             <tr>
>                 <td>用户名</td>
>                 <td class="inputs">
>                     <input name="username" type="text" id="username">
>                     <br>
>                     <span id="username_err" class="err_msg" style="display: none">用户名不太受欢迎</span>
>                 </td>
> 
>             </tr>
> 
>             <tr>
>                 <td>密码</td>
>                 <td class="inputs">
>                     <input name="password" type="password" id="password">
>                     <br>
>                     <span id="password_err" class="err_msg" style="display: none">密码格式有误</span>
>                 </td>
>             </tr>
> 
> 
>         </table>
> 
>         <div class="buttons">
>             <input value="注 册" type="submit" id="reg_btn">
>         </div>
>         <br class="clear">
>     </form>
> 
> </div>
> </body>
> </html>
> ```
>
> 
>
> 3、创建RegisterServlet类
>
> ```java
> 
> @WebServlet(value = "/registerServlet")
> public class RegisterServlet extends HttpServlet {
>     @Override
>     protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
>         //1.接收用户数据
>         String username = request.getParameter("username");
>         String password = request.getParameter("password");
> 
>         //封装用户对象
>         User user = new User();
>         user.setUsername(username);
>         user.setPassword(password);
> 
>         //2、调用 Mapper，根据用户名查询用户对象
>         //2.1 获取SqlSessionFactory对象
> /*
>         String resource = "mybatis-config.xml";
>         InputStream inputStream = Resources.getResourceAsStream(resource);
>         SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
> */
> 
>         //上述三行代码 由utils工具类替代
>         SqlSessionFactory sqlSessionFactory = SqlSessionFactoryUtils.getSqlSessionFactory();
>         //2.2 获取SqlSession对象
>         SqlSession session = sqlSessionFactory.openSession();
>         //2.3 获取Mapper
>         UserMapper mapper = session.getMapper(UserMapper.class);
> 
>         //2.4 调用方法
>         User u = mapper.selectByUsername(username);
> 
>         //3.判断用户对象 是否为null
>         if(u == null){
>            //用户名不存在，添加用户
>            mapper.add(user);
>             // 提交事务
>             session.commit();
> 
>             response.setContentType("text/html;charset=utf-8");
>             response.getWriter().write("注册成功");
>             session.close();
>         }else {
>             //用户名存在，给出提示信息
>             response.setContentType("text/html;charset=utf-8");
>             response.getWriter().write("用户名已存在");
>         }
> 
>     }
> 
>     @Override
>     protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
>         this.doGet(request, response);
>     }
> }
> ```

##### 5.3.3 代码优化

（对重复代码进行SQLSessionfactory工具类抽取，在该类中创建返回值为SqlSessionFactory的方法，之后在注册和登录的Servlet中，直接调用工具类的该方法即可。）

> 上面两个功能已经实现，但是在写Servlet的时候，因为需要使用Mybatis来完成数据库的操作，所以对于Mybatis的基础操作就出现了些重复代码，如下
>
> ```java
> String resource = "mybatis-config.xml";
> InputStream inputStream = Resources.getResourceAsStream(resource);
> SqlSessionFactory sqlSessionFactory = new 
> 	SqlSessionFactoryBuilder().build(inputStream);
> ```
>
> 有了这些重复代码就会造成一些问题:
>
> * 重复代码不利于后期的维护
> * SqlSessionFactory工厂类进行重复创建
>   * 就相当于每次买手机都需要重新创建一个手机生产工厂来给你制造一个手机一样，资源消耗非常大但性能却非常低。所以这么做是不允许的。
> * 那如何来优化呢？
>
>   * 代码重复可以抽取工具类
>   * 对指定代码只需要执行一次可以使用静态代码块

> 有了这两个方向后，代码具体该如何编写?
>
> ```java
> public class SqlSessionFactoryUtils {
> 
>     private static SqlSessionFactory sqlSessionFactory;
> 
>     static {
>         //静态代码块会随着类的加载而自动执行，且只执行一次
>         try {
>             String resource = "mybatis-config.xml";
>             InputStream inputStream = Resources.getResourceAsStream(resource);
>             sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
>         } catch (IOException e) {
>             e.printStackTrace();
>         }
>     }
> 
> 
>     public static SqlSessionFactory getSqlSessionFactory(){
>         return sqlSessionFactory;
>     }
> }
> ```
>
> 工具类抽取以后，以后在对Mybatis的SqlSession进行操作的时候，就可以直接使用
>
> ```java
> SqlSessionFactory sqlSessionFactory = SqlSessionFactoryUtils.getSqlSessionFactory();
> ```
>
> 这样就可以很好的解决上面所说的代码重复和重复创建工厂导致性能低的问题了。





## JSP

### 1，JSP 概述

> JSP（全称：Java Server Pages）：Java服务端页面。
>
> 既可以定义HTML、JS、CSS等静态内容，还可以定义 Java 代码的动态内容。



### 2，JSP入门

先在 pom.xml 中导入 jsp 依赖

```xml
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope>
</dependency>
```

webapp下新建 jsp 页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>hello jsp</h1>

    <%
        System.out.println("hello,jsp~");
    %>
</body>
</html>
```

运行tomcat，浏览器会显示

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230310111005945.png" alt="image-20230310111005945" style="zoom:50%;" />

控制台会输出 hello,jsp~



### 3，JSP 原理

> - 本质上就是一个Servlet
>
> -  JSP 在被访问时，由 JSP 容器(Tomcat)将其转换为 Java文件(Servlet)，在由 JSP 容器(Tomcat)将其编译，最终对外提供服务的其实就是这个字节码文件
>
> ![image-20230310105540243](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230310105540243.png)



### 4，JSP 脚本

> - JSP 脚本用于在 JSP 页面内定义 Java 代码
> - JSP 脚本分类
>   1. <%...%>：内容会直接放到_jspService()方法之中
>   2. <%=…%>：内容会放到out.print()中，作为out.print()的参数
>   3. <%!…%>：内容会放到_jspService()方法之外，被类直接包含

#### 4.1 案例

使用 JSP 脚本展示品牌数据

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230310113732312.png" alt="image-20230310113732312" style="zoom:80%;" />

- 在 brand.jsp 中准备一些数据，模仿其是从数据库中获取到的

```jsp
<%
    // 查询数据库
    List<Brand> brands = new ArrayList<Brand>();
    brands.add(new Brand(1,"三只松鼠","三只松鼠",100,"三只松鼠，好吃不上火",1));
    brands.add(new Brand(2,"优衣库","优衣库",200,"优衣库，服适人生",0));
    brands.add(new Brand(3,"小米","小米科技有限公司",1000,"为发烧而生",1));
%>
```

- 将 table 标签中的数据改为动态的

  ```jsp
  <table border="1" cellspacing="0" width="800">
      <tr>
          <th>序号</th>
          <th>品牌名称</th>
          <th>企业名称</th>
          <th>排序</th>
          <th>品牌介绍</th>
          <th>状态</th>
          <th>操作</th>
  
      </tr>
      
      <%
       for (int i = 0; i < brands.size(); i++) {
           //获取集合中的 每一个 Brand 对象
           Brand brand = brands.get(i);
      %>
      	 <tr align="center">
              <td>1</td>
              <td>三只松鼠</td>
              <td>三只松鼠</td>
              <td>100</td>
              <td>三只松鼠，好吃不上火</td>
              <td>启用</td>
              <td><a href="#">修改</a> <a href="#">删除</a></td>
          </tr>
      <%
       }
      %>
     
  </table>
  ```

  > 注：jsp 中的 java 代码是可以截断的



- 将 td 标签的数据 改为动态的

  ```jsp
  <table border="1" cellspacing="0" width="800">
      <tr>
          <th>序号</th>
          <th>品牌名称</th>
          <th>企业名称</th>
          <th>排序</th>
          <th>品牌介绍</th>
          <th>状态</th>
          <th>操作</th>
  
      </tr>
      
      <%
       for (int i = 0; i < brands.size(); i++) {
           //获取集合中的 每一个 Brand 对象
           Brand brand = brands.get(i);
      %>
      	 <tr align="center">
              <td><%=brand.getId()%></td>
              <td><%=brand.getBrandName()%></td>
              <td><%=brand.getCompanyName()%></td>
              <td><%=brand.getOrdered()%></td>
              <td><%=brand.getDescription()%></td>
              <td><%=brand.getStatus() == 1 ? "启用":"禁用"%></td>
              <td><a href="#">修改</a> <a href="#">删除</a></td>
          </tr>
      <%
       }
      %>
     
  </table>
  ```

  

#### 4.2 JSP 缺点

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230310142325158.png" alt="image-20230310142325158" style="zoom:80%;" />



### 5， EL 表达式

- Expression Language 表达式语言，用于简化 JSP 页面内的 Java 代码

- 主要功能：获取数据

- 语法：${expression}

  ${brands}： 获取域中存储的 key 为 brands 的数据

  

- JavaWeb中的四大域对象：

  1、 page：当前页面有效

  2、request：当前请求有效

  3、session：当前回话有效

  4、application：当前应用有效

el 表达式获取数据，会依次从这4个域中寻找，直到找到为止。

（将数据存到域中，再从域中获取数据。）

<img src="C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230310193603348.png" alt="image-20230310193603348" style="zoom:80%;" />



### 6，JSTL 标签

##### 4.4.1 简介

- JSP 标准标签库（ Jsp Standarded Tag Library ），使用标签取代 JSP 页面上的 Java 代码。

![image-20230310194457541](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230310194457541.png)



##### 4.4.2 JSTL 快速入门

​	1、导入坐标

```xml
<dependency>
  <groupId>jstl</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
```

​	2、在 JSP 页面上引入 JSTL标签库

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

​	3、使用

​	<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230311110617464.png" alt="image-20230311110617464" style="zoom:67%;" />

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230311110647494.png" style="zoom:80%;" />



### 7， MVC模式和三层架构

#### 7.1 MVC模式

> MVC 是一种分层开发的模式，其中：
>
> - M：Model，业务模型，处理业务
> - V：View，视图，界面展示
> - C：Controller，控制器，处理请求，调用模型和视图

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230311111742601.png" alt="image-20230311111742601" style="zoom:80%;" />

> MVC 好处
>
> - 职责单一，互不影响
> - 有利于分工协作
> - 有利于组件重用



#### 7.2 三层架构

> - 表现层
> - 业务逻辑层
> - 数据访问层

![image-20230311112236267](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230311112236267.png)

>  MVC更宏观，三层架构更落地。

> dao：data access object
>
> <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230311113746245.png" alt="image-20230311113746245" style="zoom:50%;" />



MVC 模式 和 三层架构：

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230311114253919.png" alt="image-20230311114253919" style="zoom:80%;" />



#### 7.3 案例

##### 7.3.1 环境准备

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230313100446496.png" style="zoom:80%;" />

Mybatis-config.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--起别名-->
    <typeAliases>
        <package name="com.duck.pojo"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/db1?useSSL=false&amp;useServerPrepStmts=true"/>
                <property name="username" value="root"/>
                <property name="password" value="password"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--扫描mapper-->
        <package name="com.duck.mapper"/>
    </mappers>
</configuration>
```

BrandMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.duck.mapper.BrandMapper">


    <resultMap id="brandResultMap" type="brand">
        <result column="brand_name" property="brandName"></result>
        <result column="company_name" property="companyName"></result>
    </resultMap>


</mapper>
```

BrandMapper:

```java
public interface BrandMapper {
    /**
     * 查询所有
     * @return
     */
    @Select("select * from tb_brand")
    @ResultMap("brandResultMap")
    List<Brand> selectAll();

}
```



##### 7.3.2 查询所有

![image-20230313114049151](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230313114049151.png)

Dao层： 

```java
public interface BrandMapper {
    /**
     * 查询所有
     * @return
     */
    @Select("select * from tb_brand")
    @ResultMap("brandResultMap")
    List<Brand> selectAll();

}
```

Service层：

```java
package com.duck.service;

public class BrandService {
    SqlSessionFactory factory = SqlSessionFactoryUtils.getSqlSessionFactory();

    /**
     * 查询所有
     * @return
     */
    public List<Brand> selectAll(){
        //调用BrandMapper.selectAll()

        //1.获取SQL Session对象
        SqlSession session = factory.openSession();
        //2.获取BrandMapper
        BrandMapper mapper = session.getMapper(BrandMapper.class);
        //3.调用方法
        List<Brand> brands = mapper.selectAll();
        session.close();

        return brands;
    }
}
```

Web层：

```java
package com.duck.web;

@WebServlet(value = "/selectAllServlet")
public class SelectAllServlet extends HttpServlet {
    private BrandService service = new BrandService();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.调用BrandService完成查询
        List<Brand> brands = service.selectAll();

        //2.存入request域中
        request.setAttribute("brands",brands);

        //3.转发到index.jsp
        request.getRequestDispatcher("/brand.jsp").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}

```



##### 7.3.3 添加

![image-20230313193516294](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230313193516294.png)

Dao层：

```java
    /**
     * 添加 brand
     * @param brand
     */

    @Insert("insert into tb_brand values(null,#{brandName},#{companyName},#{ordered},#{description},#{status})")
    @ResultMap("brandResultMap")
    void add(Brand brand);
```

Service层：

```java
    /**
     * 添加brand对象
     * @param brand
     */
    public void add(Brand brand){
        //1.获取SQL Session对象
        SqlSession session = factory.openSession();
        //2.获取BrandMapper
        BrandMapper mapper = session.getMapper(BrandMapper.class);
        //3.调用方法
        mapper.add(brand);

        //提交事务
        session.commit();
        //释放资源
        session.close();
    }
```

Web层：

```java
@WebServlet("/addServlet")
public class addServlet extends HttpServlet {
    private BrandService service = new BrandService();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //处理POST请求的乱码问题
        request.setCharacterEncoding("UTF-8");

        //1.接收表单提交的数据，封装为一个Brand对象
        String brandName = request.getParameter("brandName");
        String companyName = request.getParameter("companyName");
        String ordered = request.getParameter("ordered");
        String description = request.getParameter("description");
        String status = request.getParameter("status");

        //封装为一个Brand对象
        Brand brand = new Brand();
        brand.setBrandName(brandName);
        brand.setCompanyName(companyName);
        brand.setOrdered(Integer.parseInt(ordered));
        brand.setDescription(description);
        brand.setStatus(Integer.parseInt(status));

        //2.调用service完成添加
        service.add(brand);

        //3.转发到查询所有的servlet
        request.getRequestDispatcher("/selectAllServlet").forward(request,response);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```





##### 7.3.4 修改-回显数据

> 点击修改，需要把之前的信息 回显到表单上，再在原有数据上进行修改。

![image-20230313210440700](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230313210440700.png)

Dao层：

```java
@Select("select * from  tb_brand where id = #{id}")
@ResultMap("brandResultMap")
Brand selectById(int id);
```

Service层：

```java
public Brand selectById(int id){
    //调用BrandMapper.selectById()

    //1.获取SQL Session对象
    SqlSession session = factory.openSession();
    //2.获取BrandMapper
    BrandMapper mapper = session.getMapper(BrandMapper.class);
    //3.调用方法
    Brand brand = mapper.selectById(id);

    session.close();

    return brand;
}
```

Web层：

```java
@WebServlet("/addServlet")
public class addServlet extends HttpServlet {
    private BrandService service = new BrandService();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //处理POST请求的乱码问题
        request.setCharacterEncoding("UTF-8");

        //1.接收表单提交的数据，封装为一个Brand对象
        String brandName = request.getParameter("brandName");
        String companyName = request.getParameter("companyName");
        String ordered = request.getParameter("ordered");
        String description = request.getParameter("description");
        String status = request.getParameter("status");

        //封装为一个Brand对象
        Brand brand = new Brand();
        brand.setBrandName(brandName);
        brand.setCompanyName(companyName);
        brand.setOrdered(Integer.parseInt(ordered));
        brand.setDescription(description);
        brand.setStatus(Integer.parseInt(status));

        //2.调用service完成添加
        service.add(brand);

        //3.转发到查询所有的servlet
        request.getRequestDispatcher("/selectAllServlet").forward(request,response);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

jsp：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <input type="button" value="新增" id="add"><br>
    <hr>
    <table border="1" cellspacing="0" width="800">
        <tr>
            <th>序号</th>
            <th>品牌名称</th>
            <th>企业名称</th>
            <th>排序</th>
            <th>品牌介绍</th>
            <th>状态</th>
            <th>操作</th>
        </tr>

        <c:forEach items="${brands}" var="brand" varStatus="status">
            <tr align="center">
                <%--<td>${brand.id}</td>--%>
                <td>${status.count}</td>
                <td>${brand.brandName}</td>
                <td>${brand.companyName}</td>
                <td>${brand.ordered}</td>
                <td>${brand.description}</td>
                <c:if test="${brand.status == 1}">
                    <td>启用</td>
                </c:if>
                <c:if test="${brand.status != 1}">
                    <td>禁用</td>
                </c:if>

                <td><a href="/selectByIdServlet?id=${brand.id}">修改</a><a href="#">删除</a></td>
            </tr>
        </c:forEach>

    </table>

<script>
    document.getElementById("add").onclick = function () {
        location.href = "/addBrand.jsp";
    }
</script>
</body>
</html>
```

##### 7.3.5 修改-修改数据

![image-20230313210456209](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230313210456209.png)

Dao层：

```java
    /**
     * 修改 brand 数据
     * @param brand
     */
    @Update("update tb_brand set brand_name = #{brandName},company_name = #{companyName},ordered = #{ordered},description = #{description},status = #{status} where id = ${id}")
    @ResultMap("brandResultMap")
    void update(Brand brand);
```

Service层：

```java
    /**
     * 修改brand
     * @param brand
     */
    public void update(Brand brand){
        //1.获取SQL Session对象
        SqlSession session = factory.openSession();
        //2.获取BrandMapper
        BrandMapper mapper = session.getMapper(BrandMapper.class);
        //3.调用方法
        mapper.update(brand);

        //提交事务
        session.commit();
        session.close();
    }
```

Web层：

```java
@WebServlet("/updateServlet")
public class UpdateServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("UTF-8");

        //1.接收 id 和 brand 信息
        String id = request.getParameter("id");

        String brandName = request.getParameter("brandName");
        String companyName = request.getParameter("companyName");
        String ordered = request.getParameter("ordered");
        String description = request.getParameter("description");
        String status = request.getParameter("status");
        //封装brand
        Brand brand = new Brand(Integer.parseInt(id),brandName,companyName,Integer.parseInt(ordered),description,Integer.parseInt(status));

        //2.调用Service中的update方法
        BrandService service = new BrandService();
        service.update(brand);

        //3.转发到 查询所有Servlet
        request.getRequestDispatcher("/selectAllServlet").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

jsp:

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>修改品牌</title>
</head>
<body>
    <h3>修改品牌</h3>
    <form action="/updateServlet" method="post">

        <%--隐藏域，提交id--%>
        <input type="hidden" name="id" value="${brand.id}">

        品牌名称：<input name="brandName" value="${brand.brandName}"><br>
        企业名称：<input name="companyName" value="${brand.companyName}"><br>
        排序：<input name="ordered" value="${brand.ordered}"><br>
        描述信息：<textarea rows="5" cols="20" name="description">${brand.description}</textarea><br>
        状态：
        <c:if test="${brand.status == 0}">
            <input type="radio" name="status" value="0" checked="checked">禁用
            <input type="radio" name="status" value="1">启用<br>
        </c:if>

        <c:if test="${brand.status == 1}">
            <input type="radio" name="status" value="0">禁用
            <input type="radio" name="status" value="1" checked="checked">启用<br>
        </c:if>

        <input type="submit" value="提交">
    </form>
</body>
</html>
```



##### 7.3.5 删除

dao层：

```java
    /**
     * 添加 brand
     * @param id
     */

    @Insert("delete from tb_brand where id = #{id}")
    @ResultMap("brandResultMap")
    void delete(int id);
```

service层：

```java
    /**
     * 删除 brand 对象
     * @param id
     */
    public void delete(int id){
        //1.获取SQL Session对象
        SqlSession session = factory.openSession();
        //2.获取BrandMapper
        BrandMapper mapper = session.getMapper(BrandMapper.class);
        //3.调用方法
        mapper.delete(id);

        //提交事务
        session.commit();
        //释放资源
        session.close();

    }
```

Web层：

```java
@WebServlet(value = "/deleteByIdServlet")
public class DeleteByIdServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取id
        int id = Integer.parseInt(request.getParameter("id"));
        //调用service层的delete方法
        BrandService service = new BrandService();
        service.delete(id);
        //重定向到selectAllServlet
        request.getRequestDispatcher("/selectAllServlet").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

brand.jsp：

```jsp
    <table border="1" cellspacing="0" width="800">
        <tr>
            <th>序号</th>
            <th>品牌名称</th>
            <th>企业名称</th>
            <th>排序</th>
            <th>品牌介绍</th>
            <th>状态</th>
            <th>操作</th>
        </tr>

        <c:forEach items="${brands}" var="brand" varStatus="status">
            <tr align="center">
                <%--<td>${brand.id}</td>--%>
                <td>${status.count}</td>
                <td>${brand.brandName}</td>
                <td>${brand.companyName}</td>
                <td>${brand.ordered}</td>
                <td>${brand.description}</td>
                <c:if test="${brand.status == 1}">
                    <td>启用</td>
                </c:if>
                <c:if test="${brand.status != 1}">
                    <td>禁用</td>
                </c:if>

                <td><a href="/selectByIdServlet?id=${brand.id}">修改</a><a href="/deleteByIdServlet?id=${brand.id}">删除</a></td>
            </tr>
        </c:forEach>

    </table>
```



### 8，Cookie Session(未完成的)















### 9，Filter & Listener & ajax

#### 9.1 Filter 

##### 9.1.1 Filter概述

> - 概念：Filter 表示过滤器，是 JavaWeb三大组件（Servlet、Filter、Listener）之一。
> - 过滤器可以把资源的请求**拦截**下来，从而实现一些特殊的功能。
> - 过滤器一般完成一些通用的操作，比如：权限控制、统一编码处理、敏感字符处理等。                  

![image-20230314095955601](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314095955601.png)



##### 9.1.2 Filter快速入门

![image-20230314115603064](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314115603064.png)



##### 9.1.3 Filter执行流程

![image-20230314151811851](C:\Users\10632\AppData\Roaming\Typora\typora-user-images\image-20230314151811851.png)

接下来我们通过代码验证一下，在 `doFilter()` 方法前后都加上输出语句，如下

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20210823195828596.png" alt="image-20210823195828596" style="zoom: 67%;" />

同时在 `hello.jsp` 页面加上输出语句，如下

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314152536541.png" alt="image-20230314152536541" style="zoom:67%;" />

执行访问该资源打印的顺序是按照我们标记的标号进行打印的话，说明我们上边总结出来的流程是没有问题的。启动服务器访问 `hello.jsp` 页面，在控制台打印的内容如下：

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20210823200202153.png" alt="image-20210823200202153" style="zoom:80%;" />

以后我们可以将对请求进行处理的代码放在放行之前进行处理，而如果请求完资源后还要对响应的数据进行处理时可以在放行后进行逻辑处理。

##### 9.1.4 Filter拦截路径配置

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314153140132.png" alt="image-20230314153140132" style="zoom: 80%;" />



##### 9.1.5 过滤器链

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314153525063.png" alt="image-20230314153525063" style="zoom: 80%;" />

按照链名的自然排序执行：

![image-20230314154002804](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314154002804.png)

##### 9.1.6 案例 登录验证

> 需求：访问服务器资源时，需要先进行登录验证，
>
> 如果没有登录，则自动跳转到登录页面。

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314161322920.png" alt="image-20230314161322920" style="zoom:80%;" />



#### 9.2 Listener

##### 9.2.1 概述

> * Listener 表示监听器，是 JavaWeb 三大组件(Servlet、Filter、Listener)之一。
>
> * 监听器可以监听 `application`，`session`，`request` 三个对象创建、销毁或者往其中添加修改删除属性时自动执行代码的功能组件。
>
>   request 和 session 我们学习过。而 `application` 是 `ServletContext` 类型的对象。
>
>   `ServletContext` 代表整个web应用，在服务器启动的时候，tomcat会自动创建该对象。在服务器关闭时会自动销毁该对象。

##### 9.2.2 分类

JavaWeb 提供了8个监听器：

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314162205423.png" alt="image-20230314162205423" style="zoom:80%;" />

这里面只有 `ServletContextListener` 这个监听器后期我们会接触到，`ServletContextListener` 是用来监听 `ServletContext` 对象的创建和销毁。

`ServletContextListener` 接口中有以下两个方法

* `void contextInitialized(ServletContextEvent sce)`：`ServletContext` 对象被创建了会自动执行的方法
* `void contextDestroyed(ServletContextEvent sce)`：`ServletContext` 对象被销毁时会自动执行的方法

##### 9.2.3 代码演示

我们只演示一下 `ServletContextListener` 监听器

* 定义一个类，实现`ServletContextListener` 接口
* 重写所有的抽象方法
* 使用 `@WebListener` 进行配置

代码如下：

```java
@WebListener
public class ContextLoaderListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        //加载资源
        System.out.println("ContextLoaderListener...");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        //释放资源
    }
}
```

启动服务器，就可以在启动的日志信息中看到 `contextInitialized()` 方法输出的内容，同时也说明了 `ServletContext` 对象在服务器启动的时候被创建了。



#### 9.3 Ajax

##### 9.3.1 概述

> - `AJAX` (Asynchronous JavaScript And XML)：异步的 JavaScript 和 XML。



> - AJAX作用：
>
>   1、**与服务器进行数据交换**：通过AJAX可以给服务器发送请求，并**获取服务器响应的数据**
>
>   - 使用了 Ajax 和服务器进行通信，就可以使用 HTML + Ajax 来替换JSP页面了。
>
>   之后：
>
>   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314185717152.png" alt="image-20230314185717152" style="zoom: 67%;" />
>
>   之前：
>
>   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314163401024.png" alt="image-20230314163401024" style="zoom:67%;" />
>
>   
>
>   2、异步交换：可以在***不重新加载整个页面***的情况下，与服务器交换数据并***更新部分***网页的技术，如：搜索联想、用户名是否可用校验，等等…
>
>   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314185931853.png" alt="image-20230314185931853" style="zoom:67%;" />



> - 同步和异步：
>
>   >  浏览器页面在发送请求给服务器，在服务器处理请求的过程中，浏览器页面不能做其他的操作。只能等到服务器响应结束后才能，浏览器页面才能继续做其他的操作。
>   >
>   > ![image-20230314190603549](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314190603549.png)
>
>   > 浏览器页面发送请求给服务器，在服务器处理请求的过程中，浏览器页面还可以做其他的操作。
>   >
>   > ![image-20230314190619724](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314190619724.png)
>
>   



##### 9.3.2 快速入门

https://www.w3school.com.cn/js/js_ajax_intro.asp

分两端，服务端和浏览器端。

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314212457229.png" alt="image-20230314212457229" style="zoom:67%;" />

1. 编写AjaxServlet，并使用response输出字符串

```java
@WebServlet("/ajaxServlet")
public class AjaxServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.相应数据
        response.getWriter().write("hello ajax");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

2. 创建 XMLHttpRequest 对象：用于和服务器交换数据

```java
var xmlhttp;
if (window.XMLHttpRequest) {
    // code for IE7+, Firefox, Chrome, Opera, Safari
    xmlhttp = new XMLHttpRequest();
} 
else {
    // code for IE6, IE5 
    xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
```

3. 向服务器发送请求

```java
xmlhttp.open("GET",“url");
xmlhttp.send();//发送请求
```

4. 获取服务器响应数据

```java
xmlhttp.onreadystatechange = function () {
    if (xmlhttp.readyState == 4 && xmlhttp.status == 200){
        alert(xmlhttp.responseText);
    }
}
```



##### 9.3.3 案例

![image-20230314201315567](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314201315567.png)



#### 9.4 axios

> - Axios 对原生的AJAX进行封装，简化书写
> - 官网：https://www.axios-http.cn/docs/intro

##### 9.4.1 基本使用

axios使用是比较简单的，分以下两步：

- 引入 axios 的 js 文件

  ```html
  <script src="js/axios-0.18.0.js"></script>
  ```

- 使用 axios 发送请求，并获取响应结果

  - 发送 get 请求

    ```js
    axios({
        method:"get",
        url:"http://localhost:8080/ajax-demo1/aJAXDemo1?username=zhangsan"
    }).then(function (resp){
        alert(resp.data);
    })
    ```

  - 发送 post 请求

    ```js
    axios({
        method:"post",
        url:"http://localhost:8080/ajax-demo1/aJAXDemo1",
        data:"username=zhangsan"
    }).then(function (resp){
        alert(resp.data);
    });
    ```

`axios()` 是用来发送异步请求的，小括号中使用 js 对象传递请求相关的参数：

* `method` 属性：用来设置请求方式的。取值为 `get` 或者 `post`。
* `url` 属性：用来书写请求的资源路径。如果是 `get` 请求，需要将请求参数拼接到路径的后面，格式为： `url?参数名=参数值&参数名2=参数值2`。
* `data` 属性：作为请求体被发送的数据。也就是说如果是 `post` 请求的话，数据需要作为 `data` 属性的值。

`then()` 需要传递一个匿名函数。我们将 `then()` 中传递的匿名函数称为 ==回调函数==，意思是该匿名函数在发送请求时不会被调用，而是在成功响应后调用的函数。而该回调函数中的 `resp` 参数是对响应的数据进行封装的对象，通过 `resp.data` 可以获取到响应的数据。



##### 9.4.2 快速入门

> - 后端实现
>
>   定义一个用于接收请求的servlet，代码如下：
>
>   ```java
>   @WebServlet("/axiosServlet")
>   public class AxiosServlet extends HttpServlet {
>       @Override
>       protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
>           System.out.println("get...");
>           //1. 接收请求参数
>           String username = request.getParameter("username");
>           System.out.println(username);
>           //2. 响应数据
>           response.getWriter().write("hello Axios~");
>       }
>   
>       @Override
>       protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
>           System.out.println("post...");
>           this.doGet(request, response);
>       }
>   }
>   ```
>
> - 前端实现
>
>   * 引入 js 文件
>
>     ```js
>     <script src="js/axios-0.18.0.js"></script>
>     ```
>
>   * 发送 ajax 请求
>
>     * get 请求
>
>       ```js
>       axios({
>           method:"get",
>           url:"http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan"
>       }).then(function (resp) {
>           alert(resp.data);
>       })
>       ```
>
>     * post 请求
>
>       ```js
>       axios({
>           method:"post",
>           url:"http://localhost:8080/ajax-demo/axiosServlet",
>           data:"username=zhangsan"
>       }).then(function (resp) {
>           alert(resp.data);
>       })
>       ```
>
>     **整体页面代码如下：**
>
>     ```html
>     <!DOCTYPE html>
>     <html lang="en">
>     <head>
>         <meta charset="UTF-8">
>         <title>Title</title>
>     </head>
>     <body>
>             
>     <script src="js/axios-0.18.0.js"></script>
>     <script>
>         //1. get
>        /* axios({
>             method:"get",
>             url:"http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan"
>         }).then(function (resp) {
>             alert(resp.data);
>         })*/
>             
>         //2. post  在js中{} 表示一个js对象，而这个js对象中有三个属性
>         axios({
>             method:"post",
>             url:"http://localhost:8080/ajax-demo/axiosServlet",
>             data:"username=zhangsan"
>         }).then(function (resp) {
>             alert(resp.data);
>         })
>     </script>
>     </body>
>     </html>
>     ```
>



##### 9.4.3 请求方法别名

为了方便起见， Axios 已经为所有支持的请求方法提供了别名。如下：

* `get` 请求 ： `axios.get(url[,config])`

* `delete` 请求 ： `axios.delete(url[,config])`

* `head` 请求 ： `axios.head(url[,config])`

* `options` 请求 ： `axios.option(url[,config])`

* `post` 请求：`axios.post(url[,data[,config])`

* `put` 请求：`axios.put(url[,data[,config])`

* `patch` 请求：`axios.patch(url[,data[,config])`

而我们只关注 `get` 请求和 `post` 请求。

入门案例中可由 第一张的方式 改为 第二张图片：

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314223931444.png" alt="image-20230314223931444" style="zoom:67%;" />

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314223900549.png" alt="image-20230314223900549" style="zoom:67%;" />

```js
axios.get("http://localhost:8080/ajax-demo/axiosServlet?username=zhangsan").then(function (resp) {
    alert(resp.data);
});
```

```js
axios.post("http://localhost:8080/ajax-demo/axiosServlet","username=zhangsan").then(function (resp) {
    alert(resp.data);
})
```





![image-20230314224350325](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230314224350325.png)

#### 9.5 JSON

##### 9.5.1 概述

> - JavaScript Object Notation。JavaScript对象表示法
> - 由于其语法简单，层次结构鲜明，现多用于作为**数据载体**，在网络中进行数据传输。

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315091859699.png" alt="image-20230315091859699" style="zoom: 80%;" />

> 通过上面 js 对象格式和 json 格式进行对比，发现两个格式特别像。只不过 js 对象中的属性名可以使用引号（可以是单引号，也可以是双引号）；而 `json` 格式中的键要求必须使用双引号括起来，这是 `json` 格式的规定。
>
> 作用：由于其语法格式简单，层次结构鲜明，现多用于作为数据载体，在网络中进行数据传输。

##### 9.5.2 JSON 基础语法

![image-20230315092149865](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315092149865.png)



代码演示：

创建一个页面，在该页面的 `<script>` 标签中定义json字符串

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script>
    //1. 定义JSON字符串
    var jsonStr = '{"name":"zhangsan","age":23,"addr":["北京","上海","西安"]}'
    alert(jsonStr);

</script>
</body>
</html>
```

通过浏览器打开，页面效果如下图所示

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315093725532.png" alt="image-20230315093725532" style="zoom:67%;" />

现在我们需要获取到该 `JSON` 串中的 `name` 属性值，应该怎么处理呢？

如果它是一个 js 对象，我们就可以通过 `js对象.属性名` 的方式来获取数据。JS 提供了一个对象 `JSON` ，该对象有如下两个方法：

* `parse(str)`：将 JSON串转换为 js 对象。

  使用方式是：var jsObject = JSON.parse(jsonStr);`

* `stringify(obj)：将 js 对象转换为 JSON 串。

  使用方式是：var jsonStr=JSON.stringify(jsObject)

不过，在axios中，js对象和json对象会相互转换。



##### 9.5.3  JSON串和Java对象的相互转换

学习完 json 后，接下来聊聊 json 的作用。以后我们会以 json 格式的数据进行前后端交互。前端发送请求时，如果是复杂的数据就会以 json 提交给后端；而后端如果需要响应一些复杂的数据时，也需要以 json 格式将数据响应回给浏览器。

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315093055174.png" alt="image-20230315093055174" style="zoom:80%;" />

在后端我们就需要重点学习以下两部分操作：

* 请求数据：JSON字符串转为Java对象
* 响应数据：Java对象转为JSON字符串

接下来给大家介绍一套 API，可以实现上面两部分操作。这套 API 就是 `Fastjson`

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315101655490.png" alt="image-20230315101655490" style="zoom:80%;" />

Fastjson代码演示：

```java
    public static void main(String[] args) {
        //1.将java对象转为json字符串
        User user = new User();
        user.setId(1);
        user.setUsername("zhangsan");
        user.setPassword("123");

        String jsonString = JSON.toJSONString(user);
        System.out.println(jsonString);
        //{"id":1,"password":"123","username":"zhangsan"}

        //2.将json字符串转为java对象
        User u = JSON.parseObject("{\"id\":1,\"password\":\"123\",\"username\":\"zhangsan\"}",User.class);
        System.out.println(u);
    }
```



#### 9.6 案例

##### 9.6.1 需求

使用Axios + JSON 完成品牌列表数据查询和添加，页面效果还是如下所示：

![image-20230315104001567](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315104001567.png)

------



##### 9.6.2  查询所有功能

![image-20230315104113717](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315104113717.png)

前后端需以 JSON 格式进行数据的传递；由于此功能是查询所有的功能，前端发送 ajax 请求不需要携带参数，而后端响应数据需以如下格式的 json 数据

![image-20230315104204153](https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315104204153.png)

###### 1.  环境准备

> 使用已经写好mapper和service层的brand-demo项目，以下仅描述html和servlet的交互

###### 2.  后端实现

> 具体逻辑：
>
> * 调用 service 的 `selectAll()` 方法进行查询所有的逻辑处理
> * 将查询到的集合数据转换为 json 数据。我们将此过程称为 ***序列化***；如果是将 json 数据转换为 Java 对象，我们称之为 ***反序列化***
> * 将 json 数据响应回给浏览器。这里一定要设置响应数据的类型及字符集 `response.setContentType("text/json;charset=utf-8");`

`SelectAllServlet` 代码如下：

```java
@WebServlet(value = "/selectAllServlet")
public class SelectAllServlet extends HttpServlet {
    private BrandService brandService = new BrandService();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.调用Service查询
        List<Brand> brands = brandService.selectAll();

        //2.将集合转换为 JSON 数据      序列化
        String jsonString = JSON.toJSONString(brands);

        //3. 响应数据
        response.setContentType("text/json;charset=utf-8");
        response.getWriter().write(jsonString);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

###### 3.  前端实现

> 具体逻辑：
>
> - 引入 axios 的 js 文件：
>
>   ```js
>   <script src="js/axios-0.18.0.js"></script>
>   ```
>
> - 绑定页面加载完毕事件
>
>   在 `brand.html` 页面绑定加载完毕事件，该事件是在页面加载完毕后被触发，代码如下:
>
>   ```js
>   window.onload = function() {
>   	
>   }
>   ```
>
> - 发送异步请求：
>
>   在页面加载完毕事件绑定的匿名函数中发送异步请求，代码如下：
>
>   ```js
>    //2. 发送ajax请求
>   axios({
>       method:"get",
>       url:"http://localhost:8080/brand-demo/selectAllServlet"
>   }).then(function (resp) {
>   
>   });
>   ```
>
> - 处理相应数据：
>
>   在 `then` 中的回调函数中通过 `resp.data` 可以获取响应回来的数据，而数据格式如下
>
>   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315114951374.png" alt="image-20230315114951374" style="zoom:80%;" />
>
>   现在我们需要拼接字符串，将下面表格中的所有的 `tr` 拼接到一个字符串中，然后使用 `document.getElementById("brandTable").innerHTML = 拼接好的字符串`  就可以动态的展示出用户想看到的数据
>
>   <img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20230315115023879.png" alt="image-20230315115023879" style="zoom: 50%;" />
>
>   而表头行是固定的，所以先定义初始值是表头行数据的字符串，如下：
>
>   ```js
>   //获取数据
>   let brands = resp.data;
>   let tableData = " <tr>\n" +
>       "        <th>序号</th>\n" +
>       "        <th>品牌名称</th>\n" +
>       "        <th>企业名称</th>\n" +
>       "        <th>排序</th>\n" +
>       "        <th>品牌介绍</th>\n" +
>       "        <th>状态</th>\n" +
>       "        <th>操作</th>\n" +
>       "    </tr>";
>   ```
>
>   接下来遍历响应回来的数据 `brands` ，拿到每一条品牌数据
>
>   ```js
>   for (let i = 0; i < brands.length ; i++) {
>       let brand = brands[i];
>   	  
>   }
>   ```
>
>   紧接着就是从 `brand` 对象中获取数据并且拼接 `数据行`，累加到 `tableData` 字符串变量中
>
>   ```js
>   tableData += "\n" +
>       "    <tr align=\"center\">\n" +
>       "        <td>"+(i+1)+"</td>\n" +
>       "        <td>"+brand.brandName+"</td>\n" +
>       "        <td>"+brand.companyName+"</td>\n" +
>       "        <td>"+brand.ordered+"</td>\n" +
>       "        <td>"+brand.description+"</td>\n" +
>       "        <td>"+brand.status+"</td>\n" +
>       "\n" +
>       "        <td><a href=\"#\">修改</a> <a href=\"#\">删除</a></td>\n" +
>       "    </tr>";
>   ```
>
>   最后再将拼接好的字符串写到表格中
>
>   ```js
>   // 设置表格数据
>   document.getElementById("brandTable").innerHTML = tableData;
>   ```

brand.html 页面如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="addBrand.html"><input type="button" value="新增"></a><br>
<hr>
<table id="brandTable" border="1" cellspacing="0" width="100%">
</table>

<script src="js/axios-0.18.0.js"></script>
<script>
    //1.当页面加载完成后，发送ajax请求
    window.onload = function () {
        //1.2 发送ajax 请求
        axios({
            method:"get",
            url:"http://localhost:8081/selectAllServlet"
        }).then(function (resp){
            //获取数据
            let brands = resp.data;
            let tableData = "    <tr>\n" +
                "        <th>序号</th>\n" +
                "        <th>品牌名称</th>\n" +
                "        <th>企业名称</th>\n" +
                "        <th>排序</th>\n" +
                "        <th>品牌介绍</th>\n" +
                "        <th>状态</th>\n" +
                "        <th>操作</th>\n" +
                "    </tr>";
            for (let i = 0; i < brands.length; i++) {
                let brand = brands[i];


                tableData += "    <tr align=\"center\">\n" +
                    "        <td>"+(i+1)+"</td>\n" +
                    "        <td>"+brand.brandName+"</td>\n" +
                    "        <td>"+brand.companyName+"</td>\n" +
                    "        <td>"+brand.ordered+"</td>\n" +
                    "        <td>"+brand.description+"</td>\n" +
                    "        <td>"+brand.status+"</td>\n" +
                    "\n" +
                    "        <td><a href=\"#\">修改</a> <a href=\"#\">删除</a></td>\n" +
                    "    </tr>";
            }

            //设置表格数据
            document.getElementById("brandTable").innerHTML = tableData;
        })

    }
    
    
</script>

</body>
</html>
```



------



##### 9.6.3  添加品牌功能

###### 1. 后端实现

在 `com.duck.web` 包下创建名为 `AddServlet` 的 `servlet`，具体的逻辑如下：

* 获取请求参数

  由于前端提交的是 json 格式的数据，所以我们不能使用 `request.getParameter()` 方法获取请求参数

  * 如果提交的数据格式是 `username=zhangsan&age=23` ，后端就可以使用 `request.getParameter()` 方法获取
  * 如果提交的数据格式是 json，后端就需要通过 request 对象获取输入流，再通过输入流读取数据 

* 将获取到的请求参数（json格式的数据）转换为 `Brand` 对象

* 调用 service 的 `add()` 方法进行添加数据的逻辑处理

* 将 json 数据响应回给浏览器。

`AddServlet` 代码如下：

```java
@WebServlet("/addServlet")
public class AddServlet extends HttpServlet {

    private BrandService brandService = new BrandService();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        //1. 接收数据,request.getParameter 不能接收json的数据
       /* String brandName = request.getParameter("brandName");
        System.out.println(brandName);*/

        // 获取请求体数据
        BufferedReader br = request.getReader();
        String params = br.readLine();
        // 将JSON字符串转为Java对象
        Brand brand = JSON.parseObject(params, Brand.class);
        //2. 调用service 添加
        brandService.add(brand);
        //3. 响应成功标识
        response.getWriter().write("success");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

###### 2. 前端实现

在 `addBrand.html` 页面给 `提交` 按钮绑定点击事件，并在绑定的匿名函数中发送异步请求，代码如下：

```js
//1. 给按钮绑定单击事件
document.getElementById("btn").onclick = function () {
    //2. 发送ajax请求
    axios({
        method:"post",
        url:"http://localhost:8080/brand-demo/addServlet",
        data:formData
    }).then(function (resp) {
        // 判断响应数据是否为 success
        if(resp.data == "success"){
            location.href = "http://localhost:8080/brand-demo/brand.html";
        }
    })
}
```

现在我们只需要考虑如何获取页面上用户输入的数据即可。

首先我们先定义如下的一个 js 对象，该对象是用来封装页面上输入的数据，并将该对象作为上面发送异步请求时 `data` 属性的值。

```js
// 将表单数据转为json
var formData = {
    brandName:"",
    companyName:"",
    ordered:"",
    description:"",
    status:"",
};
```

接下来获取输入框输入的数据，并将获取到的数据赋值给 `formData` 对象指定的属性。比如获取用户名的输入框数据，并把该数据赋值给 `formData` 对象的 `brandName` 属性

```js
// 获取表单数据
let brandName = document.getElementById("brandName").value;
// 设置数据
formData.brandName = brandName;
```

说明：其他的输入框都用同样的方式获取并赋值。  但是有一个比较特殊，就是状态数据，如下图是页面内容

<img src="https://typora-fcc.oss-cn-beijing.aliyuncs.com/pictures-PicGo/image-20210831103843798.png" alt="image-20210831103843798" style="zoom:80%;" />

我们需要判断哪儿个被选中，再将选中的单选框数据赋值给 `formData` 对象的 `status` 属性，代码实现如下：

```js
let status = document.getElementsByName("status");
for (let i = 0; i < status.length; i++) {
    if(status[i].checked){
        //
        formData.status = status[i].value ;
    }
}
```

**整体页面代码如下：**

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>添加品牌</title>
</head>
<body>
<h3>添加品牌</h3>
<form action="" method="post">
    品牌名称：<input id="brandName" name="brandName"><br>
    企业名称：<input id="companyName" name="companyName"><br>
    排序：<input id="ordered" name="ordered"><br>
    描述信息：<textarea rows="5" cols="20" id="description" name="description"></textarea><br>
    状态：
    <input type="radio" name="status" value="0">禁用
    <input type="radio" name="status" value="1">启用<br>

    <input type="button" id="btn"  value="提交">
</form>

<script src="js/axios-0.18.0.js"></script>

<script>
    //1. 给按钮绑定单击事件
    document.getElementById("btn").onclick = function () {
        // 将表单数据转为json
        var formData = {
            brandName:"",
            companyName:"",
            ordered:"",
            description:"",
            status:"",
        };
        // 获取表单数据
        let brandName = document.getElementById("brandName").value;
        // 设置数据
        formData.brandName = brandName;

        // 获取表单数据
        let companyName = document.getElementById("companyName").value;
        // 设置数据
        formData.companyName = companyName;

        // 获取表单数据
        let ordered = document.getElementById("ordered").value;
        // 设置数据
        formData.ordered = ordered;

        // 获取表单数据
        let description = document.getElementById("description").value;
        // 设置数据
        formData.description = description;

        let status = document.getElementsByName("status");
        for (let i = 0; i < status.length; i++) {
            if(status[i].checked){
                //
                formData.status = status[i].value ;
            }
        }
        //console.log(formData);
        //2. 发送ajax请求
        axios({
            method:"post",
            url:"http://localhost:8080/brand-demo/addServlet",
            data:formData
        }).then(function (resp) {
            // 判断响应数据是否为 success
            if(resp.data == "success"){
                location.href = "http://localhost:8080/brand-demo/brand.html";
            }
        })
    }
</script>
</body>
</html>
```

==说明：==

`查询所有` 功能和 `添加品牌` 功能就全部实现，大家肯定会感觉前端的代码很复杂；而这只是暂时的，后面学习了 `vue` 前端框架后，这部分前端代码就可以进行很大程度的简化。







### 10，Vue&Element(未完成的)











