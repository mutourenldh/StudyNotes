#### java8知识点学习总结

### 一、lambda表达式

#### 1.lambda表达式是什么

lambda是一个匿名函数，我们可以把lambda表达式理解为一段可以传递的代码（将代码像数据一样进行传递）

- 从匿名内部类到lambda表达式的转换

  ~~~java
  public class newTest1115 {
  
      public static void main(String[] args) {
  //        匿名内部类的写法
          Runnable runnable = new Runnable(){
              @Override
              public void run() {
                  System.out.println("Hello World");
              }
          };
          runnable.run();
  //        lambda表达式的写法1
          Runnable runnable1 = () -> {
              System.out.println("Hello World111!");
          };
          runnable1.run();
  //      lambda表达式的写法2
          Runnable r1= () -> System.out.println("1111");
          r1.run();
      }
  }
  ~~~

  使用lambda表达式作为参数进行传递

  ~~~java
  public class test2 {
      public static void main(String[] args) {
  //        原来的使用匿名内部类作为参数进行传递
          TreeSet<String> t1 = new TreeSet<String>(new Comparator<String>() {
              @Override
              public int compare(String s, String t1) {
                  return Integer.compare(s.length(),t1.length());
              }
          });
  //        使用lambda表达式作为参数进行传递
          TreeSet<String> t2 = new TreeSet<>(
                  (o1,o2) -> Integer.compare(o1.length(),o2.length())
          );
      }
  }
  ~~~

#### 2.lambda表达式语法

Lambda表达式在java中引入了一个新的语法元素和操作符。这个操作符为

“ ->”,该操作符被称为lambda操作符或者箭头操作符。它将lambda分为两部分：

左侧：指定了lambda表达式需要的所有参数

右侧：指定了Lambda体，即lambda表达式要执行的功能。

1. 语法格式一：无参无返回值，lambda体只需要一条语句

   ~~~java
   	/**
   	 *  1.无参无返回值
   	 * 	() -> System.out.println("lambda表达式的第一种形式");
   	 */
   	@Test
   	public void test1() {
   		Runnable runnable = ()-> System.out.println("lambda表达式的第一种形式");
   		runnable.run();
   	}
   ~~~

2. 语法格式二：有一个参数，无返回值

   ~~~java
   	/**
   	 * 2.有一个参数，无返回值,
   	 * (x) -> System.out.println(x);
   	 * 	如果只有一个参数，则参数列表中的小括号可以省略不写
   	 * x -> System.out.println(x);
   	 */
   	@Test
   	public void test2() {
   		Consumer<String> comConsumer=(x) -> System.out.println(x);
   		comConsumer.accept("第二种形式");
           //省略参数列表中的小括号
   		Consumer<String> aaConsumer= x -> System.out.println(x);
   		aaConsumer.accept("123");
   	}
   ~~~

   

3. 语法格式三：有两个参数，有返回值，并且lambda体中有多条语句

   ~~~java
   	/**
   	 * 3.有两个参数，并且有多条实现语句，且有返回值
   	 */
   	@Test
   	public void test3() {
   //		//有两个参数，并且有多条实现语句，且有返回值
   		Comparator<Integer> aComparator=(x,y) -> {
   			System.out.println(x);
   			System.out.println(y);
   			return Integer.compare(x, y);
   		};
   		int compare = aComparator.compare(1, 2);
   		System.out.println(compare);
   		System.out.println("-----------------------");
   		//有两个参数，但是只有一条实现语句的时候，lambda表达式实现体中的大括号{}和return可以省略不写
   		Comparator<Integer> bComparator=(x,y) ->  Integer.compare(x, y);
   		int compare2 = bComparator.compare(1, 2);
   		System.out.println(compare2);
   	}
   ~~~

4. 类型推断

   ~~~java
   	@Test
   	public void test4() {
   		//lambda表达式中参数列表中的类型可以省略不写，因为jvm编译器可以通过上下文推断得出，称之为类型推断
   		Comparator<Integer> comparator= (Integer x,Integer y) -> Integer.compare(x, y);
   		int compare = comparator.compare(4, 3);
   		System.out.println(compare);
   	}
   ~~~



#### 3.函数式接口

- 只包含一个抽象方法的接口，称为函数式接口。
- 我们可以通过Lambda表达式来创建函数式接口的对象。如果Lambda表达式抛出一个受检异常，那么该异常需要在函数式接口中的抽象方法上申明。
- 我们可以在任意函数式接口上使用@FunctionalInterface注解，这样做可以检查它是否是一个函数式接口。同时javadoc也会包含一条声明，说明这个接口是一个函数式接口。

##### 1.函数式接口的应用

自定义函数式接口

~~~java
@FunctionalInterface//这个注解可以检查接口是不是函数式接口
public interface MyFunction {
	public Integer getValue(Integer x);
}
~~~

