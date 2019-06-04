

[JDK1.8 新特性（全）](https://blog.csdn.net/qq_29411737/article/details/80835658)

[JDK1.8 十大新特性详解](https://www.cnblogs.com/wangkang0320/articles/6841524.html)

jdk1.8新特性知识点：

1. **Lambda表达式**

2. **函数式接口**

3. **Stream API**

4. **接口中的默认方法(default)和静态方法(static)**

5. **新时间日期API**

6. **Optional类**

7. **方法引用和构造器调用**

8. **在jdk1.8中对hashMap等map集合的数据结构优化。hashMap数据结构的优化** 
   原来的hashMap采用的数据结构是哈希表（数组+链表），hashMap默认大小是16，一个0-15索引的数组，如何往里面存储元素，首先调用元素的hashcode 
   方法，计算出哈希码值，经过哈希算法算成数组的索引值，如果对应的索引处没有元素，直接存放，如果有对象在，那么比较它们的equals方法比较内容 
   如果内容一样，后一个value会将前一个value的值覆盖，如果不一样，在1.7的时候，后加的放在前面，形成一个链表，形成了碰撞，在某些情况下如果链表 
   无限下去，那么效率极低，碰撞是避免不了的 
   加载因子：0.75，数组扩容，达到总容量的75%，就进行扩容，但是无法避免碰撞的情况发生 
   在1.8之后，在数组+链表+红黑树来实现hashmap，当碰撞的元素个数大于8时 & 总容量大于64，会有红黑树的引入 
   除了添加之后，效率都比链表高，1.8之后链表新进元素加到末尾 
   ConcurrentHashMap (锁分段机制)，concurrentLevel,jdk1.8采用CAS算法(无锁算法，不再使用锁分段)，数组+链表中也引入了红黑树的使用



## Lambda



lambda表达式本质上是一段匿名内部类，也可以是一段可以传递的代码

先来体验一下lambda最直观的优点：简洁代码

```java
  //匿名内部类
  Comparator<Integer> cpt = new Comparator<Integer>() {
      @Override
      public int compare(Integer o1, Integer o2) {
          return Integer.compare(o1,o2);
      }
  };

  TreeSet<Integer> set = new TreeSet<>(cpt);

  System.out.println("=========================");

  //使用lambda表达式
  Comparator<Integer> cpt2 = (x,y) -> Integer.compare(x,y);
  TreeSet<Integer> set2 = new TreeSet<>(cpt2);

  //使用lambda表达式的遍历迭代
  set.forEach(x->System.out.println(x));
  
  //在 Java 8 中使用双冒号操作符(double colon operator)  
  set.forEach(System.out::println);

```

Lmabda表达式的语法总结： () -> ();



| 前置                                       | 语法                                                 |
| ------------------------------------------ | ---------------------------------------------------- |
| 无参数无返回值                             | () -> System.out.println(“Hello WOrld”)              |
| 有一个参数无返回值                         | (x) -> System.out.println(x)                         |
| 有且只有一个参数无返回值                   | x -> System.out.println(x)                           |
| 有多个参数，有返回值，有多条lambda体语句   | (x，y) -> {System.out.println(“xxx”);return xxxx;}； |
| 有多个参数，有返回值，只有一条lambda体语句 | (x，y) -> xxxx                                       |
|                                            |                                                      |


​	
口诀：左右遇一省括号，左侧推断类型省

注：当一个接口中存在多个抽象方法时，如果使用lambda表达式，并不能智能匹配对应的抽象方法，因此引入了函数式接口的概念

#### 函数式接口

函数式接口的提出是为了给Lambda表达式的使用提供更好的支持。

什么是函数式接口？ 
简单来说就是只定义了一个抽象方法的接口（Object类的public方法除外），就是函数式接口，并且还提供了注解：@FunctionalInterface

常见的四大函数式接口

Consumer<T> :消费型接口 ，有参无返回值
    void accept(T t);

```java

    public static void Consumertest(){
        changeStr("hello",(str) -> System.out.println(str));
    }

    /**
     *  Consumer<T> 消费型接口
     * @param str
     * @param con
     */
    public static void changeStr(String str, Consumer<String> con){
        con.accept(str);
    }

```

Supplier<T> :供给型接口   无参有返回值
    T get();

```java
	public static void Suppliertest() {
		String value = getValue(() -> "hello");
		System.out.println(value);
	}

	/**
	 * Supplier<T> 供给型接口
	 * 
	 * @param sup
	 * @return
	 */
	public static String getValue(Supplier<String> sup) {
		return sup.get();
	}
```

Function<T,R> :函数型接口，有参有返回值
    R apply(T t);

```java
	public static void Functiontest() {
		Long result = changeNum(100L, (x) -> x + 200L);
		System.out.println(result);
	}

	/**
	 * Function<T,R> 函数式接口
	 * 
	 * @param num
	 * @param fun
	 * @return
	 */
	public static Long changeNum(Long num, Function<Long, Long> fun) {
		return fun.apply(num);
	}
```

Predicate<T> :断言型接口   有参有返回值，返回值是boolean类型
    boolean test(T t);

```java
	/**
     * lambda配合函数式接口  Predicate接口
     * 对集合进行过滤，获取目标元素
     */
	public static void Predicatetest() {
		List<String> nameList = Arrays.asList("zhangsan", "lisi", "wangwu", "zhaoliu", "tianqi", "wangba");

		System.out.println("nameStrs which start with z");
		filter(nameList, (str) -> str.startsWith("z"));

		System.out.println("nameStrs which end with u ");
		filter(nameList, (str) -> str.endsWith("u"));

		System.out.println("print all language");
		filter(nameList, (str) -> true);

		System.out.println("print no language");
		filter(nameList, (str) -> false);

		System.out.println("Print language whose length greater than 4:");
		filter(nameList, (str) -> str.length() > 4);

	}

    public static void filter(List<String> names, Predicate<String> condition) {
        for(String name: names)  {
            if(condition.test(name)) {
                System.out.println(name + " ");
            }
        }
    }
    //使用 stream流的lambda
	public static void filter2(List<String> names, Predicate<String> condition) {
		names.stream().filter((name) -> condition.test(name)).forEach((name) -> {
			System.out.println(name + " ");
		});
	}
```

#### 方法引用

若lambda体中的内容有方法已经实现了，那么可以使用“方法引用” 
也可以理解为方法引用是lambda表达式的另外一种表现形式并且其语法比lambda表达式更加简单

(a) 方法引用 
三种表现形式： 
1. 对象：：实例方法名 
2. 类：：静态方法名 
3. 类：：实例方法名 （lambda参数列表中第一个参数是实例方法的调用 者，第二个参数是实例方法的参数时可用）

```java
public void test() {
        /**
        *注意：
        *   1.lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的函数列表和返回值               类型保持一致！
        *   2.若lambda参数列表中的第一个参数是实例方法的调用者，而第二个参数是实例方法的参数时，可               以使用ClassName::method
        *
        */
        Consumer<Integer> con = (x) -> System.out.println(x);
        con.accept(100);
// 方法引用-对象::实例方法
    Consumer<Integer> con2 = System.out::println;
    con2.accept(200);

    // 方法引用-类名::静态方法名
    BiFunction<Integer, Integer, Integer> biFun = (x, y) -> Integer.compare(x, y);
    BiFunction<Integer, Integer, Integer> biFun2 = Integer::compare;
    Integer result = biFun2.apply(100, 200);

    // 方法引用-类名::实例方法名
    BiFunction<String, String, Boolean> fun1 = (str1, str2) -> str1.equals(str2);
    BiFunction<String, String, Boolean> fun2 = String::equals;
    Boolean result2 = fun2.apply("hello", "world");
    System.out.println(result2);
}
```

(b)构造器引用 
格式：ClassName::new

```java
 public void test2() { 
    // 构造方法引用  类名::new
    Supplier<Employee> sup = () -> new Employee();
    System.out.println(sup.get());
    Supplier<Employee> sup2 = Employee::new;
    System.out.println(sup2.get());

    // 构造方法引用 类名::new （带一个参数）
    Function<Integer, Employee> fun = (x) -> new Employee(x);
    Function<Integer, Employee> fun2 = Employee::new;
    System.out.println(fun2.apply(100));
 }
```

(c)数组引用

格式：Type[]::new

```java
public void test(){
        // 数组引用
        Function<Integer, String[]> fun = (x) -> new String[x];
        Function<Integer, String[]> fun2 = String[]::new;
        String[] strArray = fun2.apply(10);
        Arrays.stream(strArray).forEach(System.out::println);
}


```


## Stream



[JDK 1.8 新特性之Stream 详解个人笔记](https://blog.csdn.net/chenhao_c_h/article/details/80691284)

https://blog.csdn.net/lidai352710967/article/details/82496783

首先对stream的操作可以分为两类，中间操作(intermediate operations)和结束操作(terminal operations):

中间操作总是会惰式执行，调用中间操作只会生成一个标记了该操作的新stream。
结束操作会触发实际计算，计算发生时会把所有中间操作积攒的操作以pipeline的方式执行，这样可以减少迭代次数。计算完成之后stream就会失效。
虽然大部分情况下stream是容器调用Collection.stream()方法得到的，但stream和collections有以下不同：

无存储。stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
为函数式编程而生。对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。
惰式执行。stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
可消费性。stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。

parallelStream 是并发流  会使用多线程并发的处理数据

有多种方式生成 Stream Source：

从 Collection 和数组

**Collection.stream()**

**Collection.parallelStream()**   

**Arrays.stream(T array) or Stream.of()**

从 BufferedReader

**java.io.BufferedReader.lines()**

静态工厂

**java.util.stream.IntStream.range()**

**java.nio.file.Files.walk()**

自己构建

**java.util.Spliterator**

其它

**Random.ints()**

**BitSet.stream()**

**Pattern.splitAsStream(java.lang.CharSequence)**

**JarFile.stream()**

如:

```java
		List<Integer> list=Arrays.asList(1,2,4,5,6);
		
		Stream<Integer> stream = list.stream();
		
		Arrays.stream(new String[]{"233","33"});
		
		Stream.of(1,2,4,5,6,3);
		
		Pattern compile = Pattern.compile(":");
		compile.splitAsStream("a:b:c");
```

#### Stream  方法使用

流(stream)的操作类型分为两种：

**Intermediate**：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。

**Terminal**：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。

在对于一个 Stream 进行多次转换操作 (Intermediate 操作)，每次都对 Stream 的每个元素进行转换，而且是执行多次，这样时间复杂度就是 N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 lazy 的，多个转换操作只会在 Terminal 操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream 里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal 操作的时候循环 Stream 对应的集合，然后对每个元素执行所有的函数。

还有一种操作被称为 **short-circuiting**。用以指：

对于一个 intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的 Stream，但返回一个有限的新 Stream。

对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。



需要注意的是，对于基本数值型，目前有三种对应的包装类型 Stream：

IntStream、LongStream、DoubleStream。当然我们也可以用 Stream<Integer>、Stream<Long> >、Stream<Double>，但是 boxing 和 unboxing 会很耗时，所以特别为这三种基本数值型提供了对应的 Stream。

数值流的构造

```java
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
IntStream.range(1, 3).forEach(System.out::println);
IntStream.rangeClosed(1, 3).forEach(System.out::println);
```


流转换为其它数据结构

```java
// 1. Array
String[] strArray1 = stream.toArray(String[]::new);
// 2. Collection
List<String> list1 = stream.collect(Collectors.toList());
List<String> list2 = stream.collect(Collectors.toCollection(ArrayList::new));
Set set1 = stream.collect(Collectors.toSet());
Stack stack1 = stream.collect(Collectors.toCollection(Stack::new));
// 3. String
String str = stream.collect(Collectors.joining()).toString();
```

一个 Stream 只可以使用一次，上面的代码只是示例，为了简洁而重复使用了数次(正常开发只能使用一次)。

Collectors 使用 : [JDK1.8新特性(五): Collectors](https://blog.csdn.net/vbirdbest/article/details/80216713)

流的操作
接下来，当把一个数据结构包装成 Stream 后，就要开始对里面的元素进行各类操作了。常见的操作可以归类如下。

**Intermediate**：

map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

**Terminal**：

forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

**Short-circuiting**：
anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit



##### map/flatMap

```java
//转换大写，wordList为单词集合List<String>类型
List<String> output = wordList.stream().map(String::toUpperCase).collect(Collectors.toList())
```

求平方数

```java
//这段代码生成一个整数 list 的平方数 {1, 4, 9, 16}。
List<Integer> nums = Arrays.asList(1, 2, 3, 4);
List<Integer> squareNums = nums.stream().map(n -> n * n).collect(Collectors.toList());
```


从上面例子可以看出，map 生成的是个 1:1 映射，每个输入元素，都按照规则转换成为另外一个元素。还有一些场景，是一对多映射关系的，这时需要 flatMap。



```java
//将最底层元素抽出来放到一起，最终 output 的新 Stream 里面已经没有 List 了，都是直接的数字。
Stream<List<Integer>> inputStream = Stream.of(
 Arrays.asList(1),
 Arrays.asList(2, 3),
 Arrays.asList(4, 5, 6)
 );
Stream<Integer> outputStream = inputStream.flatMap((childList) -> childList.stream());
List<Integer> list =outputStream.collect(Collectors.toList());
System.out.println(list.toString());
```

##### filter

filter 对原始 Stream 进行某项测试，通过测试的元素被留下来生成一个新 Stream。

```java
//留下偶数，经过条件“被 2 整除”的 filter，剩下的数字为 {2, 4, 6}。
Integer[] sixNums = {1, 2, 3, 4, 5, 6};
Integer[] evens =Stream.of(sixNums).filter(n -> n%2 == 0).toArray(Integer[]::new);
//把每行的单词用 flatMap 整理到新的 Stream，然后保留长度不为 0 的，就是整篇文章中的全部单词了。
//REGEXP为正则表达式，具体逻辑具体分析
List<String> output = reader.lines().
    flatMap(line -> Stream.of(line.split(REGEXP))).
    filter(word -> word.length() > 0).collect(Collectors.toList());
```



##### forEach

forEach 方法接收一个 Lambda 表达式，然后在 Stream 的每一个元素上执行该表达式。

```java
//打印所有男性姓名，roster为person集合类型为List<Pserson>
// Java 8
roster.stream().filter(p -> p.getGender() == Person.Sex.MALE).forEach(p -> System.out.println(p.getName()));
// Pre-Java 8
for (Person p : roster) {
    if (p.getGender() == Person.Sex.MALE) {
        System.out.println(p.getName());
    }
}
```


当需要为多核系统优化时，可以 parallelStream().forEach()，只是此时原有元素的次序没法保证，并行的情况下将改变串行时操作的行为，此时 forEach 本身的实现不需要调整，而 Java8 以前的 for 循环 code 可能需要加入额外的多线程逻辑。

另外一点需要注意，forEach 是 terminal 操作，因此它执行后，Stream 的元素就被“消费”掉了，你无法对一个 Stream 进行两次 terminal 运算。下面代码是错误的

```java
//错误代码示例，一个stream不可以使用两次
stream.forEach(element -> doOneThing(element));
stream.forEach(element -> doAnotherThing(element));
```

##### peek

相反，具有相似功能的 intermediate 操作 peek 可以达到上述目的。如下是出现在该 api javadoc 上的一个示例。

```java
// peek 对每个元素执行操作并返回一个新的 Stream
Stream.of("one", "two", "three", "four")
     .filter(e -> e.length() > 3)
     .peek(e -> System.out.println("Filtered value: " + e))
     .map(String::toUpperCase)
     .peek(e -> System.out.println("Mapped value: " + e))
     .collect(Collectors.toList());
```

##### findFirst

这是一个 termimal 兼 short-circuiting 操作，它总是返回 Stream 的第一个元素，或者空。

这里比较重点的是它的返回值类型：Optional。这也是一个模仿 Scala 语言中的概念，作为一个容器，它可能含有某值，或者不包含。使用它的目的是尽可能避免 NullPointerException。

```java
String strA = " abcd ", strB = null;
print(strA);
print("");
print(strB);
getLength(strA);
getLength("");
getLength(strB);
//输出text不为null的值
public static void print(String text) {
    // Java 8
     Optional.ofNullable(text).ifPresent(System.out::println);
    // Pre-Java 8
     if (text != null) {
        System.out.println(text);
     }
 }
//输出text的长度，避免空指针
public static int getLength(String text) {
    // Java 8
    return Optional.ofNullable(text).map(String::length).orElse(-1);
    // Pre-Java 8
    //return if (text != null) ? text.length() : -1;
}
```


在更复杂的 if (xx != null) 的情况中，使用 Optional 代码的可读性更好，而且它提供的是编译时检查，能极大的降低 NPE 这种 Runtime Exception 对程序的影响，或者迫使程序员更早的在编码阶段处理空值问题，而不是留到运行时再发现和调试。

Stream 中的 findAny、max/min、reduce 等方法等返回 Optional 值。还有例如 IntStream.average() 返回 OptionalDouble 等等

##### reduce

这个方法的主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。从这个意义上说，字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce。也有没有起始值的情况，这时会把 Stream 的前面两个元素组合起来，返回的是 Optional。

```java
// reduce用例
// 字符串连接，concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat); 
// 求最小值，minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min); 
// 求和，sumValue = 10, 有起始值
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// 求和，sumValue = 10, 无起始值，返回Optional,所以有get()方法
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// 过滤，字符串连接，concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F")
    .filter(x -> x.compareTo("Z") > 0)
    .reduce("", String::concat);
```



##### limit/skip

limit 返回 Stream 的前面 n 个元素；skip 则是扔掉前 n 个元素（它是由一个叫 subStream 的方法改名而来）。

```java
// limit 和 skip 对运行次数的影响
public void testLimitAndSkip() {
     List<Person> persons = new ArrayList();
     for (int i = 1; i <= 10000; i++) {
         Person person = new Person(i, "name" + i);
         persons.add(person);
    }
    List<String> personList2 = persons.stream()
        .map(Person::getName).limit(10).skip(3)
        .collect(Collectors.toList());
System.out.println(personList2);
}
private class Person {
    public int no;
    private String name;
    public Person (int no, String name) {
        this.no = no;
        this.name = name;
    }
     public String getName() {
        System.out.println(name);
        return name;
     }
}
//结果
name1
name2
name3
name4
name5
name6
name7
name8
name9
name10
[name4, name5, name6, name7, name8, name9, name10]
//这是一个有 10，000 个元素的 Stream，但在 short-circuiting 操作 limit 和 skip 的作用下，管道中 map 操作指定的 getName() 方法的执行次数为 limit 所限定的 10 次，而最终返回结果在跳过前 3 个元素后只有后面 7 个返回。
```


有一种情况是 limit/skip 无法达到 short-circuiting 目的的，就是把它们放在 Stream 的排序操作后，原因跟 sorted 这个 intermediate 操作有关：此时系统并不知道 Stream 排序后的次序如何，所以 sorted 中的操作看上去就像完全没有被 limit 或者 skip 一样。

```java
 List<Person> persons = new ArrayList();
 for (int i = 1; i <= 5; i++) {
     Person person = new Person(i, "name" + i);
     persons.add(person);
 }
List<Person> personList2 = persons.stream().sorted((p1, p2) -> 
p1.getName().compareTo(p2.getName())).limit(2).collect(Collectors.toList());
System.out.println(personList2);
//结果
name2
name1
name3
name2
name4
name3
name5
name4
[stream.StreamDW$Person@816f27d, stream.StreamDW$Person@87aac27]
//虽然最后的返回元素数量是 2，但整个管道中的 sorted 表达式执行次数没有像前面例子相应减少。
```

##### sorted

对 Stream 的排序通过 sorted 进行，它比数组的排序更强之处在于你可以首先对 Stream 进行各类 map、filter、limit、skip 甚至 distinct 来减少元素数量后，再排序，这能帮助程序明显缩短执行时间。

```java
List<Person> persons = new ArrayList();
 for (int i = 1; i <= 5; i++) {
     Person person = new Person(i, "name" + i);
     persons.add(person);
 }
List<Person> personList2 = persons.stream().limit(2).sorted((p1, p2) -> p1.getName().compareTo(p2.getName())).collect(Collectors.toList());
System.out.println(personList2);
//结果
name2
name1
[stream.StreamDW$Person@6ce253f1, stream.StreamDW$Person@53d8d10a]
```

##### min/max/distinct

min 和 max 的功能也可以通过对 Stream 元素先排序，再 findFirst 来实现，但前者的性能会更好，为 O(n)，而 sorted 的成本是 O(n log n)。同时它们作为特殊的 reduce 方法被独立出来也是因为求最大最小值是很常见的操作。

```java
//找出最长一行的长度
BufferedReader br = new BufferedReader(new FileReader("c:\\Service.log"));
int longest = br.lines().mapToInt(String::length).max().getAsInt();
br.close();
System.out.println(longest);
//找出全文的单词，转小写，并排序,使用 distinct 来找出不重复的单词。单词间只有空格
List<String> words = br.lines()
    .flatMap(line -> Stream.of(line.split(" ")))
    .filter(word -> word.length() > 0)
    .map(String::toLowerCase)
    .distinct().sorted()
    .collect(Collectors.toList());
br.close();
System.out.println(words);
```



##### Match

Stream 有三个 match 方法，从语义上说：

allMatch：Stream 中全部元素符合传入的 predicate，返回 true

anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true

noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

它们都不是要遍历全部元素才能返回结果。例如 allMatch 只要一个元素不满足条件，就 skip 剩下的所有元素，返回 false。

```java
//Match使用示例
List<Person> persons = new ArrayList();
persons.add(new Person(1, "name" + 1, 10));
persons.add(new Person(2, "name" + 2, 21));
persons.add(new Person(3, "name" + 3, 34));
persons.add(new Person(4, "name" + 4, 6));
persons.add(new Person(5, "name" + 5, 55));
boolean isAllAdult = persons.stream().allMatch(p -> p.getAge() > 18);
System.out.println("All are adult? " + isAllAdult);
boolean isThereAnyChild = persons.stream().anyMatch(p -> p.getAge() < 12);
System.out.println("Any child? " + isThereAnyChild);
//结果
All are adult? false
Any child? true
```

##### generate

通过实现 Supplier 接口，你可以自己来控制流的生成。这种情形通常用于随机数、常量的 Stream，或者需要前后元素间维持着某种状态信息的 Stream。把 Supplier 实例传递给 Stream.generate() 生成的 Stream，默认是串行（相对 parallel 而言）但无序的（相对 ordered 而言）。由于它是无限的，在管道中，必须利用 limit 之类的操作限制 Stream 大小。

```java
//生成10个随机数
Random seed = new Random();
Supplier<Integer> random = seed::nextInt;
Stream.generate(random).limit(10).forEach(System.out::println);
//Another way
IntStream.generate(() -> (int) (System.nanoTime() % 100)).
limit(10).forEach(System.out::println);
```


Stream.generate() 还接受自己实现的 Supplier。例如在构造海量测试数据的时候，用某种自动的规则给每一个变量赋值；或者依据公式计算 Stream 的每个元素值。这些都是维持状态信息的情形。

```java
Stream.generate(new PersonSupplier()).
    limit(10).forEach(p -> System.out.println(p.getName() + ", " + p.getAge()));
private class PersonSupplier implements Supplier<Person> {
     private int index = 0;
     private Random random = new Random();
     @Override
     public Person get() {
        return new Person(index++, "StormTestUser" + index, random.nextInt(100));
     }
}
//结果
StormTestUser1, 9
StormTestUser2, 12
StormTestUser3, 88
StormTestUser4, 51
StormTestUser5, 22
StormTestUser6, 28
StormTestUser7, 81
StormTestUser8, 51
StormTestUser9, 4
StormTestUser10, 76
```

##### iterate

iterate 跟 reduce 操作很像，接受一个种子值，和一个 UnaryOperator（例如 f）。然后种子值成为 Stream 的第一个元素，f(seed) 为第二个，f(f(seed)) 第三个，以此类推

```java
//生成等差数列
Stream.iterate(0, n -> n + 3).limit(10). forEach(x -> System.out.print(x + " "));
//结果
0 3 6 9 12 15 18 21 24 27
```


Stream.generate 相仿，在 iterate 时候管道必须有 limit 这样的操作来限制 Stream 大小。

用 Collectors 来进行 reduction 操作
java.util.stream.Collectors 类的主要作用就是辅助进行各类有用的 reduction 操作，例如转变输出为 Collection，把 Stream 元素进行归组。

##### groupingBy/partitioningBy

```java
//按照年龄归组
Map<Integer, List<Person>> personGroups = Stream.generate(new PersonSupplier())
    .limit(100).collect(Collectors.groupingBy(Person::getAge));
Iterator it = personGroups.entrySet().iterator();
while (it.hasNext()) {
     Map.Entry<Integer, List<Person>> persons = (Map.Entry) it.next();
     System.out.println("Age " + persons.getKey() + " = " + persons.getValue().size());
}
//上面的 code，首先生成 100 人的信息，然后按照年龄归组，相同年龄的人放到同一个 list 中，如下的输出：
Age 0 = 2
Age 1 = 2
Age 5 = 2
Age 8 = 1
Age 9 = 1
Age 11 = 2
……
Map<Boolean, List<Person>> children = Stream.generate(new PersonSupplier())
    .limit(100).collect(Collectors.partitioningBy(p -> p.getAge() < 18));
System.out.println("Children number: " + children.get(true).size());
System.out.println("Adult number: " + children.get(false).size());
//结果
Children number: 23 
Adult number: 77
```

##### Count 

计数是一个最终操作，返回Stream中元素的个数，返回值类型是long。

```java
long startsWithB = 
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();
System.out.println(startsWithB);    // 3
```



##### Stream 特性归纳

1. 不是数据结构

2. 它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据。

3. 它也绝不修改自己所封装的底层数据结构的数据。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素。

4. 所有 Stream 的操作必须以 lambda 表达式为参数

5. 不支持索引访问

6. 你可以请求第一个元素，但无法请求第二个，第三个，或最后一个。不过请参阅下一项。

7. 很容易生成数组或者 List

8. 惰性化

9. 很多 Stream 操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始。

10. Intermediate 操作永远是惰性化的。

11. 并行能力

12. 当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的。

13. 可以是无限的

14. 集合有固定大小，Stream 则不必。limit(n) 和 findFirst() 这类的 short-circuiting 操作可以对无限的 Stream 进行运算并很快完成。



## Collectors 

[JDK1.8新特性(五): Collectors](https://blog.csdn.net/vbirdbest/article/details/80216713)

Stream中有两个个方法collect和collectingAndThen用于对流中的数据进行处理，可以对流中的数据进行聚合操作，如：

1. 将流中的数据转成集合类型: toList、toSet、toMap、toCollection
2. 将流中的数据(字符串)使用分隔符拼接在一起：joining
3. 对流中的数据求最大值maxBy、最小值minBy、求和summingInt、求平均值averagingDouble
4. 对流中的数据进行映射处理 mapping
5. 对流中的数据分组：groupingBy、partitioningBy
6. 对流中的数据累计计算：reducing





## 接口的静态方法和默认方法

[Java 8 新特性：接口的静态方法和默认方法](https://blog.csdn.net/sun_promise/article/details/51220518)



## Optional

[JDK1.8新特性(三): 方法引用 ::和Optional](https://blog.csdn.net/vbirdbest/article/details/80207673)

[JDK1.8新特性值Optional](https://blog.csdn.net/zknxx/article/details/78586799)



## Map和日期

[JDK1.8新特性(六): Map和日期](https://blog.csdn.net/vbirdbest/article/details/80268231)



JDK1.8中Map新增了一些方法，其中一部分方法是为了简化代码的，如forEach，另外一些方法是为了防止null，使操作代码更加严谨。

```java
public interface Map<K,V> {
    // 如果key存在，则忽略put操作
    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }

    // 循环
    public void forEach(BiConsumer<? super K, ? super V> action){...}

    // 如果存在则计算：先判断key是否存在，如果key存在，将BiFunction计算的结果作为key的新值重新put进去
    public V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {}

    // 如果key不存在则将计算的结果put进去
    public V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {}

    // 只有key-value同时满足条件时才会删除
    public boolean remove(Object key, Object value){}

    // 如果key不存在，则返回默认值
    public V getOrDefault(Object key, V defaultValue) {}

    // 合并：将BiFunction的结果作为key的新值
    public V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {}
```

示例

```java
@Test
public void putIfAbsent() {
    Map<String, String> map = new HashMap<>();
    map.putIfAbsent("key", "oldValue");
    // 如果key存在，则忽略put操作
    map.putIfAbsent("key", "newValue");
    String value = map.get("key");
    System.out.println(value);
}

@Test
public void forEach() {
    Map<String, String> map = new HashMap<>();
    map.putIfAbsent("key1", "value1");
    map.putIfAbsent("key2", "value1");
    map.putIfAbsent("key3", "value1");

    map.forEach((key, value) -> System.out.println(key + ":" + value));
}

@Test
public void computeIfPresent() {
    Map<String, String> map = new HashMap<>();
    map.putIfAbsent("key1", "value1");

    // 如果存在则计算：先判断key是否存在，如果key存在，将BiFunction计算的结果作为key的新值重新put进去
    map.computeIfPresent("key1", (key, value) -> key + "=" + value);
    String value = map.get("key1");
    System.out.println(value);

    // 如果计算的结果为null，相当于从map中移除
    map.computeIfPresent("key1", (k, v) -> null);
    boolean contain = map.containsKey("key1");
    System.out.println(contain);
}

@Test
public void computeIfAbsent() {
    // 如果key不存在则将计算的结果put进去
    Map<String, String> map = new HashMap<>();
    map.computeIfAbsent("key2", v -> "value2");
    boolean contain = map.containsKey("key2");
    System.out.println(contain);
}

@Test
public void remove(){
    Map<String, String> map = new HashMap<>();
    map.putIfAbsent("key1", "value1");
    boolean result = map.remove("key1", "value2");
    System.out.println(result);
}

@Test
public void getOrDefault(){
    Map<String, String> map = new HashMap<>();
    String value = map.getOrDefault("key1", "default value");
    System.out.println(value);
}

@Test
public void merge(){
    Map<String, String> map = new HashMap<>();
    map.put("key1", "value1");

    map.merge("key1", "newValue", (value, newValue) -> value + "-" + newValue);
    String value = map.get("key1");
    // value1-newValue
    System.out.println(value);
}
```



算法的 修改详见数据结构的笔记

日期

Java 8 在包java.time下包含了一组全新的时间日期API。新的日期API和开源的Joda-Time库差不多，但又不完全一样。

#### Clock 时钟

Clock类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代 System.currentTimeMillis() 来获取当前的微秒数。某一个特定的时间点也可以使用Instant类来表示，Instant类也可以用来创建老的java.util.Date对象。

```java
@Test
public void clock(){
    Clock clock = Clock.systemDefaultZone();
    long millis = clock.millis();
    Instant instant = clock.instant();
    Date date = Date.from(instant);
System.out.println(millis);
System.out.println(date);
}
```
#### Timezones 时区

在新API中时区使用ZoneId来表示。时区可以很方便的使用静态方法of来获取到。 时区定义了到UTS时间的时间差，在Instant时间点对象到本地日期对象之间转换的时候是极其重要的。



```java
@Test
public void timezones(){
    // 获取所有可用的时区
    Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
    System.out.println(availableZoneIds);
// 获取默认的时区 Asia/Shanghai
ZoneId zoneId = ZoneId.systemDefault();
System.out.println(zoneId);

// 获取时区规则 ZoneRules[currentStandardOffset=+08:00]
ZoneRules rules = zoneId.getRules();
System.out.println(rules);
}
```
#### LocalTime 本地时间

LocalTime 定义了一个没有时区信息的时间，例如 晚上10点，或者 17:30:15。

```java
@Test
public void localTime(){
    ZoneId zoneId = ZoneId.systemDefault();
    ZoneId zoneId2 = ZoneId.of("Etc/GMT+8");
// 获取指定时区的当前时间
LocalTime now = LocalTime.now(zoneId);
LocalTime now2 = LocalTime.now(zoneId2);

// 判断一个本地时间是否在另一个本地时间之前
boolean isBefore = now.isBefore(now2);

// 获取两个本地时间小时之差
long hours = ChronoUnit.HOURS.between(now, now2);
System.out.println(hours);

// 获取两个本地时间分钟之差
long minutes = ChronoUnit.MINUTES.between(now, now2);
System.out.println(minutes);

LocalTime localTime = LocalTime.of(23, 59, 59);
System.out.println(localTime);       // 23:59:59

DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT).withLocale(Locale.GERMAN);
LocalTime local = LocalTime.parse("13:37", formatter);
System.out.println(local);  // 13:37
}
```
#### LocalDate 本地日期

LocalDate 表示了一个确切的日期，比如 2014-03-11。该对象值是不可变的，用起来和LocalTime基本一致。


```java
@Test
public void localDate(){
    LocalDate today = LocalDate.now();
    LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
    LocalDate yesterday = tomorrow.minusDays(2);
System.out.println(today + "," + tomorrow + "," + yesterday);
LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println(dayOfWeek);
DateTimeFormatter germanFormatter = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM)
                .withLocale(Locale.GERMAN);
LocalDate localDate = LocalDate.parse("24.12.2014", germanFormatter);
System.out.println(localDate);   // 2014-12-24
}
```


#### LocalDateTime 本地日期时间

LocalDateTime 同时表示了时间和日期，相当于前两节内容合并到一个对象上了。LocalDateTime和LocalTime还有LocalDate一样，都是不可变的。LocalDateTime提供了一些能访问具体字段的方法。



```java
@Test
public void localDateTime(){
    LocalDateTime localDateTime = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);
    DayOfWeek dayOfWeek = localDateTime.getDayOfWeek();
    System.out.println(dayOfWeek);
Month month = localDateTime.getMonth();
System.out.println(month);

long minuteOfDay = localDateTime.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);

Instant instant = localDateTime.atZone(ZoneId.systemDefault()).toInstant();
Date date = Date.from(instant);
System.out.println(date);

// DateTimeFormatter是不可变的，所以它是线程安全的
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime parsed = LocalDateTime.parse("2018-05-07 07:13:00", formatter);
String string = formatter.format(parsed);
System.out.println(string);
}
```



