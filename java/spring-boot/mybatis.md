# 配置

## 工具引入



首先是通过maven将mybatis引入到项目里。直接修改```pom.xml```就好了

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```

我使用的工具是vscode，在下面有个java project那里可以看到自己有没有导入成功



## 数据库的配置

在项目的配置文件里面连接数据库，直接修改``` application.properties```就好了

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/springdb?useUnicode=true&characterEncoding=utf8&useSSL=true
spring.datasource.username=iwan
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

这样字就可以在项目里面用了



# 注解使用

## entity

在enetity里面写的类差不多数据库里存放数据的格式一样，利用lombok的@Data为他生成set get 和toString的方法就行了，这个大概算是个bean

## service

在service上也要加上service的注解，这样才能被springboot识别到，然后再controller里被@Autowired装配

在这里写对数据库进行的操作，什么增删查改啥的，注意要使用@Autowired 这个注解可以实现自动装配，通过它来初始化一个mapper的对象。

## mapper

@mapper 或者@mapperscan者两种方式都可以，

mapper写的就是对数据库的操作了，通过@Select，@Insert这个注解来实现，

要**注意**的是在注解里面的“ ”里不能再有单引号否则会报错什么sql参数错误什么的异常。

这个东西有两种占位符 #和$ 再用#的时候会匹配数据的类型 所以在插入值的时候会比较方便，$是作为字符串拼接的所以 在不同的表名列名使用是比较方便

> MyBatis处理 #{ } 占位符，使用的 JDBC 对象是 PreparedStatement 对象，执行sql语句的效率更高。
> 使用 PreparedStatement 对象，能够避免 sql 注入，使得sql语句的执行更加安全。
> #{ } 常常作为列值使用，位于sql语句中等号的右侧；#{ } 位置的值与数据类型是相关的。

>MyBatis处理 ${ } 占位符，使用的 JDBC 对象是 Statement 对象，执行sql语句的效率相对于 #{ } 占位符要更低。
>${ } 占位符的值，使用的是字符串连接的方式，有 sql 注入的风险，同时也存在代码安全的问题。
>${ } 占位符中的数据是原模原样的，不会区分数据类型。
>${ } 占位符常用作表名或列名，这里推荐在能保证数据安全的情况下使用 ${ }。
