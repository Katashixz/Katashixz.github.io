---
title: Spring框架知识点整理
date: 2022-07-15 17:31:24
tags:
categories: 自主学习
cover: false
---
# 为了融入公司而学Spring...
## IOC
1. 概念
    - 控制反转IoC(Inversion of Control)是一个概念，是一种思想。
    - 由Spring容器进行对象的创建和依赖注入，程序员在使用时直接取出使用。
    - 和正转概念中，由程序员说了算不同。

## 项目搭建
1. Maven->quickstart
1. resources新建，修改专有属性,删掉启动文件
1. pom里maven.compiler.source,maven.compiler.target改成1.8
1. 添加spring依赖
    ```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.5.RELEASE</version>
    </dependency>
    ```
1. build里删除 加上
    ```xml
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
            </includes>
        </resource>
    </resources>
    ```

1. 基于xml的IOC，使用spring容器创建对象并取出对象 
    1. 创建学生对象实体Student
    1. 在配置文件中创建对象
        ```xml
        <bean id="stu" class="com.study.pojo.Student"></bean>
        ```
    1. 取出对象
        ```java
        ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
        Student stu = (Student) ac.getBean("stu");
        ```
    1. 给创建的对象赋值
        1. 使用setter注入——必须提供无参的构造方法,必须提供setXXX()方法
            - 简单类型注入，使用value属性
                ```xml
                <bean id="stu" class="com.study.pojo.Student">
                <property name="name" value="张三"></property>
                <property name="age" value="12"></property>
                </bean>
                ``` 
            - 引用类型注入，使用ref属性
                ```xml
                <!--创建学校对象-->
                <bean id="school" class="com.study.pojo.school">
                    <property name="name" value="清华大学"></property>
                    <property name="addres" value="海淀区"></property>
                </bean>
                <!--创建学生对象-->
                <bean id="stu" class="com.study.pojo.Student">
                    <property name="name" value="张三"></property>
                    <property name="age" value="12"></property>
                    <property name="school" ref="school"></property>
                </bean>
                ``` 
        1. 使用构造方法注入
            ```java
            Student stu = new Student("张三",22)
            ```
            1. 使用构造方法的参数名称进行注入值
                - pojo中没有无参构造，只有有参构造
                ```xml
                <bean id="school" class="com.study.pojo.school">
                    <constructor-arg name="name" value="清华大学"></constructor-arg>
                    <constructor-arg name="addres" value="海淀区"></constructor-arg>
                </bean>
                ```
                - 构造方法的参数名称必须和pojo的名称**一致**
            1. 使用构造方法参数的下标注入值
                - pojo中没有无参构造，只有有参构造
                ```xml
                <!--创建学生对象-->
                <bean id="stu" class="com.study.pojo.Student">
                    <constructor-arg index="0" value="王五"></constructor-arg>
                    <constructor-arg index="1" value="22"></constructor-arg>
                    <constructor-arg index="2" ref="school"></constructor-arg>
                </bean>
                ```
            1. 使用默认的构造方法的参数顺序注入值
                - 按照默认的顺序
                 ```xml
                <!--创建学生对象-->
                <bean id="stu" class="com.study.pojo.Student">
                    <constructor-arg value="王五"></constructor-arg>
                    <constructor-arg value="22"></constructor-arg>
                    <constructor-arg ref="school"></constructor-arg>
                </bean>
                ```


## 基于注解的IOC
- 也称为DI(Dependency Injection)，是IOC的具体实现的技术。
- **基于注解的IOC，必须要在applicationContext.yml里添加包扫描**
    ```xml
    <context:component-scan base-package="所在位置">
    ```
1. 创建对象的注解
    - @Component
        - 可以创建任意对象,创建的对象的默认名称是类名的**驼峰命名法**，也可以指定对象的名称@Component("指定名称")
    - @Controller
        - 专门用来创建控制器的对象，这种对象可以接收用户的请求，可以返回处理结果给客户端
    - @Service
        - 专门用来创建业务逻辑层的对象，负责向下访问数据访问层，处理完毕后的结果返回给界面层
    - @Repository
        - 专门用来创建数据访问层的对象，负责数据库中的增删改查所有操作 
