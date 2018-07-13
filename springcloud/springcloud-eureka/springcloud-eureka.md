



### 一、环境准备

springboot+mybatis整合

#### 1.创建父工程

![5](/images/5.png)

1. POM依赖

~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.haoge.cloud</groupId>
  <artifactId>springcloud</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  
  	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<junit.version>4.12</junit.version>
		<log4j.version>1.2.17</log4j.version>
		<lombok.version>1.16.18</lombok.version>
	</properties>
	<!-- 父类工程的管理 -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR1</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>1.5.9.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>5.0.4</version>
			</dependency>
			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>druid</artifactId>
				<version>1.0.31</version>
			</dependency>
			<dependency>
				<groupId>org.mybatis.spring.boot</groupId>
				<artifactId>mybatis-spring-boot-starter</artifactId>
				<version>1.3.0</version>
			</dependency>
			<dependency>
				<groupId>ch.qos.logback</groupId>
				<artifactId>logback-core</artifactId>
				<version>1.2.3</version>
			</dependency>
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>${junit.version}</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>log4j</groupId>
				<artifactId>log4j</artifactId>
				<version>${log4j.version}</version>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<finalName>firstMainCloud</finalName>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
			</resource>
		</resources>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<configuration>
					<delimiters>
						<delimit>$</delimit>
					</delimiters>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
~~~

#### 2. 创建子工程

1.创建maven子模块

在springcloud上右键创建maven子工程springcloud-api

![33](/images/33.png)

![21](/images/21.png)

2. 打包方式选择jar

![20](/images/20.png)

3. POM依赖

~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.haoge.cloud</groupId>
    <artifactId>springcloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>springcloud-api</artifactId>
</project>
~~~

4. 编写公用实体类

~~~java
@SuppressWarnings("serial")
public class Dept implements Serializable{//必须实现序列化
	
	private Long deptno;//主键
	private String dname;//部门名称
	private String db_source;// 来自那个数据库，因为微服务架构可以一个服务对应一个数据库，同一个信息被存储到不同数据库
	
}
~~~

#### 3. 创建提供者springcloud-provider-dept-8001

任然是在springcloud项目上右键选择 new maven module,打包方式选择jar   springcloud-provider-dept-8001

1. POM依赖

~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.haoge.cloud</groupId>
    <artifactId>springcloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>springcloud-provider-dept-8001</artifactId>
   <dependencies>
		<!-- 引入自己定义的api通用包，可以使用Dept部门Entity -->
		<dependency>
			<groupId>com.haoge.cloud</groupId>
			<artifactId>springcloud-api</artifactId>
			<version>${project.version}</version>
		</dependency>
		<!-- actuator主管监控信息完善 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- 将微服务provider侧注册进eureka eureka后面没有server表示是eureka的客户端-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		
		
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-core</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jetty</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
		</dependency>
		<!-- 修改后立即生效，热部署 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>springloaded</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>
	</dependencies>
</project>
~~~

2. yaml文件

~~~yaml
server:
  port: 8001    # 项目的端口号
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件
    
spring:
   profiles:
    active:
    - dev
   application:
    name: springcloud-dept  # 当前微服务向外暴露的微服务名称
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01              # 数据库名称
    username: root
    password: 123456
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200  
~~~

3. 项目结构

![32](/images/32.png)



4. 主启动程序

~~~java
@SpringBootApplication
public class DeptProvider8001_App {
	public static void main(String[] args) {
		SpringApplication.run(DeptProvider8001_App.class, args);
	}
}
~~~

5. DeptController

~~~java
@RestController
public class DeptController {
	@Autowired
	private DeptService service;
	
	/**
	 * 增加部门的方法
	 * @param dept
	 * @return
	 */
	@RequestMapping(value="/dept/add",method=RequestMethod.POST)
	public boolean add(@RequestBody Dept dept) {
		return service.add(dept);
	}
	/**
	 * 根据Id查询部门
	 * @param id
	 * @return
	 */
	@RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
	public Dept get(@PathVariable Long id) {
		return service.get(id);
	}
	/**
	 * 查询部门列表
	 * @return
	 */
	@RequestMapping(value="/dept/list",method=RequestMethod.GET)
	public List<Dept> get() {
		System.out.println(8001);
		return service.list();
	}
}
~~~

6. DeptService

~~~java
public interface DeptService
{
	public boolean add(Dept dept);

	public Dept get(Long id);

