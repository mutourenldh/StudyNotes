#### 一、项目创建

分别创建eureka-server,eureka-provider,eureka-consumer三个项目。

eureka-server项目选中web和Eureka Server模块，eureka-provider,eureka-consumer都选中web和Eureka Discovery模块

![1](/images/1.png)

#### 二、eureka-server代码示例

##### 1.配置文件

~~~yaml
server:
  port: 8761
eureka:
  instance:
    hostname: eureka-server   #eureka实例的主机名
  client:
    register-with-eureka: false   #不把自己注册到eureka上
    fetch-registry: false         #不从eureka上来获取服务的注册地址
    service-url:
      defaultZone: http://localhost:8761/eureka
~~~

##### 2.@EnableEurekaServer注解

在主启动类上使用这个注解：开启基于注解的EurekaServer

~~~java
@EnableEurekaServer
@SpringBootApplication
public class SpringbootEurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootEurekaServerApplication.class, args);
	}
}
~~~

##### 3.测试

地址：http://localhost:8761/

效果如下：

![2](/images/2.png)

#### 三、eureka-provider代码

##### 1.配置文件

~~~yaml
server:
  port: 8002
spring:
  application:
    name: provider-ticket

eureka:
  instance:
    prefer-ip-address: true    #注册服务的时候使用IP进行注册
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
~~~

##### 2. service

~~~java
@Service
public class TicketService {
	
	public String getTicket() {
		System.out.println(8002);
		return "第一张电影票";
	}

}
~~~

##### 3. controller

~~~java
@RestController
public class TicketController {
	
	@Autowired
	TicketService ticketService;
	
	@GetMapping("/ticket")
	public String getTicket() {
		String ticket = ticketService.getTicket();
		return ticket;
	}

}
~~~

##### 4.测试

地址：http://localhost:8002/ticket

访问注册中心：http://localhost:8761/，我们看到已经有一个服务提供者注册在eureka上

![3](/images/3.png)

#### 四、eureka-consumer代码

##### 1.配置文件

~~~yaml
spring:
  application:
    name: consumer-user
server:
  port: 8200

  
eureka:
  instance:
    prefer-ip-address: true    #注册服务的时候使用IP进行注册
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
~~~

##### 2. 主启动程序

~~~java
@EnableDiscoveryClient    //开启服务发现的功能
@SpringBootApplication
public class SpringbootEurekaConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootEurekaConsumerApplication.class, args);
	}
	
	//使用RestTemplate进行远程服务调用
	@Bean
	@LoadBalanced//允许使用负载均衡机制,默认是轮询机制
	public RestTemplate restTemplate() {
		return new RestTemplate();
	}
}
~~~

##### 3. controller

~~~java
@RestController
public class UserController {
//测试地址  http://localhost:8200/buy?name=zhangsan
	@Autowired
	RestTemplate restTemplate;
	@GetMapping("buy")
	public String buy(String name) {
		
		String s = restTemplate.getForObject("http://provider-ticket/ticket", String.class);
		return name+"购买了"+s;
	}
}
~~~

##### 4.测试 

http://localhost:8200/buy?name=zhangsan

##### 5.负载均衡

我们修改提供者端口号，生成端口号为8001,8002的提供者jar包。启动之后，我们可以看到在注册中心有两个提供者，访问http://localhost:8200/buy?name=zhangsan的时候，使用轮询的负载均衡机制，依次访问每个服务提供者。

![5](/images/5.png)

启动命令

~~~shell
java -jar provider-8001.jar
~~~

#### 五、springboot热部署

~~~xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
</dependency>
~~~

#### 六、springboot监控

##### 1.pom依赖

~~~xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
~~~

##### 2.配置文件

~~~yaml
#开启监控功能
management:
  security:
    enabled: false 
~~~

##### 3.监控和管理端点

| 端点名      | 描述                        |
| ----------- | --------------------------- |
| autoconfig  | 所有自动配置信息            |
| auditevents | 审计事件                    |
| beans       | 所有Bean的信息              |
| configprops | 所有配置属性                |
| dump        | 线程状态信息                |
| env         | 当前环境信息                |
| health      | 应用健康状况                |
| info        | 当前应用信息                |
| metrics     | 应用的各项指标              |
| mappings    | 应用@RequestMapping映射路径 |
| shutdown    | 关闭当前应用（默认关闭）    |
| trace       | 追踪信息（最新的http请求）  |

例如：http://localhost:8080/health

#####  4.定制端点信息

###### 4.1规则

通过endpoints+端点名+属性名    来设置

修改端点IP :endpoints.beans.id=mybeans    

关闭端点：endpoints.beans.enabled=false    

开启某个端点    

endpoints.enabled=false    //关闭所有端点

endpoints.beans.enabled=true    //开启beans端点

定制端点访问根路径    management.context-path=/manage    

例子：

~~~yaml
#开启监控功能
management:
  security:
    enabled: false    #开启监控功能
  context-path: /manager  #定制管理端点根访问路径
  port: 8181   #定制管理端点端口号
endpoints:
  beans:
    id: mybean    #定义beans端点的id
    path: /bean   #定义beans端点的访问路径
~~~

如上配置之后我们访问beans管理端点路径为：

http://localhost:8181/manager/bean

##### 5. 自定义别的组件的health信息

###### 5.1代码示例

~~~java
@Component
public class MyAppHealthIndicator implements HealthIndicator{

	@Override
	public Health health() {
		// TODO Auto-generated method stub
		
		//获取信息进行判断组件的状态
//		return Health.up().build();//代表组件健康
		return Health.down().withDetail("msg", "服务down掉了。。。").build();
	}
}
~~~

###### 5.2 原则

1）名字必须为 xxxHealthIndicator

2)  必须实现接口HealthIndicator

3)  使用@Component将组件添加在容器中

###### 5.3 测试

测试地址：http://localhost:8181/manager/health

测试结果：

![6](/images/6.png)