将函数式接口作为参数传到方法的参数列表中，然后我们在调用这个方法的时候，再使用lambda表达式实现该函数式接口即可

~~~java
public class TestLambda1 {
	/**
	 * 首先定义一个函数式接口，然后将接口作为参数传进方法中。在方法中调用接口的抽象方法
	 * 在实际调用该方法的时候，传入的参数实际上是我们通过lambda实现的函数式接口实现
	 * @param num
	 * @param mf
	 * @return
	 */
	public Integer operation(Integer num,MyFunction mf) {
		return mf.getValue(num);
	}
	
	@Test
	public void name() {
        //调用我们定义的operation方法，并且使用lambda表达式实现参数中的函数式接口，实现我们真正需要实现的业务
		Integer operation = operation(100, (x) -> x*x);
		
		Integer operation2 = operation(2, x -> x+4);
		System.out.println(operation);
		System.out.println(operation2);
	}
}
~~~

##### 2.java内置的四大核心函数式接口

1. 消费型接口

   ~~~java
   //1.消费型接口，接收一个参数，没有返回值
   //Consumer<T> 对接收到的类型为T的参数进行操作，包含方法
   void accept(T t)
   //首先定义一个方法
   public void happy(double money,Consumer<Double> com) {
   		com.accept(money);
   	}
   //之后调用这个方法，并且对参数中的消费型接口进行具体实现
   @Test
   	public void testConsumer() {
   		happy(100, x -> System.out.println("吃饭一共花了"+x+"元钱！"));
   	}
   ~~~

2. 供给型接口，无参，有返回值

```java
//1.供给型接口，没有参数，返回类型为T的对象
//Supplier<T> 供给型接口<T> 返回类型为T的对象，包含方法 
T get()
    
//供给型接口测试
	@Test
	public void testSupplier() {
		List<Integer> list=getNumList(5, () -> (int)(Math.random()*100));
		for (Integer integer : list) {
			System.out.println(integer);
		}
	}
	
	//需求：产生指定个数的整数，并且放入集合中
	public List<Integer> getNumList(Integer num,Supplier<Integer> sup){
		ArrayList<Integer> list = new ArrayList<Integer>();
		for (int i = 0; i < num; i++) {
			Integer integer = sup.get();
			list.add(integer);
			
		}
		return list;
	}
```

3. 函数型接口

```java
//1.函数型接口，对类型为T的对象进行操作，并且返回结果为R类型的对象
//包含方法 R apply(T t)
	public String dealStr(String str,Function<String, String> func) {
		return func.apply(str);
	}
	@Test
	public void name() {
		String dealStr = dealStr("abcdefg", (str) -> str.substring(1,5));
		System.out.println(dealStr);
		String dealStr2 = dealStr("\t\t\t李东浩喜欢刘倩倩      ", (str) -> str.trim());
		System.out.println(dealStr2);
	}
```

4. 断言型接口

```java
//1.断言型接口，接收一个参数，返回boolean类型
//Predicate<T> 对接收到的类型为T的参数进行操作，包含方法
boolean test(T t)
//测试断言型接口
    //	4.测试断言型接口
//	需求：判断一个字符换集合中的每个字符串的长度，返回字符串长度大于2的字符串集合
	public List<String> filterStr(List<String> list,Predicate<String> pre){
		
		ArrayList<String> list2 = new ArrayList<String>();
		for (String string : list) {
			if (pre.test(string)) {
				list2.add(string);
			}
		}
		return list2;
	}
	@Test
	private void name3() {
		List<String> list = new ArrayList<String>();
		list.add("aaaa");
		list.add("bb");
		list.add("ccc");
		list.add("ddd");
		
		List<String> filterStr = filterStr(list, (s) -> s.length()>2);
		for (String string : filterStr) {
			System.out.println(string);
		}
	}
```

##### 3.方法引用

当要传递给lambda体的操作，已经有实现的方法的时候，我们可以使用方法引用。（实现抽象方法的参数列表，必须与方法引用方法的参数列表保持一致）

方法引用：使用操作符“::” 将方法名和对象或者类的名字分割开来

主要使用方法有如下三种情况：

- 对象::实例方法
- 类::静态方法
- 类::实例方法

1.测试对象::实例方法

~~~java
//测试方法引用中的对象::实例方法名
	@Test
	public void test1() {
//		消费型接口：有一个参数，但是没有返回值
		Consumer<String> com = (x) -> System.out.println(x);
		com.accept("123");
		//打印语句实际上是调用了PrintStream中的println方法
		PrintStream ps=System.out;
		Consumer<String> com1=ps::println;
		com1.accept("倩倩");
	}
	@Test
	public void test2() {
		Employee employee = new Employee("123");
		Supplier<String> su= () -> employee.getName();
		System.out.println(su.get());
		
		
		Supplier<String> sup= employee::getName;
		String name=sup.get();
		System.out.println(name);
		
	}
