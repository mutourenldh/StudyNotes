#### 一、hystrix简介

![1](/images/1.png)

![2](/images/2.png)

![3](/images/3.png)

#### 二、服务熔断测试

##### 1. 项目搭建



###### 1. 新建springcloud-provider-dept-hystrix-8001

仿照springcloud-provider-dept-8001搭建springcloud-provider-dept-hystrix-8001项目

POM依赖

~~~xml
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
		<!-- hystrix -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
	</dependencies>
~~~

######2. yaml配置文件

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
    instance-id: deptService8001-hystrix     #对当前服务起的别名
    prefer-ip-address: true     #我们在eureka服务端查看服务名称的时候：访问路径可以显示IP地址     
info: 
  app.name: Deptservicecloud
  company.name: www.haoge.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$    

~~~

###### 3. 修改controller

~~~java
@RestController
public class DeptController {
	@Autowired
	private DeptService service;
	
	/**
	 * 根据Id查询部门
	 * @param id
	 * @return
	 */
	//一旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法
	@HystrixCommand(fallbackMethod="processHystrix_Get")
	@RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
	public Dept get(@PathVariable Long id) {
		Dept dept= service.get(id);
		if (dept==null) {
			throw new RuntimeException("该ID：" + id + "没有没有对应的信息");
		}
		return dept;
	}
	public Dept processHystrix_Get(@PathVariable("id") Long id) {
		Dept dept = new Dept();
		dept.setDname("该ID：" + id + "没有没有对应的信息,null--@HystrixCommand");
		dept.setDeptno(id);
		dept.setDb_source("no this database in MySQL");
		return dept;
	}

}
~~~

###### 4. 主启动类@EnableCircuitBreaker

~~~java
@SpringBootApplication
@EnableEurekaClient//这个注解的意思是本服务启动后会自动注册进eureka服务端中
@EnableDiscoveryClient//允许服务发现的注解
@EnableCircuitBreaker//对hystrix熔断机制的支持
public class DeptProvider8001_hystrix {
	public static void main(String[] args) {
		SpringApplication.run(DeptProvider8001_hystrix.class, args);
	}
}
~~~

###### 5. 测试

启动7001,7002,7003项目，启动springcloud-provider-dept-hystrix-8001项目，启动springcloud-consumer-dept-80

效果：

![4](/images/4.png)

###### 6. 工作过程

当我们访问不存在的数据的时候，控制器throw new RuntimeException

，这个时候在Hystrix的作用下走我们指定的方法processHystrix_Get，返回一个预期的结果。

#### 三、服务降级测试

服务降级：整体资源不够了，我们先关掉一部分，待度过难关之后再开启回来。

服务降级是在客户端完成的，和服务端没有关系。

###### 1. 修改springcloud-api

新建DeptClientServiceFallBackFactory，实现FallbackFactory\<eptClientService\>。代码如下

~~~java
@Component
public class DeptClientServiceFallBackFactory implements FallbackFactory<DeptClientService> {

	@Override
	public DeptClientService create(Throwable arg0) {
		return new DeptClientService() {

			@Override
			public Dept get(long id) {
				Dept dept = new Dept();
				dept.setDname("该ID：" + id + "没有没有对应的信息,Consumer客户端提供的降级信息,此刻服务Provider已经关闭");
				dept.setDeptno(id);
				dept.setDb_source("no this database in MySQL");
				return dept;
			}

			@Override
			public List<Dept> list() {
				// TODO Auto-generated method stub
				return null;
			}

			@Override
			public boolean add(Dept dept) {
				// TODO Auto-generated method stub
				return false;
			}
		};
	}
}
~~~

###### 2. 修改DeptClientService

~~~java
//@FeignClient(value = "springcloud-dept")
@FeignClient(value="springcloud-dept",fallbackFactory=DeptClientServiceFallBackFactory.class)
public interface DeptClientService {
	
	@RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
	public Dept get(@PathVariable("id") long id);

	@RequestMapping(value = "/dept/list", method = RequestMethod.GET)
	public List<Dept> list();

	@RequestMapping(value = "/dept/add", method = RequestMethod.POST)
	public boolean add(Dept dept);

}
~~~

###### 3. 修改springcloud-consumer-dept-feign yaml文件

~~~yaml
server:
  port: 80
  
eureka:
  client:
    register-with-eureka: false #自己不能注册
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/  
#增加的内容      
feign: 
  hystrix: 
    enabled: true
~~~

###### 4. 测试

启动7001,7002,7003三个eureka Server,然后启动springcloud-provider-dept-8001，再启动springcloud-consumer-dept-feign。

正常访问http://localhost/consumer/dept/get/1

然后故意关闭springcloud-provider-dept-8001再次访问http://localhost/consumer/dept/get/1

###### 5. 测试结果

![6](/images/6.png)

###### 6. 工作原理

将所有关于DeptClientService中方法的异常处理统一用接口处理，如上若

DeptClientService中有服务dwon掉了，我们就找DeptClientServiceFallBackFactory中对应的方法进行服务降级处理

#### 四、hystrix Dashboard

