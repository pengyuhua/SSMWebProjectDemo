#使用IDEA搭建SSM项目流程
###1、创建maven_web工程项目，创建项目时需要勾选相应的maven模块
###2、项目创建完成后，在项目根目录下新建文件夹: target ; 然后在src/main/ 下新建java以及resoureces文件夹，并且相应设置为source root 以及 resource root 文件夹 ; 打开webapp/WEB-INF 下新建jsp 文件及以存放创建的歌jsp页面。
###3、在resources文件夹下新建mapper文件夹用于存放mybatis的mapper文件；同时创建sprng文件夹用于存放整合spring、springMvc框架的配置文件；创建好spring文件夹后在其下创建spring-dao.xml ， spring-mvc.xml ， spring-service.xml 用于配置整合的框架。在resources文件夹下创建jdbc.properties、mybatis-config.xml 用于配置mybatis相关信息；
*****
####jdbc.properties
````
#jdbc.driver=com.mysql.jdbc.Driver
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis_test?serverTimezone=UTC
jdbc.username=pengyuhua
jdbc.password=pyh546482
````
***
####mybatis-config.xml
````
    <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE configuration
             PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
         <!-- 配置全局属性 -->
         <settings>
             <!-- 使用jdbc的getGeneratedKeys获取数据库自增主键值 -->
             <setting name="useGeneratedKeys" value="true" />
             <!-- 使用列别名替换列名 默认:true -->
             <setting name="useColumnLabel" value="true" />
             <!-- 开启驼峰命名转换:Table{create_time} -> Entity{createTime} -->
             <setting name="mapUnderscoreToCamelCase" value="true" />
         </settings>
     </configuration>
     
 ````
 ***
 ####spring-dao.xml：
 ````
 <?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
 
     <!-- 配置整合mybatis过程 -->
     <!-- 1.配置数据库相关参数properties的属性：${url} -->
     <context:property-placeholder location="classpath:jdbc.properties"/>
 
     <!-- 2.数据库连接池 -->
     <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
         <!-- 配置连接池属性 -->
         <property name="driverClass" value="${jdbc.driver}"/>
         <property name="jdbcUrl" value="${jdbc.url}"/>
         <property name="user" value="${jdbc.username}"/>
         <property name="password" value="${jdbc.password}"/>
 
         <!-- c3p0连接池的私有属性 -->
         <property name="maxPoolSize" value="30"/>
         <property name="minPoolSize" value="10"/>
         <!-- 关闭连接后不自动commit -->
         <property name="autoCommitOnClose" value="false"/>
         <!-- 获取连接超时时间 -->
         <property name="checkoutTimeout" value="10000"/>
         <!-- 当获取连接失败重试次数 -->
         <property name="acquireRetryAttempts" value="2"/>
     </bean>
 
     <!-- 3.配置SqlSessionFactory对象 -->
     <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
         <!-- 注入数据库连接池 -->
         <property name="dataSource" ref="dataSource"/>
         <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
         <property name="configLocation" value="classpath:mybatis-config.xml"/>
         <!-- 扫描pojo包 使用别名 -->
         <property name="typeAliasesPackage" value="com.pojo"/>
         <!-- 扫描sql配置文件:mapper需要的xml文件 -->
         <property name="mapperLocations" value="classpath:mapper/*.xml"/>
     </bean>
 
     <!-- 4.配置扫描Dao接口包，动态实现Dao接口，注入到spring容器中 -->
     <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
         <!-- 注入sqlSessionFactory -->
         <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
         <!-- 指定需要扫描Dao接口包 -->
         <property name="basePackage" value="com.dao"/>
     </bean>
````
***
####spring-mvc.xml
````
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
        <!-- 配置SpringMVC -->
        <!-- 1.开启SpringMVC注解模式 -->
        <!-- 简化配置：
            (1)自动注册DefaultAnootationHandlerMapping,AnotationMethodHandlerAdapter
            (2)提供一系列：数据绑定，数字和日期的format @NumberFormat, @DateTimeFormat, xml,json默认读写支持
        -->
        <mvc:annotation-driven />
    
        <!-- 2.静态资源默认servlet配置
            (1)加入对静态资源的处理：js,gif,png
            (2)允许使用"/"做整体映射
         -->
        <mvc:default-servlet-handler/>
    
        <!-- 3.配置jsp 显示ViewResolver -->
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
            <!--  jsp扫描包-->
            <property name="prefix" value="/WEB-INF/jsp/" />
            <property name="suffix" value=".jsp" />
        </bean>
    
        <!-- 4.扫描web相关的bean -->
        <context:component-scan base-package="com.controller" />
    
    </beans>
````
***
####spring-service.xml
````
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    
        <!-- 扫描service包下所有使用注解的类型 -->
        <context:component-scan base-package="com.service" />
    
        <!-- 配置事务管理器 -->
        <bean id="transactionManager"
              class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <!-- 注入数据库连接池 -->
            <property name="dataSource" ref="dataSource" />
        </bean>
        <!-- 配置基于注解的声明式事务 -->
        <tx:annotation-driven transaction-manager="transactionManager" />
    </beans>
