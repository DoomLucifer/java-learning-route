[TOC]

# Spring

## Spring概述

### Spring是什么

Spring是一个开源的轻量级Java SE（Java 标准版本）/Java EE（Java 企业版本）开发应用框架，根本目的：简化Java开发。应用程序是由一组相互协作的对象组成。而在传统应用程序开发中，一个完整的应用是由一组相互协作的对象组成。所以开发一个应用除了要开发业务逻辑之外，最多的是关注如何使这些对象协作来完成所需功能，而且要低耦合、高内聚。业务逻辑开发是不可避免的，那如果有个框架出来帮我们来创建对象及管理这些对象之间的依赖关系，我们不需要通过工厂和生成器来创建及管理对象之间的依赖关系，这样我们是不是减少了许多工作，加速了开发，能节省出很多时间来干其他事。Spring框架刚出来时主要就是来完成这个功能。【IOC】

### Spring能做什么

1. Spring能帮我们根据配置文件创建及组装对象之间的依赖关系。
2. Spring面向切面编程能帮助我们无耦合的实现日志记录，性能统计，安全控制。
3. Spring能非常简单的帮我们管理数据库事务。
4. Spring还提供了与第三方数据访问框架（如Hibernate、JPA）无缝集成，而且自己也提供了一套JDBC访问模板，来方便数据库访问。
5. Spring还提供与第三方Web（如Struts、JSF）框架无缝集成，而且自己也提供了一套Spring MVC框架，来方便web层搭建。
6. Spring能方便的与Java EE（如Java Mail、任务调度）整合，与更多技术整合（比如缓存框架）。

### Spring的优点

1. 轻量级的容器：以集中的、自动化的方式进行应用程序对象创建和装配，负责对象创建和装配，管理对象生命周期，能组合成复杂的应用程序。Spring容器是非侵入式的（不需要依赖任何Spring特定类），而且完全采用POJOs进行开发，使应用程序更容易测试、更容易管理。而且核心JAR包非常小，Spring3.0.5不到1M，而且不需要依赖任何应用服务器，可以部署在任何环境（Java SE或Java EE）。
2. AOP：AOP是Aspect Oriented Programming的缩写，意思是面向切面编程，提供从另一个角度来考虑程序结构以完善面向对象编程（相对于OOP），即可以通过在编译期间、装载期间或运行期间实现在不修改源代码的情况下给程序动态添加功能的一种技术。通俗点说就是把可重用的功能提取出来，然后将这些通用功能在合适的时候织入到应用程序中；比如安全，日记记录，这些都是通用的功能，我们可以把它们提取出来，然后在程序执行的合适地方织入这些代码并执行它们，从而完成需要的功能并复用了这些功能。
3. 简单的数据库事物管理：在使用数据库的应用程序当中，自己管理数据库事务是一项很让人头疼的事，而且很容易出现错误，Spring支持可插入的事务管理支持，而且无需JEE环境支持，通过Spring管理事务可以把我们从事务管理中解放出来来专注业务逻辑。
4. JDBC抽象及ORM框架支持：Spring使JDBC更加容易使用；提供DAO（数据访问对象）支持，非常方便集成第三方ORM框架，比如Hibernate等；并且完全支持Spring事务和使用Spring提供的一致的异常体系。
5. 灵活的Web层支持：Spring本身提供一套非常强大的MVC框架，而且可以非常容易的与第三方MVC框架集成，比如Struts等。
6. 简化各种技术集成：提供对Java Mail、任务调度、JMX、JMS、JNDI、EJB、动态语言、远程访问、Web Service等的集成。

### Spring基础

Spring架构图

![Spring架构图](/Users/zhengguoqiang/Projects/IdeaProjects/DoomLucifer/java-learning-route/Framework/Spring/images/spring-framework.png)

核心容器：Core、Beans、Context、EL模块

- Core模块：封装了框架依赖的最底层部分，包括资源访问、类型转换及一些常用工具类
- Beans模块：提供了框架的基础部分，包括反转控制和依赖注入。其中Bean Factory是容器核心，本质是“工厂设计模式”的实现，而且无需编程实现“单例设计模式”，单例完全由容器控制，而且提倡面向接口编程，而非面向实现编程；所有应用程序对象及对象间关系由框架管理，从而真正把你从程序逻辑中把维护对象之间的依赖关系提取出来，所有这些依赖关系都由BeanFactory来维护。
- Context模块：以Core和Beans为基础，集成Beans模块功能并添加资源绑定、数据验证、国际化**、**Java EE支持、容器生命周期、事件传播等；核心接口是ApplicationContext。
- EL模块：提供强大的表达式语言支持，支持访问和修改属性值，方法调用，支持访问及修改数组、容器和索引器，命名变量，支持算数和逻辑运算，支持从Spring 容器获取Bean，它也支持列表投影、选择和一般的列表聚合等。

AOP、Aspects模块

- AOP模块：Spring AOP模块提供了符合 *AOP Alliance*规范的面向方面的编程（aspect-oriented programming）实现，提供比如日志记录、权限控制、性能统计等通用功能和业务逻辑分离的技术，并且能动态的把这些功能添加到需要的代码中；这样各专其职，降低业务逻辑和通用功能的耦合。
- Aspects模块：提供了对AspectJ的集成，AspectJ提供了比Spring ASP更强大的功能。

数据访问、集成模块：该模块包括了JDBC、ORM、OXM、JMS和事务管理