1. 依赖注入的注解
    - 简单类型的注入
        - @Value
            - 用来给简单类型注入值
            ```java
            @Value("清华大学")
            private String name;
            ```
    - 引用类型的注入
        - @Autowired
            - 使用类型注入值，从整个Bean工厂中搜索同源类型的对象进行注入。
            - **注意**
                - 在有父子类的情况下，使用按类型注入，就意味着有多个可注入的对象，此时按照名称进行二次筛选，选中与被注入对象相同名称的对象进行注入
        - @Autowired+@Qualifier
            - 使用名称注入值，从整个Bean工厂中搜索相同名称的对象进行注入。
            - **注意**
                - 如果有父子类的情况下，直接按名称进行注入值 
1. 为应用指定多个Spring配置文件
    1. 拆分配置文件的策略
        1. 按层拆
            - applicationContext_controller.xml
            - applicationContext_service.xml
            - applicationContext_mapper.xml
        1. 按功能拆
            - applicationContext_users.xml 里面有userController,userService,userMapper
        1. 拆完之后单独写一个配置文件整合
            - 批量导入
            ```xml
            <import resource="applicationContext_*.xml"></import>
            ```

## 面向切面编程AOP
- AOP (Aspect Orient Programming)
- 切面
    - 公共的，通用的，重复的功能称为切面，面向切面编程就是将切面提取出来，单独开发，在需要调用的方法中通过动态代理的方式进行植入
- 常见术语
    1. 切面
        - 就是那些重复的，公共的，通用的功能称为切面，例如：日志、事务、权限
    1. 连接点
        - 就是目标方法，因为在目标方法中要实现目标方法的功能和切面功能
    1. 切入点
        - 多个连接点构成切入点，切入点可以说一个目标方法，可以使一个类中的所有方法， 可以是某个包下所有类中的方法切面在切入点切入
    1. 目标对象
        - 操作谁，谁就是目标对象
    1. 通知
        - 来指定切入的时机，是在目标方法执行前还是执行后还是出错时，还是环绕目标方法切入切面功能
- AspectJ框架
    - 优秀的面向切面的框架
    - 常见通知类型
        1. 前置通知@Before
        1. 后置通知@AfterReturning
        1. 环绕通知@Around
        1. 最终通知@After
        1. 定义切入点@PointCut
    - 切入点表达式
        - execution(访问权限 方法返回值 方法声明(参数) 异常类型)
        - 简化 -> execution(访问权限 方法声明(参数))
        - 用到的符号
            - \* 代码任意个任意的字符(通配符)
            - .. 如果出现在方法的参数中就代表任意参数，出现在路径中则代表本路径及其所有的子路径
        - 示例
            - execution(public * *(..)) 任意的公共方法
            - execution(* set*(..)) 任何一个以set开始的方法
            - execution(* com.xyz.service.impl.*.*(..)) 这个包下的任意类中的任意方法
            - execution(* com.xyz.service..*.*(..)) 这个包及其子包下的任意类中的任意方法 com.xyz.service.a com.xyz.service.b
            - execution(* *..service.*.*(..)) service之前可以有任意的子包
            - execution(* *.service.*.*(..)) service之前只有一个包
- 前置通知@Before使用
    - 实现步骤
        1. 创建业务接口
        1. 创建业务实现
        1. 创建切面类，实现切面方法
        1. 在applicationContext.xml文件中进行切面绑定
    - 前置通知切面方法的规范
        1. 访问权限是public
        1. 方法的返回值是void
        1. 方法名称自定义
        1. 方法没有参数，如果有也只能是JoinPoint类型
        1. 必须使用@Before来注解声明切入时机和切入点
            - 参数value 指定切入点表达式