	public List<Dept> list();
}
~~~

7.  DeptServiceImpl

~~~java
@Service
public class DeptServiceImpl implements DeptService {
	@Autowired
	private DeptDao dao;

	public boolean add(Dept dept) {
		// TODO Auto-generated method stub
		return dao.addDept(dept);
	}

	@Override
	public Dept get(Long id) {
		// TODO Auto-generated method stub
		return dao.findById(id);
	}

	@Override
	public List<Dept> list() {
		// TODO Auto-generated method stub
		return dao.findAll();
	}
}

~~~

8. DeptDao

~~~java
@Mapper
public interface DeptDao
{
	public boolean addDept(Dept dept);

	public Dept findById(Long id);

	public List<Dept> findAll();
}
~~~

9. DeptMapper.xml

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.haoge.cloud.dao.DeptDao">

	<select id="findById" resultType="com.haoge.cloud.entities.Dept" parameterType="Long">
		select deptno,dname,db_source from dept where deptno=#{deptno};
	</select>
	<select id="findAll" resultType="com.haoge.cloud.entities.Dept">
		select deptno,dname,db_source from dept;
	</select>
	<insert id="addDept" parameterType="com.haoge.cloud.entities.Dept">
		INSERT INTO dept(dname,db_source) VALUES(#{dname},DATABASE());
	</insert>

</mapper>
~~~

10 mybatis.cfg.xml（可以省略）

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

	<!-- <settings>
		<setting name="cacheEnabled" value="true" />二级缓存开启
	</settings> -->

</configuration>
~~~

11. 测试

http://localhost:8001/dept/get/2

http://localhost:8001/dept/list

#### 4. 创建消费者springcloud-consumer-dept-80

1.创建maven子项目

任然是在springcloud项目上右键选择 new maven module,打包方式选择jar   springcloud-consumer-dept-80

2. POM依赖

~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.haoge.cloud</groupId>
    <artifactId>springcloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>springcloud-consumer-dept-80</artifactId>
  
   <dependencies>
		<dependency>
			<groupId>com.haoge.cloud</groupId>
			<artifactId>springcloud-api</artifactId>
			<version>${project.version}</version>
		</dependency>
		<!-- Ribbon相关 ，负载均衡-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-ribbon</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- 修改后立即生效，热部署 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>springloaded</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>
	</dependencies>
</project>
~~~

3. yaml文件

~~~yaml
server:
  port: 80
~~~

4. 主程序

~~~java
@SpringBootApplication
public class DeptConsumer80_App {
	
	public static void main(String[] args) {
		SpringApplication.run(DeptConsumer80_App.class, args);
	}

}
~~~

5. Controller

~~~java
@RestController
public class DeptController_Consumer {
	private static final String REST_URL_PREFIX = "http://localhost:8001";
//	private static final String REST_URL_PREFIX = "http://firstmaincloud-dept";

	@Autowired
	private RestTemplate template;
	/**
	 * 使用 使用restTemplate访问restful接口非常的简单粗暴无脑。 (url, requestMap,
	 * ResponseBean.class)这三个参数分别代表 REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。
	 */
	@RequestMapping(value = "/consumer/dept/add")
	public boolean add(Dept dept) {
		return template.postForObject(REST_URL_PREFIX + "/dept/add", dept, boolean.class);
	}
	/**
	 * 根据ID查询部门的方法
	 * 
	 * @param dept
	 * @return
	 */
	@RequestMapping(value = "/consumer/dept/get/{id}")
	public Dept get(@PathVariable("id") Long id) {
		return template.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
	}
	/**
	 * 查询列表的方法
	 * 
	 * @param dept
	 * @return
	 */
	@SuppressWarnings("unchecked")
	@RequestMapping(value = "/consumer/dept/list")
	public List<Dept> list() {
		
		return template.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
	}
	// 测试@EnableDiscoveryClient,消费端可以调用服务发现
	@RequestMapping(value = "/consumer/dept/discovery")
	public Object discovery() {
		return template.getForObject(REST_URL_PREFIX + "/dept/discovery", Object.class);
	}
}
~~~

RestTemplate

提供了多种便捷访问远程HTTP服务的方法，是一种简单便捷的访问restfl的服务模板类，是spring提供的用于访问rest服务的客户端模板工具集

6. 配置类

~~~java
@Configuration //@Configuration配置   ConfigBean = applicationContext.xml
public class ConfigBean {
	@Bean
	public RestTemplate getRestTemplete() {
		return new RestTemplate();
	}
}
~~~

ConfigBean相当于我们以前写的applicationContext.xml配置文件，getRestTemplete()方法相当于我们我们以前在xml文件中定义的bean

7.测试

http://localhost/consumer/dept/get/1

http://localhost/consumer/dept/list

### 二、Spring Eureka

#### 1.简介

1.Netflix在设计Eureka的时候遵循的是AP原则

C：Consistency（强一致性）

A：Availability（可用性）

P：Partition tolerance(分区容错性)

2.Eureka是一个基于rest的服务，用于定位服务，以实现云端中间层服务发现和故障转移。功能类似于dubbo的注册中心，比如zookeeper.SpringCloud主要使用Eureka来实现服务注册和发现功能。

3.Eureka采用了CS的设计架构。Eureka Server作为服务注册功能的服务器，它是服务注册中心。

而系统中的其他微服务，使用Eureka的客户端连接到Eureka的客户端连接到Eureka Server并维持心跳连接。

4.EurekaClient是一个Java客户端，用于简化Eureka Server的交互。客户端同时具备一个内置的，使用轮询负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期为30秒），如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认为90秒）

5.架构图

![27](/images/37.png)

#### 2 Eureka Server 搭建

1、 仍然是在springcloud项目上右键选择 new maven module,打包方式选择jar  ，项目名称为 springcloud-eureka-server-7001

2、POM依赖

~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.haoge.cloud</groupId>
    <artifactId>springcloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>springcloud-eureka-server-7001</artifactId>
  <dependencies>
		<!--eureka-server服务端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
		<!-- 修改后立即生效，热部署 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>springloaded</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>
	</dependencies>
</project>
~~~

3 、yaml配置文件

~~~yaml
server: 
  port: 7001
 
eureka: 
  instance:
    hostname: localhost #eureka服务端的实例名称(单机版)
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）,
~~~

eureka.client.service-url.defaultZone 对应的属性值意义：这个地址是Eureka服务端暴露给外部的地址，外部的微服务如果想注册进这个Eureka Server中，则响应的写这个地址。

4、主启动类

~~~java
@SpringBootApplication
@EnableEurekaServer// EurekaServer服务器端启动类,接受其它微服务注册进来
public class EurekaServer7001_App {
	
	public static void main(String[] args) {
		SpringApplication.run(EurekaServer7001_App.class, args);
	}
}
~~~

5、测试，还没有服务注册

http://localhost:7001/

![44](/images/44.png)

#### 3. 服务注册

将8001项目提供的服务注册进Eureka Server中

1.8001项目添加POM 依赖

~~~xml
<!-- 将微服务provider侧注册进eureka eureka后面没有server表示是eureka的客户端-->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
~~~

2、yaml添加配置

~~~yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka   #单机版
~~~

服务提供者eureka.client.service-url.defaultZone对应的属性值意义：表示这个微服务要注册进http://localhost:7001/eureka这个地址中。

3、主启动类添加注解@EnableEurekaClient

~~~java
@SpringBootApplication
@EnableEurekaClient//这个注解的意思是本服务启动后会自动注册进eureka服务端中
public class DeptProvider8001_App {
	public static void main(String[] args) {
		SpringApplication.run(DeptProvider8001_App.class, args);
	}
}
~~~

4、测试

启动7001服务端项目，再启动8001项目。

http://localhost:7001/

![613](/images/613.png)

图示为微服务对外暴露的服务名称，是我们在8001项目配置文件中写的spring.application.name对应的属性名称。Eureka自动帮我们转为大写。

5、8001服务配置补充

~~~yaml
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka   #单机版
#      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/      
  instance:
    instance-id: deptService8001     #对当前服务起的别名
    prefer-ip-address: true     #我们在eureka服务端查看服务名称的时候：访问路径可以显示IP地址  （使用IP进行服务注册）   
~~~

效果

![53](/images/53.png)

6、8001服务显示info信息

6.1 pom依赖

~~~xml
<!-- actuator主管监控信息完善 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
~~~

6.2 主工程springcloudPOM添加依赖

~~~xml
<build>
		<finalName>firstMainCloud</finalName>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
			</resource>
		</resources>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<configuration>
					<delimiters>
						<delimit>$</delimit>
					</delimiters>
				</configuration>
			</plugin>
		</plugins>
	</build>
~~~

作用：允许yaml配置文件读取POM文件中的值。（需要以\$开头，并以\$结尾）。如下所示

6.3 yaml配置

~~~yaml
info: 
  app.name: Deptservicecloud
  company.name: www.haoge.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$  
~~~

效果：

![536](/images/536.png)

#### 4 Eureka的自我保护机制

![401](/images/401.png)

spring cloud中可以使用

enable.server.enable-self-preservation=false 禁用自我保护模式

#### 5.服务发现

1、@EnableDiscoveryClient//允许服务发现的注解

8001项目主启动类加注解：@EnableDiscoveryClient

~~~java
@SpringBootApplication
@EnableEurekaClient//这个注解的意思是本服务启动后会自动注册进eureka服务端中
@EnableDiscoveryClient//允许服务发现的注解
public class DeptProvider8001_App {
	public static void main(String[] args) {
		SpringApplication.run(DeptProvider8001_App.class, args);
	}
}
~~~

2、controller添加测试代码

~~~java
	//服务发现模块
	@Autowired
	private DiscoveryClient client;
	
	@RequestMapping(value = "/dept/discovery", method = RequestMethod.GET)
	public Object discovery()
	{
		List<String> list = client.getServices();//查询eureka中的服务都有哪些
		System.out.println("**********" + list);

		List<ServiceInstance> srvList = client.getInstances("MICROSERVICECLOUD-DEPT");
		for (ServiceInstance element : srvList) {
			System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t"
					+ element.getUri());
		}
		return this.client;
	}
