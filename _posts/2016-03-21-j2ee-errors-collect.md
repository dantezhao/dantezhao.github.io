---
layout: post
title:  "J2EE错误汇整"
categories: 编程语言
tags: jvm j2ee java
---

* content
{:toc}

## 前言

14年开始做一个javaweb项目，结果没有任何指导，也没有人带，完全自己从头来，因此遇到了很多的坑，还好当时做了大量的记录，现在整理一下。




## 错误汇总
### 错误1
```
WARN: HHH000374: Could not unbind factory from JNDI
org.hibernate.engine.jndi.JndiException: Error parsing JNDI name []
    at org.hibernate.engine.jndi.internal.JndiServiceImpl.parseName(JndiServiceImpl.java:141)
    at org.hibernate.engine.jndi.internal.JndiServiceImpl.unbind(JndiServiceImpl.java:225)
    at org.hibernate.internal.SessionFactoryRegistry.removeSessionFactory(SessionFactoryRegistry.java:139)
    at org.hibernate.internal.SessionFactoryImpl.close(SessionFactoryImpl.java:1369)
    at lee.NewsManager.main(NewsManager.java:49)
Caused by: javax.naming.NoInitialContextException: Need to specify class name in environment or system property, or as an applet parameter, or in an application resource file:  java.naming.factory.initial
    at javax.naming.spi.NamingManager.getInitialContext(Unknown Source)
    at javax.naming.InitialContext.getDefaultInitCtx(Unknown Source)
    at javax.naming.InitialContext.getURLOrDefaultInitCtx(Unknown Source)
    at javax.naming.InitialContext.getNameParser(Unknown Source)
    at org.hibernate.engine.jndi.internal.JndiServiceImpl.parseName(JndiServiceImpl.java:135)
    ... 4 more  

```

解决：
```
<session-factory name="test">
  ******
</session-factory>
把<session-factory name="test">改为<session-factoryst>
```

### 错误2

```
Exception in thread "main" org.hibernate.MappingException: Unknown entity: org.crazyit.app.domain.News
    at org.hibernate.internal.SessionFactoryImpl.getEntityPersister(SessionFactoryImpl.java:1096)
    at org.hibernate.internal.SessionImpl.getEntityPersister(SessionImpl.java:1479)
    at org.hibernate.event.internal.AbstractSaveEventListener.saveWithGeneratedId(AbstractSaveEventListener.java:117)
    at org.hibernate.event.internal.DefaultSaveOrUpdateEventListener.saveWithGeneratedOrRequestedId(DefaultSaveOrUpdateEventListener.java:209)
    at org.hibernate.event.internal.DefaultSaveEventListener.saveWithGeneratedOrRequestedId(DefaultSaveEventListener.java:55)
    at org.hibernate.event.internal.DefaultSaveOrUpdateEventListener.entityIsTransient(DefaultSaveOrUpdateEventListener.java:194)
    at org.hibernate.event.internal.DefaultSaveEventListener.performSaveOrUpdate(DefaultSaveEventListener.java:49)
    at org.hibernate.event.internal.DefaultSaveOrUpdateEventListener.onSaveOrUpdate(DefaultSaveOrUpdateEventListener.java:90)
    at org.hibernate.internal.SessionImpl.fireSave(SessionImpl.java:715)
    at org.hibernate.internal.SessionImpl.save(SessionImpl.java:707)
    at org.hibernate.internal.SessionImpl.save(SessionImpl.java:702)
    at lee.NewsManager.main(NewsManager.java:44)  

```

解决：
```
  在配置文件里面添加<mapping class="org.crazyit.app.domain.News"/>
```



### 错误3

```
Exception in thread "main" org.hibernate.HibernateException: SessionFactory was not configured for multi-tenancy
    at org.hibernate.internal.AbstractSessionImpl.<init>(AbstractSessionImpl.java:83)
    at org.hibernate.internal.SessionImpl.<init>(SessionImpl.java:243)
    at org.hibernate.internal.SessionFactoryImpl$SessionBuilderImpl.openSession(SessionFactoryImpl.java:1589)
    at org.hibernate.internal.SessionFactoryImpl.openSession(SessionFactoryImpl.java:999)
    at mt.SchemaTestMain.main(SchemaTestMain.java:63)

ERROR: HHH000231: Schema export unsuccessful  

Exception in thread "main" org.hibernate.HibernateException: The database returned no natively generated identity value
    at org.hibernate.id.IdentifierGeneratorHelper.getGeneratedIdentity(IdentifierGeneratorHelper.java:91)
    at org.hibernate.id.IdentityGenerator$GetGeneratedKeysDelegate.executeAndExtract(IdentityGenerator.java:100)
    at org.hibernate.id.insert.AbstractReturningDelegate.performInsert(AbstractReturningDelegate.java:58)
    at org.hibernate.persister.entity.AbstractEntityPersister.insert(AbstractEntityPersister.java:3032)
    at org.hibernate.persister.entity.AbstractEntityPersister.insert(AbstractEntityPersister.java:3558)
    at org.hibernate.action.internal.EntityIdentityInsertAction.execute(EntityIdentityInsertAction.java:98)
    at org.hibernate.engine.spi.ActionQueue.execute(ActionQueue.java:492)
    at org.hibernate.engine.spi.ActionQueue.addResolvedEntityInsertAction(ActionQueue.java:197)
    at org.hibernate.engine.spi.ActionQueue.addInsertAction(ActionQueue.java:181)
    at org.hibernate.engine.spi.ActionQueue.addAction(ActionQueue.java:216)
    at org.hibernate.event.internal.AbstractSaveEventListener.addInsertAction(AbstractSaveEventListener.java:334)
    at org.hibernate.event.internal.AbstractSaveEventListener.performSaveOrReplicate(AbstractSaveEventListener.java:289)
    at org.hibernate.event.internal.AbstractSaveEventListener.performSave(AbstractSaveEventListener.java:195)
    at org.hibernate.event.internal.AbstractSaveEventListener.saveWithGeneratedId(AbstractSaveEventListener.java:126)
    at org.hibernate.event.internal.DefaultSaveOrUpdateEventListener.saveWithGeneratedOrRequestedId(DefaultSaveOrUpdateEventListener.java:209)
    at org.hibernate.event.internal.DefaultSaveEventListener.saveWithGeneratedOrRequestedId(DefaultSaveEventListener.java:55)
    at org.hibernate.event.internal.DefaultSaveOrUpdateEventListener.entityIsTransient(DefaultSaveOrUpdateEventListener.java:194)
    at org.hibernate.event.internal.DefaultSaveEventListener.performSaveOrUpdate(DefaultSaveEventListener.java:49)
    at org.hibernate.event.internal.DefaultSaveOrUpdateEventListener.onSaveOrUpdate(DefaultSaveOrUpdateEventListener.java:90)
    at org.hibernate.internal.SessionImpl.fireSave(SessionImpl.java:715)
    at org.hibernate.internal.SessionImpl.save(SessionImpl.java:707)
    at org.hibernate.internal.SessionImpl.save(SessionImpl.java:702)
    at com.mt.SchemaTestMain.main(SchemaTestMain.java:45)

```
解决：
```
指定主键
```


