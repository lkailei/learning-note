## Maven

Maven是一个[项目管理工具](https://baike.baidu.com/item/项目管理工具)，它包含了一个项目对象模型 (Project Object Model)，一组标准集合，一个[项目生命周期](https://baike.baidu.com/item/项目生命周期)(Project Lifecycle)，一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段(phase)中[插件](https://baike.baidu.com/item/插件)(plugin)目标(goal)的逻辑。当你使用Maven的时候，你用一个明确定义的项目对象模型来描述你的项目，然后Maven可以应用横切的逻辑，这些逻辑来自一组共享的（或者自定义的）插件。

Maven 有一个生命周期，当你运行 mvn install 的时候被调用。这条命令告诉 Maven 执行一系列的有序的步骤，直到到达你指定的生命周期。遍历生命周期旅途中的一个影响就是，Maven 运行了许多默认的[插件](https://baike.baidu.com/item/插件)目标，这些目标完成了像编译和创建一个 JAR 文件这样的工作。

此外，Maven能够很方便的帮你管理项目报告，生成站点，管理JAR文件，等等。

apache下的纯java开发的项目管理工具，适用于管理这些依赖的jar包的工具，项目的构建，依赖管理  构建是：从源代码到编译，测试，运行，打包，部署，运行的过程

### maven包目录结构

[![HhLaTS.png](https://s4.ax1x.com/2022/02/16/HhLaTS.png)](https://imgtu.com/i/HhLaTS)

- `bin`：mvn.bat以run方式运行，mvnDeBug.bat:以debug的方式运行
                      

- `boot`：maven运行的类加载器
                       

- `conf`: settings.xml整个maven工具的核心配置工具
                       

- `lib`：maven运行的jar包；需要配置maven_Home,path，`maven-v`检查是否安装成功，

### 使用maven前的配置

1. 配置maven-path：便于通过classPath变量找到maven命令，全局都可以使用maven命令

   [![HhOAAS.png](https://s4.ax1x.com/2022/02/16/HhOAAS.png)](https://imgtu.com/i/HhOAAS)

2. 配置本地仓库位置在conf/setting.xml

   `<localRepository>D:/mavenstorage/repository</localRepository>`
   
3. 配置maven的提供商，因为使用国外镜像网络较慢，所以我们一般都走阿里的nexus镜像这样下载会更快，在conf/setting.xml配置

   ```xml
    <mirror>
         <id>alimaven</id>
         <mirrorOf>central</mirrorOf>
         <name>aliyun maven</name>
         <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
       </mirror>
   	<mirror>
         <id>alimaven</id>
         <mirrorOf>central</mirrorOf>
         <name>aliyun maven</name>
         <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
       </mirror>
       <mirror>
         <id>central</id>
         <name>Maven Repository Switchboard</name>
         <url>http://repo1.maven.org/maven2/</url>
         <mirrorOf>central</mirrorOf>
       </mirror>
       <mirror>
         <id>mvnrepo</id>
         <name>mvnrepo</name>
         <url>https://mvnrepository.com/artifact/</url>
         <mirrorOf>central</mirrorOf>
       </mirror>
   ```

经过以上简单的配置我们就可以在idea中使用maven了。

以上配置中，mirrorOf 的取值为 central，表示该配置为中央仓库的镜像，所有对于中央仓库的请求都会转到该镜像。当然，我们也可以使用以上方式配置其他仓库的镜像。另外三个元素 id、name 和 url 分别表示镜像的唯一标识、名称和地址。

以上配置中，mirrorOf 元素的取值为 * ，表示匹配所有远程仓库，所有对于远程仓库的请求都会被拦截，并跳转到 url 元素指定的地址。

为了满足一些较为复杂的需求，Maven 还支持一些更为高级的配置。

- <mirrorOf>*</mirrorOf>：匹配所有远程仓库。
- <mirrorOf>external:*</mirrorOf>：匹配所有远程仓库，使用 localhost 和 file:// 协议的除外。即，匹配所有不在本机上的远程仓库。
- <mirrorOf>repo1,repo2</mirrorOf>：匹配仓库 repo1 和 repo2，使用逗号分隔多个远程仓库。
- <mirrorOf>*,!repo1</miiroOf>：匹配所有远程仓库，repo1 除外，使用感叹号将仓库从匹配中排除。

###  maven的构建项目过程：

- `编译阶段`：compile complie阶段会将java文件编译程class文件到target目录
- `清理阶段`：clean 清理输出的class文件
    编译阶段：complie,将代码编译成class文件
                            

- `打包阶段`：package, java-->jar包，web包--->war包     
- `测试阶段`：test test阶段会将target目录下生成一个test-classes目录
- `部署项目`：install  会在target目录下生成一个jar包或者war包，同时也会经其部署到指定目录如放到mven仓库。

同样也可以组合使用 `mvn clean compile|test|install|package`

`mvn package -Dmaven.test.skip=true` 跳过单元测试和单元测试的编译  

> 使用maven好处**：一步构建，依赖管理，跨平台，提高团队效率 eclipse传统的构建项目过程：创建java代码，编译，单元测试，war包运行
>          **

### 什么是pom

项目对象模型或 POM 是 Maven 中的基本工作单元。它是一个 XML 文件，包含有关项目和 Maven 用于构建项目的配置细节的信息。它包含大多数项目的默认值。这方面的例子有构建目录(target) ; 源目录(src/main/java) ; 测试源目录(src/test/java)等等。当执行一个任务或目标时，Maven 会在工作目录中查找 POM。它读取 POM，获取所需的配置信息，然后执行目标,可以在 POM 中指定的一些配置包括项目依赖项、可执行的插件或目标、构建配置文件等等。还可以指定其他信息，如项目版本、说明、开发人员、邮件列表等。

#### 一个pom文件基本的元素

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1</version>
</project>
```

- `project `: 根

- `modelVersion `: 应该设置为4.0.0

- `groupId`:  项目的组Id

- `artifactId`:  项目Id

- `version`: 指定组级别的组件版本

### maven 在项目的使用

**maven工程项目的中的目录：**

- `src/main/java`：存放项目的.java文件
                      

- `src/main/resources`：存放项目的资源文件，如：Spring配置文件
                      

- ` src/test/java`：存放单元测试的.java 文件
                       

- `src/test/resources`：测试资源文件
                       

- `target`：项目的输入位置，编译后的class文件输出到这里
                    

- `pom.xml`：maven项目的核心配置文件

 **maven坐标就是maven对项目的jar包的定义：**

```html
  <groupId>com.leo.demo</groupId>
  <artifactId>Maven</artifactId><!--模块的名称-->
  <packaging>war</packaging><!--打包类型。war，jar，pom,-->
  <version>0.0.1-SNAPSHOT</version>
  <name>Maven Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>5.3.3.Final</version>
      </dependency>
  </dependencies>                         
```

**scope中属性：**
                            

- `provideed`：编译时需要，测试时也需要，运行时不需要，打包时不需要
                               

- `Runtime`：数据库的驱动包，编译时不需要，测试时需要，运行时不需要，打包需要
                              

- `Test`：编译时不需要，运行时不需要，测试时需要
                              

- `complie`：编译时需要，测试时需要，运行时需要，打包时需要
  
- `system`: 指定使用本地仓库的jar                     

修改Tomcat的即插件：配置端口和访问路径
                           

```xml
<build>
   <finalName>Maven</finalName>
   <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>tomcat-maven-plugin</artifactId>
        <version>1.1</version>
      <configuration>
        <port>8888</port>
        <path>/maven</path>
      </configuration>
     </plugin>
   </plugins>
</build>                              
```

当我们需要使用本地指定的jar只需要加入systemPath属性即可

```xml
<dependency>
	<groupId>htmlunit</groupId>
	<artifactId>htmlunit</artifactId>
	<version>2.21-OSGi</version>
	<scope>system</scope>
	<systemPath>${project.basedir}/libs/htmlunit-2.21-OSGi.jar</systemPath>
</dependency>
```

配置资源文件

```xml
  <build>
    <resources>
      <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
  </build>
```

配置经jar包install指定的位置

```xml
<build>
   <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
       </plugin>
       <plugin>
          <groupId>org.apache.maven.plugin</groupId>
           <artifactId>maven-antrun-plugin</artifactId>
           <execution>
               <id>copy-war-file</id>
               <phase>package</phase>
               <goals>
                   <goal>run</goal>
               </goals>
               <configuration>
                   <target>
                       <copy todir="${project.basedir}/../../../war">
                           <fileset dir="${project.basedir}/target">
                               <include name="*.war"></include>
                           </fileset>
                       </copy>
                   </target>
               </configuration>
           </execution>
       </plugin>
   </plugins>
</build>   
```

## 镜像与 Maven 私服配合使用

镜像通常会和 Maven 私服配合使用，由于 Maven 私服可以代理所有外部的公共仓库（包括中央仓库），因此对于组织内部的用户来说，使用一个私服就相当于使用了所有需要的外部仓库，这样就可以将配置集中到私服中，简化 Maven 本身的配置。这种情况下，用户所有所需的构件都可以从私服中获取，此时私服就是所有仓库的镜像。

```
`<srttings>``...``    ``<mirrors>``        ``<mirror>``            ``<id>nexus</id>``            ``<mirrorOf>*</mirrorOf>``            ``<name>nexus</name>``            ``<url>http:``//localhost:8082/nexus/content/groups/bianchengbang_repository_group/</url>``        ``</mirror>``    ``</mirrors>``...``</settings>`
```

以上配置中，mirrorOf 元素的取值为 * ，表示匹配所有远程仓库，所有对于远程仓库的请求都会被拦截，并跳转到 url 元素指定的地址。

为了满足一些较为复杂的需求，Maven 还支持一些更为高级的配置。

- `<mirrorOf>central</mirrorOf>`  表示该配置为中央仓库的镜像，所有对于中央仓库的请求都会转到该镜像

- `<mirrorOf>*</mirrorOf>`：匹配所有远程仓库。
- `<mirrorOf>external:*</mirrorOf>`：匹配所有远程仓库，使用 localhost 和 file:// 协议的除外。即，匹配所有不在本机上的远程仓库。
- `<mirrorOf>repo1,repo2</mirrorOf>`：匹配仓库 repo1 和 repo2，使用逗号分隔多个远程仓库。
- `<mirrorOf>*,!repo1</miiroOf>`：匹配所有远程仓库，repo1 除外，使用感叹号将仓库从匹配中排除。

```
`需要注意的是，由于镜像仓库完全屏蔽了被镜像仓库，当镜像仓库不稳定或者停止服务时，Maven 也无法访问被镜像仓库，因而将无法下载构件。`
```

### Maven私服

Maven 私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，用来代理位于外部的远程仓库（中央仓库、其他远程公共仓库）。

建立了 Maven 私服后，当局域网内的用户需要某个构件时，会按照如下顺序进行请求和下载。

1. 请求本地仓库，若本地仓库不存在所需构件，则跳转到第 2 步；
2. 请求 Maven 私服，将所需构件下载到本地仓库，若私服中不存在所需构件，则跳转到第 3 步。
3. 请求外部的远程仓库，将所需构件下载并缓存到 Maven 私服，若外部远程仓库不存在所需构件，则 Maven 直接报错。


此外，一些无法从外部仓库下载到的构件，也能从本地上传到私服供其他人使用。

下图中展示了 Maven 私服的用途。

[![ppUPI3T.png](https://s1.ax1x.com/2023/03/21/ppUPI3T.png)](https://imgse.com/i/ppUPI3T)

## Maven 私服优势

 Maven 私服具有以下 5 点优势：

#### 节省外网带宽

大量对于外部远程仓库的重复请求，会消耗很大量的带宽，利用 Maven 私服代理外部仓库后，能够消除对外部仓库的大量重复请求，降低外网带宽压力。

#### 下载速度更快

Maven 私服位于局域网内，从私服下载构建更快更稳定。

#### 便于部署第三方构件

有些构件是无法从任何一个远程仓库中获得的（例如，某公司或组织内部的私有构件、Oracle 的 JDBC 驱动等），建立私服之后，就可以将这些构件部署到私服中，供内部 Maven 项目使用。

#### 提高项目的稳定性，增强对项目的控制

如果不建立私服，那么 Maven 项目的构件就高度依赖外部的远程仓库，若外部网络不稳定，则项目的构建过程也会变得不稳定。
建立私服后，即使外部网络状况不佳甚至中断，只要私服中已经缓存了所需的构件，Maven 也能够正常运行。

此外，一些私服软件（如 Nexus）还提供了很多额外控制功能，例如，权限管理、RELEASE/SNAPSHOT 版本控制等，可以对仓库进行一些更加高级的控制。

#### 降低中央仓库得负荷压力

由于私服会缓存中央仓库得构件，避免了很多对中央仓库的重复下载，降低了中央仓库的负荷。

## Maven 私服搭建

能够帮助我们建立 Maven 私服的软件被称为 Maven 仓库管理器（Repository Manager），主要有以下 3 种：

- Apache Archiva
- JFrog Artifactory
- Sonatype Nexus

其中，Sonatype Nexus 是当前最流行、使用最广泛的 Maven 仓库管理器

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <!-- localRepository
     | The path to the local repository maven will use to store artifacts.
     |
     | Default: ${user.home}/.m2/repository
    <localRepository>/path/to/local/repo</localRepository>
    -->
    <localRepository>D:\mavenstorage\repository</localRepository>
    <!-- interactiveMode
     | This will determine whether maven prompts you when it needs input. If set to false,
     | maven will use a sensible default value, perhaps based on some other setting, for
     | the parameter in question.
     |
     | Default: true
    <interactiveMode>true</interactiveMode>
    -->

    <!-- offline
     | Determines whether maven should attempt to connect to the network when executing a build.
     | This will have an effect on artifact downloads, artifact deployment, and others.
     |
     | Default: false
    <offline>false</offline>
    -->

    <!-- pluginGroups
     | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
     | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
     | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
     |-->
    <pluginGroups>
        <!-- pluginGroup
         | Specifies a further group identifier to use for plugin lookup.
        <pluginGroup>com.your.plugins</pluginGroup>
        -->
    </pluginGroups>

    <!-- proxies
     | This is a list of proxies which can be used on this machine to connect to the network.
     | Unless otherwise specified (by system property or command-line switch), the first proxy
     | specification in this list marked as active will be used.
     |-->
    <proxies>
        <!-- proxy
         | Specification for one proxy, to be used in connecting to the network.
         |
        <proxy>
          <id>optional</id>
          <active>true</active>
          <protocol>http</protocol>
          <username>proxyuser</username>
          <password>proxypass</password>
          <host>proxy.host.net</host>
          <port>80</port>
          <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
        </proxy>
        -->
    </proxies>

    <!-- servers
     | This is a list of authentication profiles, keyed by the server-id used within the system.
     | Authentication profiles can be used whenever maven must make a connection to a remote server.
     |-->
    <servers>
        <!-- server
         | Specifies the authentication information to use when connecting to a particular server, identified by
         | a unique name within the system (referred to by the 'id' attribute below).
         |
         | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
         |       used together.
         |
        <server>
          <id>deploymentRepo</id>
          <username>repouser</username>
          <password>repopwd</password>
        </server>
        -->

        <!-- Another sample, using keys to authenticate.
        <server>
          <id>siteServer</id>
          <privateKey>/path/to/private/key</privateKey>
          <passphrase>optional; leave empty if not used.</passphrase>
        </server>
        -->
        <!-- <server>
          <id>nexus.qfengx.cn</id>
          <username>admin</username>
          <password>qazwsx123</password>
        </server> -->
        <!--第一个要id和下面的mirror中id一致代表拉取也需要进行身份验证-->
        <server>
            <id>maven-public</id>
            <username>admin</username>
            <password>dev@18538206096</password>
        </server>
        <server>
            <id>releases</id> <!--对应pom.xml的id=releases的仓库 发布时使用的-->
            <username>admin</username>
            <password>dev@18538206096</password>
        </server>
        <server>
            <id>snapshots</id> <!--对应pom.xml的id=snapshots仓库-->
            <username>admin</username>
            <password>dev@18538206096</password>
        </server>
    </servers>

    <!-- mirrors
     | This is a list of mirrors to be used in downloading artifacts from remote repositories.
     |
     | It works like this: a POM may declare a repository to use in resolving certain artifacts.
     | However, this repository may have problems with heavy traffic at times, so people have mirrored
     | it to several places.
     |
     | That repository definition will have a unique id, so we can create a mirror reference for that
     | repository, to be used as an alternate download site. The mirror site will be the preferred
     | server for that repository.
     |-->
    <mirrors>
        <!-- mirror
         | Specifies a repository mirror site to use instead of a given repository. The repository that
         | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
         | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
         |
        <mirror>
          <id>mirrorId</id>
          <mirrorOf>repositoryId</mirrorOf>
          <name>Human Readable Name for this Mirror.</name>
          <url>http://my.repository.com/repo/path</url>
        </mirror>
         -->
        <!--如果此处的nexus需要密码则需要在server中进行配置 id必须一致 必须和nexus上面的name 一致-->
        <mirror>
            <id>maven-public</id>
            <name>siiri Nexus</name>
            <url>http://172.16.30.20:8081/repository/maven-public/</url>
            <mirrorOf>*</mirrorOf>
        </mirror>
        <mirror>
            <id>alimaven</id>
            <mirrorOf>central</mirrorOf>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </mirror>
        <mirror>
            <id>alimaven</id>
            <mirrorOf>central</mirrorOf>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
        </mirror>
        <mirror>
            <id>central</id>
            <name>Maven Repository Switchboard</name>
            <url>http://repo1.maven.org/maven2/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
        <mirror>
            <id>mvnrepo</id>
            <name>mvnrepo</name>
            <url>https://mvnrepository.com/artifact/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>

    </mirrors>

    <!-- profiles
     | This is a list of profiles which can be activated in a variety of ways, and which can modify
     | the build process. Profiles provided in the settings.xml are intended to provide local machine-
     | specific paths and repository locations which allow the build to work in the local environment.
     |
     | For example, if you have an integration testing plugin - like cactus - that needs to know where
     | your Tomcat instance is installed, you can provide a variable here such that the variable is
     | dereferenced during the build process to configure the cactus plugin.
     |
     | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
     | section of this document (settings.xml) - will be discussed later. Another way essentially
     | relies on the detection of a system property, either matching a particular value for the property,
     | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
     | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
     | Finally, the list of active profiles can be specified directly from the command line.
     |
     | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
     |       repositories, plugin repositories, and free-form properties to be used as configuration
     |       variables for plugins in the POM.
     |
     |-->
    <profiles>
        <!-- profile
         | Specifies a set of introductions to the build process, to be activated using one or more of the
         | mechanisms described above. For inheritance purposes, and to activate profiles via <activatedProfiles/>
         | or the command line, profiles have to have an ID that is unique.
         |
         | An encouraged best practice for profile identification is to use a consistent naming convention
         | for profiles, such as 'env-dev', 'env-test', 'env-production', 'user-jdcasey', 'user-brett', etc.
         | This will make it more intuitive to understand what the set of introduced profiles is attempting
         | to accomplish, particularly when you only have a list of profile id's for debug.
         |
         | This profile example uses the JDK version to trigger activation, and provides a JDK-specific repo.
        <profile>
          <id>jdk-1.4</id>

          <activation>
            <jdk>1.4</jdk>
          </activation>

          <repositories>
            <repository>
              <id>jdk14</id>
              <name>Repository for JDK 1.4 builds</name>
              <url>http://www.myhost.com/maven/jdk14</url>
              <layout>default</layout>
              <snapshotPolicy>always</snapshotPolicy>
            </repository>
          </repositories>
        </profile>
        -->

        <!--
         | Here is another profile, activated by the system property 'target-env' with a value of 'dev',
         | which provides a specific path to the Tomcat instance. To use this, your plugin configuration
         | might hypothetically look like:
         |
         | ...
         | <plugin>
         |   <groupId>org.myco.myplugins</groupId>
         |   <artifactId>myplugin</artifactId>
         |
         |   <configuration>
         |     <tomcatLocation>${tomcatPath}</tomcatLocation>
         |   </configuration>
         | </plugin>
         | ...
         |
         | NOTE: If you just wanted to inject this configuration whenever someone set 'target-env' to
         |       anything, you could just leave off the <value/> inside the activation-property.
         |
        <profile>
          <id>env-dev</id>

          <activation>
            <property>
              <name>target-env</name>
              <value>dev</value>
            </property>
          </activation>

          <properties>
            <tomcatPath>/path/to/tomcat/instance</tomcatPath>
          </properties>
        </profile>
        -->
        <profile>
            <id>jdk-1.8</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
            </activation>
            <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
            </properties>
        </profile>

        <profile>
            <id>dev</id>
            <repositories>
                <repository>
                    <id>maven-public</id>
                    <name>siiri Nexus</name>
                    <url>http://172.16.30.20:8081/repository/maven-public/</url>
                    <!--是否下载 releases 构件-->
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </releases>
                    <!--是否下载 snapshots 构件-->
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
            </repositories>

        </profile>

    </profiles>

    <!-- activeProfiles
     | List of profiles that are active for all builds.
     |
    <activeProfiles>
      <activeProfile>alwaysActiveProfile</activeProfile>
      <activeProfile>anotherAlwaysActiveProfile</activeProfile>
    </activeProfiles>
    -->
    <activeProfiles>
        <activeProfile>dev</activeProfile>
        <activeProfile>jdk-1.8</activeProfile>
    </activeProfiles>
</settings>


```