- AspectJ框架切换JDK动态代理和CGLib动态代理
    ```xml
    <!--默认是JDK动态代理，取出时必须使用接口类型-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy> 
    <!--设置为CGLib子类代理-->
    <aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>
    ```
    - 记住使用接口来接永远不会出错
- 后置通知@AfterReturning 
    - 后置通知方法的规范
        1. 访问权限是public
        1. 方法没有返回值
        1. 方法名称自定义
        1. 方法有参数(也可以没有参数，如果目标方法没有返回值，则可以写无参的方法。这个切面方法的参数就是目标方法的返回值)
        1. 使用@AtterReturn来表明是后置通知
            - 参数value指定切入点表达式，returning 指定目标方法的返回值名称，则名必须与切面方法的参数名称一致
    - **重要**
        - 如果目标方法的返回值是8种基本类型或String类型，则不可在后置通知中改变
        - 如果是引用类型，则可以改变
- 环绕通知@Around
    - 他是通过拦截目标方法的方式，在目标方法前后增强功能的通知，是功能最强大的通知，一般事务使用此通知，它可以轻易改变目标方法的返回值。
    - 环绕通知方法的规范
        1. 访问权限是public
        1. 切面方法有返回值，此返回值就是目标方法的返回值
        1. 方法名称自定义
        1. 方法有参数，此参数就是目标方法
        1. 回避异常
        1. 使用@Around声明是环绕通知
            - 参数value指定切入点表达式
    - 代码实现，ProceedingJoinPoint用于接收方法
    ```java
    @Around(value = "execution(* com.study.s03.*.*(..))")
    public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("前切功能实现........");
        Object object = pjp.proceed(pjp.getArgs());
        System.out.println("后切功能实现........");
        return object.toString().toUpperCase();
    }
    ```
- 最终通知@After
    - 无论目标方法是否正常执行，最终通知的代码都会被执行
    - 最终通知方法的规范
        1. 访问权限是public
        1. 方法没有返回值
        1. 方法名称自定义
        1. 方法没有参数，如果有也只能是JoinPoint类型
        1. 使用@After注解表明是最终通知
            - 参数value指定切入点表达式
- @Pointcut 定义切入点别名
    - 当较多的通知增强方法使用相同的切入点表达式(execution)时，编写维护较为麻烦。所以提供了这个注解，相当于封装一下。

## 事务相关知识
- 查看源码
    - SM整合步骤
        - ![](../image/自主学习/SM%E6%95%B4%E5%90%88%E6%AD%A5%E9%AA%A4.png)
- 基于注解的事务添加步骤
    1. 在applicationContext_service.xml中添加事务管理器
    ```xml
    <!--1.添加事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--事务必须关联数据库处理，所以要配置数据源-->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    ```
    1. 在applicationContext_service.xml中添加事务的注解驱动
    ```xml
    <!--2.添加事务的注解驱动-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
    ```
    1. 在业务逻辑的实现类上添加注解@Transactional(propagation = Propagation.REQUIRED)，REQUIRED表示增删改操作时必须添加的事务传播特性
- Spring的两种事务处理方式
    1. 注解式的事务(如上)
        - 弊端
            - 若一个类里有五六十个方法，并且增删改查都有，只能在方法上添加注解，会累死人
    1. 声明式事务(必须掌握)
        - 在配置文件中添加一次，整个项目遵循事务的设定
- 面试易考**Spring中事务的五大隔离级别**
    1. 未提交读(Read Uncommitted)：允许脏读，也就是可能读取到其他会话中未提交事务修改的数据
    1. 提交读(Read Committed)：只能读取到已经提交的数据。Oracle等多数数据库默认都是该级别(不重复读)
    1. 可重复读(Repeated Read)：可重复读。在同一个事务内的查询都是事务开始时刻一致的，InmoDB默认级别。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻象读，但是innoDB解决了幻读
    1. 串行读(Serializable)：完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞
    1. 使用数据库默认的隔离级别isolation=Isolation.DEFAULT
        - MySQL: mysql默认的事务处理级别是'REPEATABLE-READ',也就是可重复读
        - Oracle:oracle 数据库支持READ,COMMITTED和SERIALIZABLE这两种事务隔离级别。