~~~

2.测试方法引用中的 类::静态方法名

~~~java
	//测试方法引用中的   类::静态方法名
	@Test
	public void test3() {
		//lambda体中内容已经有方法实现了，并且该方法是静态方法，那么我们可以使用
//		类::方法名的方式来进行方法引用
		Comparator<Integer> com=(x,y) -> Integer.compare(x, y);
		int a=com.compare(2, 3);
		System.out.println(a);
		
		Comparator<Integer> com1=Integer::compare;
		int b=com1.compare(4, 5);
		System.out.println(b);
	}
~~~

3.测试方法引用中的  类::实例方法名

这种情况使用需要有一定的条件。即如果参数列表中的第一个参数是实例方法的调用者，而第二个参数是实例方法的参数时。可以使用类名::方法名这种格式的方法进行引用

~~~java
	@Test//测试方法引用中的        类：实例方法名
	public void test4() {
		BiPredicate<String,String> bi=(x,y) -> x.equals(y);
		boolean test = bi.test("aa","bb");
		System.out.println(test);
		BiPredicate<String, String> bp=String::equals;
		boolean test2 = bp.test("mm", "n");
		System.out.println(test2);
	}
~~~

##### 4.构造器引用

格式：		ClassName::new

与函数式接口相结合，自动与函数式方法中方法兼容。可以把构造器引用赋值给定义的方法，与构造器参数列表要与接口中抽象方法的参数列表一致。

~~~java
	/**
	 * 构造器引用
	 * 格式：  类名：new
	 */
	@Test
	public void test5() {
		Supplier<Employee> sup=() -> new Employee();
		Supplier<Employee> su1=Employee::new;
		Employee emp=su1.get();
		System.out.println(emp);
		
		Function<String, Employee> fnc=(x) -> new Employee(x);
		Function<String, Employee> fnc1=Employee::new;
		Employee emp1=fnc1.apply("小李");
		System.out.println(emp1);
		
		BiFunction<String, Integer, Employee> bf=Employee::new;
		Employee emp2=bf.apply("倩倩", 10);
		System.out.println(emp2);
	}
~~~

##### 5.数组引用

格式    type[]::new

~~~java
/**
	 * 数组引用
	 * Type :: new 
	 */
	@Test
	public void test7() {
		
		Function<Integer, String[]> fun = (x) -> new String[x];
		String[] string1 = fun.apply(10);
		System.out.println(string1.length);
		
		Function<Integer, String[]> fun1 = String[]::new ;
		String[] string2 = fun1.apply(20);
		System.out.println(string2.length);
	}
~~~

#### 4.Stream

##### 1.Stream是什么

stream是数据渠道，用于操作数据源（集合，数组等）所生成的元素序列。

集合讲的是数据，流指的是计算。

**注意：**

1. Stream自己不会存储元素

2. Stream 不会改变源对象，相反他们会返回一个持有结果的新Stream
3. Stream 操作是延迟执行的。这意味这他们会等到需要结果的时候才会执行

##### 2.Stream操作的三个步骤

###### 1.创建Stream

一个数据源（如：集合、数组），获取一个流

~~~java
//测试创建strema流的几种操作
	@Test
	public void test1() {
//		1.可以通过Collection系列集合提供的stream()或者parallelStream()
		List<String> list= new ArrayList<String>();
		//串行流
		Stream<String> stream = list.stream();
		//并行流
		Stream<String> parallelStream = list.parallelStream();

//		2.通过数组中的静态方法stream()获取数组流
//        重载形式，能够处理对应基本类型的数组：
//public static IntStream stream(int[] array)
//public static LongStream stream(long[] array)
//public static DoubleStream stream(double[] array)
		Employee[] emps=new Employee[10];
		Stream<Employee> stream2 = Arrays.stream(emps);
		
//		3.通过Stream类中的静态方法of()，该方法可以接收任意数量的参数
		Stream<String> stream3 = Stream.of("aa","bb","cc");
		
//		4.使用静态方法 Stream.iterate() 和 Stream.generate(), 创建无限流
		Stream<Integer> stream4 = Stream.iterate(0,(x) -> x+2);
		stream4.limit(10).forEach(System.out::println);
		System.out.println("--------------------------");
//		5.生成
		Stream.generate(() -> Math.random())
			  .limit(5)
			  .forEach(System.out::println);
	}
~~~



###### 2.中间操作

一个中间操作链，对数据源的数据进行处理。多个中间线可以连接起来形成一个流水线，除非流水线上发生终止操作。否则中间操作不会执行任何的处理，而是在终止操作的时候一次性全部处理。称为“惰性求值”。