### 错误4

```
org.springframework.beans.factory.BeanDefinitionStoreException: Unexpected exception parsing XML document from class path resource [beans.xml]; nested exception is java.lang.NoClassDefFoundError: org/aopalliance/intercept/MethodInterceptor
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:414)
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:336)
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.loadBeanDefinitions(XmlBeanDefinitionReader.java:304)
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:181)
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:217)
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:188)
at org.springframework.beans.factory.support.AbstractBeanDefinitionReader.loadBeanDefinitions(AbstractBeanDefinitionReader.java:252)
at org.springframework.context.support.AbstractXmlApplicationContext.loadBeanDefinitions(AbstractXmlApplicationContext.java:127)
at org.springframework.context.support.AbstractXmlApplicationContext.loadBeanDefinitions(AbstractXmlApplicationContext.java:93)
at org.springframework.context.support.AbstractRefreshableApplicationContext.refreshBeanFactory(AbstractRefreshableApplicationContext.java:129)
at org.springframework.context.support.AbstractApplicationContext.obtainFreshBeanFactory(AbstractApplicationContext.java:537)
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:452)
at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:83)
at com.saas.test.UserTest.add(UserTest.java:19)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
at java.lang.reflect.Method.invoke(Unknown Source)
at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:459)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:675)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)
Caused by: java.lang.NoClassDefFoundError: org/aopalliance/intercept/MethodInterceptor
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClass(Unknown Source)
at java.security.SecureClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.defineClass(Unknown Source)
at java.net.URLClassLoader.access$100(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at org.springframework.transaction.config.TxAdviceBeanDefinitionParser.getBeanClass(TxAdviceBeanDefinitionParser.java:71)
at org.springframework.beans.factory.xml.AbstractSingleBeanDefinitionParser.parseInternal(AbstractSingleBeanDefinitionParser.java:66)
at org.springframework.beans.factory.xml.AbstractBeanDefinitionParser.parse(AbstractBeanDefinitionParser.java:61)
at org.springframework.beans.factory.xml.NamespaceHandlerSupport.parse(NamespaceHandlerSupport.java:74)
at org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseCustomElement(BeanDefinitionParserDelegate.java:1427)
at org.springframework.beans.factory.xml.BeanDefinitionParserDelegate.parseCustomElement(BeanDefinitionParserDelegate.java:1417)
at org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.parseBeanDefinitions(DefaultBeanDefinitionDocumentReader.java:174)
at org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.doRegisterBeanDefinitions(DefaultBeanDefinitionDocumentReader.java:144)
at org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader.registerBeanDefinitions(DefaultBeanDefinitionDocumentReader.java:100)
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.registerBeanDefinitions(XmlBeanDefinitionReader.java:510)
at org.springframework.beans.factory.xml.XmlBeanDefinitionReader.doLoadBeanDefinitions(XmlBeanDefinitionReader.java:392)
... 37 more
Caused by: java.lang.ClassNotFoundException: org.aopalliance.intercept.MethodInterceptor
at java.net.URLClassLoader$1.run(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
... 60 more

```

原因：

测试的时候没有加自动扫描，因为用的是注解，需要自动扫描
sessionfactory需要自动装配，装配到dao层

解决：添加红色的两部分
```
<?xml version="1.0" encoding="UTF-8"?>
<beans default-autowire="byName" xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <context:component-scan base-package="com"></context:component-scan>

```

### 错误5

```
Error occured processing XML 'org/springframework/transaction/interceptor/TransactionInterceptor'. See Error Log for more details.
```

在配制
<!-- 使用annotation定义事务  -->
<tx:annotation-driven transaction-manager="transactionManager" />
的时候可能会出现上面的错误提示。


解决：
```
加入aopalliance-1.0.jar
```

### 错误6
```
java.lang.NoClassDefFoundError: org/aspectj/weaver/reflect/ReflectionWorld$ReflectionWorldException
at java.lang.Class.getDeclaredMethods0(Native Method)
at java.lang.Class.privateGetDeclaredMethods(Unknown Source)
at java.lang.Class.getDeclaredMethods(Unknown Source)
at org.springframework.core.type.StandardAnnotationMetadata.hasAnnotatedMethods(StandardAnnotationMetadata.java:129)
at org.springframework.context.annotation.ConfigurationClassUtils.isLiteConfigurationCandidate(ConfigurationClassUtils.java:157)
at org.springframework.context.annotation.ConfigurationClassUtils.checkConfigurationClassCandidate(ConfigurationClassUtils.java:108)
at org.springframework.context.annotation.ConfigurationClassPostProcessor.processConfigBeanDefinitions(ConfigurationClassPostProcessor.java:278)
at org.springframework.context.annotation.ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(ConfigurationClassPostProcessor.java:239)
at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanDefinitionRegistryPostProcessors(PostProcessorRegistrationDelegate.java:254)
at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(PostProcessorRegistrationDelegate.java:94)
at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:606)
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:462)
at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:83)
at com.saas.test.UserTest.add(UserTest.java:19)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
at java.lang.reflect.Method.invoke(Unknown Source)
at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:459)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:675)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)
Caused by: java.lang.ClassNotFoundException: org.aspectj.weaver.reflect.ReflectionWorld$ReflectionWorldException
at java.net.URLClassLoader$1.run(Unknown Source)
at java.net.URLClassLoader$1.run(Unknown Source)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
at java.lang.ClassLoader.loadClass(Unknown Source)
... 38 more
```