- 事物模块：该模块用于Spring管理事务，只要是Spring管理对象都能得到Spring管理事务的好处，无需在代码中进行事务控制了，而且支持编程和声明性的事物管理。
- JDBC模块：提供了一个JBDC的样例模板，使用这些模板能消除传统冗长的JDBC编码还有必须的事务控制，而且能享受到Spring管理事务的好处。
- ORM模块：提供与流行的“对象-关系”映射框架的无缝集成，包括Hibernate、JPA、Ibatiss等。而且可以使用Spring事务管理，无需额外控制事务。
- OXM模块：提供了一个对Object/XML映射实现，将java对象映射成XML数据，或者将XML数据映射成java对象，Object/XML映射实现包括JAXB、Castor、XMLBeans和XStream。
- JMS模块：用于JMS(Java Messaging Service)，提供一套 “消息生产者、消息消费者”模板用于更加简单的使用JMS，JMS用于用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。

Web、Remoting模块：Web/Remoting模块包含了Web、Web-Servlet、Web-Struts、Web-Porlet模块。

- Web模块：提供了基础的web功能。例如多文件上传、集成IoC容器、远程过程访问（RMI、Hessian、Burlap）以及Web Service支持，并提供一个RestTemplate类来提供方便的Restful services访问。
- Web-Servlet模块：提供了一个Spring MVC Web框架实现。Spring MVC框架提供了基于注解的请求资源注入、更简单的数据绑定、数据验证等及一套非常易用的JSP标签，完全无缝与Spring其他技术协作。
- Web-Structs模块：提供了与Struts无缝集成，Struts1.x 和Struts2.x都支持

Test模块：Spring支持Junit和TestNG测试框架，而且还额外提供了一些基于Spring的测试功能，比如在测试Web框架时，模拟Http请求的功能。

## IOC

### IOC基础

#### IOC是什么

IOC：Inversion of Control 即“控制反转”。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

- 控制：传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；谁控制谁？当然是IoC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）。
- 反转：有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。

#### IOC能做什么

IoC不是一种技术，只是一种思想，一个重要的面向对象编程的法则，它能指导我们如何设计出松耦合、更优良的程序。传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。

其实IoC对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC容器来创建并注入它所需要的资源了。

IoC很好的体现了面向对象设计法则之一—— 好莱坞法则：“别找我们，我们找你”；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找。

#### IOC和DI

DI—Dependency Injection，即“依赖注入”：是组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

**谁依赖谁** ：应用程序依赖于IOC容器

**为什么需要依赖** ：应用程序需要IOC容器来提供对象需要的外部资源

**谁注入谁** ：IOC容器注入应用程序某个对象依赖的外部资源

**注入了什么** ：注入某个对象所需要的外部资源（包括对象、资源、常量数据等）

IoC和DI的关系：其实它们是同一个概念的不同角度描述，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC 而言，**“依赖注入”** 明确描述了“被注入对象依赖IoC容器配置依赖对象”。

### IOC容器

#### IOC容器概念

IoC容器就是具有依赖注入功能的容器，IoC容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。应用程序无需直接在代码中new相关的对象，应用程序由IoC容器进行组装。在Spring中BeanFactory是IoC容器的实际代表者。

Spring IoC容器如何知道哪些是它管理的对象呢？这就需要配置文件，Spring IoC容器通过读取配置文件中的配置元数据，通过元数据对应用中的各个对象进行实例化及装配。一般使用基于xml配置文件进行配置元数据，而且Spring与配置文件完全解耦的，可以使用其他任何可能的方式进行配置元数据，比如注解、基于java文件的、基于属性文件的配置都可以。

#### Bean

由IoC容器管理的那些组成你应用程序的对象我们就叫它Bean， Bean就是由Spring*容器*初始化、装配及管理的对象，除此之外，bean就与应用程序中的其他对象没有什么区别了。那IoC怎样确定如何实例化Bean、管理Bean之间的依赖关系以及管理Bean呢？这就需要配置元数据，在Spring中由BeanDefinition代表。

#### 详解IOC容器

在Spring Ioc容器的代表就是org.springframework.beans包中的BeanFactory接口，BeanFactory接口提供了IoC容器最基本功能；而org.springframework.context包下的ApplicationContext接口扩展了BeanFactory，还提供了与Spring AOP集成、国际化处理、事件传播及提供不同层次的context实现 (如针对web应用的WebApplicationContext)。简单说， BeanFactory提供了IoC容器最基本功能，而 ApplicationContext 则增加了更多支持企业级功能支持。ApplicationContext完全继承BeanFactory，因而BeanFactory所具有的语义也适用于ApplicationContext。

容器实现：

- XMLBeanFactory：BeanFactory实现，提供基本的IoC容器功能，可以从classpath或文件系统等获取资源；

```java
//1.
File file = new File("fileSystemConfig.xml");
Resource resource = new FileSystemResource(file);
//读取资源配置实例化一个IOC容器
BeanFactory beanFactory = new XmlBeanFactory(resource);
//2.
Resource resource = new ClassPathResource("classpath.xml");
BeanFactory beanFactory = new XmlBeanFactory(resource);
```

- ClassPathXmlApplicationContext：ApplicationContext实现，从classpath获取配置文件；

```java
BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath.xml");
```

- FileSystemXmlApplicationContext：ApplicationContext实现，从文件系统获取配置文件。

```java
BeanFactory beanFactory = new FileSystemXmlApplicationContext("fileSystemConfig.xml");
```

ApplicationContext接口获取Bean方法简介：