##### 1. 简介

![11](/images/11.png)

##### 2. 项目搭建

###### 1. 新建maven子项目

新建maven子项目 springcloud-consumer-hystrix-dashboard

###### 2. POM依赖

~~~xml
<dependencies>
		<!-- 自己定义的api -->
		<dependency>
			<groupId>com.haoge.cloud</groupId>
			<artifactId>springcloud-api</artifactId>
			<version>${project.version}</version>
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
		<!-- Ribbon相关 -->
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
		<!-- feign相关 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>
		<!-- hystrix和 hystrix-dashboard相关 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
		</dependency>
	</dependencies>
~~~

###### 3. YAML文件

~~~yaml
server:
  port: 9001
~~~

###### 4. 主启动类

~~~java
@SpringBootApplication
@EnableHystrixDashboard
public class DeptConsumer_DashBoard_App {
	
	public static void main(String[] args) {
		SpringApplication.run(DeptConsumer_DashBoard_App.class, args);
	}
}
~~~

所有provider项目（8001,8002,8003）都需要有监控组件。

~~~xml
<!-- actuator主管监控信息完善 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
~~~

###### 5. 测试

启动项目springcloud-consumer-hystrix-dashboard

访问 http://localhost:9001/hystrix

![7](/images/7.png)



分别启动7001,7002,7003 eureka server三个项目，再启动springcloud-provider-dept-hystrix-8001

访问：http://localhost:8001/dept/get/2

访问：http://localhost:8001/hystrix.stream

效果：

![1532057862379](/images/8.png)

###### 6. 使用图形化监控界面

![1532057911753](/images/9.png)

![1532057992480](/images/10.png)

![1532064503720](/images/720.png)

![1532064514748](/images/748.png)

![1532064528464](/images/464.png)

#### 五、Zuul

##### 1. Zuul简介

![958](/images/958.png)

##### 2. 项目搭建

新建maven子模块springcloud-zuul-gateway-9527

###### 1. POM依赖

~~~xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.haoge.cloud</groupId>
    <artifactId>springcloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <artifactId>springcloud-zuul-gateway-9527</artifactId>
  
  <dependencies>
		<!-- zuul路由网关 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<!-- actuator监控 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- hystrix容错 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<!-- 日常标配 -->
		<dependency>
			<groupId>com.haoge.cloud</groupId>
			<artifactId>springcloud-api</artifactId>
			<version>${project.version}</version>
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
		<!-- 热部署插件 -->
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

###### 2. yaml文件

~~~yaml
server: 
  port: 9527
 
spring: 
  application:
    name: springcloud-zuul-gateway
 
eureka: 
  client: 
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka  
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true 
    
zuul: 
#  ignored-services: springcloud-dept 
  prefix: /haoge
  ignored-services: "*"
  routes: 
    mydept.serviceId: springcloud-dept
    mydept.path: /mydept/**

info:
  app.name: Deptservicecloud
  company.name: www.haoge.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$

~~~

###### 3. 主程序

~~~java
@SpringBootApplication
@EnableZuulProxy
public class Zuul_9527_StartSpringCloudApp {

	public static void main(String[] args) {
		SpringApplication.run(Zuul_9527_StartSpringCloudApp.class, args);
	}
}
~~~

###### 4. 配置host

~~~shell
127.0.0.1 myzuul.com
~~~

###### 5. 测试

启动三个eureka server 7001,7002,7003项目。再启动springcloud-provider-dept-8001。再启动springcloud-zuul-gateway-9527

访问http://localhost:7001/

![1532066680323](/images/323.png)

即zuul也会把自己作为服务注册在eureka中。

访问数据：

不启用路由：http://localhost:8001/dept/get/2

启用路由：http://myzuul.com:9527/springcloud-dept/dept/get/2

##### 3. zuul路由映射规则

###### 1. 配置用mydept代替springcloud-dept

~~~yaml
zuul: 
  routes: 
    mydept.serviceId: springcloud-dept
    mydept.path: /mydept/**
~~~

此时

http://myzuul.com:9527/springcloud-dept/dept/get/2

http://myzuul.com:9527/mydept/dept/get/2

都可以访问

###### 2. 禁用服务

~~~yaml
#禁用springcloud-dept服务（禁用单个服务）
ignored-services: springcloud-dept 
#禁用所有的服务名称
ignored-services: "*"
~~~

示例如下：

~~~yaml
zuul: 
  ignored-services: springcloud-dept 	
#  ignored-services: "*"
  routes: 
    mydept.serviceId: springcloud-dept
    mydept.path: /mydept/**
~~~

此时http://myzuul.com:9527/springcloud-dept/dept/get/2不可以访问

http://myzuul.com:9527/mydept/dept/get/2可以访问

###### 3. 增加前缀

访问服务之前增加haoge前缀。

~~~yaml
zuul: 
  ignored-services: springcloud-dept 	
  prefix: /haoge
  routes: 
    mydept.serviceId: springcloud-dept
    mydept.path: /mydept/**
~~~

此时访问路径改为：

http://myzuul.com:9527/haoge/mydept/dept/get/2