[TOC]

# Spring Core源码阅读

## ClassPathXmlApplicationContext类图

![ClassPathXmlApplicationContext.class](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/ResourceLoader.jpg)

ResourceLoader是加载资源的一种方式，使用了策略模式。

ClassPathXmlApplicationContext类构造器源码：

```java
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
    //父类构造器
    super(parent);
    this.setConfigLocations(configLocations);
    if(refresh) {
    	this.refresh();
	}
}
```

### 构造器调用顺序

```java
super(parent)-->AbstractXmlApplicationContext(ApplicationContext parent)-->AbstractRefreshableConfigApplicationContext(ApplicationContext parent)-->AbstractApplicationContext(ApplicationContext parent){
    this();
    this.setParent(parent);
}
//this()
public AbstractApplicationContext() {

	this.logger = LogFactory.getLog(this.getClass());
    this.id = ObjectUtils.identityToString(this);
    this.displayName = ObjectUtils.identityToString(this);
    this.beanFactoryPostProcessors = new ArrayList();
    this.active = new AtomicBoolean();
    this.closed = new AtomicBoolean();
    this.startupShutdownMonitor = new Object();
    this.applicationListeners = new LinkedHashSet();
    this.resourcePatternResolver = this.getResourcePatternResolver();
}
//this.resourcePatternResolver = this.getResourcePatternResolver();
//ResourcePatternResolver继承ResourceLoader(只支持获取单个Resource)接口，添加了获取多个Resource的方法，PathMatchingResourcePatternResolver是spring为ResourcePatternResolver接口提供的默认实现，基于模式匹配，默认使用AntPathMatcher进行路径匹配
protected ResourcePatternResolver getResourcePatternResolver() {
    //PathMatchingResourcePatternResolver支持Ant风格的路径解析
	return new PathMatchingResourcePatternResolver(this);
}
```

### 设置配置文件路径

调用AbstractRefreshableConfigApplicationContext.setConfigLocations方法

```java
public void setConfigLocations(String... locations) {
	if(locations != null) {
		Assert.noNullElements(locations, "Config locations must not be null");
        this.configLocations = new String[locations.length];

        for(int i = 0; i < locations.length; ++i) {
			this.configLocations[i] = this.resolvePath(locations[i]).trim();
        }
	} else {
		this.configLocations = null;
	}
}
//resolvePath(String path)方法用于将占位符(placeholder)解析成实际的地址，比如new ClassPathXmlApplicationContext("classpath:config.xml"),"classpath:"则会被解析
protected String resolvePath(String path) {
	return this.getEnvironment().resolveRequiredPlaceholders(path);
}
//getEnvironment方法来自于ConfigurableApplicationContext接口,返回ConfigurableEnvironment,AbstractApplicationContext对该方法进行了实现
public ConfigurableEnvironment getEnvironment() {
	if(this.environment == null) {
		this.environment = this.createEnvironment();
	}
	return this.environment;
}
```

#### Environment接口

继承体系：图

Environment接口表示当前应用所处的环境，从接口的方法可以看出，其主要和profile、property相关

##### profile

Spring profile特性是从3.1开始的，解决了不同环境使用不用的配置并且能够无缝切换（运行阶段完成）。用maven profiles也能够实现，实现方式是在构建阶段将对应环境的配置文件编译到可部署的应用中。这种方式的问题是要为每种环境重新构建应用，生产环境重新构建可能会引入bug。

```xml
<beans profile="develop">  
    <context:property-placeholder location="classpath*:jdbc-develop.properties"/>  
</beans>  
<beans profile="production">  
    <context:property-placeholder location="classpath*:jdbc-production.properties"/>  
</beans>  
<beans profile="test">  
    <context:property-placeholder location="classpath*:jdbc-test.properties"/>  
</beans>
```

启动代码用如下代码设置profile：

```java
context.getEnvironment().setActiveProfiles("dev")
```

基于注解方式的实现参考[Spring Profiles example](http://www.mkyong.com/spring/spring-profiles-example/)

##### property

这里的property指的是程序运行时的一些参数

> properties files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc Properties objects,Maps, and so on.

##### Environment构造器

```java
public AbstractEnvironment() {
	this.propertySources = new MutablePropertySources(this.logger);
    this.propertyResolver = new PropertySourcesPropertyResolver(this.propertySources);
    this.customizePropertySources(this.propertySources);
    if(this.logger.isDebugEnabled()) {
		this.logger.debug("Initialized " + this.getClass().getSimpleName() + " with PropertySources " + this.propertySources);
	}
}
```

###### PropertySources接口

