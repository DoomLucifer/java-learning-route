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

> 注：java中类是单继承的，接口是支持多继承的

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

PropertySource接口代表了键值对的Property来源。继承体系：

![PropertySource继承体系](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/PropertySource.jpg)



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

#### 路径placeholder处理

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

##### PropertyResolver接口

![PropertyResolver继承体系图](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/PropertyResolver.jpg)

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

#### prepareRefresh

```java
protected void prepareRefresh() {
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);
    if(this.logger.isInfoEnabled()) {
        this.logger.info("Refreshing " + this);
    }
	//空实现
    this.initPropertySources();
    //属性校验
    this.getEnvironment().validateRequiredProperties();
    this.earlyApplicationEvents = new LinkedHashSet();
}
```

##### 属性校验

AbstractEnvironment.validateRequiredProperties:

```java
public void validateRequiredProperties() throws MissingRequiredPropertiesException {
    this.propertyResolver.validateRequiredProperties();
}
```

AbstractPropertyResolver.validateRequiredProperties:

```java
public void validateRequiredProperties() {
    MissingRequiredPropertiesException ex = new MissingRequiredPropertiesException();
    Iterator var2 = this.requiredProperties.iterator();

    while(var2.hasNext()) {
        String key = (String)var2.next();
        if(this.getProperty(key) == null) {
            ex.addMissingRequiredProperty(key);
        }
    }

    if(!ex.getMissingRequiredProperties().isEmpty()) {
        throw ex;
    }
}
```

requiredProperties是通过setRequiredProperties方法设置的，保存在一个list里面，默认是空的，也就是不需要校验任何属性。

#### BeanFactory创建

obtainFreshBeanFactory调用AbstractRefreshableApplicationContext.refreshBeanFactory():

```java
protected final void refreshBeanFactory() throws BeansException {
    if(this.hasBeanFactory()) {
        this.destroyBeans();
        this.closeBeanFactory();
    }

    try {
        DefaultListableBeanFactory beanFactory = this.createBeanFactory();
        beanFactory.setSerializationId(this.getId());
        this.customizeBeanFactory(beanFactory);
        this.loadBeanDefinitions(beanFactory);
        Object var2 = this.beanFactoryMonitor;
        synchronized(this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    } catch (IOException var5) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + this.getDisplayName(), var5);
    }
}
```

##### BeanFactory接口

此接口实际上就是Bean容器，继承体系：

继承体系图

##### BeanFactory定制

AbstractRefreshableApplicationContext.customizeBeanFactory方法用于给子类提供一个自由配置的机会，默认实现：

```java
protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
    if(this.allowBeanDefinitionOverriding != null) {
        beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding.booleanValue());
    }

    if(this.allowCircularReferences != null) {
        beanFactory.setAllowCircularReferences(this.allowCircularReferences.booleanValue());
    }

}
```

##### Bean加载

AbstractXmlApplicationContext.loadBeanDefinitions()，bean加载的核心方法

```java
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
    this.initBeanDefinitionReader(beanDefinitionReader);
    this.loadBeanDefinitions(beanDefinitionReader);
}
```

###### EntityResolver

部分继承体系：

图

EntityResolver接口在org.xml.sax中定义，DelegatingEntityResolver用于schema和dtd的解析

###### BeanDefinitionReader

继承体系：

图

###### 路径解析(Ant)

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
    Resource[] configResources = this.getConfigResources();
    if(configResources != null) {
        reader.loadBeanDefinitions(configResources);
    }
	//AbstractRefreshableConfigApplicationContext中初始化configLocations
    String[] configLocations = this.getConfigLocations();
    if(configLocations != null) {
        reader.loadBeanDefinitions(configLocations);
    }

}
```

AbstractBeanDefinitionReader.loadBeanDefinitions

```java
public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {
    Assert.notNull(locations, "Location array must not be null");
    int counter = 0;
    String[] var3 = locations;
    int var4 = locations.length;

    for(int var5 = 0; var5 < var4; ++var5) {
        String location = var3[var5];
        counter += this.loadBeanDefinitions(location);
    }

    return counter;
}
```

之后调用：

```java
public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {
    return this.loadBeanDefinitions(location, (Set)null);
}

