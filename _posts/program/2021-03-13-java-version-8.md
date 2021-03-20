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

**排序**

|函数|参数|说明|
|-|-|-|
|store|-|-|自然排序|
|store(Comparator<T> c)|int compare(T t1, T t2)|定制排序|


**筛选和切片**

|函数|参数|说明|
|-|-|-|
|filter|boolean test(T t)|过滤元素|
|limit|获取数量|截断流, 不需要迭代所有集合|
|skip|跳过数量|跳过指定数量获取元素|
|distince|-|通过元素的 hashCode 和 equals 去除重复元素|

### 终止操作

|函数|参数|说明|
|-|-|-|
|allMatch|boolean test(T t)|检查是否匹配所有元素|
|anyMatch|boolean test(T t)|检查是否至少匹配一个元素|
|noneMatch|boolean test(T t)|检查是否没有匹配所有元素|
|findFirst|-|返回第一个元素|
|findAny|-|返回任意元素|
|count|-|返回元素总数|
|max(Comparator<T> c)|int compare(T t1, T t2)|返回最大值|
|min(Comparator<T> c)|int compare(T t1, T t2)|返回最小值|


**归约**

|函数|参数|说明|
|-|-|-|
|reduce(T identity, BinaryOperator)||将流中元素反复结合得到一个值|
|reduce(BinaryOperator)|-|将流中元素反复结合得到一个值|

```java
/**
 * 采用 reduce(T identity, BinaryOperator operator)
 * identity: 表示起始值
 * operator: 表示递归操作
 *  _x: 第一次就是起始值(identity), 之后就是每次`(_x, _y) -> _x + _y`的结果
 *  _y: 流循环中每次下标递增的新值
 *
 *  例如:
 *  流数据: 1, 2, 3
 *  起始值: 0
 *  下面计算流程:
 *  0 + 1 = 1
 *  1 + 2 = 3
 *  3 + 3 = 6
 *  结果返回: 6
 */
Stream.of(1, 2, 3).reduce(0, (_x, _y) -> _x + _y);
```
**收集**

|函数|参数|说明|
|-|-|-|
|collect|Collector<T, A, R>|流中元素汇总|

> Collectors收集器工具类

### 并行流

工作窃取模式

#### 传统方式

定义并行任务类

```java
/**
 * 1. 继承RecursiveTask<T>;
 * 2. 定义拆分规则;
 */
class ForkJoinCalculate extends RecursiveTask<Long> {

  // 定义拆分条件
  private final Long THRESHOLD = 1000L;

  // 定义开始行
  private Long begin;

  // 定义结束行
  private Long end;

  public ForkJoinCalculate(Long begin, Long end) {
      this.begin = begin;
      this.end = end;
  }

  @Override
  protected Long compute() {
      // 获取当前处理行数
      Long length = end - begin;

      if (length <= THRESHOLD) {
          // 小于拆分条件, 不在拆分, 进行处理。
          Long sum = 0L;

          for (Long i = 0l; begin <= end; i++) {
              sum += i;
          }

          // 返回结果
          return sum;
      } else {
          // 大于拆分条件, 继续拆分。总行的一般, 分担任务。
          Long middle = (begin + end) / 2;

          // 拆分子任务, 压入线程队列
          ForkJoinCalculate leftFork = new ForkJoinCalculate(begin, middle);
          leftFork.fork();


          ForkJoinCalculate rightFork = new ForkJoinCalculate(middle + 1, end);
          rightFork.fork();

          // 合并左右结果
          return leftFork.join() + rightFork.join();
      }

  }
}
```

执行并行任务

```java
// 定义任务
ForkJoinCalculate forkJoinTask = new ForkJoinCalculate(0L, 1000000L);

// 创建线程池执行任务
ForkJoinPool pool = new ForkJoinPool();
Long sum = pool.invoke(forkJoinTask);
```

#### Java8 并行流

Stream API 可以通过 parallel(并行流) 与 sequential(顺序流) 切换并行流与顺序流

```java
LongStream.rangClosed(0, 1000000L).parallel().reduce(0, Long::sum);
```

## Optional

是一个容器类, 代表一个值存在或不存在。可以有效避免控制异常。

|函数|参数|说明|
|-|-|-|
|of(T t)|T|创建实例|
|empty|-|创建空实例|
|ofNullable(T t)|t不为null, 创建实例, 否则创建空实例|
|isPresent|-|判断是否包含值|
|orElse(T t)|T|如果调用实例包含值, 返回该值。否则返回t|
|orElseGet(Supplier s)|Supplier|如果调用实例包含值, 返回该值。否则返回s获取值|
|map(Function f)|Function|如果有值返回实例, 否则返回空实例|
|flatMap(Function f)|Function|同map|

```java

Optional<Demo> optional = Optional.ofNullable(new Demo("名称", "地址"));

// 存在值正常返回
Demo demo = optional.orElse(new Demo());
// 利用 Supplier 创建对象
Demo demo = optional.orElseGet(() -> new Demo());

// 处理数据获取结果
Optional<String> title = optional.map((_d) -> d.getTitle());
// 需要在包装一层 Optional
Optional<String> title = optional.flatMap((_d) -> Optional.of(d.getTitle()));
```

## 接口默认方法与静态方法

```java
public interface DefaultInterface {
  // 接口默认方法
  default String getTitle() {
    return "标题";
  }

  // 接口静态方法
  static void show() {
    System.out.println("接口静态方法");
  }
}
```

## 新时间日期

* LocalDate 日期

* LocalTime 时间

* LocalDateTime 日期时间

```java
// 指定时区获取时间
LocalDateTime ldt = LocalDateTime.now(ZoneId.of("Asia/Shanghai"));
```

* Instant 时间戳

```java
// 默认获取 UTC 时区
Instant ins = Instant.now();
// 获取时间戳(毫秒)
ins.toEpochMilli();
// 北京时间
OffsetDataTime newIns = ins.atOffset(ZoneOffset.ofHours(8));

// 计算时间间隔
Duration duration = Duration.between(Instant.now(), Instant.now());
Duration duration = Duration.between(LocalTime.now(), LocalTime.now());

// 计算日期间隔
Period period = Period.between(LocalDate.now(), LocalDate.now());
```

### TemporalAdjuster 时间校正器 & TemporalAdjusters 支撑工具类

```java
LocalDateTime ldt = LocalDateTime.now();

// 获取下一个周日
LocalDateTime nextDay = ldt.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));
```

### DateTimeFormatter 格式化时间&日期

```java
// 定义日期格式
DateTimeFormatter dft = DateTimeFormatter.ofPattern("yyyy-MM-dd");

// 日期时间格式化
String dataString = LocalDateTime.now().format(dft);
// 或
String dataString = dft.format(LocalDateTime.now());

// 转换
LocalDateTime newDateTime = LocalDateTime.now();
LocalDateTime transferDateTime = newDateTime.parse("2021-01-01", dft);

// 日期转换
LocalDate.parse("20161218", dft);
```