缺少jar包引起的
aspectjweaver-1.8.5.jar


### 错误7

```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'staffDAOImpl' defined in file [C:\zhao\workspace\saas\build\classes\com\saas\dao\impl\StaffDAOImpl.class]: Initialization of bean failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sessionFactory' defined in class path resource [beans.xml]: Invocation of init method failed; nested exception is org.hibernate.AnnotationException: No identifier specified for entity: com.saas.po.Staff
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:547)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:755)
at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:757)
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:480)
at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:83)
at saas.StaffTest.before(StaffTest.java:22)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
at java.lang.reflect.Method.invoke(Unknown Source)
at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:24)
at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:459)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:675)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sessionFactory' defined in class path resource [beans.xml]: Invocation of init method failed; nested exception is org.hibernate.AnnotationException: No identifier specified for entity: com.saas.po.Staff
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1574)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:539)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireByName(AbstractAutowireCapableBeanFactory.java:1240)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1190)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
... 34 more
Caused by: org.hibernate.AnnotationException: No identifier specified for entity: com.saas.po.Staff
at org.hibernate.cfg.InheritanceState.determineDefaultAccessType(InheritanceState.java:277)
at org.hibernate.cfg.InheritanceState.getElementsToProcess(InheritanceState.java:224)
at org.hibernate.cfg.AnnotationBinder.bindClass(AnnotationBinder.java:775)
at org.hibernate.cfg.Configuration$MetadataSourceQueue.processAnnotatedClassesQueue(Configuration.java:3845)
at org.hibernate.cfg.Configuration$MetadataSourceQueue.processMetadata(Configuration.java:3799)
at org.hibernate.cfg.Configuration.secondPassCompile(Configuration.java:1412)
at org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1846)
at org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1930)
at org.springframework.orm.hibernate4.LocalSessionFactoryBuilder.buildSessionFactory(LocalSessionFactoryBuilder.java:372)
at org.springframework.orm.hibernate4.LocalSessionFactoryBean.buildSessionFactory(LocalSessionFactoryBean.java:454)
at org.springframework.orm.hibernate4.LocalSessionFactoryBean.afterPropertiesSet(LocalSessionFactoryBean.java:439)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1633)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1570)
... 43 more

```

解决：给staffDAOImpl类添加@Re注解




### 错误8

```
执行插入操作时，提示空指针错误，但是数据仍然可以插入成功。
ERROR: HHH000299: Could not complete schema update
java.lang.NullPointerException
    at org.hibernate.tool.hbm2ddl.SuppliedConnectionProviderConnectionHelper.prepare(SuppliedConnectionProviderConnectionHelper.java:51)
    at org.hibernate.tool.hbm2ddl.SchemaUpdate.execute(SchemaUpdate.java:219)
    at org.hibernate.tool.hbm2ddl.SchemaUpdate.execute(SchemaUpdate.java:203)
    at org.hibernate.internal.SessionFactoryImpl.<init>(SessionFactoryImpl.java:522)
    at org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1859)
    at org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1930)
    at org.springframework.orm.hibernate4.LocalSessionFactoryBuilder.buildSessionFactory(LocalSessionFactoryBuilder.java:372)
    at org.springframework.orm.hibernate4.LocalSessionFactoryBean.buildSessionFactory(LocalSessionFactoryBean.java:454)
    at org.springframework.orm.hibernate4.LocalSessionFactoryBean.afterPropertiesSet(LocalSessionFactoryBean.java:439)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1633)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1570)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:539)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:736)
    at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:757)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:480)
    at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
    at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:83)
    at com.saas.test.StaffTest.add(StaffTest.java:22)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
    at java.lang.reflect.Method.invoke(Unknown Source)
    at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:47)
    at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
    at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:44)
    at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
    at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:271)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:70)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:50)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:238)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:63)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:236)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:53)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:229)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:309)
    at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
    at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:459)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:675)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)

```

分析：
原因不详，但是问题出在这一行，http://stackoverflow.com/questions/15636595/hibernate-4-multitenancy-spring-3-schema-export-error上分析是hibernate的错误，目前hibernate貌似不支持multitenanay的自动创建表行为。
<property name="hbm2ddl.auto">update</property>  
这是hibernate的一个bug，目前尚未解决。https://hibernate.atlassian.net/browse/HHH-7395

解决：
删除该行即可


### 错误9：

```
HTTP Status 404 - /streetManager/index.jsp
 发布首页出现404错误，web.xml没有错误。
```

解决
在IE中提示“404”错误有以下三种情况
I．未部署Web应用
II．URL输入错误
排错方法：
首先，查看URL的IP地址和端口号是否书写正确。
其次，查看上下文路径是否正确 Project--------Properties------MyElipse-----Web-----
Web Context-root检查这个路径名称是否书写正确。
最后，检查一下文件名称是否书写正确。
III．目录不能被引用
排错方法：
在 Eclipse的“包资源管理器(Package Explorer)”检查文件存放的位置。由于META-INF
WEB-INF文件夹下的内容无法对外发布，所以，如果你引用了带这两个目录的文件，肯定是不允许。例如：http://localhost:8080/guestbook/WEB-INF/index.html就是错误的
文件位置存放错误。