~~~

3、测试

http://localhost:8001/dept/discovery

4、80项目controller测试代码

~~~java
// 测试@EnableDiscoveryClient,消费端可以调用服务发现
	@RequestMapping(value = "/consumer/dept/discovery")
	public Object discovery() {
		return template.getForObject(REST_URL_PREFIX + "/dept/discovery", Object.class);
	}
~~~

测试：http://localhost/consumer/dept/discovery

#### 6. Eureka集群配置

1、仿照7001项目创建两个Eureka Server项目，端口分别为7002,7003

2、7001Server配置文件

~~~yaml
server: 
  port: 7001
 
eureka: 
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
#    hostname: localhost #eureka服务端的实例名称(单机版)
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
~~~

7001Server配置文件

~~~yaml
server: 
  port: 7002
 
eureka: 
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
#    hostname: localhost #eureka服务端的实例名称(单机版)
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7003.com:7003/eureka/
~~~

7003Server配置文件

~~~yaml
server: 
  port: 7003
 
eureka: 
  instance:
    hostname: eureka7003.com #eureka服务端的实例名称
#    hostname: localhost #eureka服务端的实例名称(单机版)
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7001.com:7001/eureka/
~~~

8001服务提供者yaml配置文件

~~~yaml
server:
  port: 8001    # 项目的端口号
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件
    