//第二个参数为null
public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
    ResourceLoader resourceLoader = this.getResourceLoader();
    if(resourceLoader == null) {
        throw new BeanDefinitionStoreException("Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
    } else {
        int loadCount;
        //参见ResourceLoader类图，ClassPathXMLApplicationContext实现了ResourcePatternResolver接口
        if(!(resourceLoader instanceof ResourcePatternResolver)) {
            // Can only load single resources by absolute URL
            Resource resource = resourceLoader.getResource(location);
            loadCount = this.loadBeanDefinitions((Resource)resource);
            if(actualResources != null) {
                actualResources.add(resource);
            }

            if(this.logger.isDebugEnabled()) {
                this.logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");
            }

            return loadCount;
        } else {
            try {
                //Resource pattern matching available
                Resource[] resources = ((ResourcePatternResolver)resourceLoader).getResources(location);
                loadCount = this.loadBeanDefinitions(resources);
                if(actualResources != null) {
                    Resource[] var6 = resources;
                    int var7 = resources.length;

                    for(int var8 = 0; var8 < var7; ++var8) {
                        Resource resource = var6[var8];
                        actualResources.add(resource);
                    }
                }

                if(this.logger.isDebugEnabled()) {
                    this.logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
                }

                return loadCount;
            } catch (IOException var10) {
                throw new BeanDefinitionStoreException("Could not resolve bean definition resource pattern [" + location + "]", var10);
            }
        }
    }
}
```

getResource的实现在AbstractApplicationContext：

```java
public Resource[] getResources(String locationPattern) throws IOException {
    //AbstractApplicationContext构造器中初始化ResourcePatternResolver接口的实现PathMatchingResourcePatternResolver对象
    return this.resourcePatternResolver.getResources(locationPattern);
}
```

PathMatchingResourcePatternResolver是ResourceLoader继承体系的一部分。

```java
public Resource[] getResources(String locationPattern) throws IOException {
    Assert.notNull(locationPattern, "Location pattern must not be null");
    if(locationPattern.startsWith("classpath*:")) {
        //matcher是一个AntPathMatcher对象
        return this.getPathMatcher().isPattern(locationPattern.substring("classpath*:".length()))?this.findPathMatchingResources(locationPattern):this.findAllClassPathResources(locationPattern.substring("classpath*:".length()));
    } else {
        int prefixEnd = locationPattern.startsWith("war:")?locationPattern.indexOf("*/") + 1:locationPattern.indexOf(58) + 1;
        return this.getPathMatcher().isPattern(locationPattern.substring(prefixEnd))?this.findPathMatchingResources(locationPattern):new Resource[]{this.getResourceLoader().getResource(locationPattern)};
    }
}
```

isPattern:

```java
@Override
public boolean isPattern(String path) {
    return (path.indexOf('*') != -1 || path.indexOf('?') != -1);
}
```

通过上面两个方法可以看出配置文件路径是支持ant风格的，也就是可以这么写：

```java
new ClassPathXmlApplicationContext("con*.xml");
```

###### 配置文件加载

入口方法在AbstractBeanDefinitionReader的loadBeanDefinitions方法里

```java
//加载
Resource[] resources = ((ResourcePatternResolver)resourceLoader).getResources(location);
//解析
int loadCount = this.loadBeanDefinitions(resources);
```

最终逐个调用XmlBeanDefinitionReader的loadBeanDefinitions方法

```java
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
    return this.loadBeanDefinitions(new EncodedResource(resource));
}
```

Resource是代表一种资源的接口，类图如下：

EncodedResource扮演的其实是一个装饰器的模式，为InputStreamSource添加了字符编码（虽然默认为null），这样为我们自定义xml配置文件的编码方式提供了机会。

之后关键的源码只有两行：

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
    InputStream inputStream = encodedResource.getResource().getInputStream();
    InputSource inputSource = new InputSource(inputStream);
    return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
}
```

