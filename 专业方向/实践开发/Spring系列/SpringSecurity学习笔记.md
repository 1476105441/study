# SpringSecurity学习笔记



## @EnableWebSecurity注解

作用有两个：

   	 1. 加载了WebSecurityConfiguration配置类, 配置安全认证策略。 
     2. 加载了AuthenticationConfiguration, 配置了认证信息。 



是否要使用这个注解：

 * 如果项目中未添加start，要启用SpringSecurity得要加上这个注解

* 如果项目中添加了start，既不需要添加SpringSecurity注解了，以 spring-boot-start-security为例，在SecurityAutoConfiguration这个类中，看到导入SpringBootWebSecurityConfiguration这个类 

  原文链接：https://blog.csdn.net/mingtiandexia/article/details/88910370?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-2-88910370.pc_agg_new_rank&utm_term=%E4%B8%8D%E5%8A%A0EnableWebSecurity&spm=1000.2123.3001.4430



## WebSecurityConfigurerAdapter类

> 此处我有一个疑问，是不是一定要用配置类继承这个类？还是普通类也可以？配置类继承一定要加上@EnableWebSecurity注解吗？

### 作用：

  * WebSecurityConfigurerAdapter 类是个适配器, 在配置的时候,需要我们自己写个配置类去继承他,然后编写自己所特殊需要的配置，注意，这个配置类上面可以加上@EnableWebSecurity注解



### 使用方法：

* 编写一个自己的配置类继承WebSecurityConfigurerAdapter，重写configure方法来配置自己的认证信息和认证策略

	​	



在观看学习视频时发现这么一个现象：

	* 当安全配置上方没有添加@EnableWebSecurity注解时，如下图：

![1646656419061](D:\wjs\Documents\SpringSecurity学习笔记.assets\1646656419061.png)

​	启动项目后，访问资源文件时是需要登陆验证的

* 当安全配置类上方添加了@EnableWebSecurity注解时，如下图：

![1646656533772](D:\wjs\Documents\SpringSecurity学习笔记.assets\1646656533772.png)

​	必须要将super那部分放开，才会启用安全认证功能

> 综上，我得出结论：
>
> ​	当你没有重新提供一个安全配置类时，框架使用内部自带的安全配置类，所以不需要你再配置一些安全信息，只需要写新增的安全配置即可。
>
> ​	但是当你提供了一个安全配置类之后，会覆盖掉框架自带的安全配置类，此时你就需要使用super的方法来做一些安全配置。





## @PostConstruct注解

* 该注解不是spring中的注解，而是Java自带的注解

* Java中该注解的说明：@PostConstruct该注解被用来修饰一个非静态的void（）方法。被@PostConstruct修饰的方法会在服务器加载[Servlet](https://so.csdn.net/so/search?q=Servlet&spm=1001.2101.3001.7020)的时候运行，并且只会被服务器执行一次。PostConstruct在构造函数之后执行，init（）方法之前执行。 

* 通常我们会是在Spring框架中使用到@PostConstruct注解 该注解的方法在整个Bean初始化中的执行顺序：

  Constructor(构造方法) -> @Autowired(依赖注入) -> @PostConstruct(注释的方法)

附上一篇该注解的详细说明：https://blog.csdn.net/qq360694660/article/details/82877222





### 安全配置类中的方法

**requestMatchers()** 与**authorizeRequests()** 区别：https://blog.csdn.net/join_null/article/details/119390280



authorizeRequests().antMathers("**").anonymous() : 

​		匿名访问，未登录状态下可以访问，已登录状态下不能访问

authorizeRequests().antMathers("**").permitAll():





### Authentication对象

spring security中的对象，可以获取登录的用户信息

详细说明在此：https://www.cnblogs.com/summerday152/p/13636285.html





### 修改用户权限

https://blog.csdn.net/liuyuncd/article/details/117551867