- Object getBean(String name)：根据名称返回一个Bean，客户端需要自己进行类型转换；
- T getBean(String name,Class<T> requiredType)：根据名称和指定的类型返回一个Bean，客户端无需自己进行类型转换，如果类型转换失败，容器抛出异常；
- T getBean(Class<T> requiredType)：根据指定的类型返回一个Bean，客户端无需自己进行类型转换，如果没有或有多于一个Bean存在容器将抛出异常；
- Map<String,T> getBeansOfType(Class<T> type)：根据指定的类型返回一个键值为名字和值为Bean对象的 Map，如果没有Bean对象存在则返回空的Map。

#### IOC容器如何工作

1. 准备配置文件：在配置文件中声明Bean定义也就是为Bean配置元数据。
2. 由IOC容器进行解析元数据：IoC容器的Bean Reader读取并解析配置文件，根据定义生成BeanDefinition配置元数据对象，IoC容器根据BeanDefinition进行实例化、配置及组装Bean。
3. 实例化IOC容器：由客户端实例化容器，获取需要的Bean。

![img](/Users/zhengguoqiang/Projects/IdeaProjects/DoomLucifer/java-learning-route/Framework/Spring/images/spring-ioc-xml.png)

### IOC配置

#### XML配置

```xml
<beans>  
    <import resource=”resource1.xml”/>  
    <bean id=”bean1”class=””></bean>  
    <bean id=”bean2”class=””></bean>  
	<bean name=”bean2”class=””></bean>  
    <alias alias="bean3" name="bean2"/>  
    <import resource=”resource2.xml”/>  
</beans>  
```

1. <bean>标签用来进行Bean定义
2. alias用于定义Bean别名
3. import用于导入其他配置文件的Bean定义，这是为了加载多个配置文件，当然也可以把这些配置文件构造为一个数组（new String[] {“config1.xml”, config2.xml}）传给ApplicationContext实现进行加载多个配置文件，那一个更适合由用户决定；这两种方式都是通过调用Bean Definition Reader 读取Bean定义，内部实现没有任何区别。<import>标签可以放在<beans>下的任何位置，没有顺序关系。

#### Bean的配置

Spring IoC容器目的就是管理Bean，这些Bean将根据配置文件中的Bean定义进行创建，而Bean定义在容器内部由BeanDefinition对象表示，该定义主要包含以下信息：

1. 全限定类名：用于定义Bean的实现类
2. Bean行为定义：定义了Bean在容器中的行为；包括作用域（单例、原型创建）、是否惰性初始化及生命周期等
3. Bean创建方式定义：说明是通过构造器还是工厂方法创建Bean
4. Bean之间关系定义：即对其他bean的引用，也就是依赖关系定义，这些引用bean也可以称之为同事bean 或依赖bean，也就是依赖注入。

Bean定义只有“全限定类名”在当使用构造器或静态工厂方法进行实例化bean时是必须的，其他都是可选的定义。难道Spring只能通过配置方式来创建Bean吗？回答当然不是，某些SingletonBeanRegistry接口实现类实现也允许将那些非BeanFactory创建的、已有的用户对象注册到容器中，这些对象必须是共享的，比如使用DefaultListableBeanFactory 的registerSingleton() 方法。不过建议采用元数据定义。

#### Bean的命名

每个Bean可以有一个或多个id（或称之为标识符或名字），在这里我们把第一个id称为“标识符”，其余id叫做“别名”；这些id在IoC容器中必须唯一。如何为Bean指定id呢，有以下几种方式；

1. 不指定id，只配置必须的全限定类名，由IoC容器为其生成一个标识，客户端必须通过接口“T getBean(Class<T> requiredType)”获取Bean；
2. 指定id，必须在IOC容器中唯一
3. 指定name，这样name就是标识符，必须在IOC容器中唯一
4. 指定id和name，id就是标识符，而name就是别名，必须在Ioc容器中唯一；

```xml
<bean id=”bean1”name=”alias1”  
class=” cn.javass.spring.chapter2.helloworld.HelloImpl”/>  
<!-- 如果id和name一样，IoC容器能检测到，并消除冲突 -->  
<bean id="bean3" name="bean3" class="cn.javass.spring.chapter2.helloworld.HelloImpl"/>  
```

```java
@Test  
public void test4() {  
BeanFactory beanFactory =  
new ClassPathXmlApplicationContext("chapter2/namingbean4.xml");  
    //根据id获取bean  
    HelloApi bean1 = beanFactory.getBean("bean1", HelloApi.class);  
    bean1.sayHello();  
    //根据别名获取bean  
    HelloApi bean2 = beanFactory.getBean("alias1", HelloApi.class);  
    bean2.sayHello();  
    //根据id获取bean  
    HelloApi bean3 = beanFactory.getBean("bean3", HelloApi.class);  
    bean3.sayHello();  
    String[] bean3Alias = beanFactory.getAliases("bean3");  
    //因此别名不能和id一样，如果一样则由IoC容器负责消除冲突  
    Assert.assertEquals(0, bean3Alias.length);  
}
```

5. 指定多个name，多个name用“，”、“；”、“ ”分割，第一个被用作标识符，其他的（alias1、alias2、alias3）是别名，所有标识符也必须在Ioc容器中唯一；

```xml
<bean name=” bean1;alias11,alias12;alias13 alias14”  
      class=” cn.javass.spring.chapter2.helloworld.HelloImpl”/>     
<!-- 当指定id时，name指定的标识符全部为别名 -->  
<bean id="bean2" name="alias21;alias22"  
class="cn.javass.spring.chapter2.helloworld.HelloImpl"/>  
```

6. 使用<alias>标签指定别名，别名也必须在IoC容器中唯一

```xml
<bean name="bean" class="cn.javass.spring.chapter2.helloworld.HelloImpl"/>  
<alias alias="alias1" name="bean"/>  
<alias alias="alias2" name="bean"/>       
```