InputSource是org.xml.sax的类

doLoadBeanDefinitions:

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource) {
    Document doc = doLoadDocument(inputSource, resource);
    return registerBeanDefinitions(doc, resource);
}
```

doLoadDocument:

```java
protected Document doLoadDocument(InputSource inputSource, Resource resource) {
    return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
        getValidationModeForResource(resource), isNamespaceAware());
}
```

documentLoader是一个DefaultDocumentLoader对象，此类是DocumentLoader接口的唯一实现。getEntityResolver方法返回ResourceEntityResolver，上面说过了。errorHandler是一个SimpleSaxErrorHandler对象。

校验模型其实就是确定xml文件使用xsd方式还是dtd方式来校验，忘了的话左转度娘。Spring会通过读取xml文件的方式判断应该采用哪种。

NamespaceAware默认false，因为默认配置了校验为true。

DefaultDocumentLoader.loadDocument:

```java
@Override
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
    ErrorHandler errorHandler, int validationMode, boolean namespaceAware) {
    //这里就是老套路了，可以看出，Spring还是使用了dom的方式解析，即一次全部load到内存
    DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
    DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
    return builder.parse(inputSource);
}
```

createDocumentBuilderFactory:

```java
protected DocumentBuilderFactory createDocumentBuilderFactory(int validationMode, boolean namespaceAware) throws ParserConfigurationException {
    DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
    factory.setNamespaceAware(namespaceAware);
    if(validationMode != 0) {
        //此方法设为true进度dtd有效，xsd(scheme)无效
        factory.setValidating(true);
        if(validationMode == 3) {
            //开启xsd(scheme)支持
            factory.setNamespaceAware(true);

            try {
                factory.setAttribute("http://java.sun.com/xml/jaxp/properties/schemaLanguage", "http://www.w3.org/2001/XMLSchema");
            } catch (IllegalArgumentException var6) {
                ParserConfigurationException pcex = new ParserConfigurationException("Unable to validate using XSD: Your JAXP provider [" + factory + "] does not support XML Schema. Are you running on Java 1.4 with Apache Crimson? Upgrade to Apache Xerces (or Java 1.5) for full XSD support.");
                pcex.initCause(var6);
                throw pcex;
            }
        }
    }

    return factory;
}
```

###### Bean解析

XmlBeanDefinitionReader.registerBeanDefinitions:

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    BeanDefinitionDocumentReader documentReader = this.createBeanDefinitionDocumentReader();
    int countBefore = this.getRegistry().getBeanDefinitionCount();
    documentReader.registerBeanDefinitions(doc, this.createReaderContext(resource));
    return this.getRegistry().getBeanDefinitionCount() - countBefore;
}
```

createBeanDefinitionDocumentReader:

```java
protected BeanDefinitionDocumentReader createBeanDefinitionDocumentReader() {
    return (BeanDefinitionDocumentReader)BeanDefinitionDocumentReader.class.cast(BeanUtils.instantiateClass(this.documentReaderClass));
}
```

documentReaderClass默认是DefaultBeanDefinitionDocumentReader，这其实也是策略模式，通过setter方法可以更换其实现。

注意cast方法，代替了强转。

createReaderContext:

```java
public XmlReaderContext createReaderContext(Resource resource) {
    return new XmlReaderContext(resource, this.problemReporter, this.eventListener, this.sourceExtractor, this, this.getNamespaceHandlerResolver());
}
```

problemReporter是一个FailFastProblemReporter对象。

eventListener是EmptyReaderEventListener对象，此类里的方法都是空实现。

sourceExtractor是NullSourceExtractor对象，直接返回空，也是空实现。

getNamespaceHandlerResolver默认返回DefaultNamespaceHandlerResolver对象，用来获取xsd对应的处理器。