### 错误10
```
HTTP Status 500 - Error instantiating servlet class org.springframework.web.servlet.DispatcherServlet


type Exception report


message Error instantiating servlet class org.springframework.web.servlet.DispatcherServlet


description The server encountered an internal error that prevented it from fulfilling this request.


exception


javax.servlet.ServletException: Error instantiating servlet class org.springframework.web.servlet.DispatcherServlet


org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)


org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)


org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:610)


org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:518)


org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1091)


org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:668)


org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223)


org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1517)


org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1474)


java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)


java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)


org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)


java.lang.Thread.run(Unknown Source)


root cause


java.lang.ClassNotFoundException: org.springframework.web.servlet.DispatcherServlet


org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1305)


org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1157)


org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)


org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)


org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:610)


org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:518)


org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1091)


org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:668)


org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223)


org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1517)


org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1474)


java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)


java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)


org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)


java.lang.Thread.run(Unknown Source)


note The full stack trace of the root cause is available in the Apache Tomcat/8.0.21 logs.


DispatcherServlet
```

找不到该类，是在web-xml中配置的，原因是wen-inf lib中没有，copy进去需要jar包就行


### 错误11

```
HTTP Status 500 - Servlet.init() for servlet springmvc threw exception
type Exception report


message Servlet.init() for servlet springmvc threw exception


description The server encountered an internal error that prevented it from fulfilling this request.


exception


javax.servlet.ServletException: Servlet.init() for servlet springmvc threw exception
org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:610)
org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:518)
org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1091)
org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:668)
org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1517)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1474)
java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
java.lang.Thread.run(Unknown Source)
root cause


org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'staffDao' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'sessionFactory' while setting bean property 'sessionFactory'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sessionFactory' defined in class path resource [beans.xml]: Invocation of init method failed; nested exception is java.lang.NoClassDefFoundError: org/hibernate/engine/jdbc/connections/spi/MultiTenantConnectionProvider
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:336)
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:108)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1456)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1197)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:304)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:300)
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:195)
org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:703)
org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:760)
org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:482)
org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:658)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:624)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:672)
org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:543)
org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:484)
org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)
javax.servlet.GenericServlet.init(GenericServlet.java:158)
org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:610)
org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:518)
org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1091)
org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:668)
org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1517)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1474)
java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
java.lang.Thread.run(Unknown Source)
root cause


org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sessionFactory' defined in class path resource [beans.xml]: Invocation of init method failed; nested exception is java.lang.NoClassDefFoundError: org/hibernate/engine/jdbc/connections/spi/MultiTenantConnectionProvider
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1553)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:539)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:304)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:300)
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:195)
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:328)
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:108)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1456)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1197)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:304)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:300)
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:195)
org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:703)
org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:760)
org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:482)
org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:658)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:624)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:672)
org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:543)
org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:484)
org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)
javax.servlet.GenericServlet.init(GenericServlet.java:158)
org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:610)
org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:518)
org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1091)
org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:668)
org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1517)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1474)
java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
java.lang.Thread.run(Unknown Source)
root cause


java.lang.NoClassDefFoundError: org/hibernate/engine/jdbc/connections/spi/MultiTenantConnectionProvider
java.lang.ClassLoader.defineClass1(Native Method)
java.lang.ClassLoader.defineClass(Unknown Source)
java.security.SecureClassLoader.defineClass(Unknown Source)
org.apache.catalina.loader.WebappClassLoaderBase.findClassInternal(WebappClassLoaderBase.java:2472)
org.apache.catalina.loader.WebappClassLoaderBase.findClass(WebappClassLoaderBase.java:854)
org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1274)
org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1157)
org.hibernate.service.classloading.internal.ClassLoaderServiceImpl$AggregatedClassLoader.findClass(ClassLoaderServiceImpl.java:290)
java.lang.ClassLoader.loadClass(Unknown Source)
java.lang.ClassLoader.loadClass(Unknown Source)
java.lang.Class.forName0(Native Method)
java.lang.Class.forName(Unknown Source)
org.hibernate.service.classloading.internal.ClassLoaderServiceImpl.classForName(ClassLoaderServiceImpl.java:146)
org.hibernate.service.jdbc.connections.internal.MultiTenantConnectionProviderInitiator.initiateService(MultiTenantConnectionProviderInitiator.java:84)
org.hibernate.service.jdbc.connections.internal.MultiTenantConnectionProviderInitiator.initiateService(MultiTenantConnectionProviderInitiator.java:43)
org.hibernate.service.internal.StandardServiceRegistryImpl.initiateService(StandardServiceRegistryImpl.java:69)
org.hibernate.service.internal.AbstractServiceRegistryImpl.createService(AbstractServiceRegistryImpl.java:176)
org.hibernate.service.internal.AbstractServiceRegistryImpl.initializeService(AbstractServiceRegistryImpl.java:150)
org.hibernate.service.internal.AbstractServiceRegistryImpl.getService(AbstractServiceRegistryImpl.java:131)
org.hibernate.engine.jdbc.internal.JdbcServicesImpl.buildJdbcConnectionAccess(JdbcServicesImpl.java:228)
org.hibernate.engine.jdbc.internal.JdbcServicesImpl.configure(JdbcServicesImpl.java:89)
org.hibernate.service.internal.StandardServiceRegistryImpl.configureService(StandardServiceRegistryImpl.java:75)
org.hibernate.service.internal.AbstractServiceRegistryImpl.initializeService(AbstractServiceRegistryImpl.java:159)
org.hibernate.service.internal.AbstractServiceRegistryImpl.getService(AbstractServiceRegistryImpl.java:131)
org.hibernate.cfg.Configuration.buildTypeRegistrations(Configuration.java:1818)
org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1776)
org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1861)
org.springframework.orm.hibernate4.LocalSessionFactoryBuilder.buildSessionFactory(LocalSessionFactoryBuilder.java:343)
org.springframework.orm.hibernate4.LocalSessionFactoryBean.buildSessionFactory(LocalSessionFactoryBean.java:431)
org.springframework.orm.hibernate4.LocalSessionFactoryBean.afterPropertiesSet(LocalSessionFactoryBean.java:416)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1612)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1549)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:539)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:304)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:300)
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:195)
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:328)
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:108)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1456)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1197)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:304)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:300)
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:195)
org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:703)
org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:760)
org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:482)
org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:658)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:624)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:672)
org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:543)
org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:484)
org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)
javax.servlet.GenericServlet.init(GenericServlet.java:158)
org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:610)
org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:518)
org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1091)
org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:668)
org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1517)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1474)
java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
java.lang.Thread.run(Unknown Source)
root cause


java.lang.ClassNotFoundException: org.hibernate.engine.jdbc.connections.spi.MultiTenantConnectionProvider
org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1305)
org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1157)
java.lang.ClassLoader.defineClass1(Native Method)
java.lang.ClassLoader.defineClass(Unknown Source)
java.security.SecureClassLoader.defineClass(Unknown Source)
org.apache.catalina.loader.WebappClassLoaderBase.findClassInternal(WebappClassLoaderBase.java:2472)
org.apache.catalina.loader.WebappClassLoaderBase.findClass(WebappClassLoaderBase.java:854)
org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1274)
org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1157)
org.hibernate.service.classloading.internal.ClassLoaderServiceImpl$AggregatedClassLoader.findClass(ClassLoaderServiceImpl.java:290)
java.lang.ClassLoader.loadClass(Unknown Source)
java.lang.ClassLoader.loadClass(Unknown Source)
java.lang.Class.forName0(Native Method)
java.lang.Class.forName(Unknown Source)
org.hibernate.service.classloading.internal.ClassLoaderServiceImpl.classForName(ClassLoaderServiceImpl.java:146)
org.hibernate.service.jdbc.connections.internal.MultiTenantConnectionProviderInitiator.initiateService(MultiTenantConnectionProviderInitiator.java:84)
org.hibernate.service.jdbc.connections.internal.MultiTenantConnectionProviderInitiator.initiateService(MultiTenantConnectionProviderInitiator.java:43)
org.hibernate.service.internal.StandardServiceRegistryImpl.initiateService(StandardServiceRegistryImpl.java:69)
org.hibernate.service.internal.AbstractServiceRegistryImpl.createService(AbstractServiceRegistryImpl.java:176)
org.hibernate.service.internal.AbstractServiceRegistryImpl.initializeService(AbstractServiceRegistryImpl.java:150)
org.hibernate.service.internal.AbstractServiceRegistryImpl.getService(AbstractServiceRegistryImpl.java:131)
org.hibernate.engine.jdbc.internal.JdbcServicesImpl.buildJdbcConnectionAccess(JdbcServicesImpl.java:228)
org.hibernate.engine.jdbc.internal.JdbcServicesImpl.configure(JdbcServicesImpl.java:89)
org.hibernate.service.internal.StandardServiceRegistryImpl.configureService(StandardServiceRegistryImpl.java:75)
org.hibernate.service.internal.AbstractServiceRegistryImpl.initializeService(AbstractServiceRegistryImpl.java:159)
org.hibernate.service.internal.AbstractServiceRegistryImpl.getService(AbstractServiceRegistryImpl.java:131)
org.hibernate.cfg.Configuration.buildTypeRegistrations(Configuration.java:1818)
org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1776)
org.hibernate.cfg.Configuration.buildSessionFactory(Configuration.java:1861)
org.springframework.orm.hibernate4.LocalSessionFactoryBuilder.buildSessionFactory(LocalSessionFactoryBuilder.java:343)
org.springframework.orm.hibernate4.LocalSessionFactoryBean.buildSessionFactory(LocalSessionFactoryBean.java:431)
org.springframework.orm.hibernate4.LocalSessionFactoryBean.afterPropertiesSet(LocalSessionFactoryBean.java:416)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1612)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1549)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:539)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:304)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:300)
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:195)
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:328)
org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveValueIfNecessary(BeanDefinitionValueResolver.java:108)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1456)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1197)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:304)
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:300)
org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:195)
org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:703)
org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:760)
org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:482)
org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:658)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:624)
org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:672)
org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:543)
org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:484)
org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:136)
javax.servlet.GenericServlet.init(GenericServlet.java:158)
org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502)
org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:79)
org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:610)
org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:518)
org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1091)
org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:668)
org.apache.coyote.http11.Http11NioProtocol$Http11ConnectionHandler.process(Http11NioProtocol.java:223)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1517)
org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(NioEndpoint.java:1474)
java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
java.lang.Thread.run(Unknown Source)
```


