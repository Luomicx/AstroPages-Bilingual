---
title: Java学习 - Java8 新特性
pubDatetime: 2025-08-05T22:20:40Z
description: Java8 新特性学习笔记
featured: false
draft: false
tags:
  - Java
---
# Java8 新特性

之前一直知道有 Java8 的新特性，包括 lambda 表达式，函数式接口，Stream API，Optional 处理的容器。这些内容有的使用过，但是都是一知半解，这里进行系统的学习一下。

## Lambda 表达式 (Lambda Expressions)

这是 Java 8 最具标志性的新特性。

- **是什么？** Lambda 表达式，可以理解为 **一段可以被传递的代码**，或者说是一个 **匿名函数**。它允许你将一个函数作为方法的参数进行传递。

- **为什么需要它？** 在 Java 8 之前，如果你想传递一段代码逻辑（比如一个排序规则），你必须创建一个实现了特定接口的匿名内部类的实例。代码非常冗长和笨重。Lambda 表达式极大地简化了这种写法。

- **怎么用？** Lambda 表达式的核心语法是 `(parameters) -> expression` 或 `(parameters) -> { statements; }`。

  - `parameters`: 方法的参数列表。
  - `->`: Lambda 操作符，读作 "goes to"。
  - `expression` 或 `statements`: 方法体。

  **【代码对比】** 让我们来看一个最经典的例子：给一个列表排序。

  **Java 8 之前 (使用匿名内部类):**

  ```java
  List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
  
  Collections.sort(names, new Comparator<String>() {
      @Override
      public int compare(String a, String b) {
          return a.compareTo(b);
      }
  });
  
  System.out.println(names); // [anna, mike, peter, xenia]
  ```

  这里我们为了传递 `a.compareTo(b)` 写了 **5** 行代码，使用 **lambda 表达式之后** 的代码是这样的：

  ```java
  List<String> names = Arrays.asList("John", "Danny", "周杰伦", "陈奕迅");
  
  //1.最完整的 Lambda 写法
  Collections.sort(names, (String a, String b) -> {
      return a.compareTo(b);
  });
  
  // 2. 类型推断，编译器是知道 a 和 b 是 String 类型，可以省略
  Collections.sort(names, (a, b) -> {
      return a.compareTo(b);
  });
  
  // 3. 单行方法体： 如果方法只有一行 可以省略大括号 {} 和 return
  Collections.sort(names, (a, b) -> a.compareTo(b));
  
  System.out.println(names);
  ```

  这样的话，我们就从 5 行代码优化到了 1 行代码，这个时候我们的代码意图就比较清晰了，让 sort 方法使用 a.compareTo(b) 这个规则来排序。


## 函数式接口（Functional Interfaces）

- **是什么？** 一个 **有且仅有一个抽象方法** 的接口，就是函数式接口。为了让编译器进行检查，我们通常会给它加上 `@FunctionalInterface` 注解。

  - 正是因为抽象方法中只有一个方法，所以我们才能省略@Override 函数声明等内容
  - 在 `java.util.function` 包下定义了 Java 8 的丰富的函数式接口
  - 这也是对于 Java 一门 典型的 OOP 语言向 OOF （面向函数编程） 的跨越
  - 在函数式编程语言中，函数被当做一等公民对待。在将函数作为一等公民的编程语言中，Lambda 表达式的类型也是函数。但是在 Java 中，Lambda 表达式是对象，而不是函数，他们必须依附于一类特别的对象类型--函数式接口！
  - 简单来说，在 Java8 中 Lambda 表达式就是一个函数式接口的实例。这就是 Lambda 表达式和函数式接口的关系。也就是说，只要一个对象是函数式接口的实例，那么该对象就可以使用 Lambda 表达式来表示。
  - 在以前用匿名实现类的现在都可以用 Lambda 表达式来编写。

- **为什么需要它？** 函数式接口是 Lambda 表达式的 **目标类型**。换句话说，一个 Lambda 表达式必须被赋值给一个函数式接口类型的变量。上面排序例子中的 `Comparator` 接口就是一个函数式接口（尽管它有多个方法，但只有一个抽象方法 `compare`）。

- **怎么用？** Java 8 在 `java.util.function` 包中内置了四大核心函数式接口，你需要熟练掌握它们：

  1. **`Predicate<T>` - 断言型接口**
     - `boolean test(T t)`: 传入一个参数，返回一个布尔值。常用于过滤。
     - **示例**: `Predicate<String> isEmpty = s -> s.isEmpty();`
  2. **`Consumer<T>` - 消费型接口**
     - `void accept(T t)`: 传入一个参数，没有返回值（消费掉）。常用于打印、修改等操作。
     - **示例**: `Consumer<String> printer = s -> System.out.println(s);`
  3. **`Function<T, R>` - 功能型接口**
     - `R apply(T t)`: 传入一个 T 类型的参数，返回一个 R 类型的结果。常用于转换、映射。
     - **示例**: `Function<String, Integer> lengthFunc = s -> s.length();`
  4. **`Supplier<T>` - 供给型接口**
     - `T get()`: 不接收参数，返回一个结果。常用于创建对象。
     - **示例**: `Supplier<String> newString = () -> new String();`

  现在你再看 Lambda 表达式，就会明白它背后是如何与这些接口匹配的。