XmlReaderContext的作用感觉就是这一堆参数的容器，糅合到一起传给DocumentReader，并美其名为Context。可以看出，Spring中到处都是策略模式，大量操作被抽象成接口。

DefaultBeanDefinitionDocumentReader.registerBeanDefinitions:

```java
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
    this.readerContext = readerContext;
    this.logger.debug("Loading bean definitions");
    Element root = doc.getDocumentElement();
    this.doRegisterBeanDefinitions(root);
}
```

doRegisterBeanDefinitions:

```java
protected void doRegisterBeanDefinitions(Element root) {
    BeanDefinitionParserDelegate parent = this.delegate;
    this.delegate = this.createDelegate(this.getReaderContext(), root, parent);
    //默认的命名空间即
    //http://www.springframework.org/schema/beans
    if(this.delegate.isDefaultNamespace(root)) {
        //检查profile属性
        String profileSpec = root.getAttribute("profile");
        if(StringUtils.hasText(profileSpec)) {
            //profile属性可以以，分割
            String[] specifiedProfiles = StringUtils.tokenizeToStringArray(profileSpec, ",; ");
            if(!this.getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
                if(this.logger.isInfoEnabled()) {
                    this.logger.info("Skipped XML bean definition file due to specified profiles [" + profileSpec + "] not matching: " + this.getReaderContext().getResource());
                }

                return;
            }
        }
    }

    this.preProcessXml(root);
    this.parseBeanDefinitions(root, this.delegate);
    this.postProcessXml(root);
    this.delegate = parent;
}
```

delegate的作用在与处理beans标签的嵌套，其实Spring配置文件是可以写成这样的：

```xml
<?xml version="1.0" encoding="UTF-8"?>    
<beans>    
    <bean class="base.SimpleBean"></bean>
    <beans>
        <bean class="java.lang.Object"></bean>
    </beans>
</beans>
```

xml(schema)的命名空间其实类似于java的包名，命名空间采用URL，比如Spring的是这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>    
<beans xmlns="http://www.springframework.org/schema/beans"></beans>
```

xmlns属性就是xml规范定义的用来设置命名空间的。这样设置了之后其实里面的bean元素全名就相当于http://www.springframework.org/schema/beans:bean，可以有效的防止命名冲突。命名空间可以通过规范定义的org.w3c.dom.Node.getNamespaceURI方法获得。

注意一下profile的检查，AbstractEnvironment.acceptsProfiles:

```java
@Override
public boolean acceptsProfiles(String... profiles) {
    Assert.notEmpty(profiles, "Must specify at least one profile");
    for (String profile : profiles) {
        if (StringUtils.hasLength(profile) && profile.charAt(0) == '!') {
            if (!isProfileActive(profile.substring(1))) {
                return true;
            }
        } else if (isProfileActive(profile)) {
            return true;
        }
    }
    return false;
}
```

原理很简单，注意从源码可以看出，**profile属性支持!取反**。

preProcessXml方法是个空实现，供子类去覆盖，**目的在于给子类一个把我们自定义的标签转为Spring标准标签的机会**, 想的真周到。

DefaultBeanDefinitionDocumentReader.parseBeanDefinitions:

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    if(delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();

        for(int i = 0; i < nl.getLength(); ++i) {
            Node node = nl.item(i);
            if(node instanceof Element) {
                Element ele = (Element)node;
                if(delegate.isDefaultNamespace(ele)) {
                    this.parseDefaultElement(ele, delegate);
                } else {
                    delegate.parseCustomElement(ele);
                }
            }
        }
    } else {
        delegate.parseCustomElement(root);
    }

}
```

可见，对于非默认命名空间的元素交由delegate处理。

###### 默认命名空间解析