出现这个错误的原因可能有多个，我的是因为没有web-inf的jar包没有导入完全








### 错误12
```
No mapping found for HTTP request with URI [/saas/view/userlist.do] in DispatcherServlet with name 'springmvc'
```
标签错误，html结尾多了>

### 错误13

```
HTTP Status 500 - An exception occurred processing JSP page /view/staff_list.jsp at line 25
type Exception report


message An exception occurred processing JSP page /view/staff_list.jsp at line 25


description The server encountered an internal error that prevented it from fulfilling this request.


exception


org.apache.jasper.JasperException: An exception occurred processing JSP page /view/staff_list.jsp at line 25


22:
23: <c:forEach items="${list} " var="staff">
24: <tr>
25: <th>${staff.address}</th>
26: <th>${staff.name}</th>
27: <th>${staff.telephone}</th>
28: </tr>




Stacktrace:
org.apache.jasper.servlet.JspServletWrapper.handleJspException(JspServletWrapper.java:567)
org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:469)
org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:396)
org.apache.jasper.servlet.JspServlet.service(JspServlet.java:340)
javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
org.springframework.web.servlet.view.InternalResourceView.renderMergedOutputModel(InternalResourceView.java:168)
org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:303)
org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1244)
org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1027)
org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:971)
org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:893)
org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:966)
org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:857)
javax.servlet.http.HttpServlet.service(HttpServlet.java:622)
org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:842)
javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
root cause


javax.el.PropertyNotFoundException: Property 'address' not found on type java.lang.String
javax.el.BeanELResolver$BeanProperties.get(BeanELResolver.java:268)
javax.el.BeanELResolver$BeanProperties.access$300(BeanELResolver.java:221)
javax.el.BeanELResolver.property(BeanELResolver.java:355)
javax.el.BeanELResolver.getValue(BeanELResolver.java:95)
org.apache.jasper.el.JasperELResolver.getValue(JasperELResolver.java:110)
org.apache.el.parser.AstValue.getValue(AstValue.java:167)
org.apache.el.ValueExpressionImpl.getValue(ValueExpressionImpl.java:184)
org.apache.jasper.runtime.PageContextImpl.proprietaryEvaluate(PageContextImpl.java:936)
org.apache.jsp.view.staff_005flist_jsp._jspx_meth_c_005fforEach_005f0(staff_005flist_jsp.java:166)
org.apache.jsp.view.staff_005flist_jsp._jspService(staff_005flist_jsp.java:121)
org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:431)
org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:396)
org.apache.jasper.servlet.JspServlet.service(JspServlet.java:340)
javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
org.springframework.web.servlet.view.InternalResourceView.renderMergedOutputModel(InternalResourceView.java:168)
org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:303)
org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1244)
org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1027)
org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:971)
org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:893)
org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:966)
org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:857)
javax.servlet.http.HttpServlet.service(HttpServlet.java:622)
org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:842)
javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
note The full stack trace of the root cause is available in the Apache Tomcat/8.0.21 logs.




--------------------------------------------------------------------------------


Apache Tomcat/8.0.21

```

