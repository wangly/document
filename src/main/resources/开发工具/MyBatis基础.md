身世：iBatis升级版，本文主要介绍MyBatis升级前后，使用方式的区别。 
升级前，DAO层的结构为：DAO（接口）→ DAOImpl（实现）→ SQLMap.xml，数据模型DO、参数类Param、结果集包装类Result，以及指定数据源等参数的全局配置文件。
DAO：定义了操作数据库的接口。
DAOImpl：利用持有数据源的数据模板（如jdbcTemplate、sqlSessionTemplate），调用执行sql的方法。有时包括参数检查、结果类型转换等处理。
SQLMap.xml：sql与程序的映射文件，主要包含sql脚本。
由于DAOImpl主要负责调用template的方法执行sql，职责单一。因此，升级后的MyBatis通过mapper文件的namespace，指定mapper对应的接口，省去了DAOImpl，减轻了开发工作量。使用这种方式，需要指定Mapper接口的扫描路径，如下配置：
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
   <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryMybatis" />
   <property name="basePackage" value="com.meituan.hotel.pms.dao.mapper"/>
</bean>
<bean id="sqlSessionFactoryMybatis" class="org.mybatis.spring.SqlSessionFactoryBean">
   <property name="dataSource" ref="hotelDataSource"/>
   <!-- 配置扫描Domain的包路径 -->
   <property name="typeAliasesPackage" value="com.meituan.hotel.pms.dao.mapper"/>
   <!-- 配置扫描Mapper XML的位置 -->
   <property name="mapperLocations" value="classpath*:spring/dao/mapper/*.xml"/>
   <!-- 配置mybatis配置文件的位置 -->
   <property name="configLocation" value="classpath:spring/dao/mybatis-config-new.xml"/>
</bean>
注意：basePackage、mapperLocations
其中，mybatis-config文件配置项如下：
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
        <setting name="multipleResultSetsEnabled" value="true"/>
        <setting name="useColumnLabel" value="true"/>
        <setting name="defaultExecutorType" value="SIMPLE"/>
        <setting name="defaultStatementTimeout" value="25000"/>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="callSettersOnNulls" value="true"/>
    </settings>
    <!--分页插件-->
    <plugins>
        <plugin interceptor="com.meituan.ia.contract.interceptor.PageInterceptor" />
        <!--<plugin interceptor="com.meituan.hotel.ia.lib.crypto.interceptor.EncryptInterceptor" />-->
    </plugins>
</configuration>
properties --- 用于提供一系列的键值对组成的属性信息，该属性信息可以用于整个配置文件中。
settings --- 用于设置 MyBatis 的运行时方式，比如是否启用延迟加载等。
typeAliases --- 为 Java 类型指定别名，可以在 XML 文件中用别名取代 Java 类的全限定名。
typeHandlers --- 在 MyBatis 通过 PreparedStatement 为占位符设置值，或者从 ResultSet 取出值时，特定类型的类型处理器会被执行。
objectFactory --- MyBatis 通过 ObjectFactory 来创建结果对象。可以通过继承 DefaultObjectFactory 来实现自己的 ObjectFactory 类。
plugins --- 用于配置一系列拦截器，用于拦截映射 SQL 语句的执行。可以通过实现 Interceptor 接口来实现自己的拦截器，例如PageInterceptor。
environments --- 用于配置数据源信息，包括连接池、事务属性等。支持配置多套环境，便于开放、发布的各个阶段环境的切换。pms中是将数据源配置单独与mysql-datasource.xml文件中。
mappers --- 程序中所有用到的 SQL 映射文件都在这里列出，这些映射 SQL 都被 MyBatis 管理。