|            函数式接口            | 参数类型 | 返回类型 |                             用途                             |
| :------------------------------: | :------: | :------: | :----------------------------------------------------------: |
|       Consumer 消费型接口        |    T     |   void   |     对类型为 T 的对象应用操作，包含方法：void accept(T t)      |
|       Supplier 供给型接口        |    无    |    T     |             返回类型为 T 的对象，包含方法：T get()             |
|     Function <T, R> 函数型接口     |    T     |    R     | 对类型为 T 的对象应用操作，并返回结果。结果是 R 类型的对象。包含方法：R apply(T t) |
|       Predicate 断定型接口        |    T     | boolean  | 确定类型为 T 的对象是否满足某约束，并返回 boolean 值。包含方法：boolean test(T t) |
|        BiFunction <T,U,R>         |   T, U   |    R     | 对类型为 T, U 参数应用操作，返回 R 类型的结果。包含方法为：Rapply(T t, U u); |
|  UnaryOperator(Function 子接口)   |    T     |    T     | 对类型为 T 的对象进行一元运算，并返回 T 类型的结果。包含方法为：Tapply(T t); |
| BinaryOperator(BiFunction 子接口) |   T, T    |    T     | 对类型为 T 的对象进行二元运算，并返回 T 类型的结果。包含方法为：Tapply(T t1, T t2); |
|         BiConsumer <T,U>          |   T, U    |   void   |    对类型为 T, U 参数应用操作。包含方法为：voidaccept(Tt, Uu)    |
|         BiPredicate <T,U>         |   T, U    | boolean  |                包含方法为：booleantest(Tt, Uu)                |
|          ToIntFunction           |    T     |   int    |                       计算 int 值的函数                        |
|          ToLongFunction          |    T     |   long   |                       计算 long 值的函数                       |
|         ToDoubleFunction         |    T     |  double  |                      计算 double 值的函数                      |
|           IntFunction            |   int    |    R     |                     参数为 int 类型的函数                      |
|           LongFunction           |   long   |    R     |                     参数为 long 类型的函数                     |
|          DoubleFunction          |  double  |    R     |                    参数为 double 类型的函数                    |

这有俩例子：

```java
public void happyTime(double money, Consumer<Double> consumer) {
    consumer.accept(money);
}

@Test
public void test01(){
    // 1.以前的写法
    happyTime(1241, new Consumer<Double>() {
        @Override
        public void accept(Double money) {
            System.out.println("可恶的Java，让我回家吧， 花费：" + money);
        }
    });

    // 2.Lambda表达式，将之前的代码压缩为一行
    happyTime(516, money -> System.out.println("学习太累了，奖励自己一套掠夺2.0， 花了我" + money + "大洋"));
```

```java
public List<String> filterString(List<String> strings, Predicate<String> predicate) {
    ArrayList<String> res = new ArrayList<>();
    for (String string : strings) {
        if (predicate.test(string)) {
            res.add(string);
        }
    }
    return res;
}

@Test
public void test02() {
    List<String> strings = Arrays.asList("东京", "西京", "南京", "北京", "天津", "中京");
    // 1.以前的写法
    List<String> list = filterString(strings, new Predicate<String>() {
        @Override
        public boolean test(String s) {
            return s.contains("京");
        }
    });
    System.out.println(list);

    // 2.现在的写法
    List<String> res = filterString(strings, s -> s.contains("京"));
    System.out.println(res);
}
```

还有一个实际应用的例子，是在黑马点评中的一个解决缓存击穿的代码实现