解决
仔细检查出错位置，我的原因是多了一个空格
<c:forEach items="${list} " var="staff">

### 错误14

```
type Exception report


message Unable to compile class for JSP:


description The server encountered an internal error that prevented it from fulfilling this request.


exception


org.apache.jasper.JasperException: Unable to compile class for JSP:






An error occurred at line: [73] in the generated java file: [D:\workspace\sts\.metadata\.plugins\org.eclipse.wst.server.core\tmp1\work\Catalina\localhost\maven\org\apache\jsp\view\staff_005flist_jsp.java]


The method getDispatcherType() is undefined for the type HttpServletRequest






Stacktrace:


org.apache.jasper.compiler.DefaultErrorHandler.javacError(DefaultErrorHandler.java:102)


org.apache.jasper.compiler.ErrorDispatcher.javacError(ErrorDispatcher.java:198)


org.apache.jasper.compiler.JDTCompiler.generateClass(JDTCompiler.java:450)


org.apache.jasper.compiler.Compiler.compile(Compiler.java:361)


org.apache.jasper.compiler.Compiler.compile(Compiler.java:336)


org.apache.jasper.compiler.Compiler.compile(Compiler.java:323)


org.apache.jasper.JspCompilationContext.compile(JspCompilationContext.java:585)


org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:363)


org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:396)


org.apache.jasper.servlet.JspServlet.service(JspServlet.java:340)


javax.servlet.http.HttpServlet.service(HttpServlet.java:729)


org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)


org.springframework.web.servlet.view.InternalResourceView.renderMergedOutputModel(InternalResourceView.java:168)


org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:303)


org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1244)


org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1027)


org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:971)


org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:893)


org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:966)


org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:857)


javax.servlet.http.HttpServlet.service(HttpServlet.java:622)


org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:842)


javax.servlet.http.HttpServlet.service(HttpServlet.java:729)


org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)

```



错误原因是因为在maven依赖中多了一个servlet-api.jar的jar包，具体原因不太清楚，但是如果加在pom文件中的话，发布会报错，删掉即可。




### 错误15

```
七月 31, 2015 12:18:20 上午 org.apache.catalina.core.StandardContext listenerStart
严重: Exception sending context initialized event to listener instance of class org.springframework.web.context.ContextLoaderListener
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'helloWorld': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: protected javax.validation.Validator com.saas.sysadmin.common.base.BaseController.validator; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type [javax.validation.Validator] found for dependency: expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:334)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1210)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:755)
    at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:757)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:480)
    at org.springframework.web.context.ContextLoader.configureAndRefreshWebApplicationContext(ContextLoader.java:403)
    at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:306)
    at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:106)
    at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:5016)
    at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5528)
    at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
    at org.apache.catalina.core.StandardContext.reload(StandardContext.java:4033)
    at org.apache.catalina.loader.WebappLoader.backgroundProcess(WebappLoader.java:425)
    at org.apache.catalina.core.ContainerBase.backgroundProcess(ContainerBase.java:1345)
    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.processChildren(ContainerBase.java:1546)
    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.processChildren(ContainerBase.java:1556)
    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.processChildren(ContainerBase.java:1556)
    at org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.run(ContainerBase.java:1524)
    at java.lang.Thread.run(Thread.java:744)
Caused by: org.springframework.beans.factory.BeanCreationException: Could not autowire field: protected javax.validation.Validator com.saas.sysadmin.common.base.BaseController.validator; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type [javax.validation.Validator] found for dependency: expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:561)
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88)
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:331)
    ... 24 more
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type [javax.validation.Validator] found for dependency: expected at least 1 bean which qualifies as autowire candidate for this dependency. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.raiseNoSuchBeanDefinitionException(DefaultListableBeanFactory.java:1301)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1047)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:942)
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:533)
    ... 26 more

```

错误原因是没有validatorbean


解决：
```
    <bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean" />

```



### 错误16