#### 实例化Bean

Spring IoC容器如何实例化Bean呢？传统应用程序可以通过new和反射方式进行实例化Bean。而Spring IoC容器则需要根据Bean定义里的配置元数据使用反射机制来创建Bean。在Spring IoC容器中根据Bean定义创建Bean主要有以下几种方式：

1. 使用构造器实例化Bean：这是最简单的方式，Spring IoC容器即能使用默认空构造器也能使用有参数构造器两种方式创建Bean，如以下方式指定要创建的Bean类型：

```xml
<!-- 使用空构造器进行定义，使用此种方式，class属性指定的类必须有空构造器 -->
<bean name="bean1" class="cn.javass.spring.chapter2.HelloImpl2"/>
```

```xml
<!-- 使用有参数构造器进行定义，使用此中方式，可以使用< constructor-arg >标签指定构造器参数值，其中index表示位置，value表示常量值，也可以指定引用，指定引用使用ref来引用另一个Bean定义 -->
<bean name="bean2" class="cn.javass.spring.chapter2.HelloImpl2">  
<!-- 指定构造器参数 -->  
     <constructor-arg index="0" value="Hello Spring!"/>  
</bean>  
```

2. 使用静态工厂方式实例化Bean，使用这种方式除了指定必须的class属性，还要指定factory-method属性来指定实例化Bean的方法，而且使用静态工厂方法也允许指定方法参数，spring IoC容器将调用此属性指定的方法来获取Bean，配置如下所示：

```java
//静态工厂类HelloApiStaticFactory
public class HelloApiStaticFactory {  
    //工厂方法  
       public static HelloApi newInstance(String message) {  
              //返回需要的Bean实例  
           return new HelloImpl2(message);  
}  
} 
```

```xml
<!-- 使用静态工厂方法 -->  
<bean id="bean3" class="cn.javass.spring.chapter2.HelloApiStaticFactory" factory-method="newInstance">  
     <constructor-arg index="0" value="Hello Spring!"/>  
</bean>  
```

3. 使用实例工厂方法实例化Bean，使用这种方式不能指定class属性，此时必须使用factory-bean属性来指定工厂Bean，factory-method属性指定实例化Bean的方法，而且使用实例工厂方法允许指定方法参数，方式和使用构造器方式一样，配置如下：

```java
package cn.javass.spring.chapter2;  
public class HelloApiInstanceFactory {  
public HelloApi newInstance(String message) {  
          return new HelloImpl2(message);  
   }  
}
```

```xml
<!—1、定义实例工厂Bean -->  
<bean id="beanInstanceFactory"  
class="cn.javass.spring.chapter2.HelloApiInstanceFactory"/>  
<!—2、使用实例工厂Bean创建Bean -->  
<bean id="bean4"  
factory-bean="beanInstanceFactory"  
     factory-method="newInstance">  
 <constructor-arg index="0" value="Hello Spring!"></constructor-arg>  
</bean>  
```

## DI

### DI配置

#### 依赖和依赖注入

**泛化** ：表示类与类之间的继承关系、接口与接口之间的继承关系；

**实现** ：表示类对接口的实现；

**依赖** ：当类与类之间有使用关系时就属于依赖关系，不同于关联关系，依赖不具有“拥有关系”，而是一种“相识关系”，只在某个特定地方（比如某个方法体内）才有关系。

**关联** ：表示类与类或类与接口之间的依赖关系，表现为“拥有关系”；具体到代码可以用实例变量来表示；

**聚合** ：属于是关联的特殊情况，体现部分-整体关系，是一种弱拥有关系；整体和部分可以有不一样的生命周期；是一种弱关联；

**组合** ：属于是关联的特殊情况，也体现了体现部分-整体关系，是一种强“拥有关系”；整体与部分有相同的生命周期，是一种强关联；

Spring IoC容器的依赖有两层含义：**Bean依赖容器**和**容器注入Bean的依赖资源**：

**Bean依赖容器** ：也就是说Bean要依赖于容器，这里的依赖是指容器负责创建Bean并管理Bean的生命周期，正是由于由容器来控制创建Bean并注入依赖，也就是控制权被反转了，这也正是IoC名字的由来，**此处的有依赖是指Bean和容器之间的依赖关系**。

**容器注入Bean的依赖资源** ：容器负责注入Bean的依赖资源，依赖资源可以是Bean、外部文件、常量数据等，在Java中都反映为对象，并且由容器负责组装Bean之间的依赖关系，**此处的依赖是指Bean之间的依赖关系，可以认为是传统类与类之间的“关联”、“聚合”、“组合”关系**。

依赖注入的好处：

1. **动态替换Bean依赖对象，程序更灵活：**替换Bean依赖对象，无需修改源文件：应用依赖注入后，由于可以采用配置文件方式实现，从而能随时动态的替换Bean的依赖对象，无需修改java源文件；
2. **更好的实践面向接口编程，代码更清晰：**在Bean中只需指定依赖对象的接口，接口定义依赖对象完成的功能，通过容器注入依赖实现；
3. **更好实践优先使用对象组合，而不是类继承：** 因为IoC容器采用注入依赖，也就是组合对象，从而更好的实践对象组合。

> 采用对象组合，Bean的功能可能由几个依赖Bean的功能组合而成，其Bean本身可能只提供少许功能或根本无任何功能，全部委托给依赖Bean，对象组合具有动态性，能更方便的替换掉依赖Bean，从而改变Bean功能；
>
> 而如果采用类继承，Bean没有依赖Bean，而是采用继承方式添加新功能，，而且功能是在编译时就确定了，不具有动态性，而且采用类继承导致Bean与子Bean之间高度耦合，难以复用。