```java
public <R, ID> R queryWithPassThrough(
    String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallBack, Long time, TimeUnit unit) {
    // 1.从Redis查询商铺缓存
    String key = keyPrefix + id;
    String json = stringRedisTemplate.opsForValue().get(key);
    // 2.判断是否存在
    if (StrUtil.isNotBlank(json)) {
        // 3.存在直接返回
        return JSONUtil.toBean(json, type);
    }
    // 判断缓存命中的是否为空值（解决缓存穿透）
    if (json != null) {
        return null;
    }
    // 4.查询数据库 -> 存在 or 不存在
    // 这里就是实现了一个 Function 接口， 两个泛型分别接收传入类型和返回类型的参数
    // 这里传入的函数就是
    R r = dbFallBack.apply(id);
    // 5.不存在，返回错误
    if (r == null) {
        // 将空值写入Redis
        stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
        return null;
    }
    // 6.存在，写入Redis
    this.set(key, r, time, unit);
    // 7.返回
    return r;
}

// 调用
Shop shop = cacheClient.queryWithPassThrough(
    CACHE_SHOP_KEY, 
    id, 
    Shop.class, 
    this::getById, 
    CACHE_SHOP_TTL, 
    TimeUnit.MINUTES
);

// 使用 Lambda 表达式
Shop shop = cacheClient.queryWithPassThrough(
    CACHE_SHOP_KEY, 
    id, 
    Shop.class, 
    (queryId) -> { return this.getById(queryId); }, // <-- Lambda 表达式
    CACHE_SHOP_TTL, 
    TimeUnit.MINUTES
);
```

## 方法引用和构造器引用

- 当要传递给 Lambda 体的操作，已经有实现的方法了，可以使用方法引用
- 方法引用可以看做会 Lambda 表达式的深层次表达，换句话说，方法引用就是 Lambda 表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法，可以认为是 Lambda 表达式的一个语法糖
- 要求：实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的方法的参数列表和返回值类型保持一致
- 格式：使用操作符 `::` 将类或对象与方法名分割开来
- 有如下三种使用情况
  1. 对象:: 实例方法名
  2. 类:: 静态方法名
  3. 类:: 实例方法名

### 方法引用的使用情况

- 方法引用的使用
  1. 使用情境：当要传递给 Lambda 体的操作，已经有实现的方法了，可以使用方法引用！
  2. 方法引用，本质上就是 Lambda 表达式，而 Lambda 表达式作为函数式接口的实例。所以方法引用，也是函数式接口的实例。
  3. 使用格式： 类(或对象) :: 方法名
  4. 具体分为如下的三种情况：
     - 情况 1：对象 :: 非静态方法
     - 情况 2：类 :: 静态方法
     - 情况 3：类 :: 非静态方法
  5. 方法引用使用的要求：要求接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同！（针对于情况 1 和情况 2）

## 强大的 Stream API

### Stream API 概述

- Java8 中有两个最为重要的改变，第一个就是 Lambda 表达式，另外一个则是 Stream API
- Stream API(java.util.stream)把真正的函数式编程风格引入到 Java 中，这是目前为止对 Java 类库最好的补充，因为 Stream API 可以极大地提高程序员生产力，让程序员写出高效、简洁的代码
- Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。
- 使用 Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询，也可以使用 Stream API 来并行执行操作。简言之，Stream API 提供了一种高效且易于使用的处理数据的方式
- 为什么要使用 Stream API
  - 实际开发中，项目中多数数据源都是来自 MySQL、Oracle 等。但现在数据源可以更多了，有 MongDB、Redis 等，而这些 NoSQL 的数据就需要 Java 层面去处理。我就是学完 Redis 再来补票的..
  - Stream 和 Collection 集合的区别：Collection 是一种静态的内存数据结构，而 Stream 是有关计算的。前者是主要面向内存，存储在内存中，后者主要是面向 CPU，通过 CPU 实现计算（这也就是为什么一旦执行终止操作之后，Stream 就不能被再次使用，得重新创建一个新的流才行）
- 小结
  1. Stream 关注的是对数据的运算，与 CPU 打交道；
     集合关注的是数据的存储，与内存打交道
  2. Stream 自己不会存储数据；
     Stream 不会改变源对象，相反，他们会返回一个持有结果的新 Stream
     Stream 操作是延迟执行的，这意味着他们会等到需要结果的时候才执行
  3. Stream 执行流程
     - Stream 实例化
     - 一系列中间操作（过滤、映射、…）
     - 终止操作
  4. 说明
     - 一系列中间操作链，对数据源的数据进行处理
     - 一旦执行终止操作，就执行中间操作链，并产生结果，之后，不会再被使用

### Stream 的操作

**创建 Stream (Source)**: 从集合、数组等数据源创建一个流。

1. - `list.stream()`
   - `Arrays.stream(array)`
   - `Stream.of("a", "b", "c")`
2. **中间操作 (Intermediate Operations)**: 对流进行处理和转换，可以有多个。这些操作是 **惰性求值** 的，只有当触发终止操作时才会执行。
   - `filter(Predicate<T> predicate)`: **过滤**，保留满足条件的元素。
   - `map(Function<T, R> mapper)`: **映射/转换**，将流中每个元素转换成另一种形态。
   - `sorted()`: **排序**。
   - `distinct()`: **去重**。
   - `limit(long maxSize)`: **截断**，只取前 N 个元素。
