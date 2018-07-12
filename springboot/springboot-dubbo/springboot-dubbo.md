#### 一、安装zookeeper

~~~shell
#下载zookeeper镜像
docker pull zookeeper
#启动zookeeper镜像
docker run --name zk01 -p 2181:2181 --restart always -d zookeeper

~~~

#### 二、POM依赖

~~~xml
	<!-- 引入dubbo和zk客户端依赖 -->
		<dependency>
			<groupId>com.alibaba.boot</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>0.1.0</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
		<dependency>
			<groupId>com.github.sgroschupf</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.1</version>
		</dependency>
~~~

#### 三、provider

##### 1.配置文件

~~~properties
#项目名称
dubbo.application.name=provider-ticket
#注册中心地址
dubbo.registry.address=zookeeper://47.105.103.45:2181
#扫描的service包
dubbo.scan.base-packages=com.haoge.dubbo.ticket.service
~~~

##### 2.service

~~~java
//接口
public interface TicketService {
	public String getTicket();
}
~~~

~~~java
//接口实现
//使用service注解将服务发布到注册中心中去
@Service//com.alibaba.dubbo.config.annotation.Service
@Component
public class TicketServiceImpl implements TicketService{
	public String getTicket() {
		return "第一部电影";
	}
}
~~~

#### 四、consumer

##### 1.配置文件

~~~properties
dubbo.application.name=consumer-user
dubbo.registry.address=zookeeper://47.105.103.45:2181
~~~

##### 2.引用提供者

即将提供者接口也在consumer项目中拷贝一份，企业开发中可以将提供者接口做成API

![1](/images/1.png)

##### 3.consumer代码

~~~java
@Service
public class UserService {
	@Reference//引用ticketService
	TicketService ticketService;
	public void hello() {
		System.out.println("买到票了:"+ticketService.getTicket());
	}
}
~~~

##### 4.测试代码

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootDubboConsumerApplicationTests {

	@Autowired
	UserService userService;

	@Test
	public void contextLoads() {
		userService.hello();
	}
}

~~~



