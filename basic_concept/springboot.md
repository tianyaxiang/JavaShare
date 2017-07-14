#  SpringBoot
## SpringBoot 
使用Spring Boot快速构建应用
随着Spring 4新版本的发布，Spring Boot这个新的子项目得到了广泛的关注，因为不管是Spring 4官方发布的新闻稿还是针对首席架构师Adrian Colyer的专访，都对这个子项目所带来的生产率提升赞誉有加。

Spring Boot充分利用了JavaConfig的配置模式以及“约定优于配置”的理念，能够极大的简化基于Spring MVC的Web应用和REST服务开发。

Spring 4倡导微服务的架构，针对这一理念，近来在微博上也有一些有价值的讨论，如这里和这里。微服务架构倡导将功能拆分到离散的服务中，独立地进行部署，Spring Boot能够很方便地将应用打包成独立可运行的JAR包，因此在开发模式上很契合这一理念。目前，Spring Boot依然是0.5.0的里程碑版本，因此相关的文档尚不完善，本文将会以一个简单的样例来介绍基于这个项目的开发过程。

要Spring Boot进行功能开发，需要使用Gradle或者Maven作为构建工具。在本例中，我们会使用Eclipse和Maven插件进行开发。要使用Spring Boot，首先创建一个Maven工程，并修改Maven主要的配置文件pom.xml，如下所示：

<parent>
       <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>0.5.0.M7</version>
 </parent>
  <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring3</artifactId>
        </dependency>
    </dependencies>

    <properties>
        <start-class>com.levin.Application</start-class>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestone</id>
            <url>http://repo.spring.io/libs-milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-milestone</id>
            <url>http://repo.spring.io/libs-milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
在上面的配置中，需要将工程的parent设置为spring-boot-starter-parent，并添加对spring-boot-starter-web的依赖，这样我们就无需设置各个依赖项及其版本信息了。并且在构建中要声明使用spring-boot-maven-plugin这个插件，它会对Maven打包形成的JAR进行二次修改，最终产生符合我们要求的内容结构。

在我们的应用中将要发布一个REST服务，显示一个基本的用户信息，首先定义一个简单的模型类：

	
	package com.levin;

public class Person {
	private String name;
	private String email;
	
	public Person(String name, String email) {
		this.name = name;
		this.email = email;
	}
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	
}
接下来，我们需要声明一个Spring MVC的Controller，响应对实体的请求：

@Controller
public class ShowPersonController {
    @RequestMapping("/showPerson")
    public @ResponseBody Person showPerson() {
        return new Person("levinzhang","levinzhang1981@gmail.com");
    }
}
这个类与我们在使用Spring MVC定义Controller时并无任何差别。接下来，我们需要声明一个主类启动这个应用程序：

@ComponentScan
@EnableAutoConfiguration
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
这个类的main方法中使用了SpringApplication帮助类，并以Application这个类作为配置来启动Spring的应用上下文。在这个类中使用了ComponentScan以及EnableAutoConfiguration注解，其中ComponentScan注解会告知Spring扫描指定的包来初始化Spring Bean，这能够确保我们声明的Bean能够被发现。EnableAutoConfiguration将会启动自动配置模式，在我们的配置中会将对Tomcat的依赖级联进来，因此在应用启动时将会自动启动一个嵌入式的Tomcat，因为在样例中使用了Spring MVC，所以也会自动注册所需的DispatcherServlet，这都不需要类似web.xml这样的配置。

在Eclipse中要运行这个应用的话，可以直接以Java Application的形式来运行这个main函数，此时会启动应用，我们在浏览器中可以看到如下的运行效果，这就是我们想要的REST服务：



在开发调试完成之后，可以将应用打成JAR包的形式，在Eclipse中可以直接使用Maven插件的package命令，最终会形成一个可运行的JAR包。我们使用java –jar命令就可以运行这个JAR包了。所呈现出的效果与在调试期是一样的。现在看一下这个JAR包解压后的目录结构：



这个JAR包与传统JAR包的不同之处在于里面有一个名为lib的目录，在这个目录中包含了这个简单应用所依赖的其他JAR包，其中也包含内置的嵌入式Tomcat，正是使用它，才能发布服务和访问Web资源。除了我们编写的源码所编译形成的CLASS以外，在org目录下还有许多Spring所提供的CLASS，正是依赖这些CLASS，才能够加载位于lib目录下JAR中的类。这样的加载机制与在OSGi bundle中声明Bundle-Classpath很类似，不过在OSGi中会由容器来负责加载指定路径下的类。这大致阐述了这样一个JAR包能够发布服务的原因。

如果我们想要使用HTML、JSP等Web资源的话，在Controller中直接返回对应的视图就可以了。

如果我们想要将这个JAR包转换成可以在Servlet容器中部署的WAR的话，就不能依赖于Application的main函数了，而是要以类似于web.xml文件配置的方式来启动Spring应用上下文，此时我们需要声明这样一个类：

public class HelloWebXml extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

}
这个类的作用与在web.xml中配置负责初始化Spring应用上下文的监听器作用类似，只不过在这里不需要编写额外的XML文件了。

如果要将最终的打包形式改为WAR的话，还需要对pom.xml文件进行修改，除了需要将packaging的值修改为war以外，还需要对依赖进行适当的配置（这一部分在Spring Boot的样例和文档中均未提及，提醒大家注意）：

<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId> 
           <exclusions>
           	<exclusion>
           		<groupId>org.springframework.boot</groupId>
        		<artifactId>spring-boot-starter-tomcat</artifactId>
           	</exclusion>
           </exclusions> 
</dependency>
在这里需要移除对嵌入式Tomcat的依赖，这样打出的WAR包中，在lib目录下才不会包含Tomcat相关的JAR包，否则将会出现启动错误。另外，在移除对Tomcat的依赖后，为了保证编译正确，还需要添加对servlet-api的依赖，因此添加如下的配置：

		
<dependency>
        	<groupId>org.apache.tomcat</groupId>
        	<artifactId>tomcat-servlet-api</artifactId>
        	<version>7.0.42</version>
        	<scope>provided</scope>
</dependency>
在这里将scope属性设置为provided，这样在最终形成的WAR中不会包含这个JAR包，因为Tomcat或Jetty等服务器在运行时将会提供相关的API类。此时，执行mvn package命令就会得到一个WAR文件，我们可以直接将其放到Tomcat下运行（需要7.0.42版本以上）。

以上介绍了基于Spring Boot开发应用的过程，目前它的文档尚不完善，但是在GitHub上有不少的样例，包括与Spring Data集成访问数据库（关系型以及非关系型）、安全、WebSocket等，读者感兴趣可以下载运行，需要注意的是有些样例中使用的是0.5.0.M6版本，这个版本有问题，运行时会出错，建议手动修改为0.5.0.M7或快照版本。

基于以上的介绍，希望读者能够对Spring Boot这个新项目有所了解。它简化了JAR包管理和相关基础设施环境的配置，能够帮助我们快速开发Web应用或构建REST服务，希望它能够尽快完善成熟，更多地用于实践，提升开发效率。

参考资料

http://projects.spring.io/spring-boot/

http://projects.spring.io/spring-boot/docs/README.html

http://spring.io/guides/gs/spring-boot/

http://spring.io/guides/gs/actuator-service/

http://spring.io/guides/gs/convert-jar-to-war/