4. **增加Bean可复用性：** 依赖于对象组合，Bean更可复用且复用更简单
5. **降低Bean之间耦合：** 由于我们完全采用面向接口编程，在代码中没有直接引用Bean依赖实现，全部引用接口，而且不会出现显示的创建依赖对象代码，而且这些依赖是由容器来注入，很容易替换依赖实现类，从而降低Bean与依赖之间耦合；
6. **代码结构更清晰**

#### 依赖注入实现方式

1. 构造器注入：就是容器实例化Bean时注入那些依赖，通过在在Bean定义中指定构造器参数进行注入依赖，包括实例工厂方法参数注入依赖，但静态工厂方法参数不允许注入依赖；
2. setter方法注入：通过setter方法进行依赖注入

详细讲述请看原文地址：http://jinnianshilongnian.iteye.com/blog/1415277

### 循环依赖

- 构造器循环依赖

表示通过构造器注入构成的循环依赖，此依赖是无法解决的，只能抛出BeanCurrentlyInCreationException异常表示循环依赖。

如在创建CircleA类时，构造器需要CircleB类，那将去创建CircleB，在创建CircleB类时又发现需要CircleC类，则又去创建CircleC，最终在创建CircleC时发现又需要CircleA；从而形成一个环，没办法创建。

Spring容器将每一个正在创建的Bean 标识符放在一个“当前创建Bean池”中，Bean标识符在创建过程中将一直保持在这个池中，因此如果在创建Bean过程中发现自己已经在“当前创建Bean池”里时将抛出BeanCurrentlyInCreationException异常表示循环依赖；而对于创建完毕的Bean将从“当前创建Bean池”中清除掉。

- setter循环依赖

对于setter注入造成的依赖是通过Spring容器提前暴露刚完成构造器注入但未完成其他步骤（如setter注入）的Bean来完成的，而且只能解决单例作用域的Bean循环依赖。

通过提前暴露一个单例工厂方法，从而使其他Bean能引用到该Bean。

```java
addSingletonFactory(beanName, new ObjectFactory() {  
    public Object getObject() throws BeansException {  
        return getEarlyBeanReference(beanName, mbd, bean);  
    }  
});  
```

具体步骤：

1. Spring容器创建单例“circleA” Bean，首先根据无参构造器创建Bean，并暴露一个“ObjectFactory ”用于返回一个提前暴露一个创建中的Bean，并将“circleA” 标识符放到“当前创建Bean池”；然后进行setter注入“circleB”；
2. Spring容器创建单例“circleB” Bean，首先根据无参构造器创建Bean，并暴露一个“ObjectFactory”用于返回一个提前暴露一个创建中的Bean，并将“circleB” 标识符放到“当前创建Bean池”，然后进行setter注入“circleC”；
3. Spring容器创建单例“circleC” Bean，首先根据无参构造器创建Bean，并暴露一个“ObjectFactory ”用于返回一个提前暴露一个创建中的Bean，并将“circleC” 标识符放到“当前创建Bean池”，然后进行setter注入“circleA”；进行注入“circleA”时由于提前暴露了“ObjectFactory”工厂从而使用它返回提前暴露一个创建中的Bean；
4. 最后在依赖注入“circleB”和“circleA”，完成setter注入。

 对于“prototype”作用域Bean，Spring容器无法完成依赖注入，因为“prototype”作用域的Bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。

原文地址：http://jinnianshilongnian.iteye.com/blog/1415278

### DI其他相关

#### 延迟初始化Bean

​	延迟初始化也叫做惰性初始化，指不提前初始化Bean，而是只有在真正使用时才创建及初始化Bean。

​       	配置方式很简单只需在<bean>标签上指定 “lazy-init” 属性值为“true”即可延迟初始化Bean。

​       	Spring容器会在创建容器时提前初始化“singleton”作用域的Bean，“singleton”就是单例的意思即整个容器每个Bean只有一个实例，后边会详细介绍。Spring容器预先初始化Bean通常能帮助我们提前发现配置错误，所以如果没有什么情况建议开启，除非有某个Bean可能需要加载很大资源，而且很可能在整个应用程序生命周期中很可能使用不到，可以设置为延迟初始化。

​       延迟初始化的Bean通常会在第一次使用时被初始化；或者在被非延迟初始化Bean作为依赖对象注入时在会随着初始化该Bean时被初始化，因为在这时使用了延迟初始化Bean。

​       容器管理初始化Bean消除了编程实现延迟初始化，完全由容器控制，只需在需要延迟初始化的Bean定义上配置即可，比编程方式更简单，而且是无侵入代码的。

```xml
<bean id="helloApi"  
class="cn.javass.spring.chapter2.helloworld.HelloImpl"  
lazy-init="true"/>  
```

#### depends-on

depends-on是指指定Bean初始化及销毁时的顺序，使用depends-on属性指定的Bean要先初始化完毕后才初始化当前Bean，由于只有“singleton”Bean能被Spring管理销毁，所以当指定的Bean都是“singleton”时，使用depends-on属性指定的Bean要在指定的Bean之后销毁。

```xml
<bean id="helloApi" class="cn.javass.spring.chapter2.helloworld.HelloImpl"/>  
<bean id="decorator"  
    class="cn.javass.spring.chapter3.bean.HelloApiDecorator"  
    depends-on="helloApi">  
    <property name="helloApi"><ref bean="helloApi"/></property>  
</bean>  
```

#### 自动装备

#### 依赖检查

#### 方法注入

1. 查找方法注入