3. **终止操作 (Terminal Operations)**: 触发流的计算并产生最终结果。一个流只能有一个终止操作。
   - `collect(Collector c)`: **收集**，将流中的元素收集到集合、数组等中。这是最常用的终止操作。
   - `forEach(Consumer<T> action)`: **遍历** 流中每个元素。
   - `count()`: **计数**。
   - `anyMatch()`, `allMatch()`, `noneMatch()`: **匹配**。
   - `findFirst()`, `findAny()`: **查找**。
   - `reduce()`: **归约**，将流中元素反复结合起来得到一个值。

**【代码对比】** 来看一个典型的业务需求：

- 有一个 `User` 列表，找出所有类型为“VIP”的用户，
- 按年龄倒序排序，
- 取出他们的名字，
- 最后将名字放入一个新的 List 中。

Java 8 之前（命令式编程）：

```java
// 假设 User 类有 getType(), getAge(), getName() 方法
List<User> users = ...;
List<User> vipUsers = new ArrayList<>();
for (User user : users) {
    if ("VIP".equals(user.getType())) {
        vipUsers.add(user);
    }
}

Collections.sort(vipUsers, new Comparator<User>() {
    @Override
    public int compare(User u1, User u2) {
        return u2.getAge().compareTo(u1.getAge()); // 倒序
    }
});

List<String> names = new ArrayList<>();
for (User user : vipUsers) {
    names.add(user.getName());
}
// `names` 就是最终结果

```

代码很繁琐，定义了多个中间集合，逻辑分散在多个循环和方法调用中。

**使用 Stream API (声明式编程):**

```java
List<String> names = users.stream()                       // 1. 创建流
    .filter(user -> "VIP".equals(user.getType()))   // 2. 过滤 VIP 用户
    .sorted(Comparator.comparing(User::getAge).reversed()) // 3. 按年龄倒序
    .map(User::getName)                               // 4. 提取名字
    .collect(Collectors.toList());                    // 5. 收集成 List
// `names` 就是最终结果
```

代码就像一条流水线，一气呵成，清晰地描述了“要做什么”，而不是“要怎么一步步做”。其中 `User::getAge` 和 `User::getName` 是 **方法引用**，是 Lambda 表达式的一种更简洁的写法。

## Optional 类

- **是什么？** `Optional<T>` 是一个 **容器类**，它代表一个值 **存在或不存在**。

- **为什么需要它？** 在 Java 8 之前，我们经常通过返回 `null` 来表示一个值不存在。这导致了 infamous `NullPointerException` (NPE)。调用方总是需要写 `if (result != null)` 这样的防御性代码，既丑陋又容易忘记。`Optional` 强迫你思考和处理“值不存在”的情况，使代码更健壮。

- **怎么用？** **创建 Optional:**

  - `Optional.of(value)`: 为一个非空值创建 Optional。如果 value 是 null，会抛出 NPE。
  - `Optional.ofNullable(value)`: 为一个可能为 null 的值创建 Optional。
  - `Optional.empty()`: 创建一个空的 Optional。

  **使用 Optional:** `Optional` 的精髓在于它提供了一系列函数式方法来处理值，而不是简单粗暴地 `get()`。

  - `isPresent()`: 判断值是否存在。（传统用法，但尽量避免）

  - `ifPresent(Consumer<T> consumer)` : 如果值存在，就执行 consumer 的逻辑。
    ```java
    optionalUser.ifPresent(user -> System.out.println(user.getName()));
    ```

  - `orElse(T other)`: 如果值存在，返回值；否则返回 `other` 这个默认值。
    
    ```java
    User user = optionalUser.orElse(new User("default"));
    ```
    
  - `orElseGet(Supplier<T> other)`: 类似 `orElse`，但默认值是通过 Supplier 动态生成的，更高效。
  
  - `orElseThrow(Supplier<? extends X> exceptionSupplier)`: 如果值不存在，就抛出指定的异常。
  
  - `map(Function<T, R> mapper)`: 如果值存在，就对其进行转换。返回一个新的 `Optional<R>`。
  
  **【最佳实践】** 不要把 `Optional` 当作 `if (get() != null)` 的替代品。要善用 `map`, `ifPresent`, `orElse` 等链式方法，写出流畅、安全的代码。
  
  **错误示范:**
  
  ```java
  Optional<User> optUser = ...;
  if (optUser.isPresent()) {
      User user = optUser.get();
      // ... use user
  }
  ```
  
  **推荐用法:**
  
  ```java
  Optional<User> optUser = ...;
  String name = optUser
  	.map(User::getName)
  	.orElse("Unknown");
  ```

## 完结撒花

这就是差不多所有的内容了，Java8 让开发变得更加高效和易懂，还是很好的特性的。