- 筛选和切片

  ```java
  /**
  	 *筛选和切片
  	 *filter 接收lambda，从流中排除某些元素
  	 *limit————截断流，使其元素不超过给定的数量
  	 *skip(n) ————跳过元素，返回一个扔掉了前N个元素的流。如果流中的元素不满足n个，则返回一个空流。与limit(n)进行互补
  	 *distinct 筛选，通过流所生成的hashcode()和equals()去除重复元素
  	 */
  	//内部迭代：迭代操作由Stream API完成
  	@Test
  	public void test2() {
  		//中间操作，不会执行任何操作,测试filter方法
  		Stream<Employee>  stream= emps.stream().filter(e -> {
  			System.out.println("Stream API的中间操作");
  			return e.getAge()>35;
  		});
  		//终止操作，一次性执行全部内容，即“惰性求值”
  		stream.forEach(System.out::println);
  		
  	}
  	@Test
  	public void test3() {
  		//测试limit方法
  		emps.stream().filter(e -> {
  			System.out.println("短路");
  			return e.getSalary()>=5000;
  		}).limit(2)
  		.forEach(System.out::println);
  		
  	}
  	@Test
  	public void test4() {
  		//测试skip方法和distinct方法
  		emps.stream().filter(e -> {
  			System.out.println("短路");
  			return e.getSalary()>=5000;
  		}).skip(2)
  		.distinct()
  		.forEach(System.out::println);
  	}
  ```

