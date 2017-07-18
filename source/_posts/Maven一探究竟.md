---
title: Maven一探究竟
toc: true
date: 2016-11-10 16:56:17
category: 
	- 技术贴
	- Maven
tags: 
    - 继承
    - Maven
---


# 概述
Maven 作为一款自动构建工具，深受广大web开发者的喜爱。日常工作中其实经常会出现一些构建相关问题，博主针对一些问题也单独一篇文章来记录遇到的问题，而Maven本身一些新特性还是有必要去研究一下的。 

## Maven生命周期
![Maven生命周期](/img/maven_lifecycle.jpg)

上图是IDEA工具中的maven生命周期模型，实际上Maven是有三套生命周期的，简单介绍一下：

**clean生命周期**
主要目的是清理项目
pre-clean    ：执行清理前的工作；
clean    ：清理上一次构建生成的所有文件；
post-clean    ：执行清理后的工作

**default生命周期**
定义了真正构建时所需要执行的所有步骤，它是生命周期中最核心的部分
validate
initialize
generate-sources
process-sources: 处理项目主资源文件。一般来说，是对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中
generate-resources
process-resources
compile: 编译项目的主源码。一般来说，是编译src/main/Java目录下的Java文件至项目输出的主classpath目录中
process-classes
generate-test-sources
process-test-sources: 处理项目测试资源文件。一般来说，是对src/test/resources目录的内容进行变量替换等工作后，复制到项目输出的测试classpath目录中
generate-test-resources
process-test-resources
test-compile: 编译项目的测试代码，一般来说，是编译src/test/java目录下的Java文件至项目输出的测试classpath目录中
process-test-classes
test: 使用单元测试框架运行测试，测试代码不会打包或部署
prepare-package
package: 接受编译好的代码，打包成可发布的格式，如JAR
pre-integration-test
integration-test
post-integration-test
verify
install: 将包安装到Maven本地仓库，供本地其他Maven项目使用
deploy: 将最终的包复制到远程仓库，供其他开发人员和Maven项目使用

**site生命周期**
建立和发布项目站点，Maven能够基于POM所包含的信息，自动生成站点
pre-site
site    ：生成项目的站点文档；
post-site
site-deploy    ：发布生成的站点文档


## 插件
直接截图

![Maven-Clean生命周期](/img/maven_clean.jpg)
![Maven-Default生命周期](/img/maven_default.jpg)
![Maven-Site生命周期](/img/maven_site.jpg)


## 怎么用
<!--more-->
Maven其实很强大，提供了很高级的特性，一来方便我们自动构建项目，二来可以对整个项目进行一个全局性的管理。但是博主一直觉得Maven没什么好学的，也就没有系统研究过，这阵子正好项目需要做zk配置管理服务，就规整了一下项目中的一些配置，才知道之前的pom配置有多恶心。

### 依赖
用Maven最熟悉的要数dependency标签了 普通使用方法就不再详述了，主要介绍以下几种功能,随着项目开发以及功能迭代，之前的小项目演化成了一个多模块的大系统，模块与模块之间以及模块与第三方工具包之前都会存在很多依赖关系，如何梳理清楚这些依赖对于项目日常维护与扩展性都有很大帮助。

#### 非继承依赖方式管理POM
一般我们都会在父pom中配置n多dependency,然后子pom将需要引入的jar给dependency进来就ok了，例如
父pom
```
        <dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-tx</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-web</artifactId>
				<version>${spring.version}</version>
			</dependency>
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>${spring.version}</version>
			</dependency>
```


子pom不需要引入spring mvc模块
```
    <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
		</dependency>
```
看起来是不是很方便？但是通常大项目都会是这样的

```
xxx-base（父项目）
    xxx-common（公用组件，常量）
    xxx-utils（工具包）
    xxx-cache（缓存包）
    xxx-external（外部依赖包）

    xxx-api（接口服务）
    xxx-server（核心业务服务）
    …
```
这就出现两个问题
其一，父pom会无比巨大，因为不只是dependency插件，项目中肯定还有profile,如果项目还有全球化什么的，那就可想而知了。
其二，他们的依赖关系基本上都是 api/server 依赖于common,utils等模块，然后他们又都是base的子项目工程，是不是发现问题了？
如上述那种场景，api子pom只需要引入自己需要的jar，但它同时要引入xxx-common，而common中的一些依赖包恰恰是它不需要的。。。是不是就回到了原点。

楼主的解决方法：
```
抽出一个公共模块 xxx-dependency
将父pom关于dependency的全部依赖放在dependency中维护，减少父pom维护成本。

其他模块
    <dependencyManagement>
	    <dependencies>
	        <dependency>
	          <groupId>com.letv</groupId>
  			  <artifactId>lepush-dependency</artifactId>
  			  <version>1.0</version>
	          <type>pom</type>
	          <scope>import</scope>
	        </dependency>
	    </dependencies>
    </dependencyManagement>
```
做完了这一步，父pom的长度会大大减少，维护成本会小很多，解决了问题一，而问题二可以参见下文：禁止传递依赖

#### 禁止传递依赖
上文说到xxx-common模块中存在一些jar,xxx-api模块不需要，那么这么配置就解决了问题

```
    <dependency>
			<groupId>com.xxx</groupId>
			<artifactId>xxxx-common</artifactId>
			<version>0.0.1-SNAPSHOT</version>
			<exclusions>
				<exclusion>
					<groupId>*</groupId>
					<artifactId>*</artifactId>
				</exclusion>
			</exclusions>
		</dependency>	
```