spring:
   profiles:
    active:
    - dev
   application:
    name: springcloud-dept  # 当前微服务向外暴露的微服务名称
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01              # 数据库名称
    username: root
    password: 123456
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
eureka:
  client: #客户端注册进eureka服务列表内
    service-url: 
#      defaultZone: http://localhost:7001/eureka   #单机版
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/      
  instance:
    instance-id: deptService8001     #对当前服务起的别名
    prefer-ip-address: true     #我们在eureka服务端查看服务名称的时候：访问路径可以显示IP地址     
info: 
  app.name: Deptservicecloud
  company.name: www.haoge.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$ 
~~~

3、配置host文件

127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com

4、测试。分别启动7001,7002,7003，8001项目

分别访问

http://eureka7001.com:7001/

http://eureka7002.com:7002/

http://eureka7003.com:7003/

效果

![450](/images/450.png)

#### 7. Eureka和zookeeper的比较

Zookeeper保证的是CP原则

Eureka保证的是AP原则

在电商网站的高并发访问情况下，比如双11当天。高可用性比强一致性更加重要。

1.数据库ACID介绍

![245](/images/245.png)

2、CAP理论介绍

传统的数据库如mysql满足的是CA原则

![249](/images/249.png)



![82](/images/82.png)

由于在当前的网络硬件肯定会出现延迟丢包等问题，所以分区容错性，即P使我们必须要实现的。所以我们必须在A和C中间进行权衡。

![94](/images/94.png)

![41](/images/41.png)

