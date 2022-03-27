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