即import，alias，bean，嵌套的beans四种元素。parseDefaultElement:

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    if(delegate.nodeNameEquals(ele, "import")) {
        this.importBeanDefinitionResource(ele);
    } else if(delegate.nodeNameEquals(ele, "alias")) {
        this.processAliasRegistration(ele);
    } else if(delegate.nodeNameEquals(ele, "bean")) {
        this.processBeanDefinition(ele, delegate);
    } else if(delegate.nodeNameEquals(ele, "beans")) {
        this.doRegisterBeanDefinitions(ele);
    }
}
```

- import

写法示例：

```xml
<import resource="CTIContext.xml"/>
<import resource="customerContext.xml"/>
```

importBeanDefinitionResource套路和之前的配置文件加载完全一样，不过注意被import进来的文件是先于当前文件 被解析的。

- alias

加入有一个bean名为componentA-dataSource，但是另一个组件想以componentB-dataSource的名字使用，就可以这样定义:

```xml
<alias name="componentA-dataSource" alias="componentB-dataSource"/>
```

processAliasRegistration核心源码：

```java
protected void processAliasRegistration(Element ele) {
    String name = ele.getAttribute("name");
    String alias = ele.getAttribute("alias");
    getReaderContext().getRegistry().registerAlias(name, alias);
    getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
}
```

从前面的源码可以发现，registry其实就是DefaultListableBeanFactory，它实现了BeanDefinitionRegistry接口。registerAlias方法的实现在SimpleAliasRegistry:

```java
public void registerAlias(String name, String alias) {
    Assert.hasText(name, "'name' must not be empty");
    Assert.hasText(alias, "'alias' must not be empty");
    Map var3 = this.aliasMap;
    synchronized(this.aliasMap) {
        //名字和别名一样
        if(alias.equals(name)) {
            //ConcurrentHashMap
            this.aliasMap.remove(alias);
        } else {
            String registeredName = (String)this.aliasMap.get(alias);
            if(registeredName != null) {
                if(registeredName.equals(name)) {
                    return;
                }

                if(!this.allowAliasOverriding()) {
                    throw new IllegalStateException("Cannot register alias '" + alias + "' for name '" + name + "': It is already registered for name '" + registeredName + "'.");
                }
            }

            this.checkForAliasCircle(name, alias);
            this.aliasMap.put(alias, name);
        }
    }
}
```

所以别名关系的保存使用Map完成，key为别名，value为本来的名字。

- bean

DefaultBeanDefinitionDocumentReader.processBeanDefinition:

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if(bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // Register the final decorated instance.
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, this.getReaderContext().getRegistry());
        } catch (BeanDefinitionStoreException var5) {
            this.getReaderContext().error("Failed to register bean definition with name '" + bdHolder.getBeanName() + "'", ele, var5);
        }
		// Send registration event.
        this.getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }

}
```

最终调用BeanDefinitionParserDelegate.parseBeanDefinitionElement(Element ele, BeanDefinition containingBean)，源码较长，分部分说明。

第一部分：id & name处理

首先获取到id和name属性，**name属性支持配置多个，以逗号分隔，如果没有指定id，那么将以第一个name属性值代替。id必须是唯一的，name属性其实是alias的角色，可以和其它的bean重复，如果name也没有配置，那么其实什么也没做**。

```java
String id = ele.getAttribute("id");
String nameAttr = ele.getAttribute("name");
List<String> aliases = new ArrayList<String>();
if (StringUtils.hasLength(nameAttr)) {
    //按,分隔
    String[] nameArr = StringUtils.tokenizeToStringArray
        (nameAttr, ",;");
    aliases.addAll(Arrays.asList(nameArr));
}
String beanName = id;
if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
    //name的第一个值作为id
    beanName = aliases.remove(0);
}
//默认null
if (containingBean == null) {
    //校验id是否已重复，如果重复直接抛异常
    //校验是通过内部一个HashSet完成的，出现过的id都会保存进此Set
    checkNameUniqueness(beanName, aliases, ele);
}
```

beanName 生成

如果name和id属性都没有指定，那么Spring会自己生成一个, BeanDefinitionParserDelegate.parseBeanDefinitionElement:

```java
beanName = this.readerContext.generateBeanName(beanDefinition);
String beanClassName = beanDefinition.getBeanClassName();
if(beanClassName != null && beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() && !this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
    aliases.add(beanClassName);
}
```

可见，Spring同时会把类名作为其别名。

最终调用的是BeanDefinitionReaderUtils.generateBeanName:

```java
public static String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry, boolean isInnerBean) throws BeanDefinitionStoreException {
    String generatedBeanName = definition.getBeanClassName();
    if(generatedBeanName == null) {
        if(definition.getParentName() != null) {
            generatedBeanName = definition.getParentName() + "$child";
        } else if(definition.getFactoryBeanName() != null) {
            //工厂方法产生的bean
            generatedBeanName = definition.getFactoryBeanName() + "$created";
        }
    }

    if(!StringUtils.hasText(generatedBeanName)) {
        throw new BeanDefinitionStoreException("Unnamed bean definition specifies neither 'class' nor 'parent' nor 'factory-bean' - can't generate bean name");
    } else {
        String id = generatedBeanName;
        if(isInnerBean) {
            // Inner bean: generate identity hashcode suffix.
            id = generatedBeanName + "#" + ObjectUtils.getIdentityHexString(definition);
        } else {
            // Top-level bean: use plain class name.
        	// Increase counter until the id is unique.
            for(int counter = -1; counter == -1 || registry.containsBeanDefinition(id); id = generatedBeanName + "#" + counter) {
                ++counter;
            }
        }

        return id;
    }
}
```

第二部分：bean解析

方法实现：this.parseBeanDefinitionElement(ele,beanName,containingBean)

1. 首先获取到bean的class属性和parent属性，配置了parent之后，当前bean会继承父bean的属性。之后根据class和parent创建BeanDefinition对象。

```java
public AbstractBeanDefinition parseBeanDefinitionElement(Element ele, String beanName, BeanDefinition containingBean) {
    this.parseState.push(new BeanEntry(beanName));
    String className = null;
    if(ele.hasAttribute("class")) {
        className = ele.getAttribute("class").trim();
    }

    try {
        String parent = null;
        if(ele.hasAttribute("parent")) {
            parent = ele.getAttribute("parent");
        }
		//创建BeanDefinition
        AbstractBeanDefinition bd = this.createBeanDefinition(className, parent);
        this.parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
        bd.setDescription(DomUtils.getChildElementValueByTagName(ele, "description"));
        this.parseMetaElements(ele, bd);
        this.parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
        this.parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
        this.parseConstructorArgElements(ele, bd);
        this.parsePropertyElements(ele, bd);
        this.parseQualifierElements(ele, bd);
        bd.setResource(this.readerContext.getResource());
        bd.setSource(this.extractSource(ele));
        AbstractBeanDefinition var7 = bd;
        return var7;
    } catch (ClassNotFoundException var13) {
        this.error("Bean class [" + className + "] not found", ele, var13);
    } catch (NoClassDefFoundError var14) {
        this.error("Class that bean class [" + className + "] depends on not found", ele, var14);
    } catch (Throwable var15) {
        this.error("Unexpected failure during bean definition parsing", ele, var15);
    } finally {
        this.parseState.pop();
    }

    return null;
}
```

```java
protected AbstractBeanDefinition createBeanDefinition(String className, String parentName) throws ClassNotFoundException {
    return BeanDefinitionReaderUtils.createBeanDefinition(parentName, className, this.readerContext.getBeanClassLoader());
}
```

BeanDefinition的创建在BeanDefinitionReaderUtils.createBeanDefinition:

```java
public static AbstractBeanDefinition createBeanDefinition(String parentName, String className, ClassLoader classLoader) throws ClassNotFoundException {
    GenericBeanDefinition bd = new GenericBeanDefinition();
    bd.setParentName(parentName);
    if(className != null) {
        if(classLoader != null) {
            bd.setBeanClass(ClassUtils.forName(className, classLoader));
        } else {
            bd.setBeanClassName(className);
        }
    }
    return bd;
}
```

