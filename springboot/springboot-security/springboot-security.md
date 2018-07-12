#### 一、环境搭建

新建spring项目，选择web和thymeleaf模块

POM文件切换thymeleaf版本

~~~xml
<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
		<thymeleaf-layout-dialect.version>2.3.0</thymeleaf-layout-dialect.version>
		<thymeleaf-extras-springsecurity4.version>3.0.2.RELEASE</thymeleaf-extras-springsecurity4.version>
</properties>
~~~

将SpringSecurity实验文件夹中的controller文件和templates下的文件分别拷贝至项目对应路径下。启动项目。访问http://localhost:8080/

效果如下：

![1](/images/1.png)

#### 二、maven依赖

~~~xml
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<!-- 引入security模块 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.thymeleaf.extras/thymeleaf-extras-springsecurity4 -->
		<dependency>
			<groupId>org.thymeleaf.extras</groupId>
			<artifactId>thymeleaf-extras-springsecurity4</artifactId>
		</dependency>

</dependencies>
~~~

#### 三、代码示例

##### 1.项目结构

![2](/images/2.png)

##### 2.配置类代码



~~~java
//编写security的配置类，标注注解@EnableWebSecurity并且继承WebSecurityConfigurerAdapter
@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter{
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// TODO Auto-generated method stub
//		super.configure(http);
		//定制请求的授权规则,允许所有人访问/请求，允许有VIP1角色的访问level1下的所有请求
		http.authorizeRequests().antMatchers("/").permitAll()
		.antMatchers("/level1/**").hasRole("VIP1")
		.antMatchers("/level2/**").hasRole("VIP2")
		.antMatchers("/level3/**").hasRole("VIP3");
		//用userlogin处理登录请求，请求用户名user,密码pwd
		//开启自动配置的登录功能，security会自动生成登录页面，此处用的是定制的登录页面
	//发送/login的get请求默认来到security自动生成的登录页面,发送login的post请求用来处理登录逻辑
        http.formLogin().usernameParameter("user").passwordParameter("pwd").loginPage("/userlogin");
		//备注：
		//如果重定向到/login?error表示登录失败
		//默认情况下，/login的get请求跳转到登录页面，/login的post请求用来处理登录逻辑
		//如果我们定制了loginPage，则loginPage对应的get请求跳转到登录页面，loginPage对应的post请求用来处理登录逻辑
		
		//注销功能
		http.logout().logoutSuccessUrl("/");//注销成功之后跳转的连接
//		访问/logout表示用户注销，清空session
//		默认注销成功之后会返回/login？logout页面
		
		//开启记住我功能,自动生成的登录页面会出现记住我功能，并且参数name为remeber
		http.rememberMe().rememberMeParameter("remeber");
		//登陆成功之后，将cookie发送给浏览器进行保存，以后再访问的时候带着这个cookie就可以免登陆
		//点击注销只有，删除cookie
	}
	//定义认证规则
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		// TODO Auto-generated method stub
		//	super.configure(auth);
		//允许账号zhangsan,密码123456登录，该账号的角色为VIP1,VIP2
		auth.inMemoryAuthentication().withUser("zhangsan").password("123456").roles("VIP1","VIP2")
		.and().withUser("lisi").password("123456").roles("VIP1","VIP3")
		.and().withUser("wangwu").password("123456").roles("VIP3","VIP2");
		
	}
}
~~~

##### 3.页面代码

###### 3.1引入thymeleaf命名空间

~~~html
<html xmlns:th="http://www.thymeleaf.org"
	xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
~~~

###### 3.2 首页代码

~~~html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
	xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1 align="center">欢迎光临武林秘籍管理系统</h1>
<!-- 未登陆的情况 -->
<!-- isAuthenticated()判断是否登录成功 -->
<div sec:authorize="!isAuthenticated()">
<h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
</div>
<!-- 登陆的情况 -->
<div sec:authorize="isAuthenticated()">
<!-- sec:authentication="name" 取出登录名 -->
<h2 align="center"><span sec:authentication="name"></span>您好，您的角色有：
<!-- sec:authentication="name" 取出登录用户角色 -->
<span sec:authentication="principal.authorities"></span></h2>
<form action="" th:action="@{/logout}" method="post">
	<input type="submit" value="注销"/>
</form>
</div>

<hr>
<!-- hasRole('VIP1') 判断是否有VIP1角色，如果有，则显示包含内容 -->
<div sec:authorize="hasRole('VIP1')">
<h3>普通武功秘籍</h3>
<ul>
	<li><a th:href="@{/level1/1}">罗汉拳</a></li>
	<li><a th:href="@{/level1/2}">武当长拳</a></li>
	<li><a th:href="@{/level1/3}">全真剑法</a></li>
</ul>
</div>

<div sec:authorize="hasRole('VIP2')">
<h3>高级武功秘籍</h3>
<ul>
	<li><a th:href="@{/level2/1}">太极拳</a></li>
	<li><a th:href="@{/level2/2}">七伤拳</a></li>
	<li><a th:href="@{/level2/3}">梯云纵</a></li>
</ul>
</div>
<div sec:authorize="hasRole('VIP3')">
<h3>绝世武功秘籍</h3>
<ul>
	<li><a th:href="@{/level3/1}">葵花宝典</a></li>
	<li><a th:href="@{/level3/2}">龟派气功</a></li>
	<li><a th:href="@{/level3/3}">独孤九剑</a></li>
</ul>
</div>
</body>
</html>
~~~

###### 3.3 登录页代码

~~~html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h1 align="center">欢迎登陆武林秘籍管理系统</h1>
	<hr>
	<div align="center">
		<form th:action="@{/userlogin}" method="post">
			用户名:<input name="user"/><br>
			密码:<input name="pwd"><br/>
			<input type="checkbox" name="remeber">记住我<br/>
			<input type="submit" value="登陆">
		</form>
	</div>
</body>
</html>
~~~

