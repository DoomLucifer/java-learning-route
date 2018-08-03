# IOC&DI
> Ioc--Inversion of Control，即"控制反转".
- 控制
> 传统JavaSE中，直接在对象内部通过new进行创建对象，是程序主动创建依赖对象；而IOC是由IOC容器来控制对象的创建，也就是有容器来控制外部资源的获取（不只是对象还包括文件等）。
- 反转
> 传统应用程序是由我们自己在对象中主动创建依赖对象，是正转；而反转则是由容器来创建及注入依赖对象，为何是反转？因为容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；依赖对象的获取被反转了。

- 传统JavaSE
![传统JavaSE](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/JavaSE-traditional.png)
- IOC容器
![094c4afc.png](:storage/04fc9ff9-0442-4c12-b585-bf742f0ab7c4/094c4afc.png)
- DI(Dependency Injection)
> 依赖注入：组件之间依赖关系有容器在运行期决定，形象的说，即有容器动态的将某个依赖关系注入到组件中。
- Bean
> 就是由spring容器初始化、装配及管理的对象
- IOC容器代表
    1. BeanFactory接口 提供了IOC容器最基本功能
    2. ApplicationContext接口 扩展了BeanFactory，增加了更多企业级功能如与Spring AOP集成、国际化处理、事件传播及提供不同层次的context实现(如web应用的WebApplicationContext)，ApplicationContext接口继承ListableBeanFactory接口，ListableBeanFactory接口继承BeanFactory接口。
- IOC容器实现
    1. XmlBeanFactory:BeanFactory实现，提供基本的IOC容器功能，从classpath或文件系统获取资源
    2. ClassPathXmlApplicationContext:ApplicatonContext实现，从classpath获取配置文件
    3. FileSystemXmlApplicationContext:ApplicationContext实现，从文件系统获取配置文件
- IOC容器解析元数据
> IOC容器的Bean Reader读取并解析配置文件，根据定义生成BeanDefinition配置元数据对象，IOC容器根据BeanDefinition进行实例化、配置及组装Bean
- Bean的配置
> Bean在容器内部由BeanDefinition对象表示，主要包含一下信息：
    1. 全限定类名：定义Bean的实现类
    2. Bean行为定义：定义Bean在容器中的行为；包括作用域(单例、原型)、是否惰性初始化及生命周期等
    3. Bean创建方式定义
    4. Bean之间关系定义：即对其他bean的引用，也就是依赖关系定义；
- 实例化Bean
> IOC容器根据Bean定义里的配置元数据使用反射机制来创建Bean
实例化Bean的几种方式：
    1. 构造器实例化Bean
    2. 静态工厂方式实例化Bean，指定factory-method属性
    3. 实例工厂方式实例化Bean，不能指定class属性，必须使用factory-bean属性指定工厂bean，factory-method属性指定实例化bean的方法
- IOC容器的依赖含义
1. Bean依赖容器，指容器负责创建Bean并管理Bean的生命周期
2. 容器注入Bean的依赖资源，指容器负责注入Bean的依赖资源，依赖资源可以是Bean、外部文件、常量数据等
- DI(依赖注入)带来的好处
1. 动态替换bean依赖对象，程序更灵活
2. 更好实践面向接口编程，代码更清晰
3. 更好实践优先使用对象组合，而不是类继承
4. 增加Bean可复用性
5. 降低Bean之间耦合
6. 代码结构更清晰
- 延迟初始化Bean
> 延迟初始化也叫惰性初始化，指不提前初始化bean，而是只有在真正使用时才创建及初始化bean。实现方式：只需在<bean>标签上指定lazy-init属性为true
- depends-on
> 指定bean初始化及销毁时的顺序，使用depends-on属性指定的bean要先初始化完毕后才初始化当前bean，由于只有singleton的bean能被spring容器管理销毁，所以当指定的bean都是singleton时，使用depends-on属性指定的bean要在当前的bean之后销毁
- 自动装配 通过设置<bean>标签的属性autowire="byName"
1. default:<beans>标签中使用default-autowire属性指定
2. no
3. byName:只能用于setter注入
4. byType:只能用于setter注入，如果找到多个bean将优先注入<bean>标签primary属性为true的bean；对于集合类型找到多个bean，将注入所有匹配的候选者，对于其他类型遇到这种情况需要使用autowire-candidate属性为false来让指定的bean放弃作为自动装配的候选者，或者使用primary属性为true来指定某个bean为首选bean
5. constructor:只能用于构造器注入

*配置注入的数据会覆盖自动装配注入的数据*

- Bean的作用域
> 作用域在面向对象程序设计中是指对象或变量之间的可见范围。而在spring容器中是指创建的bean对象相对于其他bean对象的请求可见范围。
> spring提供singleton和prototype两种基本作用域，另外提供request、session、global session三种web作用域；
- singleton
> 在IOC容器中只会存在一个实例，而且其完整生命周期完全由spring容器管理；spring是注册表单例设计模式的实现，消除了编程式单例，而且对代码是非入侵式。
> 注册表方式：首先将需要单例的实例通过唯一键注册到注册表，然后通过键来获取单例，注册表实现了spring接口SingletonBeanRegistry，该接口定义了操作共享的单例对象；
- 资源访问
1. Resource接口提供对底层外部资源的一致性访问，该接口继承自InputStreamSource接口，并扩展了一些方法用于资源访问
2. isOpen:返回当前Resource代表的底层资源是否已经打开，如果返回true，则只能被读取一次然后关闭以避免资源泄漏；
3. ByteArrayResource代表byte[]数组资源
4. InputStreamResource代表java.io.InputStream字节流，只能读取一次字节流，即isOpen永远返回true
5. FileSystemResource代表java.io.File资源，getInputStream方法返回底层文件的字节流，isOpen永远返回false，表示可以多次读取底层文件的字节流
6. ClassPathResource代表classpath路径的资源
7. UrlResource代表Url资源
    http：http协议访问web资源如new UrlResource("http://地址")
    ftp：ftp协议访问资源如new UrlResource("ftp://地址")
    file：file协议访问本地文件系统资源如new URLResource("file:d:/test.txt")
8. ServletContextResource代表web应用资源
- 加载Resource
> ResourceLoader接口用于返回Resource对象；
```java
public interface ResourceLoader{
    Resource getResource(String location);
    ClassLoader getClassLoader();
}
```
> ResourceLoader加载资源时需要使用前缀来指定资源的类型，"classpath:path"表示返回ClasspathResource,"http://path"和"file:path"表示返回UrlResource资源，如果不加前缀则需要根据当前上下文来决定，DefaultResourceLoader默认实现可以加载classpath资源
- Resource通配符路径
> spring支持Ant风格路径匹配
> " ？"：匹配一个字符
> " * " ：匹配零个或多个字符
> classpath*：表示支持加载所有匹配的类路径Resource；Spring提供ResourcePatternResolver接口来加载多个Resource，该接口继承了ResourceLoader并添加了Resource[] getResources(String locationPattern)用来加载多个Resource
> spring提供了一个ResourcePatternResolver实现PathMatchingResourcePatternResolver,基于模式匹配，默认使用AntPathMatcher进行路径匹配，它除了支持ResourceLoader支持的前缀外，还额外支持classpath:用于加载所有匹配的类路径Resource，ResourceLoader不支持前缀classpath:
- SpEL表达式
![875da03d.png](:storage/04fc9ff9-0442-4c12-b585-bf742f0ab7c4/875da03d.png)