又称为Lookup方法注入，用于注入方法返回结果，也就是说能通过配置方式替换方法返回结果。使用<lookup-method name="方法名" bean="bean名字"/>配置；其中name属性指定方法名，bean属性指定方法需返回的Bean。

方法注入主要用于处理“singleton”作用域的Bean需要其他作用域的Bean时，采用Spring查找方法注入方式无需修改任何代码即能获取需要的其他作用域的Bean。

2. 替换方法注入

也叫“MethodReplacer”注入，和查找注入方法不一样的是，他主要用来替换方法体。

原文地址：http://jinnianshilongnian.iteye.com/blog/1415461

### Bean的作用域

什么是作用域呢？即“scope”，在面向对象程序设计中一般指对象或变量之间的可见范围。而在Spring容器中是指其创建的Bean对象相对于其他Bean对象的请求可见范围。

Spring提供“singleton”和“prototype”两种基本作用域，另外提供“request”、“session”、“global session”三种web作用域；Spring还允许用户定制自己的作用域。

#### 基本作用域

1. singleton：指“singleton”作用域的Bean只会在每个Spring IoC容器中存在一个实例，而且其完整生命周期完全由Spring容器管理。对于所有获取该Bean的操作Spring容器将只返回同一个Bean。

> GoF单例设计模式指“**保证一个类仅有一个实例，并提供一个访问它的全局访问点**”，介绍了两种实现：通过在类上定义静态属性保持该实例和通过注册表方式。

- 通过在类上定义静态属性保持该实例：一般指一个Java虚拟机 ClassLoader装载的类只有一个实例，一般通过类静态属性保持该实例，这样就造成需要单例的类都需要按照单例设计模式进行编码；Spring没采用这种方式，因为该方式属于侵入式设计；代码样例如下：

```java
public class Singleton{
    //1.私有化构造器
    private Singleton(){}
    //2.单例缓存者，惰性初始化，第一次使用时初始化
    private static class InstanceHolder {
        private static final Singleton instance = new Singleton();
    }
    //3.提供全局访问
    public static Singleton getInstance(){
        return InstanceHolder.instance;
    }
}
```

- 通过注册表方式：首先将需要单例的实例通过唯一键注册到注册表，然后通过键来获取单例，让我们直接看实现吧，注意本注册表实现了Spring接口“SingletonBeanRegistry”，该接口定义了操作共享的单例对象，Spring容器实现将实现此接口；所以共享单例对象通过“registerSingleton”方法注册，通过“getSingleton”方法获取，消除了编程方式单例，注意在实现中不考虑并发：

```java
public class SingletonBeanRegister implements SingletonBeanRegistry{
    //单例Bean缓存池
    private final Map<String,Object> beans = new HashMap<String,Object>();
    public boolean containsSingleton(String beanName){
        return beans.containsKey(beanName);
    }
    public Object getSingleton(String beanName){
        return beans.get(beanName);
    }
    public int getSingletonCount(){
        return beans.size();
    }
    @Override  
    public String[] getSingletonNames() {  
        return BEANS.keySet().toArray(new String[0]);  
    }  
    @Override  
    public void registerSingleton(String beanName, Object bean) {  
        if(BEANS.containsKey(beanName)) {  
            throw new RuntimeException("[" + beanName + "] 已存在");  
        }  
        BEANS.put(beanName, bean);  
    }
}
```

Spring是注册表单例设计模式的实现，消除了编程式单例，而且对代码是非入侵式。

接下来让我们看看在Spring中如何配置单例Bean吧，在Spring容器中如果没指定作用域默认就是“singleton”，配置方式通过scope属性配置，具体配置如下：

```xml
<bean class="com.doom.lucifer.Test" scope="singleton"/>
```

 Spring管理单例对象在Spring容器中存储如下所示，Spring不仅会缓存单例对象，Bean定义也是会缓存的，对于惰性初始化的对象是在首次使用时根据Bean定义创建并存放于单例缓存池。

![img](/Users/zhengguoqiang/Projects/IdeaProjects/DoomLucifer/java-learning-route/Framework/Spring/images/spring-singleton.png)

2. Prototype:即原型，指每次向Spring容器请求获取Bean都返回一个全新的Bean，相对于“singleton”来说就是不缓存Bean，每次都是一个根据Bean定义创建的全新Bean。

> GoF原型设计模式，指用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

Spring中的原型和GoF中介绍的原型含义是不一样的：

         GoF通过用原型实例指定创建对象的种类，而Spring容器用Bean定义指定创建对象的种类；

         GoF通过拷贝这些原型创建新的对象，而Spring容器根据Bean定义创建新对象。

其相同地方都是根据某些东西创建新东西，而且GoF原型必须显示实现克隆操作，属于侵入式，而Spring容器只需配置即可，属于非侵入式。

Bean定义注册表：

```java

package cn.javass.spring.chapter3;  
import java.util.HashMap;  
import java.util.Map;  
public class BeanDifinitionRegister {  
    //bean定义缓存，此处不考虑并发问题  
	private final Map<String, BeanDefinition> DEFINITIONS =  
 		new HashMap<String, BeanDefinition>();  
    public void registerBeanDefinition(String beanName, BeanDefinition bd) {  
        //1.本实现不允许覆盖Bean定义  
        if(DEFINITIONS.containsKey(bd.getId())) {  
            throw new RuntimeException("已存在Bean定义，此实现不允许覆盖");  
        }  
        //2.将Bean定义放入Bean定义缓存池  
        DEFINITIONS.put(bd.getId(), bd);  
    }  
    public BeanDefinition getBeanDefinition(String beanName) {  
        return DEFINITIONS.get(beanName);  
    }  
	public boolean containsBeanDefinition(String beanName) {        
 		return DEFINITIONS.containsKey(beanName);  
    }  
}  
```

