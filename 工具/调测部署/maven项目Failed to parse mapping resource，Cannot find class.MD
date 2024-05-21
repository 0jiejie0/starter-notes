# maven项目 Application run failed，NestedIOException: Failed to parse mapping resource，Cannot find class

## 背景

一次很普通的springboot项目部署，使用idea maven工具打包。

## 问题描述

无任何多余和不正常的操作，打好jar包后，运行程序闪退，错误日志：

```log
2024-05-21 20:41:56.472 [main] ERROR org.springframework.boot.SpringApplication - Application run failed
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'firstSqlSessionFactory' defined in class path resource [com/lightsoft/config/DataSourceConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.ibatis.session.SqlSessionFactory]: Factory method 'firstSqlSessionFactory' threw exception; nested exception is org.springframework.core.NestedIOException: Failed to parse mapping resource: 'URL [jar:file:/D:/ICU/api/icu/icu_server-1.0.jar!/BOOT-INF/classes!/mapper/MidUserMapper.xml]'; nested exception is org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. The XML location is 'URL [jar:file:/D:/ICU/api/icu/icu_server-1.0.jar!/BOOT-INF/classes!/mapper/MidUserMapper.xml]'. Cause: org.apache.ibatis.builder.BuilderException: Error resolving class. Cause: org.apache.ibatis.type.TypeException: Could not resolve type alias 'com.lightsoft.entity.MidUserEntity'.  Cause: java.lang.ClassNotFoundException: Cannot find class: com.lightsoft.entity.MidUserEntity
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:590)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1247)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1096)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:535)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:495)
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:317)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:315)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:759)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:869)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:550)
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140)
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:780)
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:412)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:333)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1277)
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1265)
	at com.lightsoft.App.main(App.java:17)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
	at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
	at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
	at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51)
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.ibatis.session.SqlSessionFactory]: Factory method 'firstSqlSessionFactory' threw exception; nested exception is org.springframework.core.NestedIOException: Failed to parse mapping resource: 'URL [jar:file:/D:/ICU/api/icu/icu_server-1.0.jar!/BOOT-INF/classes!/mapper/MidUserMapper.xml]'; nested exception is org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. The XML location is 'URL [jar:file:/D:/ICU/api/icu/icu_server-1.0.jar!/BOOT-INF/classes!/mapper/MidUserMapper.xml]'. Cause: org.apache.ibatis.builder.BuilderException: Error resolving class. Cause: org.apache.ibatis.type.TypeException: Could not resolve type alias 'com.lightsoft.entity.MidUserEntity'.  Cause: java.lang.ClassNotFoundException: Cannot find class: com.lightsoft.entity.MidUserEntity
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:185)
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:582)
	... 26 common frames omitted
Caused by: org.springframework.core.NestedIOException: Failed to parse mapping resource: 'URL [jar:file:/D:/ICU/api/icu/icu_server-1.0.jar!/BOOT-INF/classes!/mapper/MidUserMapper.xml]'; nested exception is org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. The XML location is 'URL [jar:file:/D:/ICU/api/icu/icu_server-1.0.jar!/BOOT-INF/classes!/mapper/MidUserMapper.xml]'. Cause: org.apache.ibatis.builder.BuilderException: Error resolving class. Cause: org.apache.ibatis.type.TypeException: Could not resolve type alias 'com.lightsoft.entity.MidUserEntity'.  Cause: java.lang.ClassNotFoundException: Cannot find class: com.lightsoft.entity.MidUserEntity
	at org.mybatis.spring.SqlSessionFactoryBean.buildSqlSessionFactory(SqlSessionFactoryBean.java:523)
	at org.mybatis.spring.SqlSessionFactoryBean.afterPropertiesSet(SqlSessionFactoryBean.java:380)
	at org.mybatis.spring.SqlSessionFactoryBean.getObject(SqlSessionFactoryBean.java:547)
	at com.lightsoft.config.DataSourceConfig.firstSqlSessionFactory(DataSourceConfig.java:100)
	at com.lightsoft.config.DataSourceConfig$$EnhancerBySpringCGLIB$$44d71b4e.CGLIB$firstSqlSessionFactory$0(<generated>)
	at com.lightsoft.config.DataSourceConfig$$EnhancerBySpringCGLIB$$44d71b4e$$FastClassBySpringCGLIB$$f32696d6.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:228)
	at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:365)
	at com.lightsoft.config.DataSourceConfig$$EnhancerBySpringCGLIB$$44d71b4e.firstSqlSessionFactory(<generated>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154)
	... 27 common frames omitted
Caused by: org.apache.ibatis.builder.BuilderException: Error parsing Mapper XML. The XML location is 'URL [jar:file:/D:/ICU/api/icu/icu_server-1.0.jar!/BOOT-INF/classes!/mapper/MidUserMapper.xml]'. Cause: org.apache.ibatis.builder.BuilderException: Error resolving class. Cause: org.apache.ibatis.type.TypeException: Could not resolve type alias 'com.lightsoft.entity.MidUserEntity'.  Cause: java.lang.ClassNotFoundException: Cannot find class: com.lightsoft.entity.MidUserEntity
	at org.apache.ibatis.builder.xml.XMLMapperBuilder.configurationElement(XMLMapperBuilder.java:120)
	at org.apache.ibatis.builder.xml.XMLMapperBuilder.parse(XMLMapperBuilder.java:92)
	at org.mybatis.spring.SqlSessionFactoryBean.buildSqlSessionFactory(SqlSessionFactoryBean.java:521)
	... 40 common frames omitted
Caused by: org.apache.ibatis.builder.BuilderException: Error resolving class. Cause: org.apache.ibatis.type.TypeException: Could not resolve type alias 'com.lightsoft.entity.MidUserEntity'.  Cause: java.lang.ClassNotFoundException: Cannot find class: com.lightsoft.entity.MidUserEntity
	at org.apache.ibatis.builder.BaseBuilder.resolveClass(BaseBuilder.java:118)
	at org.apache.ibatis.builder.xml.XMLMapperBuilder.resultMapElement(XMLMapperBuilder.java:265)
	at org.apache.ibatis.builder.xml.XMLMapperBuilder.resultMapElement(XMLMapperBuilder.java:252)
	at org.apache.ibatis.builder.xml.XMLMapperBuilder.resultMapElements(XMLMapperBuilder.java:244)
	at org.apache.ibatis.builder.xml.XMLMapperBuilder.configurationElement(XMLMapperBuilder.java:116)
	... 42 common frames omitted
Caused by: org.apache.ibatis.type.TypeException: Could not resolve type alias 'com.lightsoft.entity.MidUserEntity'.  Cause: java.lang.ClassNotFoundException: Cannot find class: com.lightsoft.entity.MidUserEntity
	at org.apache.ibatis.type.TypeAliasRegistry.resolveAlias(TypeAliasRegistry.java:120)
	at org.apache.ibatis.builder.BaseBuilder.resolveAlias(BaseBuilder.java:149)
	at org.apache.ibatis.builder.BaseBuilder.resolveClass(BaseBuilder.java:116)
	... 46 common frames omitted
Caused by: java.lang.ClassNotFoundException: Cannot find class: com.lightsoft.entity.MidUserEntity
	at org.apache.ibatis.io.ClassLoaderWrapper.classForName(ClassLoaderWrapper.java:200)
	at org.apache.ibatis.io.ClassLoaderWrapper.classForName(ClassLoaderWrapper.java:89)
	at org.apache.ibatis.io.Resources.classForName(Resources.java:261)
	at org.apache.ibatis.type.TypeAliasRegistry.resolveAlias(TypeAliasRegistry.java:116)
	... 48 common frames omitted
```

重新运行idea代码，也是如此。

## 原因分析

但是在idea的控制台日志中，发现其文件路径位于target目录，这是编译的缓存目录，推测是版本切换导致缓存没跟上出错了。

## 解决方案

点击maven工具中的`clean`，点击绿色小虫子`debug`，点击maven工具中的`package`，重新打包部署，恢复正常。
