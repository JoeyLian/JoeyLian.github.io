---
title: Java-Lambda与Stream简记
date: 2020-01-14
categories:
- PPL
tags:
- Java
- 函数式编程
- Lambda
- 流计算
typora-copy-images-to: ..\images
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>



## Java-Lambda与Stream简记

手册向简记 Java8 中的Lambda表达式与Stream。

有空补上Stream 的理解 : )

### Lambda 表达式

**基本语法**：

```java
(parameters) -> expression
(parameters) -> { statements; }
```

"->"操作符将Lambda表达式分为两个部分：左侧为参数列表，右侧为Lambda内容。

可以直接将 Lambda 表达式作为参数传入函数

```java
List features = Arrays.asList("Lambdas", "Default Method", "Stream API");
features.forEach(n -> System.out.println(n));	//foreach 后面介绍
```

### 函数式接口

既然 Lambda 表达式可以作为参数，那他一定有类型，也就是Java中的函数式接口。

**四类基本的函数式接口**：

- `Function<T, R>：R apply(T t)；` 输入类型T返回类型R。
- `Consumer<T>：void accept(T t)；` 输入类型T，消费掉，无返回。
- `Predicate<T>：boolean test(T t)；` 输入类型T，并进行条件判断，返回true 或 false。
- `Supplier<T>：T get()；` 无输入，产生一个T类型的返回值。

**使用**

``` java
//有一个参数，无返回值
Consumer<String> consumer = (x) -> {
    System.out.println(x);
};
consumer.accept("hello");
```

``` java
//多个参数，有返回值, 可以省略类型
//BiFunction<Integer, Integer, Integer> function1 = (x, y) -> x + y;
BiFunction<Integer, Integer, Integer> function1 = (Integer x, Integer y) -> {
    return x + y;
};
int result1 = function1.apply(10, 10);
```

lambda表达式作为参数

``` java
public static void filter(List names, Predicate condition) {
    names.stream().filter((name) -> (condition.test(name))).forEach((name) -> {
        System.out.println(name + " ");
    });
}
```

### Stream

##### 0. 使用

定义一个数组，在数组后接入 stream 与流中的操作，用 . 连接, 返回流操作结束后的结果

``` java
final List<String> strings = Arrays.asList("ab", "a", "abc", "b", "bc");
//串行
long count1 = strings.stream()
            .filter(s -> {
                    System.out.println("thread:" + Thread.currentThread().getId());
                    return s.startsWith("a");
             })
            .count();
System.out.println(count1);
//并行
long count2 = strings.parallelStream()
       .filter(s -> {
                    System.out.println("thread:" + Thread.currentThread().getId());
                    return s.startsWith("a");
        })
        .count();
System.out.println(count2);
```

##### 1. map  --  处理序列 -- 参数为 Function 类型

使用传入的Function对象对Stream中的所有元素进行处理，返回的Stream对象中的元素为原元素处理后的结果。

``` java
//map，平方数
List<Integer> nums = Arrays.asList(1, 2, 3, 4);
Stream<Integer> integerStream = nums.stream().map(n -> n * n);
Collection<Integer> squareNums = integerStream.collect(Collectors.toList());
squareNums.forEach(integer -> System.out.println(integer));
```

##### 2. filter -- 过滤 -- 参数为 Predicate 类型

filter 对原始 Stream 进行某项测试，通过测试的元素被留下来生成一个新 Stream

``` java
final List<String> strings = Arrays.asList("ab", "a", "abc", "b", "bc");
strings.stream()
       .filter(s -> s.startsWith("a"))
       .forEach(System.out::println);
```

##### 3. peek -- 遍历

peek，遍历Stream中的元素，和 forEach 类似，区别是peek不会“消费”掉Stream，而forEach会消费掉Stream；

``` c
//peek
Stream.of("one", "two", "three", "four")	//这也是个操作，初始化流
       .peek(e -> System.out.println("原来的值: " + e))
       .map(String::toUpperCase)
       .peek(e -> System.out.println("转换后的值: " + e))
       .collect(Collectors.toList());
```

##### 4. forEach -- 遍历

对所有元素进行迭代处理，无返回值

```java
Arrays.asList("ab", "a", "abc", "b", "bc").stream()	//只有foreach似乎可以不用.stream
	.forEach(s -> {System.out.println(s);});
```

##### 5. anyMatch -- 参数为 Predicate 类型

只要其中有一个元素满足传入的Predicate时返回True，否则返回False。操作只要anyMatch中的条件成立后，就不再执行。

```java
//anyMatch
boolean anyMatchReturn = Arrays.asList("ab", "a", "abc", "b", "bc").stream()
                .peek(s -> System.out.println(s))
                .anyMatch(s -> s.startsWith("b"));
System.out.println(anyMatchReturn);
//最后还会输出一个 true
//输出为 ab  a  abc  b  true
```

类似还有

**allMatch**： 所有元素均满足传入的Predicate时返回True，否则False。只要allMatch条件有一个为false，中间操作将终止执行。

**noneMatch**：所有元素均不满足传入的Predicate时返回True，否则False。只要allMatch条件有一个为true，中间操作将终止执行。

##### 6. reduce -- 规约

通过累加器accumulator，对前面的序列进行累计操作，并最终返回一个值。累加器accumulator有两个参数，第一个是前一次累加的结果，第二个是前面集合的下一个元素。

1. **T reduce(T identity, BinaryOperator accumulator)** ：这里的identity是初始值。

   下面将会把几个字符组装成一个字符串

```java
String concat = Stream.of("A", "B", "C", "D")
	.reduce("H",(x, y) -> {
                    System.out.println("x=" + x + ", y=" + y);
                    return x.concat(y);
                });
System.out.println(concat);
// 输出为
//x=H, y=A
//x=HA, y=B
//x=HAB, y=C
//x=HABC, y=D
//HABCD
```

2. **Optional reduce(BinaryOperator accumulator)**：由于没有初始值，这里输出Optional类型，避免空指针

   ``` java
   Optional<String> concat2Optional = Stream.of("A", "B", "C", "D")
       .reduce((x, y) -> {
                       System.out.println("x=" + x + ", y=" + y);
                       return x.concat(y);
                   });
   System.out.println(concat2Optional.orElse("default"));
   // 输出为
   //x=A, y=B
   //x=AB, y=C
   //x=ABC, y=D
   //ABCD
   ```

##### 7. collect -- 收集 (转化) 

collect方法可以通过收集器collector将流转化为其他形式，已经在Collectors类中封装了多种 collector。

``` java
//collect，拼接字符串
String collect1 = Stream.of("A", "B", "C", "D")
      .collect(Collectors.joining());
System.out.println(collect1);
```

**例子**：

``` java
// 为每个订单加上12%的税后求总值
// 老方法：
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double total = 0;
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    total = total + price;
}
System.out.println("Total : " + total);
 
// 新方法：
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double bill = costBeforeTax.stream()
    .map((cost) -> cost + .12*cost)
    .reduce((sum, cost) -> sum + cost);
System.out.println("Total : " + bill);
// waw
```





参考：


1. [java8 -函数式编程](https://my.oschina.net/thinwonton/blog/2992751  "With a Title")系列四篇
2. [Java8 lambda表达式10个示例](https://www.cnblogs.com/coprince/p/8692972.html)














