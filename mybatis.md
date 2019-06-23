# mybatis 学习笔记

![1556284748821](mybaties_images/1556284748821.png)

## 一、入门案例



### 1. 配置文件设置

```xml
<!--pom.xml-->
<dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.1</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.42</version>
    </dependency>
```



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="mybatis">
        <environment id="mybatis">
            <!--配置四个基本-->
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <!--指向本地MySQ的Lmybatis数据库 -->
                <property name="url" value="jdbc:mysql:///mybatis" />
                <property name="username" value="root" />
                <property name="password" value="123456" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--映射Mapper文件-->
       <mapper resource="mapper/userMapper.xml" />
    </mappers>
</configuration>
```

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace 命名空间，要指向dao类（一些也叫mapper类）-->
<mapper namespace="cn.forlkc.dao.Userdao">
    <!--
    id： 为方法名
    resultType: 为返回类型
    parameterType： 为参数类型
    -->
    <select id="findAll" resultType="cn.forlkc.entitis.User">
        select * from user
    </select>

    <select id="findById" resultType="cn.forlkc.entitis.User" parameterType="int">
        select * from user where userId = #{id}
    </select>
    
    <delete id="deleteById" parameterType="int">
        delete from user where userId = #{id}
    </delete>
    
    <insert id="insert" parameterType="cn.forlkc.entitis.User">
        insert into user(userName,password) value (#{userName}, #{password})
    </insert>

    <update id="update" parameterType="cn.forlkc.entitis.User">
        update user set password = #{password} where userName = #{userName}
    </update>
</mapper>
```

### 2. 测试文件及事务管理

```java
public void testMybatis() throws IOException {
    //开启事务
        String resource = "sqlMapConfig.xml";
        InputStream inputStream = null;
        try {
             /**
   * Returns a resource on the classpath as a Stream object
   *
   * @param resource The resource to find
   * @return The resource
   * @throws java.io.IOException If the resource cannot be found or read
   */
            inputStream = Resources.getResourceAsStream(resource);
        } catch (IOException e) {
            e.printStackTrace();
        }
    //创建session工厂
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    //开启事务
        SqlSession session = sqlSessionFactory.openSession();
    
//方法1：获取mapper，利用mapper直接操作数据库
        Userdao userdao = session.getMapper(Userdao.class);
        userdao.deleteById(1);
        User use = new User();
//方法2：直接事务查询；cn.forlkc.dao.Userdao：命名空间；findById：方法
    //直接配置，不需要在dao层写接口方法
        use = session.selectOne("cn.forlkc.dao.Userdao.findById",2);
        System.out.println(use);
        use.setPassword("159");
        use.setUserName("asdasdasd");
        userdao.insert(use);
        use.setPassword("123456");
        userdao.update(use);
        List<User> users = userdao.findAll();
        users.add(userdao.findById(2));
        for (User user : users){
            System.out.println(user);
        }

		//事务关闭
        session.close();
        inputStream.close();
    }
```

```java
//事务接口里面的方法
public interface SqlSession extends Closeable {
    <T> T selectOne(String var1);

    <T> T selectOne(String var1, Object var2);

    <E> List<E> selectList(String var1);

    <E> List<E> selectList(String var1, Object var2);

    <E> List<E> selectList(String var1, Object var2, RowBounds var3);

    <K, V> Map<K, V> selectMap(String var1, String var2);

    <K, V> Map<K, V> selectMap(String var1, Object var2, String var3);

    <K, V> Map<K, V> selectMap(String var1, Object var2, String var3, RowBounds var4);

    <T> Cursor<T> selectCursor(String var1);

    <T> Cursor<T> selectCursor(String var1, Object var2);

    <T> Cursor<T> selectCursor(String var1, Object var2, RowBounds var3);

    void select(String var1, Object var2, ResultHandler var3);

    void select(String var1, ResultHandler var2);
     /**
   * Retrieve a single row mapped from the statement key and parameter
   * using a {@code ResultHandler} and {@code RowBounds}.
   * @param statement Unique identifier matching the statement to use.
   * @param rowBounds RowBound instance to limit the query results
   * @param handler ResultHandler that will handle each retrieved row
   */

   void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler);

    int insert(String var1);

    int insert(String var1, Object var2);

    int update(String var1);

    int update(String var1, Object var2);

    int delete(String var1);

    int delete(String var1, Object var2);

    void commit();

    void commit(boolean var1);

    void rollback();

    void rollback(boolean var1);

    List<BatchResult> flushStatements();

    void close();

    void clearCache();

    Configuration getConfiguration();

    <T> T getMapper(Class<T> var1);

    Connection getConnection();
}
```

