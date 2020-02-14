# SSM搭建

在配置SSM之前，还需要做一些准备工作。

添加所需的packages：

<img src="./screenshots/chapter2/packages.png" style="margin-left:0;" />

然后在pom.xml中添加所需的依赖，由于太多就不罗列了，大致可以分为以下几类：单元测试，日志，Spring，MyBatis，MySQL，还有一些工具类等等。

接着我们继续添加所需的配置文件：

<img src="screenshots/chapter2/config.png" style="margin-left:0;" />

### jdbc.properties

``` txt
jdbc.driver = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/o2o?useUnicode=true&characterEncoding=utf8
jdbc.username=root
jdbc.password=4012
```

这是一个通过jdbc连接MySQL的配置文件，里面是连接所需的参数。

### mybatis-config.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 配置全局属性 -->
	<settings>
		<!-- 使用jdbc的getGeneratedKeys获取数据库自增主键值 -->
		<setting name="useGeneratedKeys" value="true" />

		<!-- 使用列标签替换列别名 默认:true -->
		<setting name="useColumnLabel" value="true" />

		<!-- 开启驼峰命名转换:Table{create_time} -> Entity{createTime} -->
		<setting name="mapUnderscoreToCamelCase" value="true" />
	</settings>
	<plugins>
		<plugin interceptor="com.imooc.o2o.dao.split.DynamicDataSourceInterceptor">
		</plugin>
	</plugins>
</configuration>
```

配置了MyBatis的相关参数。

### spring-*.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
	<!-- 配置整合mybatis过程 -->
	<!-- 1.配置数据库相关参数properties的属性：${url} -->
	<context:property-placeholder location = "classpath:jdbc.properties"/>
	
	<!-- 2.数据库连接池 -->
	<bean id="dataSource" abstract="true"
		class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
		<!-- 配置连接池属性 -->
		<property name = "driverClass" value = "${jdbc.driver}" />
		<property name = "jdbcUrl" value = "${jdbc.url}" />
		<property name = "user" value = "${jdbc.username}" />
		<property name = "password" value = "${jdbc.password}" />
		<!-- c3p0连接池的私有属性 -->
		<property name="maxPoolSize" value="30" />
		<property name="minPoolSize" value="10" />
		<property name="initialPoolSize" value="10"/> 
		<!-- 关闭连接后不自动commit -->
		<property name="autoCommitOnClose" value="false" />
		<!-- 获取连接超时时间 -->
		<property name="checkoutTimeout" value="10000" />
		<!-- 当获取连接失败重试次数 -->
		<property name="acquireRetryAttempts" value="2" />
	</bean>

	<!-- 3.配置SqlSessionFactory对象 -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- 注入数据库连接池 -->
		<property name="dataSource" ref="dataSource" />
		<!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
		<property name="configLocation" value="classpath:mybatis-config.xml" />
		<!-- 扫描entity包 使用别名 -->
		<property name="typeAliasesPackage" value="com.xinyuan.o2o.entity" />
		<!-- 扫描sql配置文件:mapper需要的xml文件 -->
		<property name="mapperLocations" value="classpath:mapper/*.xml" />
	</bean>

	<!-- 4.配置扫描Dao接口包，动态实现Dao接口，注入到spring容器中 -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 注入sqlSessionFactory -->
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
		<!-- 给出需要扫描Dao接口包 -->
		<property name="basePackage" value="com.xinyuan.o2o.dao" />
	</bean>
</beans>

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">
    <!-- 扫描service包下所有使用注解的类型 -->
    <context:component-scan base-package="com.xinyuan.o2o.service" />

    <!-- 配置事务管理器 -->
    <bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource" />
    </bean>

    <!-- 配置基于注解的声明式事务 -->
    <tx:annotation-driven transaction-manager="transactionManager" />
</beans>

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd">
	<!-- 配置SpringMVC -->
	<!-- 1.开启SpringMVC注解模式 -->
	<mvc:annotation-driven />

	<!-- 2.静态资源默认servlet配置 (1)加入对静态资源的处理：js,gif,png (2)允许使用"/"做整体映射 -->
	<mvc:resources mapping="/resources/**" location="/resources/" />
	<mvc:default-servlet-handler />

	<!-- 3.定义视图解析器 -->
	<bean id="viewResolver"
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/html/"></property>
		<property name="suffix" value=".html"></property>
	</bean>
	<!-- 文件上传解析器 -->
	<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="defaultEncoding" value="utf-8"></property>
		<!-- 1024 * 1024 * 20 = 20M -->
		<property name="maxUploadSize" value="20971520"></property>
		<property name="maxInMemorySize" value="20971520"></property>
	</bean>

	<!-- 4.扫描web相关的bean -->
	<context:component-scan base-package="com.xinyuan.o2o.web" />
</beans>
```

按自底向上的配置依次配置DAO层，Service层和Controller层。

DAO层负责和数据库通信，SqlSessionFactory对象用工厂模式创建连接池。这里的DAO层核心是各实体类的Mapper文件，Mapper文件中配置了各个类对数据库所需进行的sql语句映射。

Service层负责业务逻辑，简单说就是如何去调用DAO层接口。为了确保原子操作，需要启用事务管理器。

Controller层负责一些具体的业务流程控制，要调用到Service层接口。个人理解是它们分别对应上层逻辑和底层逻辑实现。DispatcherServlet负责拦截请求并匹配到不同的Controller完成对应操作。

此外还有一个View层，其实就是前端的HTML（静态）或者jsp页面（动态）。

### web.xml

然后更新WEB-INF下的web.xml文件，设置使用Spring里面的DispatcherServlet类来处理请求。这个类作用有点像路由(router)。

``` xml
<servlet>
    <servlet-name>spring-dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring/spring-*.xml</param-value>
    </init-param>
</servlet>
<servlet-mapping>
    <servlet-name>spring-dispatcher</servlet-name>
    <!-- 默认匹配所有请求 -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

最后强调，初学SSM要了解其架构和思想，而不要拘泥于配置和参数。

参考资料：

> https://www.cnblogs.com/verlen11/p/5349747.html
>
> https://blog.csdn.net/tianhumin/article/details/54949636