- 映射

  ~~~java
  /**
  	map(Function f) 接受一个函数作为参数，该函数会被映射到每个元素上，并将其映射成一个新的元素
  	mapToDouble(ToDoubleFunction f)类似于map方法，但是会产生一个新的DoubleStream
  	mapToInt(ToIntFunction f)类似于map方法，但是会产生一个新的IntStream
  	mapToLong(ToLongFunction f)类似于map方法，但是会产生一个新的LongStream
  	flatMap(Function f) 接受一个函数作为参数，将流中的每个值都转换成另一个流，然后把所有的流连接成一个流
  *
  /
  ~~~

  ~~~java
  	@Test
  	public void test5() {
  		//测试map方法
  		List<String> list=Arrays.asList("aaa","bbb","ccc","ddd","eee");
  		list.stream().map((str) -> str.toUpperCase())
  			.forEach(System.out::println);
  		System.out.println("--------------------------");
  		Stream<Employee> stream=emps.stream();
  		Stream<String> map2 = stream.map(Employee::getName);
  		map2.distinct().forEach(System.out::println);
  		System.out.println("------------------------");
  		System.out.println("---------测试map 和flatMap---------------");
  		//测试map 和flatMap
  		//区别：map方法会把aaa，bbb分别加到执行中间操作之后创建的新流中
  		//flatMap会把 “aaa”拆分成三个新的元素a加入到新流中
  		Stream<Stream<Character>> stream3=list.stream().map(TestStream1::filterCharter);
  		stream3.forEach(sm -> sm.forEach(System.out::println));
  		
  		System.out.println("---------测试flatMap---------------");
  		Stream<Character> sm = list.stream().flatMap(TestStream1::filterCharter);
  		sm.forEach(System.out::println);
  	}
  	
  	public static Stream<Character> filterCharter(String str) {
  		ArrayList<Character> list = new ArrayList<Character>();
  		for (Character character : str.toCharArray()) {
  			list.add(character);
  		}
  		return list.stream();
  	}
  ~~~

- 排序

  sorted()：产生一个新流，其中按照自然顺序排序

  sorted(Comparator comp)：产生一个新流，其中按照比较器顺序排序

  ~~~java
  	//测试排序方法
  	@Test
  	public void test7() {
  		List<String> list = Arrays.asList("ccc","aaa","bbb","ddd");
  		list.stream()
  			.sorted()//sorted无参方法：自然排序
  			.forEach(System.out::printf);
  
  		//定制化排序，在sorted接口中传入比较器实现类，进行排序
  		emps.stream()
  			.sorted((e1,e2) -> {
  				if (e1.getAge()==e2.getAge()) {
  					return e1.getName().compareTo(e2.getName());
  
  				}else {
  					return Integer.compare(e1.getAge(), e2.getAge());
  				}
  			});
  	}
  ~~~

  

###### 3.终止操作（终端操作）

终止操作会从流的流水线生成结果，其结果可以不是任何流的值，如list,integer,甚至void

- 查找和匹配

  ```java
      测试流的终止操作查找和匹配
      allMatch（Predicate p）———检查是否匹配所有元素			anyMatch(Predicate p)———检查是否至少匹配一个元素		noneMatch(Predicate p)————检查是否没有匹配所有元素
      findFirst————返回第一个元素
      findAny————返回当前流中的任意元素
      count————返回流中元素的总个数max(Comparator c)————返回流中的最大值
      min(Comparator c)—————返回流中的最小值
      forEach(Consumer c) 内部迭代，使用Collection接口需要用户自己去做迭代，成为外部迭代，相反Stream Api使用内部迭代。它帮你把迭代做了注意：流进行了终止操作后，不能再次使用
  ```

- 规约

  ~~~java
  reduce(T iden, BinaryOperator b) 可以将流中的元素反复结合起来得到一个新值，返回一个T
  reduce(BinaryOperator b) 可以将流中的元素反复结合起来，得到一个新值，返回Optional<T>
  ~~~

- 收集

  collect (Collector c) 将流转换成其他形式，接收一个Collector接口的实现，用于给Stream中的元素进行汇总。

  Collector接口中方法的实现决定了如何对流执行收集操作（如收集到List,Set，Map中），但是Collectors实用类提供了很多静态方法，可以方便的创建常见收集器实例。

**流终止操作的测试**

~~~java

public class TestStream2 {
	
	List<Employee> emps=Arrays.asList(
			new Employee(101, "张三", 18, 9999.99,Status.BUSY),
			new Employee(102, "李四", 59, 6666.66,Status.FREE),
			new Employee(103, "王五", 28, 3333.33,Status.VOCATION),
			new Employee(104, "赵六", 8, 7777.77,Status.BUSY),
			new Employee(104, "赵六", 8, 7777.77,Status.BUSY),
			new Employee(105, "田七", 38, 5555.55,Status.FREE)
			);
	@Test
	public void test1() {
		//测试allMatch方法
		boolean bl1 = emps.stream()
			.allMatch((e) -> e.getStatus().equals(Status.BUSY));
		System.out.println(bl1);
		//anyMatch
		boolean anyMatch = emps.stream().anyMatch(e -> e.getStatus().equals(Status.FREE));
		System.out.println(anyMatch);
		//noneMatch
		boolean noneMatch = emps.stream().noneMatch(e -> e.getStatus().equals(Status.VOCATION));
		System.out.println(noneMatch);
		//findFirst
		Optional<Employee> findFirst = emps.stream()
										.sorted((e1,e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
										.findFirst();
		System.out.println(findFirst.get());
		//findAny
		Optional<Employee> findAny = emps.stream().filter(e1 -> e1.getAge()>30)
				.findAny();
		System.out.println(findAny.get());
		//count方法
		long count = emps.stream().filter(e-> e.getStatus().equals(Status.FREE)).count();
		System.out.println(count);
		//max方法   返回流中的最大值
		Optional<Employee> max = emps.stream().max((e1,e2) -> Integer.compare(e1.getAge(),e2.getAge()));
		System.out.println(max.get());
		//min方法 返回流中的最小值
		Optional<Double> max2 = emps.stream().map(Employee::getSalary).min(Double::compareTo);
		System.out.println(max2.get());
	
	}
	/**
	 * 归约
	 * 测试reduce方法(T identity,BinaryOperator)/reduce(BinaryOperator)
	 * ——可以将流中的元素反复结合起来得到一个值
	 */
	@Test
	public void test3() {

		List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
		//将list中的元素进行相加
		Optional<Integer> reduce = list.stream().reduce((x,y)-> x+y);
		System.out.println(reduce.get());
		
		//10为起始值，从10开始，加上list中的所有元素
		Integer reduce2 = list.stream().reduce(10,(x,y)-> x+y);
		System.out.println(reduce2);
		
		Optional<Double> reduce3 = emps.stream().map(Employee::getSalary)
					 .reduce(Double::sum);
		System.out.println(reduce3.get());
	}
	
	/**
	 * collect  收集
	 * 将流装换为其他形式，接收一个Collector接口的实现，用于给Stream中元素做汇总的方法
	 */
	@Test
	public void test4() {
		//将emps集合数据收集到list集合中
		List<String> list = emps.stream().map(Employee::getName)
											.collect(Collectors.toList());
		list.forEach(System.out::printf);
		
		System.out.println("-----------------------------------------");
		//将emps集合数据收集到set集合中
		Set<String> set = emps.stream().map(Employee::getName).collect(Collectors.toSet());
		set.forEach(System.out::println);
		
		System.out.println("---------------------------");
		//将emps集合数据收集到指定的集合中
		HashSet<String> collect = emps.stream().map(Employee::getName).collect(Collectors.toCollection(HashSet::new));
		collect.forEach(System.out::printf);
	}
	/**
	 * collect  收集
	 * 将流装换为其他形式，接收一个Collector接口的实现，用于给Stream中元素做汇总的方法
	 */
	@Test
	public void test5() {
		//求流中的元素的总数
		Long count = emps.stream().collect(Collectors.counting());
		System.out.println(count);
		//平均数
		Double avg = emps.stream().collect(Collectors.averagingDouble(Employee::getSalary));
		System.out.println(avg);
		//求和
		Double sum = emps.stream().collect(Collectors.summingDouble(Employee::getSalary));
		System.out.println(sum);
		//summarizingDouble方法将平均数，总数，最大值，最小值都查出来了
		DoubleSummaryStatistics collect = emps.stream().collect(Collectors.summarizingDouble(Employee::getSalary));
		System.out.println(collect.getSum());
	}
	/**
	 * collect  收集
	 * 将流装换为其他形式，接收一个Collector接口的实现，用于给Stream中元素做汇总的方法
	 */
	@Test
	public void test6() {
		//测试收集最大值的操作
		Optional<Double> maxBy = emps.stream().map(Employee::getSalary)
					.collect(Collectors.maxBy(Double::compare));
		System.out.println(maxBy.get());
		
		Optional<Employee> collect = emps.stream()
		.collect(Collectors.maxBy((e1,e2) -> Double.compare(e1.getSalary(),e2.getSalary())));
		System.out.println(collect.get());
		
		Optional<Employee> collect2 = emps.stream().collect(Collectors.minBy((e1,e2)->Double.compare(e1.getSalary(), e2.getSalary())));
		System.out.println(collect2.get());
		//测试收集最小值的操作
		Optional<Double> opMinBy = emps.stream().map(Employee::getSalary).collect(Collectors.minBy(Double::compare));
		System.out.println(opMinBy.get());
	}
	//collect  收集：将流转换为其他形式，接受一个Collector接口的实现，用于给Stream中做元素汇总的方法
	//java8给Collector内置了一个接口实现Collectors,在这个接口中内置了很多方法，可以供我们方便调用
	//分组，根据某个属性值对流进行分组，属性为K，结果为V
	@Test
	public void test7() {
//		groupingBy中函数式接口实现要求一个参数，一个返回值
		Map<Status, List<Employee>> collect = emps.stream().collect(Collectors.groupingBy(e->e.getStatus()));
		System.out.println(collect);
		//多级分组1
		Map<Status, Map<Boolean, List<Employee>>> collect2 = emps.stream().
				collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy(e -> e.getSalary()>8000)));
		System.out.println(collect2);
		System.out.println("--------------------------");
		//多级分组2
		Map<Status, Map<String, List<Employee>>> collect3 = emps.stream().collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy((e) -> {
			if (e.getAge() >=70) {
				return "老年";
			}else if(e.getAge() >=35) {
				return "中年";
			}else {
				return "少年";
			}
		})));
		System.out.println(collect3);
		
	}
	
	//collect 收集：将流转换为其他形式，接收一个Collectors接口的实现，用于给Stream中元素做汇总的方法
