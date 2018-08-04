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

![Environment类图](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/Environment.jpg)



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

![PropertySources类图](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/PropertySources.jpg)

该接口是PropertySource的容器，默认的MutablePropertySources实现内部含有一个CopyOnWriteArrayList作为存储载体。

StandardEnvironment.customizePropertySources方法

```java
public static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";
public static final String SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties";

public StandardEnvironment() {
}

protected void customizePropertySources(MutablePropertySources propertySources) {
	propertySources.addLast(new MapPropertySource("systemProperties", this.getSystemProperties()));
	propertySources.addLast(new SystemEnvironmentPropertySource("systemEnvironment", this.getSystemEnvironment()));
}
```

###### PropertySource接口

PropertySource接口代表了键值对的Property来源。继承体系：(图)

AbstractEnvironment.getSystemProperties方法

```java
public Map<String, Object> getSystemProperties() {
    try {
        return System.getProperties();
    } catch (AccessControlException var2) {
        return new ReadOnlySystemAttributesMap() {
            protected String getSystemAttribute(String attributeName) {
                try {
                    return System.getProperty(attributeName);
                } catch (AccessControlException var3) {
                    if(AbstractEnvironment.this.logger.isInfoEnabled()) {
                        AbstractEnvironment.this.logger.info("Caught AccessControlException when accessing system property '" + attributeName + "'; its value will be returned [null]. Reason: " + var3.getMessage());
                    }

                    return null;
                }
            }
        };
    }
}
```

这里的实现很有意思，如果安全管理器阻止获取全部的系统属性，那么会尝试获取单个属性的可能性，如果还不行就抛异常了。

AbstractEnvironment.getSystemEnvironment方法也是一个套路，不过最终调用的是System.getenv()，可以获取jvm和OS的一些版本信息。

###### 路径placeholder处理

AbstractEnvironment.resolveRequiredPlaceholders方法

```java
public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
    //text即配置文件路径，比如classpath:config.xml
    return this.propertyResolver.resolveRequiredPlaceholders(text);
}
```

propertyResolver是一个PropertySourcesPropertyResolver对象

AbstractEnvironment的构造方法里：

```java
this.propertyResolver = new PropertySourcesPropertyResolver(this.propertySources);
```

###### PropertyResolver接口

PropertyResolver继承体系图

此接口用来解析PropertyResource

AbstractPropertyResolver.resolveRequiredPlaceholders方法

```java
public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {
    if(this.strictHelper == null) {
        this.strictHelper = this.createPlaceholderHelper(false);
    }

    return this.doResolvePlaceholders(text, this.strictHelper);
}
```

```java
private PropertyPlaceholderHelper createPlaceholderHelper(boolean ignoreUnresolvablePlaceholders) {
    return new PropertyPlaceholderHelper(this.placeholderPrefix, this.placeholderSuffix, this.valueSeparator, ignoreUnresolvablePlaceholders);
}
```

```java
private String doResolvePlaceholders(String text, PropertyPlaceholderHelper helper) {
    return helper.replacePlaceholders(text, new PlaceholderResolver() {
        public String resolvePlaceholder(String placeholderName) {
            return AbstractPropertyResolver.this.getPropertyAsRawString(placeholderName);
        }
    });
}
```

到这里代码还有进行xml配置文件的解析，这里placeholder表示的是资源前缀，比如：

```java
System.setProperty("spring","classpath");
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("${spring}:config.xml");
SimpleBean bean = context.getBean(SimpleBean.class);
```

这样就可以正确解析。placeholder的替换其实就是字符串操作，这里只说一下正确的属性是怎么来的。实现的关键在于PropertySourcesPropertyResolver.getProperty:

```java
protected String getPropertyAsRawString(String key) {
    return (String)this.getProperty(key, String.class, false);
}

protected <T> T getProperty(String key, Class<T> targetValueType, boolean resolveNestedPlaceholders) {
    if(this.propertySources != null) {
        Iterator var4 = this.propertySources.iterator();
        while(var4.hasNext()) {
            PropertySource<?> propertySource = (PropertySource)var4.next();
            Object value = propertySource.getProperty(key);
            if(value != null) {
                if(resolveNestedPlaceholders && value instanceof String) {
                    value = this.resolveNestedPlaceholders((String)value);
                }
                this.logKeyFound(key, propertySource, value);
                return this.convertValueIfNecessary(value, targetValueType);
            }
        }
    }
    return null;
}
```

其实就是从System.getProperty和System.getenv获取，但是由于环境变量是无法自定义的，所以其实只能通过System.setProperty指定。

> 注意，classpath:xxx这种写法的classpath前缀到目前为止还没有被处理

### refresh

Spring bean解析就在此方法中

AbstractApplicationContext.refresh方法

```java
public void refresh() throws BeansException, IllegalStateException {
    Object var1 = this.startupShutdownMonitor;
    synchronized(this.startupShutdownMonitor) {
        this.prepareRefresh();
        ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
        this.prepareBeanFactory(beanFactory);

        try {
            this.postProcessBeanFactory(beanFactory);
            this.invokeBeanFactoryPostProcessors(beanFactory);
            this.registerBeanPostProcessors(beanFactory);
            this.initMessageSource();
            this.initApplicationEventMulticaster();
            this.onRefresh();
            this.registerListeners();
            this.finishBeanFactoryInitialization(beanFactory);
            this.finishRefresh();
        } catch (BeansException var9) {
            if(this.logger.isWarnEnabled()) {
                this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
            }

            this.destroyBeans();
            this.cancelRefresh(var9);
            throw var9;
        } finally {
            this.resetCommonCaches();
        }

    }
}
```