```
严重: Exception sending context initialized event to listener instance of class org.springframework.web.context.ContextLoaderListener
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'helloWorld': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: com.saas.sysadmin.model.sys.service.TenantService com.saas.sysadmin.model.sys.controller.HelloWorld.tenantService; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantServiceImpl': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private com.saas.sysadmin.model.sys.dao.TenantDao com.saas.sysadmin.model.sys.service.impl.TenantServiceImpl.tenantDao; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantDaoImpl' defined in file [D:\workspace\sts\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\wtpwebapps\sysadmin\WEB-INF\classes\com\saas\sysadmin\model\sys\dao\impl\TenantDaoImpl.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: 'sessionFactory' or 'hibernateTemplate' is required
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:334)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1210)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:755)
    at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:757)
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:480)
    at org.springframework.web.context.ContextLoader.configureAndRefreshWebApplicationContext(ContextLoader.java:403)
    at org.springframework.web.context.ContextLoader.initWebApplicationContext(ContextLoader.java:306)
    at org.springframework.web.context.ContextLoaderListener.contextInitialized(ContextLoaderListener.java:106)
    at org.apache.catalina.core.StandardContext.listenerStart(StandardContext.java:5016)
    at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5528)
    at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
    at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1575)
    at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1565)
    at java.util.concurrent.FutureTask.run(FutureTask.java:262)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
    at java.lang.Thread.run(Thread.java:744)
Caused by: org.springframework.beans.factory.BeanCreationException: Could not autowire field: com.saas.sysadmin.model.sys.service.TenantService com.saas.sysadmin.model.sys.controller.HelloWorld.tenantService; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantServiceImpl': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private com.saas.sysadmin.model.sys.dao.TenantDao com.saas.sysadmin.model.sys.service.impl.TenantServiceImpl.tenantDao; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantDaoImpl' defined in file [D:\workspace\sts\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\wtpwebapps\sysadmin\WEB-INF\classes\com\saas\sysadmin\model\sys\dao\impl\TenantDaoImpl.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: 'sessionFactory' or 'hibernateTemplate' is required
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:561)
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88)
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:331)
    ... 22 more
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantServiceImpl': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private com.saas.sysadmin.model.sys.dao.TenantDao com.saas.sysadmin.model.sys.service.impl.TenantServiceImpl.tenantDao; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantDaoImpl' defined in file [D:\workspace\sts\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\wtpwebapps\sysadmin\WEB-INF\classes\com\saas\sysadmin\model\sys\dao\impl\TenantDaoImpl.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: 'sessionFactory' or 'hibernateTemplate' is required
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:334)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1210)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1120)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1044)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:942)
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:533)
    ... 24 more
Caused by: org.springframework.beans.factory.BeanCreationException: Could not autowire field: private com.saas.sysadmin.model.sys.dao.TenantDao com.saas.sysadmin.model.sys.service.impl.TenantServiceImpl.tenantDao; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantDaoImpl' defined in file [D:\workspace\sts\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\wtpwebapps\sysadmin\WEB-INF\classes\com\saas\sysadmin\model\sys\dao\impl\TenantDaoImpl.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: 'sessionFactory' or 'hibernateTemplate' is required
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:561)
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88)
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:331)
    ... 35 more
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'tenantDaoImpl' defined in file [D:\workspace\sts\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\wtpwebapps\sysadmin\WEB-INF\classes\com\saas\sysadmin\model\sys\dao\impl\TenantDaoImpl.class]: Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: 'sessionFactory' or 'hibernateTemplate' is required
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1574)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:539)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:476)
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:303)
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:299)
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:194)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1120)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1044)
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:942)
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:533)
    ... 37 more
Caused by: java.lang.IllegalArgumentException: 'sessionFactory' or 'hibernateTemplate' is required
    at org.springframework.orm.hibernate4.support.HibernateDaoSupport.checkDaoConfig(HibernateDaoSupport.java:117)
    at org.springframework.dao.support.DaoSupport.afterPropertiesSet(DaoSupport.java:44)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1633)
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1570)
    ... 47 more  

```

我的错误是因为该Dao继承了HibernateDaoSupport，因此需要对他注入sessionfactory，有两种方法，一种是spring中直接配置一个bean

```
<bean id="tenantDaoImpl" class="com.saas.sysadmin.model.sys.dao.impl.TenantDAOImpl" >
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>  
```

第二种方式：
```
在com.saas.sysadmin.model.sys.dao.impl.TenantDAOImpl新建一个方法即可：
@Resource(name = "sessionFactory")
    public void setSuperSessionFactory(SessionFactory sessionFactory){
    super.setSessionFactory(sessionFactory);
    }  

```




### 错误17
```
javax.el.PropertyNotFoundException: Property 'status' not readable on type com.saas.admin.modules.sys.entity.Order
    at javax.el.BeanELResolver$BeanProperty.read(BeanELResolver.java:357)
    at javax.el.BeanELResolver$BeanProperty.access$000(BeanELResolver.java:305)
    at javax.el.BeanELResolver.getValue(BeanELResolver.java:97)
    at org.apache.jasper.el.JasperELResolver.getValue(JasperELResolver.java:104)
    at org.apache.el.parser.AstValue.getValue(AstValue.java:183)
    at org.apache.el.ValueExpressionImpl.getValue(ValueExpressionImpl.java:184)
    at org.apache.jasper.runtime.PageContextImpl.proprietaryEvaluate(PageContextImpl.java:944)
    at org.apache.jsp.WEB_002dINF.view.sysadmin.orderListAll_jsp._jspx_meth_c_005fforEach_005f0(orderListAll_jsp.java:494)
    at org.apache.jsp.WEB_002dINF.view.sysadmin.orderListAll_jsp._jspService(orderListAll_jsp.java:361)
    at org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
    at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:439)
    at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:395)
    at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:339)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.catalina.core.ApplicationDispatcher.invoke(ApplicationDispatcher.java:748)
    at org.apache.catalina.core.ApplicationDispatcher.processRequest(ApplicationDispatcher.java:486)
    at org.apache.catalina.core.ApplicationDispatcher.doForward(ApplicationDispatcher.java:411)
    at org.apache.catalina.core.ApplicationDispatcher.forward(ApplicationDispatcher.java:338)
    at org.springframework.web.servlet.view.InternalResourceView.renderMergedOutputModel(InternalResourceView.java:168)
    at org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:303)
    at org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1244)
    at org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1027)
    at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:971)
    at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:893)
    at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:966)
    at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:857)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:624)
    at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:842)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:85)
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:220)
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:170)
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)
    at org.apache.catalina.valves.AccessLogValve.invoke(AccessLogValve.java:957)
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:423)
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1079)
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:620)
    at org.apache.tomcat.util.net.AprEndpoint$SocketProcessor.doRun(AprEndpoint.java:2516)
    at org.apache.tomcat.util.net.AprEndpoint$SocketProcessor.run(AprEndpoint.java:2505)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:744)  
```

