# WEB 开发



##拦截器

### SpringBoot 中使用拦截器

1. 创建一个类，实现HandlerIntercept接口
2. 注册拦截器对象，采用一个配置文件类实现WebMvcConfigurer接口，重写addIntercepts方法，在该方法中添加拦截器对象

### SpringMVC中使用拦截器

1. 创建类，实现HandlerIntercept接口
2. 在SpringMVC配置文件中声明拦截器对象



---





## Servlet

### SpringBoot 中使用Servlet

1. 创建类，继承HttpServlet，重写doGet或doPost方法
2. 在配置类中添加一个返回值是ServletRegistrationBean的方法，返回的是用于注册Servlet对象的类，该类的构造方法需要传递一个Servlet对象和该Servlet所管理的请求地址；也可以采用set方法来设置。



### SpringMVC中使用Servlet

​	springMVC中不推荐使用Servlet来处理请求，通常是采用中央处理器这一个Servlet来转发其他的请求，如果要使用自己写的Servlet，使用步骤与普通web项目中一致



### 普通web项目中使用Servlet

1. 创建类，继承HttpServlet，重写doGet或doPost方法
2. 在web项目的核心配置文件web.xml中声明该Servlet



---





## 过滤器

### SpringBoot中使用过滤器

1. 创建类，实现Filter接口，重写doFilter方法
2. 在配置类中注册自定义过滤器对象，写一个返回值为FilterRegistrationBean的方法，返回值就是用来注册过滤器的一个对象，给该对象配置要注册的过滤器和过滤器所要管理的路径

SpringBoot中使用字符集过滤器：

![1644300834770](C:\Users\wjs\AppData\Local\Temp\1644300834770.png)

注意：由于SpringBoot中已经默认设置了CharacterEncodingFilter，字符编码为ISO-8859-1，如果要自己配置一个字符集过滤器，需要在SpringBoot配置文件中声明不适用默认的字符集过滤器

![1644300984483](C:\Users\wjs\AppData\Local\Temp\1644300984483.png)



还可以直接修改默认的字符集过滤器：

![1644301071231](C:\Users\wjs\AppData\Local\Temp\1644301071231.png)



### SpringMVC中使用过滤器

​	与普通web项目一致

注意：在SpringMVC中为了防止请求参数和响应参数的乱码，采用框架自带的一个过滤器CharacterEncodingFilter

![1644299907892](C:\Users\wjs\AppData\Local\Temp\1644299907892.png)



### 普通web项目使用过滤器

1. 创建类，实现Filter接口，重写doFilter方法
2. 在web.xml文件中声明过滤器



### RequestBody和ResponseBody

RequestBody注解用于接受json字符串，并把它转换成对象

ResponseBody注解作用是将结果写入到htpp响应包的响应体中

RestController注解写在Controller上面，表示Controller中的所有方法都带上一个ResponseBody注解

在springMVC中，配置文件中添加了注解驱动之后，会自动引入消息转换器，可以将java对象转换成json格式的字符串，然后再配合ResponseBody注解写入到响应协议包中



---







# ORM操作数据库



## 使用mybatis

1. 在项目中添加mybatis起步依赖
2. 若不将mapper文件与dao接口分开存放，需要在pom.xml文件中指定将mapper文件包含到classpath中
3. 写application.properties文件，配置数据库连接信息

数据库配置信息：

![1644303023461](C:\Users\wjs\AppData\Local\Temp\1644303023461.png)





## 创建dao接口实现类的两种方式

### 1、使用@Mapper注解

在dao接口上使用@Mapper注解

```java
@Mapper
public interface StudentDao{
    int insert(Student sutdent);
}
```

若使用该方法，每个dao接口上都要添加上@Mapper注解，比较麻烦



### 2、使用@MapperScan注解

在配置文件类上使用，使用注解的属性basePackages来指定要创建代理对象的dao接口位置

![1644305336679](C:\Users\wjs\AppData\Local\Temp\1644305336679.png)

可以指定多个包

![1644305385248](C:\Users\wjs\AppData\Local\Temp\1644305385248.png)





### Mapper文件与Dao接口分开管理

如果为了方便管理，可以将Mapper文件放到resources目录下，分开管理的步骤如下：

1. 在resources目录下创建一个mapper目录，将mapper文件放入其中

2. 在application.properties文件中指定mapper文件位置

   ```properties
   #指定mapper文件位置
   mybatis.mapper-locations=classpath:mapper/*.xml
   #指定mybatis的日志
   mybaits.configuration.log-impl=org.apache.ibatis.logging.stdout.StOutImpl
   ```

3. 在pom.xml文件中指定把resources目录中的文件编译到目标目录中





## 事务



### Spring框架中的事务

####  1、注解式声明事务，适合中小项目中使用

使用步骤：以下两步都在spring配置文件(xml)中完成

1. 声明事务管理器对象

   ```xml
   <bean id="transactionManager" 
         class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="datasource" ref="数据源的id"/>
   </bean>
   ```

2. 开启事务注解驱动

   ```xml
   <tx:annotation-driven transaction-manager="transactionManager"/>
   ```

3. 配置完以上两步之后就可以在方法上使用@Transactional注解来配置事务，指定事务的隔离级别，传播行为，超时等



#### 2、在Spring配置文件中声明事务，方法和事务配置完全隔离，适合大型项目使用

使用步骤：

1. 要使用aspectj框架(AOP面向切面编程)，导入aspectj依赖

2. 声明事务管理器对象(与第一种方式一致)

   ```xml
   <bean id="transactionManager" 
         class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="datasource" ref="数据源的id"/>
   </bean>
   ```

3. 声明方法需要的事务类型

   ```xml
   <tx:advice id="myAdvice" transactionManager="transactionManager">
      <!--attributes:配置事务属性--> 
       <tx:attributes>
           <!--method:给具体的方法配置事务，可以有多个-->
           <!--name:方法名称，可以带有通配符，不带报名和类名
   		   propagation:传播行为
   		   isolation:隔离级别
   		   rollback-for:指定异常的类名，发生该异常时事务回滚
   		-->
           <tx:method name="" propagation="" isolation="" rollback-for=""/>
       </tx:attributes>
   </tx:advice>
   ```

4. 配置aop：指定哪些类要创建代理

   ```xml
   <aop:config>
       <!--切入点表达式，指定哪个类中的方法使用事务-->
       <aop:point-cut id="servicePT" expression="execution(* *..service..*.*(..))" />
       <!--配置增强器，关联advice和pointcut-->
       <aop:advisor advice-ref="myAdvice" pointcut-ref="servicePT" />
   </aop:config>
   ```

   

   

   ### SpringBoot中使用事务

   在SpringBoot中支持使用Spring框架中的两种事务处理方式

   使用注解式事务：

   1. 在配置类上加上注解@EnableTransactionManagement，启动事务管理器
   2. 在要做事务管理的方法上添加@Transactional注解