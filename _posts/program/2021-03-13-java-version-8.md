---
comment: false
aside:
  toc: true

title: Java8 新特新
date: 2021-03-13 10:00
tags: Java
---

## Lambda 表达式

### 语法

通过箭头(->)操作符, 将内容分为左右两个部分。左边为参数, 右边为执行代码块。

> Lambda 使用类型推断。表达式的参数类型可以省略不写, 因为JVM编译器会通过上下文推断数据类型。

### 样例

```java
// 无参数, 无返回值
Runnable runnable = () -> System.out.println("Hello Lambda!");

// 有参数, 无返回值
Consumer<String> consumer = (_x) -> System.out.println(_x);
// 一个参数, 参数括号可以默认不写
Consumer<String> consumer = _x -> System.out.println(_x);

// 有参数, 有返回值
Comparable<Integer> comparable = (x, y) -> Integer.compare(x, y);
// 存在多条语句
Comparable<Integer> comparable = (x, y) -> {
  System.out.println("多条执行行");
  return Integer.compare(x, y);
};
```

### 函数式接口

> 接口中只有一个抽象方法的接口。用 @FunctionalInterface 注解修饰。Lambda方式成为除匿名内部类的另一种实现方式。

```java
// 定义一个函数式接口
@FunctionalInterface
public interface FunctionInterface<T> {

  void handler(T object);

}

// 函数式接口使用
FunctionInterface<String> interface = (_object) -> System.out.println("输出内容:" + String.valueOf(_object));
```

#### Java8 函数接口

|接口|函数|内容|
|-|-|-|
|Consumer<T>|void accept(T t)|消费接口|
|Supplier<T>|T get()|供给型接口|
|Function<T, R>|R apply(T t)|函数型接口|
|Predicate<T>|boolean test(T t)|断言型接口|

**扩展接口**

|接口|函数|内容|
|-|-|-|
|BiFunction<T, U, R>|R apply(T t, U u)|-|
|UnaryOperator<T>|T apply(T t)|Function子接口|
|BinaryOperator<T>|T apply(T t1, T t2)|Function子接口|
|BiConsumer<T, U>|void accept(T t, U u)|-|

##### Consumer<T>

> 消费型接口, 有去无回(有参无返回)

```java
// 自定一个调用消费接口的函数
public void log(String log, Consumer<String> consumer) {
  consumer.accept(log);
}

// 调用
public void callMethod() {
  log("这是演示消费接口输出", (_log) -> System.out.println(_log));
}
```

##### Supplier<T>

> 供给型接口, 无限索取(无参有返回)

```java
// 自定一个调用供给接口的函数
public String make(Supplier<String> supplier) {
  return supplier.get();
}

// 调用
public void callMethod() {
  make(() -> String.valueOf(Math.random()));
}
```

##### Function<T, R>

> 函数型接口, 有来有往(有参有返回)

```java
// 自定调用函数接口的函数
public String handler(String origin, Function<String, String> fun) {
  return fun.apply(origin);
}

// 调用
public void callMethod() {
  String target = handler("需要处理的字符串", (_origin) -> {
    return _origin.replaceAll("需要", "已经");
  });
  System.out.println(target)); // 已经处理的字符串
}
```

##### Predicate<T>

> 断言型接口

```java
// 定义调用断言接口的函数
public List<Integer> filter(List<Integer> originArray, Predicate<Integer> pre) {
  List<Integer> resultArray = new ArrayList<>();

  for (Integer origin: originArray) {
    if (pre.test(origin)) {
      resultArray.add(origin);
    }
  }

  return resultArray;
}

// 调用
public void callMethod() {
  List<Integer> originArray = Arrays.asList(3, 6, 9, 13, 15, 18);

  List<Integer> resultArray = filter(originArray, (_origin) -> {
    return _origin.compareTo(10) >= 0 ? true : false;
  });
  System.out.println(resultArray));
}
```

### 方法引用

Lambda 体中的实例方法已经实现, 可以使用`方法引用`{:.success}。

> 调用方法的参数和返回值类型, 要与函数式接口中抽象的方法参数和返回值一直。

#### 语法

* `对象`{:.info}::`实例方法名`{:.info}

```java
// 样例一, 改造前
Consumer<String> consumer = (_x) -> System.out.println(_x);
// 改造后, 改用方法引用方式
Consumer<String> consumer = System.out::println;

consumer.accept("输出内容");

// 样例二
Employee employee = new Employee();
// 改造前
Supplier<String> supplier = () -> emp.getName();
// 改造后
Supplier<String> supplier = emp::getName;

String name = supplier.get();
```

* `类`{:.info}::`静态方法名`{:.info}

```java
Comparator<Integer> comparator = (_x, _y) -> Integer.compare(_x, _y);

Comparator<Integer> comparator = Integer::compare;

comparator.compare(6, 9);
```

* `类`{:.info}::`实例方法名`{:.info}

```java
BiPredicate<String, String> bi = (_x, _y) -> _x.equals(_y);

// 写法有固定规则。 _x 作为调用对象, _y 作为比较对象
BiPredicate<String, String> bi = String::equals;

bi.test("x", "y")
```

* 构造器引用

ClassName::new

```java
String::new
```

* 数组引用

Type::new

```java
Integer[]::new
```

## Stream

* 创建流

* 中间操作

* 终止操作(终端操作)


### 创建流

* 通过 Collection 集合提供 `stream` 或 `parallelStream`

```java
// 第一种方式
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream();

// 第二种方式
String[] array = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);

// 第三种方式
Strema<String> stream = Stream.of("a", "b", "c");

// 第四种方式, 无限流
// 迭代
Strema<Integer> stream = Stream.iterate(0, (_x) -> _x + 2);
// 生成
Strema<Integer> stream = Stream.generate(() -> Marh.random());
```

### 中间操作

中间操作可以连接起来形成一个流水线, 除非流水线上触发终止操作, 否则中间操作不会执行。在执行终止操作时一次处理, 又称`惰性求值`。

**映射**

|函数|参数|说明|
|-|-|-|
|map|R apply(T t)|将元素转换成其他形式或提取信息|
|flatMap|R apply(Stream<T> stream)|将流中的每一个值转换成另一个流, 再把所有流连成一个流|

**筛选和切片**

|函数|参数|说明|
|-|-|-|
|filter|boolean test(T t)|过滤元素|
|limit|获取数量|截断流, 不需要迭代所有集合|
|skip|跳过数量|跳过指定数量获取元素|
|distince|-|通过元素的 hashCode 和 equals 去除重复元素|