````
***
###4.添加POM 依赖
####pom.xml
````
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>SSMWebProject</groupId>
    <artifactId>SSMWebProjectDemo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <name>SSMWebProjectDemo Maven Webapp</name>
    <!-- FIXME change it to the project's website -->
    <!--<url>http://www.example.com</url>-->

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
        <jdbc.connector.version>8.0.13</jdbc.connector.version>
        <log4j2.version>2.11.1</log4j2.version>
        <!--<log4j.version>1.2.17</log4j.version>-->
        <!--<slf4j.version>1.7.7</slf4j.version>-->
        <!--<spring.version>5.1.3</spring.version>-->
        <!--<log4j2.version>2.6.2</log4j2.version>-->
    </properties>

    <!-- 远程仓库配置 -->
    <repositories>
        <repository>
            <id>apache.snapshots</id>
            <name>Apache Snapshot Repository</name>
            <url>https://repository.apache.org/snapshots</url>
            <releases>
                <enabled>false</enabled>
            </releases>
        </repository>
    </repositories>

    <dependencyManagement>
        <dependencies>
            <!--  使用Spring bom统一版本 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-framework-bom</artifactId>
                <version>5.1.3.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!--<dependency>-->
            <!--<groupId>junit</groupId>-->
            <!--<artifactId>junit</artifactId>-->
            <!--<version>4.11</version>-->
            <!--<scope>test</scope>-->
        <!--</dependency>-->

        <!-- LOg4j 2 依赖  -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
                <version>${log4j2.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>${log4j2.version}</version>
        </dependency>

        <!-- log start -->
        <!--<dependency>-->
        <!--<groupId>log4j</groupId>-->
        <!--<artifactId>log4j</artifactId>-->
        <!--<version>${log4j.version}</version>-->
        <!--</dependency>-->

        <!--<dependency>-->
        <!--<groupId>org.slf4j</groupId>-->
        <!--<artifactId>slf4j-api</artifactId>-->
        <!--<version>${slf4j.version}</version>-->
        <!--</dependency>-->

        <!--<dependency>-->
        <!--<groupId>org.slf4j</groupId>-->
        <!--<artifactId>slf4j-log4j12</artifactId>-->
        <!--<version>${slf4j.version}</version>-->
        <!--</dependency>-->
        <!-- log end -->

        <!--  JUnit5 依赖-->
        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-launcher</artifactId>
            <version>1.0.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.0.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.vintage</groupId>
            <artifactId>junit-vintage-engine</artifactId>
            <version>4.12.1</version>
            <scope>test</scope>
        </dependency>

        <!-- 格式化对象，方便输出日志 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.1.41</version>
        </dependency>

        <!-- JSOON封装包 -->
        <dependency>
            <groupId>net.minidev</groupId>
            <artifactId>json-smart</artifactId>
            <version>2.3</version>
            <scope>compile</scope>
        </dependency>

        <!-- 映入JSON -->
        <dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.9.13</version>
        </dependency>

        <!-- 上传组件包 -->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.4</version>
        </dependency>
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.9</version>
        </dependency>

        <!--  Mybatis 依赖-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
        </dependency>
        <!-- mybatis/spring包 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.2.2</version>
        </dependency>

        <!-- 数据库连接驱动依赖引入 -->
        <!-- 导入Mysql数据库链接jar包 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${jdbc.connector.version}</version>
        </dependency>

        <!-- 导入Oracle数据库链接jar包 -->
        <!--<dependency>-->
        <!--<groupId>com.oracle</groupId>-->
        <!--<artifactId>ojdbc14</artifactId>-->
        <!--<version>10.2.0.2.0</version>-->
        <!--</dependency>-->

        <!-- 数据库连接池 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>

        <!-- 导入dbcp的jar包，用来在applicationContext.xml中配置数据库 -->
        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.2.2</version>
        </dependency>

        <!-- JSTL标签类 -->
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!--  JEE 依赖  -->
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.13</version>
        </dependency>

        <!--  Spring 依赖 -->
        <!--  Spring core  -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
         <!--Spring Context -->
        <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-oxm</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
        </dependency>

        <!-- Servlet web -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.4</version>
        </dependency>

    </dependencies>

    <build>
        <finalName>SSMWebProjectDemo</finalName>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
            <plugins>
                <plugin>
                    <artifactId>maven-clean-plugin</artifactId>
                    <version>3.1.0</version>
                </plugin>
                <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
                <plugin>
                    <artifactId>maven-resources-plugin</artifactId>
                    <version>3.0.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.0</version>
                </plugin>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <version>2.22.1</version>
                </plugin>
                <plugin>
                    <artifactId>maven-war-plugin</artifactId>
                    <version>3.2.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-install-plugin</artifactId>
                    <version>2.5.2</version>
                </plugin>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>

````

###5.添加运行环境：点击run菜单下的edit configuration 选项添加tomcat运行环境

***
###6.找到index.jsp文件右键运行即可，至此项目搭建即可完成。