//	分区，根据true or false进行分区
	@Test
	public void test8() {
		Map<Boolean, List<Employee>> collect = emps.stream().collect(Collectors.partitioningBy(e -> e.getSalary()>7000));
		System.out.println(collect);
	}
	
	//collect 收集：将流转换为其他形式，接收一个Collectors接口的实现，用于给Stream中元素做汇总的方法
//	测试join方法
	@Test
	public void test10() {
		String collect = emps.stream()
		.map(Employee::getName)
		.collect(Collectors.joining());
		String collect1 = emps.stream()
				.map(Employee::getName)
				.collect(Collectors.joining(","));
		String collect2 = emps.stream()
				.map(Employee::getName)
				.collect(Collectors.joining(",","--","==="));
		System.err.println(collect);
		System.err.println(collect1);
		System.err.println(collect2);
	}
	//测试流在进行了终止操作之后就不可以继续使用
	@Test
	public void test2() {
		
		Stream<Employee> stream = emps.stream()
				 .filter((e) -> e.getStatus().equals(Status.FREE));
				
				long count = stream.count();
				
				stream.map(Employee::getSalary)
					.max(Double::compare);
	}
}
~~~

##### 3.并行流和穿行流

并行流即将一个内容分成多个数据块，并且用 不同的线程分别处理每个数据块的流。

Stream API可以声明性的通过parallel()和sequential()在并行流和顺序流之间进行切换。

### 二、java8中的其它知识点

#### 1.Fork/Join框架

Fork/Join 框架：就是在必要的情况下，将一个大任务，进行拆分(fork)成若干个小任务（拆到不可再拆时），再将一个个的小任务运算的结果进行 join 汇总.

```java
@Test
	public void test3(){
		Instant startInstant = Instant.now();
		long start = System.currentTimeMillis();
		//range，需要传入开始节点和结束节点两个参数，返回的是一个有序的LongStream。包含开始节点和结束节点两个参数之间所有的参数，间隔为1.
//		 rangeClosed的功能和range类似。
		//两者的区别：rangeClosed包含最后的节点，但是range不包含最后的节点
		//并行流就是把一个内容分成多个数据块，并且用不同 的线程分别处理每个数据块的流
		//java8中我们可以申明性地通过parallel()和sequential()在并行流和顺序流之间进行切换
		Long sum = LongStream.rangeClosed(0L, 100000000L)
							 .parallel()//并行计算
							 .sequential()//串行计算
							 .sum();
		System.out.println(sum);
		Instant endInstant = Instant.now();
		long end = System.currentTimeMillis();
		System.out.println("耗费的时间为: " + Duration.between(startInstant, endInstant).toMillis()); //2061-2053-2086-18926
	}
```

