### 1.springboot自动扫描包

spring boot自动扫包功能扫描的是主启动类所在包及其子包下的所有组件。此功能是@**SpringBootApplication**中的@**AutoConfigurationPackage**所起的作用

### **2.spring应用启动源码解析**

启动流程：

##### **1、创建SpringApplication对象**

```java
initialize(sources);
private void initialize(Object[] sources) {
    //保存主配置类
    if (sources != null && sources.length > 0) {
        this.sources.addAll(Arrays.asList(sources));
    }
    //判断当前是否一个web应用
    this.webEnvironment = deduceWebEnvironment();
    //从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来
    setInitializers((Collection) getSpringFactoriesInstances(
        ApplicationContextInitializer.class));
    //从类路径下找到ETA-INF/spring.factories配置的所有ApplicationListener
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //从多个配置类中找到有main方法的主配置类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

![](E:/BaiduNetdiskDownload/Spring%20Boot%20%E7%AC%94%E8%AE%B0/images/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180306145727.png)

![](E:/BaiduNetdiskDownload/Spring%20Boot%20%E7%AC%94%E8%AE%B0/images/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180306145855.png)

```java
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
    //获得类加载器
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<String>(
	//从指定位置加载SpringFactories	SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		//创建SpringFactories实例
            List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
				classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```

```java
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";	

public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		try {
            //从指定位置META-INF/spring.factories获得资源
			Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			List<String> result = new ArrayList<String>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
				String factoryClassNames = properties.getProperty(factoryClassName);
				result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
			}
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() +
					"] factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