定义BeanFactory：

```java

package cn.javass.spring.chapter3;  
import org.springframework.beans.factory.config.SingletonBeanRegistry;  
public class DefaultBeanFactory {  
    //Bean定义注册表  
    private BeanDifinitionRegister DEFINITIONS = new BeanDifinitionRegister();  
   
    //单例注册表  
    private final SingletonBeanRegistry SINGLETONS = new SingletonBeanRegister();  
     
    public Object getBean(String beanName) {  
        //1.验证Bean定义是否存在  
        if(!DEFINITIONS.containsBeanDefinition(beanName)) {  
            throw new RuntimeException("不存在[" + beanName + "]Bean定义");  
        }  
        //2.获取Bean定义  
        BeanDefinition bd = DEFINITIONS.getBeanDefinition(beanName);  
        //3.是否该Bean定义是单例作用域  
        if(bd.getScope() == BeanDefinition.SCOPE_SINGLETON) {  
            //3.1 如果单例注册表包含Bean，则直接返回该Bean  
            if(SINGLETONS.containsSingleton(beanName)) {  
                return SINGLETONS.getSingleton(beanName);  
            }  
            //3.2单例注册表不包含该Bean，  
            //则创建并注册到单例注册表，从而缓存  
            SINGLETONS.registerSingleton(beanName, createBean(bd));  
            return SINGLETONS.getSingleton(beanName);  
        }  
        //4.如果是原型Bean定义,则直接返回根据Bean定义创建的新Bean，  
		//每次都是新的，无缓存  
        if(bd.getScope() == BeanDefinition.SCOPE_PROTOTYPE) {  
            return createBean(bd);  
        }  
        //5.其他情况错误的Bean定义  
        throw new RuntimeException("错误的Bean定义");  
    } 
```

```java
public void registerBeanDefinition(BeanDefinition bd) {  
     DEFINITIONS.registerBeanDefinition(bd.getId(), bd);  
 }  
  
 private Object createBean(BeanDefinition bd) {  
     //根据Bean定义创建Bean  
     try {  
         Class clazz = Class.forName(bd.getClazz());  
         //通过反射使用无参数构造器创建Bean  
         return clazz.getConstructor().newInstance();  
     } catch (ClassNotFoundException e) {  
         throw new RuntimeException("没有找到Bean[" + bd.getId() + "]类");  
     } catch (Exception e) {  
         throw new RuntimeException("创建Bean[" + bd.getId() + "]失败");  
     }  
 } 
```

Spring管理原型对象在Spring容器中存储如下图所示，Spring不会缓存原型对象，而是根据Bean定义每次请求返回一个全新的Bean：

![img](/Users/zhengguoqiang/Projects/IdeaProjects/DoomLucifer/java-learning-route/Framework/Spring/images/spring-prototype.png)

#### Web应用中的作用域

- request作用域：**表示每个请求需要容器创建一个全新Bean**。比如提交表单的数据必须是对每次请求新建一个Bean来保持这些表单数据，请求结束释放这些数据。
- session作用域：**表示每个会话需要容器创建一个全新Bean**。比如对于每个用户一般会有一个会话，该用户的用户信息需要存储到会话中，此时可以将该Bean配置为web作用域。
- globalSession作用域：类似于session作用域，只是其用于portlet环境的web应用。如果在非portlet环境将视为session作用域。

#### 自定义作用域

scope接口：

```java
package org.springframework.beans.factory.config;  
import org.springframework.beans.factory.ObjectFactory;  
public interface Scope {  
       Object get(String name, ObjectFactory<?> objectFactory);  
       Object remove(String name);  
       void registerDestructionCallback(String name, Runnable callback);  
       Object resolveContextualObject(String key);  
       String getConversationId();  
} 
```

1）**Object get(String name, ObjectFactory<?> objectFactory)**：用于从作用域中获取Bean，其中参数objectFactory是当在当前作用域没找到合适Bean时使用它创建一个新的Bean； 

2）**void registerDestructionCallback(String name, Runnable callback)**：用于注册销毁回调，如果想要销毁相应的对象则由Spring容器注册相应的销毁回调，而由自定义作用域选择是不是要销毁相应的对象；

3）**Object resolveContextualObject(String key)**：用于解析相应的上下文数据，比如request作用域将返回request中的属性。 

4）**String getConversationId()**：作用域的会话标识，比如session作用域将是sessionId。

原文地址：http://jinnianshilongnian.iteye.com/blog/1415463

## 资源 Resource

### Resource基础

#### Resource接口

Spring的Resource接口代表底层外部资源，提供了对底层外部资源的一致性访问接口。

```java
public interface InputStreamSource{
    InputStream getInputStream() throws IOException;
}
```

```java
public interface Resource extends InputStreamSource {
    boolean exists();
    boolean isReadable();
    boolean isOpen();
    URL getURL() throws IOException;  
    URI getURI() throws IOException;  
    File getFile() throws IOException;  
    long contentLength() throws IOException;  
    long lastModified() throws IOException;  
    Resource createRelative(String relativePath) throws IOException;  
    String getFilename();  
    String getDescription(); 
}
```

1. InputStreamSource接口解析：

**getInputStream**：每次调用都将返回一个新鲜的资源对应的java.io. InputStream字节流，调用者在使用完毕后必须关闭该资源。

2. Resource接口继承InputStreamSource接口，并提供一些便利方法：

   **exists**：返回当前Resource代表的底层资源是否存在，true表示存在。