#### 2.时间日期API

LocalDate,LocalTime,LocalDateTime是不可变的实例对象，是线程安全的。他们提供了简单的时间日期，并不包含当前的时间信息，也不包含与时区相关的信息。

###### 1.提供的方法

- now()：静态方法。根据当前时间创建对象

  ~~~java
  LocalDate localDate=LocalDate.now();
  LocalTime time2 = LocalTime.now();
  LocalDateTime time3 = LocalDateTime.now();
  ~~~

- of()：静态方法。根据指定时间/日期创建对象

  ~~~java
   LocalDate of = LocalDate.of(2019, 2, 2);
   LocalTime of1 = LocalTime.of(12, 12, 12);
   LocalDateTime of2 = LocalDateTime.of(2019, 2, 2, 2, 2, 2);
  ~~~

- plusDays,plusWeeks,plusMonths,plusYears      

  plus方法：从当前对象添加几天，几周，几年，几分钟，几小时等

- minus开头的方法：与plus开头的方法相反，从当前对象减去几天，几年，几分钟，几小时等

- withDayOfMonth 将指定的属性值修改为指定的值并且返回新的对象

- getDayOfMonth,getDayOfYear 等方法，获得月份天数，获得年份天数

- getMonth 获得月份的month枚举值

- isBefore , isAfter 比较两个LocalDate对象

- isLeapYear  判断是否是闰年

###### 2.Instant时间戳

~~~java
		Instant ins1 = Instant.now();
		System.out.println(ins1);
		//对该时间偏离的时区数
		OffsetDateTime odt = 									ins1.atOffset(ZoneOffset.ofHours(8));
		System.out.println(odt);
		//显示从1970年到指定时间的毫秒数
		System.out.println(ins1.toEpochMilli());
		//显示从1970年增加多长的时间
		Instant ins2 = Instant.ofEpochSecond(60);
		System.out.println(ins2);
~~~

###### 3.duration和period

duration:计算两个时间之间的间隔

period:计算两个日期之间的间隔