- 事务管理器用来生成相应技术的连接+执行语句的对象
    - 如果使用MyBatis，必须使用DataSourceTransactionManager类完成处理 
- 事物的传播特性解析
    - 特性
        1. PROPAGATION_REQUIRED
            - 必备包含事务(增删改必用)
        1. PROPAGATION_REQUIRES_NEW
            - 自己新开事务，不管之前是否有事务
        1. PROPAGATION_SUPPORTS
            - 支持事务，如果加入的方法有事务，则支持，如果没有，不单开事务
        1. PROPAGATION_NERVER
            - 不能运行在事务中，如果包在事务中，抛异常
        1. PROPAGATION_NOT_SUPPORTED
            - 不支持事务，运行在非事务环境。如果包在事务中会暂时挂起事务
        1. PROPAGATION_MANDATORY
            - 必须包在事务中，没有事务则抛异常
        1. PROPAGATION_NESTED
            - 嵌套事务
    - 情况为UsersServiceImpl中insert业务嵌套Accounts中的insert业务，Accounts中的insert里有1/0的错误抛出。
        - ![](../image/自主学习/Spring%E4%BA%8B%E5%8A%A1%E7%9A%84%E4%BC%A0%E6%92%AD%E7%89%B9%E6%80%A7.png)
- 声明式事务(开发常用)
    - 步骤
        1. 添加配置文件
            1. 导入applicationContext_mapper.xml
                ```xml
                <import resource="applicationContext_mapper.xml"></import>
                ```
            1. 添加包扫描
                ```xml
                <context:component-scan base-package="com.study.service.impl"></context:component-scan>
                ```
            1. 添加事务管理器
                ```xml
                <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
                    <property name="dataSource" ref="dataSource"></property>
                </bean>
                ```
            1. 配置事务切面
                ```xml
                <tx:advice id="myadvice" transaction-manager="transactionManager">
                    <tx:attributes>
                        <tx:method name="*select*" read-only="true"/>
                        <tx:method name="*find*" read-only="true"/>
                        <tx:method name="*search*" read-only="true"/>
                        <tx:method name="*get*" read-only="true"/>
                        <tx:method name="*insert*" propagation="REQUIRED"/>
                        <tx:method name="*add*" propagation="REQUIRED"/>
                        <tx:method name="*save*" propagation="REQUIRED"/>
                        <tx:method name="*set*" propagation="REQUIRED"/>
                        <tx:method name="*update*" propagation="change"/>
                        <tx:method name="*change*" propagation="REQUIRED"/>
                        <tx:method name="*modify*" propagation="REQUIRED"/>
                        <tx:method name="*delete*" propagation="REQUIRED"/>
                        <tx:method name="*remove*" propagation="REQUIRED"/>
                        <tx:method name="*drop*" propagation="REQUIRED"/>
                        <tx:method name="*clear*" propagation="REQUIRED"/>
                        <tx:method name="*" propagation="SUPPORTS"/>
                    </tx:attributes>
                </tx:advice>
                ```
            1. 绑定切面和切入点
                ```xml
                <aop:config>
                    <aop:pointcut id="mycut" expression="execution(* com.study.service.impl.*.*(..))"></aop:pointcut>
                    <aop:advisor advice-ref="myadvice" pointcut-ref"mycut"></aop:advisor>
                </aop:config>
                ```

    - 要求项目中的方法命名有规范
        1. 完成增加操作包含 add save insert set
        1. 更新操作包含 update change modify
        1. 删除操作包含 delete drop remove clear
        1. 查询操作包含 select find search get
    - 如果有特殊需求，可以搭配使用注解式事务，设置优先级

# 完结撒花