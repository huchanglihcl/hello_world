
- [1流的概念](#1流的概念)
- [2创建流的5种方法](#2创建流的5种方法)
- [3 Stream中间操作](#3-Stream中间操作)
- [4 终止Stream操作](#4-终止Stream操作)
      

##1流的概念
   Stream是Java8中处理集合的关键抽象概念，它可以指定希望对集合的操作，可以执行复杂的查找、过滤和映射数据等操作。
   使用Stream API 对集合的数据进行操作，类似于SQL执行的数据库查询，也可以用来并行执行操作，其提供了一种高效且易于使用的处理数据方式。
+ Stream自身不会存储元素。
+ Stream不会改变数据源对象，相反会返回产生一个持有结果的新Stream。
+ Steam操作是延迟执行的，这意味着他们会等到需要结果的时候才执行。

## 2创建流的5种方法
### 2.1通过Collection 系列集合提供的串行流：stream()、并行流： paralleStream()
```java
List<String> list = new ArrayList<>();
Stream<String> stream1 = list.stream();
```

### 2.2 通过Arrays中的静态方法stream(T[] array) 获取数组流
Arrays.stream(T[] array)的源码:
```java
public static <T> Stream<T> stream(T[] array) {
     return stream(array, 0, array.length);
 }
```
用例：
```java
Stu[] stus = new Stu[10];
Stream<Stu> stream2 = Arrays.stream(stus);
/*
  public static <T> Stream<T> stream(T[] array) {
     return stream(array, 0, array.length);
 }
 */
```
### 2.3通过Stream类中的静态方法 of()
Stream.of() 源码：
```java
//1.单参泛型of
 public static<T> Stream<T> of(T t) {
    return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
//2.可变参数
@SafeVarargs
@SuppressWarnings("varargs") // Creating a stream from an array is safe
public static<T> Stream<T> of(T... values) {
    return Arrays.stream(values);
}
```
用例：
```java
Stream<String> stream3 = Stream.of("hxh", "aj", "hhh");
```
### 2.4使用Stream类的静态方法 iterate 创建无限流
iterate方法：
Stream<T> iterate(final T seed, final UnaryOperator<T> f)
参数 seed 种子起始值，UnaryOperator 函数式接口 继承Function<T,T> 此时参数类型符合返回值类型一致
用例：
```java
//4.使用Stream类的静态方法 iterate 创建无限流
//Stream<T> iterate(final T seed, final UnaryOperator<T> f) 
//参数 seed 种子起始值，
// UnaryOperator 函数式接口 继承Function<T,T> 此时参数类型符合返回值类型一致

Stream<Integer> stream4 = Stream.iterate(0, (x) -> x + 2);
//中间操作和终止操作
stream4.limit(5).forEach(System.out::println);
```
### 2.5 使用Stream类的静态方法 generate创建无限流
generate方法参数为Supplier<T> 供给型接口
```java
//5.使用Stream类的静态方法 generate 创建无限流
//参数为Supplier<T> 供给型接口
Stream<Double> generateStream = Stream.generate(() -> Math.random());
generateStream.limit(5).forEach(System.out::println);
//0.4762976596937549
//0.08577913333772513
//0.32149010682857515
//0.31059489250233197
//0.45181354173159927
```
## 3 Stream中间操作
**注意点：**
+ 若只有中间操作，则不会执行
+ 只有终止操作执行后，所有的中间操作一次执行，此时就称为延迟加载或者惰性求值
### 3.1 filter-过滤
Stream<T> filter(Predicate<? super T> predicate)
断言型接口参数 即条件判断过滤
用例： 
先创建一个Stu类List集合
```java
List<Stu> stuList = Arrays.asList(
        new Stu(1,"hh",22),
        new Stu(2,"aa",22),
        new Stu(3,"bb",32),
        new Stu(4,"cc",42),
        new Stu(5,"dd",52)
);
//取age>30的Stu元素
//若只有中间操作，则不会执行
Stream<Stu> stuStream = stuList.stream()
    .filter((i) -> i.getAge() > 40);
//终止操作 执行后，所有的中间操作一次执行，此时就称为延迟加载或者惰性求值
stuStream.forEach(System.out::println);
//Stu{id=4, name='cc', age=42}
//Stu{id=5, name='dd', age=52}
```
### 3.2 limit-限定元素数量
通过截断流，使流中元素个数不超过指定数量
```java
stuList.stream()
        .filter((s) ->{
            System.out.println("测试迭代几次");
            return s.getAge()>40;
        }).limit(1).forEach(System.out::println);
  
```
结果：
```
测试迭代几次
测试迭代几次
测试迭代几次
测试迭代几次
Stu{id=4, name='cc', age=42}
```
结果发现：先通过filter()过滤，迭代到想要的过滤结果后，再根据limit(n)，
直接截断流，后续操作不继续，限制其流中元素个数为n，此操作称为短路操作，短路操作也用于提高效率；
所以前3次元素不在结果中，但都进行迭代判断，打印了3次后后面的元素再次进行迭代，
发现元素满足过滤条件，但limit限制只要一个，即最后一次迭代后直接截断流，结果为第一个满足过滤条件的元素。
### 3.3 skip-跳过元素

返回一个跳过前n个元素的流，若流中元素不足n个，则返回一个空流。 
其与limit(n)互补
```java
//skip(n)  跳过前n个元素
stuList.stream()
    .skip(2).forEach(System.out::println);
/* 结果：
    Stu{id=3, name='bb', age=32}
    Stu{id=4, name='cc', age=42}
    Stu{id=5, name='dd', age=52}
 */
stuList.stream().skip(6).forEach(System.out::println);
//流中元素个数总数为5，小于6，则返回空流，没有结果值
```
### 3.4 distinct-去重
通过流所生成元素的hashCode()和equals()来去除重复元素
```java
List<Stu> stuList = Arrays.asList(
        new Stu(1,"hh",22),
        new Stu(2,"aa",22),
        new Stu(3,"bb",32),
        new Stu(4,"cc",42),
        new Stu(4,"cc",42),
        new Stu(4,"cc",42),
        new Stu(4,"cc",42),
        new Stu(5,"dd",52)
);
//此时，在Stu类中生成重写hashCode()和equals()方法：
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Stu stu = (Stu) o;
    return Objects.equals(id, stu.id) &&
            Objects.equals(name, stu.name) &&
            Objects.equals(age, stu.age);
}

@Override
public int hashCode() {

    return Objects.hash(id, name, age);
}
结果：
Stu{id=1, name='hh', age=22}
Stu{id=2, name='aa', age=22}
Stu{id=3, name='bb', age=32}
Stu{id=4, name='cc', age=42}
Stu{id=5, name='dd', age=52}
```
+ distinct()去重原理为通过流所生成元素的hashCode()和equals()来去除重复元素。
### 3.5  map映射

- 接收Lambda，将元素转换成其他形式或提取信息。 
- 接收一个Function<? super T, ? extends R> mapper函数作为参数，该函数会被应用到每个元素上，并将其映射到一个新的元素。

```java
//map映射
List<String> stringList = Arrays.asList("aa", "bb", "cc", "dd");
stringList.stream()
    .map((x)->x.length()).forEach(System.out::println);
stringList.stream()
    .map((x)->x.toUpperCase()).forEach(System.out::println);

结果：
2
2
2
2
AA
BB
CC
DD    
```
从结果看出，流中的每个元素都应用了map()里的参数中的Function函数，并返回经过Function处理的元素。
+ map映射的重要应用为：即类似SQL中的映射，获取对象中的某些属性（即数据库中的某些字段）
### 3.6 flatMap映射
+ 接收一个函数作为参数，将流中的每个值都转换成另一个流，然后把所有流连接成一个流。每个部分流中的每个值成单独小流，再串成一个整体流。

对比map映射： 
1. map映射是将集合中的部分流添加到整体流中，而flatMap映射是将集合中的部分流中的每个元素单独一个个地添加到整体流中。 
2. map映射: Stream<Stream<Character>> , flatMap映射：Stream<Character>
```java
/**
 * 字符串拆分成字符后组成一个字符类型的流
 * @param str
 * @return
 */
public static Stream<Character> filterCharacter(String str){
    List<Character> characterList = new ArrayList<>();
    for (Character ch: str.toCharArray()
         ) {
        characterList.add(ch);
    }
    return characterList.stream();
}
List<String> stringList = Arrays.asList("aa", "bb", "cc", "dd");
Stream<Stream<Character>> st1 = stringList.stream()
    .map(TestStream::filterCharacter);
//此时流的内容为 {{"aa"},{"bb"},{"cc"},{"dd"}} 4个单独的字符流对象组成的流
st1.forEach(System.out::println);
/*  再次遍历后
    结果：4个流对象 即 Stream<Character>  {{"aa"},{"bb"},{"cc"},{"dd"}}
    java.util.stream.ReferencePipeline$Head@470e2030
    java.util.stream.ReferencePipeline$Head@3fb4f649
    java.util.stream.ReferencePipeline$Head@33833882
    java.util.stream.ReferencePipeline$Head@200a570f
 */
System.out.println("----------------");

Stream<Character> st2 = stringList.stream().flatMap(TestStream::filterCharacter);
//此时流的内容为{"a","a","b","b","c","c","d","d"}
st2.forEach(System.out::println);
/*  再次遍历后
    结果直接返回了单个的字符流
    a
    a
    b
    b
    c
    c
    d
    d
 */

```
### 3.7 自然排序
sorted()
+ 自然排序(Comparable方式)，按照字典顺序进行排序
+ 按照实现的Comparable中的compare to()方法
```java
List<String> stringList = Arrays.asList("ee", "bb", "ff", "dd","哈哈","啊");
//根据String类中Comparable方式进行默认排序，即compare to()方法
stringList.stream()
    .sorted().forEach(System.out::println);
/*结果：
bb
dd
ee
ff
哈哈
啊*/    
```
### 3.8 指定排序
sorted(Comparator com)
+ 根据实现Comparator接口的指定方法进行排序

```java
stuList.stream().sorted(
            (a,b) ->{
                if (a.getAge().equals(b.getAge())){
                    return a.getName().compareTo(b.getName());
                }else{
                    return a.getAge().compareTo(b.getAge());
                }
            }
    ).forEach(System.out::println);
```
## 4 终止Stream操作
终止操作，执行中间链操作，并产生结果
### 4.1 allMatch-检查是否匹配所有元素
+ 匹配 match 利用断言型函数接口，返回boolean值 是否匹配
+ 查找返回容器类 Optional类型 避免空指针异常

返回结果： 
- true 匹配到了所有的元素 注意：和noneMatch()的false 结果代表集合不同 
- false 没有匹配到所有的元素 说明匹配到条件集合中的真子集

```java
boolean b = stuList.stream()
    .allMatch((e) -> e.getAge() > 20);
System.out.println(b);//true

boolean b1 = stuList.stream()
    .noneMatch((e) -> e.getAge() > 20);
System.out.println(b1);//flase

boolean b2 = stuList.stream()
    .noneMatch((e) -> e.getAge() > 40);
System.out.println(b2);//flase
```
### 4.2 anyMatch-检查是否至少匹配一个元素
返回结果： 
- true 匹配到了条件集合中的真子集元素，一个或者多个 
- false 一个元素都没有匹配到，空集
```java
boolean hhh = stuList.stream()
    .anyMatch((e) -> e.getName().equals("hhh"));
System.out.println(hhh);//false

boolean hh = stuList.stream()
    .anyMatch((e) -> e.getName().equals("hh"));
System.out.println(hhh);//true
```
### 4.3 noneMatch-检查是否所有元素都没有匹配到
返回结果： 
- true 所有元素都没有匹配到，空集 
- false 不是所有的元素都没有匹配到 即匹配到了元素,有匹配到的元素即返回false，真子集
```java
boolean b1 = stuList.stream()
    .noneMatch((e) -> e.getAge() > 20);
System.out.println(b1);//flase

boolean b2 = stuList.stream()
    .noneMatch((e) -> e.getAge() > 40);
System.out.println(b2);//flase

boolean b3 = stuList.stream()
    .noneMatch((e) -> e.getAge() > 50);
System.out.println(b3);//此时集合中只有一个元素能匹配到，返回了false

boolean b4 = stuList.stream()
    .noneMatch((e) -> e.getAge() > 60);
System.out.println(b3);//所有的元素都没有匹配到  返回了true
```
### 4.4 findFirst-返回第一个元素
```java
Optional<Stu> first = stuList.stream().findFirst();
System.out.println(first.get());// Stu{id=1, name='hh', age=22}
```
返回第一个元素，用Optional集合类来封装，避免了空指针异常
### 4.5 findAny-返回当前流中的任意一个元素
```java
//从集合中随便找个age>30的Stu对象  可以使用串行流stream,也可以使用parallelStream 并行流
Optional<Stu> any = stuList.parallelStream()
    .filter((e) -> e.getAge() > 30).findAny();
System.out.println(any.get());//Stu{id=4, name='cc', age=42}
Optional<Stu> any1 = stuList.stream()
    .filter((e) -> e.getAge() > 30).findAny();
System.out.println(any1.get());//Stu{id=3, name='bb', age=32}
```
### 4.6 count-返回流中元素总个数
long count();
```java
long count = stuList.stream().count();
System.out.println(count);//8
```
### 4.7 max-返回流中的最大值
```java
//根据年龄大小进行正序排序找出最大值
Optional<Stu> max = stuList.parallelStream()
    .max((a, b) -> Integer.compare(a.getAge(), b.getAge()));
System.out.println(max.get()); //Stu{id=5, name='dd', age=52}

//根据年龄大小进行倒序排序找出最大值
Optional<Stu> max1 = stuList.parallelStream()
    .max((a, b) -> Integer.compare(b.getAge(), a.getAge()));
System.out.println(max1.get()); //Stu{id=1, name='hh', age=22}

//提取最大年龄 先映射提取集合中每个对象的年龄 再直接进行max方法比较 最后返回一个年龄值
Optional<Integer> maxAge = stuList.parallelStream()
    .map(Stu::getAge)
    .max(Integer::compare);
System.out.println(maxAge.get()); //52
```
### 4.7 min-返回流中的最小值
```java
//根据年龄大小进行正序排序找出最小值
Optional<Stu> min = stuList.parallelStream()
    .min((a, b) -> Integer.compare(a.getAge(), b.getAge()));
System.out.println(min.get());//Stu{id=1, name='hh', age=22}

//根据年龄大小进行倒序排序找出最小值
Optional<Stu> min1 = stuList.parallelStream()
    .min((a, b) ->  Integer.compare(b.getAge(), a.getAge()));
System.out.println(min1.get()); //Stu{id=5, name='dd', age=52}
```
### 4.8 reduce-将流中元素反复结合起来，得到一个值
```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
//利用reduce归约函数，可以指定归约规则，将集合中的元素数值进行求和操作等
//以0为起始值，对集合中的各个值进行相加
Integer sum = list.stream().reduce(0, (x, y) -> x + y);
System.out.println(sum);//45

//求出stuList集合中的年龄总和
//此时利用Integer类中的静态方法sum求和 无起始值，有可能为空，
// 则返回值自动变为Optional容器类封装过后的值
Optional<Integer> ageSumOp = stuList.stream()
    .map(Stu::getAge)
    .reduce(Integer::sum);
System.out.println(ageSumOp.get());//296
```
+ 此时利用Integer类中的静态方法sum求和 无起始值,有可能为空，则返回值自动变为Optional容器类封装过后的值。
### 4.9 collect-将流转换为其他形式
+ 接收一个Collector接口的实现，用于Stream中元素做汇总的方法。
+ 利用Collectors实用工具类中提供的很多静态实现Collector接口的方法，进行相应的转换收集操作。
```java
//收集stuList集合中的所有name值，转换为list集合
List<String> nameList = stuList.stream().map(Stu::getName)
    .collect(Collectors.toList());
nameList.forEach(System.out::println);
/*
    结果： 
    hh
    aa
    b