~~~java
//Duration测试
Instant ins1 = Instant.now();
		try {
			Thread.sleep(1000L);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		Instant ins2 = Instant.now();
		//Duration 计算两个时间之间的间隔
		Duration between = Duration.between(ins1, ins2);
		System.out.println(between.toMillis());
		System.out.println("-------------------------");
		LocalTime date1 = LocalTime.now();
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		LocalTime date2 = LocalTime.now();
		System.out.println(Duration.between(date1,date2).toMillis());
~~~

~~~java
//Period测试
		LocalDate date1 = LocalDate.of(2015, 1, 1);
		LocalDate date2 = LocalDate.now();
		Period between = Period.between(date1, date2);
		System.out.println(between);
		System.out.println(between.getYears());
		System.out.println(between.getMonths());
		System.out.println(between.getDays());
~~~

###### 4.TemporalAdjuster

TemporalAdjuster：时间矫正器 有时候我们可能需要获取例如：将星期调整到下个周日等操作

TemporalAdjusters：该类通过静态方法提供了大量的常用TemporalAdjuster的实现

~~~java
	@Test
	public void test5() {
		LocalDateTime ldt1 = LocalDateTime.now();
		System.out.println(ldt1);

		LocalDateTime ldt2 = ldt1.withDayOfMonth(10);
		System.out.println(ldt2);
		//获取当前日期的下一个周日
		LocalDateTime ldt3 = ldt1.with(TemporalAdjusters.next(DayOfWeek.FRIDAY));
		System.out.println(ldt3);

		//自定义：下一个工作日
		LocalDateTime ldt5 = ldt1.with((l) -> {
			LocalDateTime ldt4 = (LocalDateTime) l;
			DayOfWeek dayOfWeek = ldt4.getDayOfWeek();
			if (dayOfWeek.equals(DayOfWeek.FRIDAY)) {
				return ldt4.plusDays(3);
			} else if (dayOfWeek.equals(DayOfWeek.SUNDAY)) {
				return ldt4.plusDays(2);
			} else {
				return ldt4.plusDays(1);
			}
		});
		System.out.println(ldt5);
	}
~~~

###### 5.DateTimeFormatter

```
DateTimeFormatter:格式化时间/日期
@Test
	public void test6() {
		DateTimeFormatter dtf=DateTimeFormatter.ISO_DATE;
		LocalDateTime ldt = LocalDateTime.now();

		String strDate = ldt.format(dtf);
		System.out.println(strDate);

		System.out.println("---------------------------");

		DateTimeFormatter dtf2 = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");

		String strDate2 = dtf2.format(ldt);
		System.out.println(strDate2);

		LocalDateTime parse = ldt.parse(strDate2, dtf2);
		System.out.println(parse);
	}
```

###### 6.时区处理

```java
//java8中加入了对时区的支持，带时区的时间ZonedDate,ZonedTime,ZonedDateTime
//Zoneld:该类中包含了所有时区的信息
//getAvailableZonelds() 可以获取所有时区信息
//of(id) 用指定的时区信息获取Zoneld对象
	@Test
	public void test7(){
		Set<String> set = ZoneId.getAvailableZoneIds();
		set.forEach(System.out::println);
	}
	@Test
	public void test8(){
		LocalDateTime ldt = LocalDateTime.now(ZoneId.of("Europe/Tallinn"));
		System.out.println(ldt);
		ZonedDateTime zonedDateTime = ldt.atZone(ZoneId.of("Asia/Shanghai"));
		System.out.println(zonedDateTime);
	}
```

###### 7.与传统日期类的转换

java.time.Instant和java.util.Date可以相互转

~~~java
// 		java.time.Instant和java.util.Date可以相互转
        Date date = new Date();
        Instant instant = date.toInstant();
        Date from = Date.from(instant);
//      Instant和TimeStamp相互转换
        Timestamp timestamp = new Timestamp(2019);
        Instant instant1 = timestamp.toInstant();
        Timestamp from1 = Timestamp.from(instant1);
//      LocalDate和Date
        LocalDate now = LocalDate.now();
        java.sql.Date date1 =java.sql.Date.valueOf(now);
        LocalDate localDate = date1.toLocalDate();
        //LocalTime和java.sql.Time
        LocalTime now1 = LocalTime.now();
        Time time = Time.valueOf(now1);
        LocalTime localTime = time.toLocalTime();
        //java.time.format.DateTimeFormatter和java.text.DateFormat
        DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-MM-DD HH:mm:ss");
        Format format1 = format.toFormat();

~~~

##### 3.接口中的静态方法和默认方法

###### 1.默认方法

java8中允许接口中包含具有具体实现的方法，该方法称为默认方法。默认方法使用 default 关键字修饰。

类优先原则

 1.如果一个类，既继承了父类A，又实现了接口B，并且父类A和接口B中有一个同名方法，则调用该类中的这个方法时，实际调用的是父类中的该实现方法。接口中的同名方法会被忽略
 2.如果一个类实现了两个接口A和B，并且A和B中有一个同名和同参的方法，则在该类中必须覆盖这个方法来解决冲突

###### 2.静态方法

接口中允许添加静态方法

~~~java
public interface MyInterface {
	default String getName() {
		return "lidonghao";
	}
	public static void show() {
		System.out.println("接口中的静态方法");
	}
}
~~~

##### 4.Optional类

java.util.Optional是一个容器 类，用来表示一个值存在或者不存在。原来用null来表示一个值不存在，现在我们使用Optional可以更好的表达这个概念。并且可以避免空指针异常。

常用方法如下：

~~~java
//		1.of方法，创建一个optional实例
		Optional<String> opt1 = Optional.of("123");
		System.out.println(opt1.get());
//		2.empty()方法，创建一个空的optional实例
		Optional<Employee> opt2 = Optional.empty();
//		System.out.println(opt2.get());
//		3.ofNullable<T t>方法  如果t不为空，则创建optional实例，否则创建空实例 
		Optional<String> opt3 = Optional.ofNullable("456");
//		System.out.println(opt3.get());
//		4.isPresent()方法判断是否包含值
//		System.out.println(opt3.isPresent());
//		System.out.println(opt2.isPresent());
		//orElse(T t) 如果调用对象包含值，则返回该值。否则返回t
		Employee orElse = opt2.orElse(new Employee());
		System.out.println(orElse);
		//orElseGet(Supplier s) 如果调用对象包含值，则返回该值，否则返回s获取的值
		Employee orElseGet = opt2.orElseGet(() -> new Employee("123"));
		System.out.println(orElseGet);
//		map(Function f)如果有值对其进行处理，并返回处理后的optional,否则返回Optional.empty()
//		flatMap(Function mapper) 与map类似，要求返回的值必须是Optional
~~~

~~~java
//1.of方法，创建一个optional实例
		Optional<Employee> ofNullable = Optional.ofNullable(new Employee("倩宝贝", 18));
//		map(Function f)如果有值对其进行处理，并返回处理后的optional,否则返回Optional.empty()
//		Optional<String> map = ofNullable.map(e -> e.getName());
//		System.out.println(map.get());

//		flatMap(Function mapper) 与map类似，要求返回的值必须是Optional
		Optional<String> flatMap = ofNullable.flatMap(e -> Optional.of(e.getName()));
		System.out.println(flatMap.get());
~~~