​       **isReadable**：返回当前Resource代表的底层资源是否可读，true表示可读。

​       **isOpen**：返回当前Resource代表的底层资源是否已经打开，如果返回true，则只能被读取一次然后关闭以避免资源泄露；常见的Resource实现一般返回false。

​       **getURL**：如果当前Resource代表的底层资源能由java.util.URL代表，则返回该URL，否则抛出IOException。

​       **getURI**：如果当前Resource代表的底层资源能由java.util.URI代表，则返回该URI，否则抛出IOException。

​       **getFile**：如果当前Resource代表的底层资源能由java.io.File代表，则返回该File，否则抛出IOException。

​       **contentLength**：返回当前Resource代表的底层文件资源的长度，一般是值代表的文件资源的长度。

​       **lastModified**：返回当前Resource代表的底层资源的最后修改时间。

​       **createRelative**：用于创建相对于当前Resource代表的底层资源的资源，比如当前Resource代表文件资源“d:/test/”则createRelative（“test.txt”）将返回表文件资源“d:/test/test.txt”Resource资源。

​       **getFilename**：返回当前Resource代表的底层文件资源的文件路径，比如File资源“file://d:/test.txt”将返回“d:/test.txt”，而URL资源http://www.javass.cn将返回“”，因为只返回文件路径。

​       **getDescription**：返回当前Resource代表的底层资源的描述符，通常就是资源的全路径（实际文件名或实际URL地址）。

Resource接口提供了足够的抽象，足够满足我们日常使用。而且提供了很多内置Resource实现：ByteArrayResource、InputStreamResource 、FileSystemResource 、UrlResource 、ClassPathResource、ServletContextResource、VfsResource等。

### 访问Resource

#### ResourceLoader接口

ResourceLoader接口用于返回Resource对象；其实现可以看作是一个生产Resource的工厂类

```java
public interface ResourceLoader{
    Resource getResource(String location);
    ClassLoader getClassLoader();
}
```

getResource接口用于根据提供的location参数返回相应的Resource对象；而getClassLoader则返回加载这些Resource的ClassLoader。

Spring提供了一个适用于所有环境的DefaultResourceLoader实现，可以返回ClassPathResource、UrlResource；还提供一个用于web环境的ServletContextResourceLoader，它继承了DefaultResourceLoader的所有功能，又额外提供了获取ServletContextResource的支持。

ResourceLoader在进行加载资源时需要使用前缀来指定需要加载：“classpath:path”表示返回ClasspathResource，“http://path”和“file:path”表示返回UrlResource资源，如果不加前缀则需要根据当前上下文来决定，DefaultResourceLoader默认实现可以加载classpath资源

```java
@Test  
public void testResourceLoad() {  
    ResourceLoader loader = new DefaultResourceLoader();  
    Resource resource = 			   loader.getResource("classpath:cn/javass/spring/chapter4/test1.txt");  
    //验证返回的是ClassPathResource  
    Assert.assertEquals(ClassPathResource.class, resource.getClass());  
    Resource resource2 = loader.getResource("file:cn/javass/spring/chapter4/test1.txt");  
    //验证返回的是ClassPathResource  
    Assert.assertEquals(UrlResource.class, resource2.getClass());  
    Resource resource3 = loader.getResource("cn/javass/spring/chapter4/test1.txt");  
    //验证返默认可以加载ClasspathResource  
    Assert.assertTrue(resource3 instanceof ClassPathResource);  
}  
```

 对于目前所有ApplicationContext都实现了ResourceLoader，因此可以使用其来加载资源。

**ClassPathXmlApplicationContext**：不指定前缀将返回默认的ClassPathResource资源，否则将根据前缀来加载资源；

**FileSystemXmlApplicationContext**：不指定前缀将返回FileSystemResource，否则将根据前缀来加载资源；

**WebApplicationContext**：不指定前缀将返回ServletContextResource，否则将根据前缀来加载资源；

**其他：**不指定前缀根据当前上下文返回Resource实现，否则将根据前缀来加载资源。

#### ResourceLoaderAware接口

ResourceLoaderAware是一个标记接口，用于通过ApplicationContext上下文注入ResourceLoader。

```java
public interface ResourceLoaderAware{
    void setResourceLoader(ResourceLoader resourceLoader);
}
```

注：ApplicationContext就是个ResourceLoader

#### 注入Resource

Spring提供了一个PropertyEditor “ResourceEditor”用于在注入的字符串和Resource之间进行转换。因此可以使用注入方式注入Resource。

ResourceEditor完全使用ApplicationContext根据注入的路径字符串获取相应的Resource，说白了还是自己做还是容器帮你做的问题。

原文地址：http://jinnianshilongnian.iteye.com/blog/1416321

### Resource通配符

#### 通配符加载Resource







- 

> Ioc--Inversion of Control，即"控制反转".
- 控制

> 传统JavaSE中，直接在对象内部通过new进行创建对象，是程序主动创建依赖对象；而IOC是由IOC容器来控制对象的创建，也就是有容器来控制外部资源的获取（不只是对象还包括文件等）。
- 反转

> 传统应用程序是由我们自己在对象中主动创建依赖对象，是正转；而反转则是由容器来创建及注入依赖对象，为何是反转？因为容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；依赖对象的获取被反转了。

- 传统JavaSE

![传统JavaSE](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/JavaSE-traditional.png)

- IOC容器

![IOC容器](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/Spring-IOC.png)

## DI(Dependency Injection)

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
## SpEL表达式

![SpEL原理](https://raw.githubusercontent.com/garaiya/java-learning-route/master/Part1/Spring/images/SpEL.png)