2. 解析bean的其它属性，其实就是读取其配置，调用相应的setter方法保存在BeanDefinition中

```java
this.parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
```

3. 解析bean的description子元素

```xml
<bean id="b" name="one, two" class="base.SimpleBean">
    <description>SimpleBean</description>
</bean>
```

4. 解析Meta子元素

```xml
<bean id="b" name="one, two" class="base.SimpleBean">
    <meta key="name" value="garaiya"/>
</bean>
```

注释上说，这样可以将任意的元数据附到对应的bean definition上。解析过程源码:

```java
public void parseMetaElements(Element ele, BeanMetadataAttributeAccessor attributeAccessor) {
    NodeList nl = ele.getChildNodes();
    for (int i = 0; i < nl.getLength(); i++) {
        Node node = nl.item(i);
        if (isCandidateElement(node) && nodeNameEquals(node, META_ELEMENT)) {
            Element metaElement = (Element) node;
            String key = metaElement.getAttribute(KEY_ATTRIBUTE);
            String value = metaElement.getAttribute(VALUE_ATTRIBUTE);
             //就是一个key, value的载体，无他
            BeanMetadataAttribute attribute = new BeanMetadataAttribute(key, value);
             //sourceExtractor默认是NullSourceExtractor，返回的是空
            attribute.setSource(extractSource(metaElement));
            attributeAccessor.addMetadataAttribute(attribute);
        }
    }
}
```

AbstractBeanDefinition继承自BeanMetadataAttributeAccessor类，底层使用了一个LinkedHashMap保存metadata。这个metadata具体是做什么暂时还不知道

5. lookup-method解析

此标签的作用在于当一个bean的某个方法被设置为lookup-method后，**每次调用此方法时，都会返回一个新的指定bean的对象**。用法示例:

```xml
<bean id="apple" class="cn.com.willchen.test.di.Apple" scope="prototype"/>
<!--水果盘-->
<bean id="fruitPlate" class="cn.com.willchen.test.di.FruitPlate">
    <lookup-method name="getFruit" bean="apple"/>
</bean>
```

数据保存在Set中，对应的类是MethodOverrides。可以参考：

[Spring-lookup-method方式实现依赖注入](http://www.cnblogs.com/ViviChan/p/4981619.html)

6. replace-method解析

此标签用于替换bean里面的特定的方法实现，替换者必须实现Spring的MethodReplacer接口。用法示例：

```xml
<bean name="replacer" class="springroad.deomo.chap4.MethodReplace" />  
<bean name="testBean" class="springroad.deomo.chap4.LookupMethodBean">
    <replaced-method name="test" replacer="replacer">
        <arg-type match="String" />
    </replaced-method>  
</bean> 
```

arg-type的作用是指定替换方法的参数类型，因为接口的定义参数都是Object的。参考：[Spring.net 1.3.2 学习20--方法注入之替换方法注入](https://blog.csdn.net/lee576/article/details/8725548)

解析之后将数据放在ReplaceOverride对象中，里面有个LinkedList专门用于保存arg-type。

7. 构造参数(constructor-arg)解析

```xml
<bean class="base.SimpleBean">
    <constructor-arg>
        <value type="java.lang.String">Cat</value>
    </constructor-arg>
</bean>
```

type一般不需要指定，除了泛型集合那种。除此之外，constructor-arg还支持name, index, ref等属性，可以具体的指定参数的位置等。构造参数解析后保存在BeanDefinition内部一个ConstructorArgumentValues对象中。如果设置了index属性，那么以Map<Integer, ValueHolder>的形式保存，反之，以List的形式保存。

8. Property解析

用来为bean的属性赋值，支持value和ref两种形式

```xml
<bean class="base.SimpleBean">
    <property name="name" value="garaiya" />
</bean>
```

9. ceshi 
10. 