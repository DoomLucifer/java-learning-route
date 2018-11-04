[TOC]

# Maven 

## Maven Tutorial

### Maven Website

> 官网地址：[click me](http://maven.apache.org)

### What is a Build Tool？

> 构建工具就是一种使与软件项目相关的所有内容自动化构建的工具，典型的构建软件工程包含以下几步：

1. Generating source code
2. Generating documentation from the source code
3. Compiling source code
4. Packaging compiled code into JAR files or ZIP files
5. Installing the packaged code on a server,in a repository or somewhere else.

### Maven Overview - Core Concepts

> Maven的核心就是POM（Project Object Model）文件

Maven核心概念预览图

1. Build Life Cycles，Phases and Goals

>  Maven的构建过程被分成构建生命周期，阶段和目标。构建生命周期由一系列的阶段组成，每个阶段又由一系列的目标组成。Maven命令就是一个构建周期或者阶段或者目标的名字，如果一个生命周期被执行则该生命周期下的所有阶段都会被执行，如果一个阶段被执行则该阶段之前的所有预定义的阶段也会被执行。例如clean compile package install

2. Dependencies and Repositories

> 依赖就是项目里使用的外部JAR文件，如果依赖不在本地仓库，Maven就会从远程中央仓库下载并保存到本地仓库。

3. Build Plugins

> 构建插件用于往构建阶段里插入额外的目标

4. Build Profiles

> 不同的环境使用不同的构建配置文件

### Maven vs Ant

Ant使用命令式方法，侧重于怎样去构建；Maven使用声明式方法，侧重于构建什么，而怎么去构建被定义在Maven Build Life Cycles,Phases and Goals.

### Effective POM

> 当POM文件都有多层继承关系时，无法知道子项目执行时有效POM文件的结构，可以通过mvn help:effective-pom 命令得到有效的POM

### Maven Settings File

> Maven有两个配置文件：
>
> Maven安装目录：$M2_HOME/conf/settings.xml
>
> 用户home目录：${user.home}/.m2/settings.xml
>
> 如果两个文件都存在，用户home目录下的配置将会覆盖Maven安装目录下的配置

### Running Maven

>  mvn dependency:copy-dependecies：这个命令式构建阶段命令和目标命令的组合，中间用冒号分隔，该命令执行dependency构建阶段的copy-dependencies目标

### External Dependencies

> 外部依赖就是一个不在maven库里（包括本地库、远程库和中央仓库）的依赖（jar文件），大多数依赖对于项目来说是外部的，但很少数对于库是外部的

配置外部依赖：

```xml
<dependency>
  <groupId>mydependency</groupId>
  <artifactId>mydependency</artifactId>
  <scope>system</scope>
  <version>1.0</version>
  <systemPath>${basedir}\war\WEB-INF\lib\mydependency.jar</systemPath>
</dependency>
```

systemPath属性设置包含依赖的jar文件的位置，${basedir}指向POM文件所在的目录

### Maven Repositories

Maven有三种库：

1. 本地库
2. 中央库
3. 远程库

maven按照上面库的顺序去查找依赖，首先在本地库，然后在中央库，如果pom里指定了远程再去远程库查找

Maven 库类型及其位置图

Remote Repository配置

```xml
<repositories>
   <repository>
       <id>jenkov.code</id>
       <url>http://maven.jenkov.com/maven2/lib</url>
   </repository>
</repositories>
```

### Maven Build Life Cycles,Phases and Goals

- Build Life Cycles

Maven有三个内置的构建生命周期：

1. default
2. clean
3. site

default处理编译和打包相关的；clean处理清除临时文件，如source文件，class文件，jar文件等；site处理项目文档相关的；

- Build Phases

| Build Phase | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `validate`  | Validates that the project is correct and all necessary information is available. This also makes sure the dependencies are downloaded. |
| `compile`   | Compiles the source code of the project.                     |
| `test`      | Runs the tests against the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed. |
| `package`   | Packs the compiled code in its distributable format, such as a JAR. |
| `install`   | Install the package into the local repository, for use as a dependency in other projects locally. |
| `deploy`    | Copies the final package to the remote repository for sharing with other developers and projects. |

注：当执行install时，在构建阶段序列里install之前的所有构建阶段将被执行，然后在执行install

### Maven Build Profiles

```xml
<profiles>
    <profile>
        <id>test</id>
        <activation>...</activation>
        <build>...</build>
        <modules>...</modules>
        <repositories>...</repositories>
        <pluginRepositories>...</pluginRepositories>
        <dependencies>...</dependencies>
        <reporting>...</reporting>
        <dependencyManagement>...</dependencyManagement>
        <distributionManagement>...</distributionManagement>
    </profile>
</profiles>
```

注：profile标签下的元素会覆盖pom文件里相同元素的值

## Maven Archetypes

### Available Maven Archetypes

```xml
mvn archetype:generate
```

该条指令将会输出所有可用的原型，因为没有指定具体的原型

### Named Archetypes

- Eclipse

```xml
mvn eclipse:eclipse
```

注：该命令在执行前需要在项目根目录有一个POM文件，然后在pom里生成原型

- Idea Archetype

```xml
mvn idea:idea
```