非常奇葩的一个错误，检查到最后发现原因是在实体类中，的getStatus方法不小心删除了，也就是没有get的方法，因此说找不到


### 错误18：

```
严重: Begin event threw error
java.lang.NoClassDefFoundError: javax/servlet/http/HttpServletRequest
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClass(ClassLoader.java:620)
at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:124)
at java.net.URLClassLoader.defineClass(URLClassLoader.java:260)
at java.net.URLClassLoader.access$100(URLClassLoader.java:56)
at java.net.URLClassLoader$1.run(URLClassLoader.java:195)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(URLClassLoader.java:188)
at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
at java.lang.ClassLoader.loadClass(ClassLoader.java:251)
at java.lang.ClassLoader.loadClassInternal(ClassLoader.java:319)
at java.lang.Class.getDeclaredMethods0(Native Method)
at java.lang.Class.privateGetDeclaredMethods(Class.java:2365)
at java.lang.Class.privateGetPublicMethods(Class.java:2488)
at java.lang.Class.getMethods(Class.java:1406)
at org.apache.tomcat.util.IntrospectionUtils.findMethods(IntrospectionUtils.java:812)
at org.apache.tomcat.util.IntrospectionUtils.setProperty(IntrospectionUtils.java:268)
at org.apache.catalina.startup.SetAllPropertiesRule.begin(SetAllPropertiesRule.java:66)
at org.apache.tomcat.util.digester.Digester.startElement(Digester.java:1276)
at com.sun.org.apache.xerces.internal.parsers.AbstractSAXParser.startElement(AbstractSAXParser.java:533)
at com.sun.org.apache.xerces.internal.parsers.AbstractXMLDocumentParser.emptyElement(AbstractXMLDocumentParser.java:220)
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanStartElement(XMLDocumentFragmentScannerImpl.java:872)
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl$FragmentContentDispatcher.dispatch(XMLDocumentFragmentScannerImpl.java:1693)
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanDocument(XMLDocumentFragmentScannerImpl.java:368)
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:834)
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:764)
at com.sun.org.apache.xerces.internal.parsers.XMLParser.parse(XMLParser.java:148)
at com.sun.org.apache.xerces.internal.parsers.AbstractSAXParser.parse(AbstractSAXParser.java:1242)
at org.apache.tomcat.util.digester.Digester.parse(Digester.java:1562)
at org.apache.catalina.startup.Catalina.load(Catalina.java:504)
at org.apache.catalina.startup.Catalina.load(Catalina.java:538)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
at java.lang.reflect.Method.invoke(Method.java:585)
at org.apache.catalina.startup.Bootstrap.load(Bootstrap.java:260)
at org.apache.catalina.startup.Bootstrap.main(Bootstrap.java:412)
java.lang.reflect.InvocationTargetException
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
at java.lang.reflect.Method.invoke(Method.java:585)
at org.apache.catalina.startup.Bootstrap.load(Bootstrap.java:260)
at org.apache.catalina.startup.Bootstrap.main(Bootstrap.java:412)
Caused by: java.lang.NoClassDefFoundError: javax/servlet/http/HttpServletRequest
at java.lang.ClassLoader.defineClass1(Native Method)
at java.lang.ClassLoader.defineClass(ClassLoader.java:620)
at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:124)
at java.net.URLClassLoader.defineClass(URLClassLoader.java:260)
at java.net.URLClassLoader.access$100(URLClassLoader.java:56)
at java.net.URLClassLoader$1.run(URLClassLoader.java:195)
at java.security.AccessController.doPrivileged(Native Method)
at java.net.URLClassLoader.findClass(URLClassLoader.java:188)
at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
at java.lang.ClassLoader.loadClass(ClassLoader.java:251)
at java.lang.ClassLoader.loadClassInternal(ClassLoader.java:319)
at java.lang.Class.getDeclaredMethods0(Native Method)
at java.lang.Class.privateGetDeclaredMethods(Class.java:2365)
at java.lang.Class.privateGetPublicMethods(Class.java:2488)
at java.lang.Class.getMethods(Class.java:1406)
at org.apache.tomcat.util.IntrospectionUtils.findMethods(IntrospectionUtils.java:812)
at org.apache.tomcat.util.IntrospectionUtils.setProperty(IntrospectionUtils.java:268)
at org.apache.catalina.startup.SetAllPropertiesRule.begin(SetAllPropertiesRule.java:66)
at org.apache.tomcat.util.digester.Digester.startElement(Digester.java:1276)
at com.sun.org.apache.xerces.internal.parsers.AbstractSAXParser.startElement(AbstractSAXParser.java:533)
at com.sun.org.apache.xerces.internal.parsers.AbstractXMLDocumentParser.emptyElement(AbstractXMLDocumentParser.java:220)
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanStartElement(XMLDocumentFragmentScannerImpl.java:872)
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl$FragmentContentDispatcher.dispatch(XMLDocumentFragmentScannerImpl.java:1693)
at com.sun.org.apache.xerces.internal.impl.XMLDocumentFragmentScannerImpl.scanDocument(XMLDocumentFragmentScannerImpl.java:368)
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:834)
at com.sun.org.apache.xerces.internal.parsers.XML11Configuration.parse(XML11Configuration.java:764)
at com.sun.org.apache.xerces.internal.parsers.XMLParser.parse(XMLParser.java:148)
at com.sun.org.apache.xerces.internal.parsers.AbstractSAXParser.parse(AbstractSAXParser.java:1242)
at org.apache.tomcat.util.digester.Digester.parse(Digester.java:1562)
at org.apache.catalina.startup.Catalina.load(Catalina.java:504)
at org.apache.catalina.startup.Catalina.load(Catalina.java:538)
... 6 more
```
解决：

把servlet-api.jar放到路径里面。

还有一种情况是，导入别人项目后，需要等一个刷新的过程，可能量三分钟再运行就ok了。

***
2016-03-21 10:27:00 hzct
