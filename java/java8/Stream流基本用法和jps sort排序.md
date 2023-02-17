---
title: java8新特性流
date: 2023-02-17 17:08:00
categories:
- java
tags:
- stream
- sort
---

### Sort排序

#### 1、Pageable实现单属性排序

```java
Sort sort = new Sort(Direction.ASC, "seqNum");
Pageable pageable = new PageRequest(pageNum, pageSize, sort);
```

>Sort第一个参数表示查询结果以asc升序排序或者是desc降序排序，第二个参数表示根据该"seqNum"字段排序。
>
>PageRequest构造方法传入pageNum,pageSize,sort。

#### 2、多属性排序

```java
Sort.Order order1 = new Sort.Order(Sort.Direction.ASC, "authStatus");
Sort.Order order2 = new Sort.Order(Sort.Direction.DESC, "createTime");
List<Sort.Order> list = new ArrayList<>();
list.add(order1);
list.add(order2);
Sort sort =  new Sort(list);
Pageable pageable = new PageRequest(page, pageSize, sort);
```

#### 3、new Sort爆红和PageRequest构造方法过时

springboot最新版本不支持Sort构造方法写入，Sort的构造器私有了private。调用`Sort.by()`方法创建Sort

```java
Sort sort =Sort.by(Direction.ASC, "seqNum");
```

PageRequest构造方法过时，调用PageRequest.of方法创建新的pageable。

```java
Pageable pageable = PageRequest.of(page, pageSize, sort);
```

### SerializerFeature枚举类

SerializerFeature规定了实体对象转换位json数据的一些格式化输出要求。

| 枚举类型               | 含义                                     |
| ---------------------- | ---------------------------------------- |
| UseSingleQuotes        | 使用单引号而不是双引号,默认为false       |
| WriteNullStringAsEmpty | 字符类型字段如果为null,输出为"",而非null |
| WriteMapNullValue      | 是否输出值为null的字段,默认为false       |

```java
// 第一个参数为需要转换为JSON的对象类，第二个参数为SerializerFeature的参数设置，对word中的数据进行枚举处理。
JSONObject.toJSONString(word, SerializerFeature.UseSingleQuotes);
JSONObject.toJSONString(word, SerializerFeature.WriteNullStringAsEmpty);
JSONObject.toJSONString(word, SerializerFeature.WriteMapNullValue);

```

### Stream流

Stream流将要处理的元素看作一种流，在流的过程中，对流中的元素进行操作。

#### Stream特性

对流的操作分为两种

1. 中间操作，每次返回一个新的流。
2. 终端操作，每个流只能进行一次终端操作，终端操作结束后流无法再次使用。

特性

1. stream不存储数据，按照规则对数据进行计算。
2. 不改变数据源，一般情况产生新的集合或值。
3. 具有延迟执行特性。

#### Stream流的创建

stream流可以由数组或集合创建

* 数组

```java
int[] array={1,3,5,6,8};
IntStream stream = Arrays.stream(array);
```

* 集合

```java
List<String> list = Arrays.asList("a", "b", "c");
// 创建一个顺序流
Stream<String> stream = list.stream();
// 创建一个并行流
Stream<String> parallelStream = list.parallelStream();
```

其次可以使用stream的静态方法：`of()、interate()、generate()`创建流。

`stream`和`parallelStream`区分：`stream`是顺序流，由主线程按顺序对流执行操作；`parrallelStream`是并行流，内部以多线程并行执行的方式对流进行操作（流中的数据处理没有顺序要求）。

#### Stream使用

>Optional

Optional类是一个可以为null的容器对象，值存在则`isPrensent()`返回`true`。

##### 遍历、匹配(foreach/find/match) 和筛选(filter)

```java
List<Integer> list = Arrays.asList(7, 6, 9, 3, 8, 2, 1);
// 遍历输出符合条件的元素
list.stream().filter(x -> x > 6).forEach(System.out::println);
// 匹配第一个
Optional<Integer> findFirst = list.stream().filter(x -> x > 6).findFirst();
// 匹配任意（适用于并行流）
Optional<Integer> findAny = list.parallelStream().filter(x -> x > 6).findAny();
// 是否包含符合特定条件的元素
boolean anyMatch = list.stream().anyMatch(x -> x > 6);
```

`filter()`过滤流中元素大于6的值，`findFirst()`匹配到第一个大于6的元素，`findAny()`随机匹配流中大于6的元素，`anyMatch()`判断流中是否存在大于6的元素。

##### 聚合(max/min/count)

```java
List<String> list = Arrays.asList("adnm", "admmt", "pot", "xbangd", "weoujgsd");
Optional<String> max = list.stream().max(Comparator.comparing(String::length));
Optional<String> min = list.stream().min(Comparator.comparing(String::length));
long count = list.stream().distinct().count();
System.out.println("最长的字符串：" + max.get());
System.out.println("最短的字符串：" + min.get());
System.out.println("字符串的个数：" + count);
```

在`max()和min()`方法中传入比较器`Comparator`获取当前比较结果的`max和min`值。`distinct()`方法用于除去流中重复的元素。

##### 映射(map/flatMap)

映射可以将一个流的元素按照一定的规则映射到另一个流中。分为`map`和`flatMap`：

1. `map：`将接收的函数作为参数，应用到每一个元素上，并映射成一个新的元素。
2. `flatMap：`接收一个函数作为参数，将流中的每个值都换成另一个流，然后连接所有流成为一个流。

```java
String[] strArr = { "abcd", "bcdd", "defde", "fTr" };
List<String> strList = Arrays.stream(strArr).map(String::toUpperCase).collect(Collectors.toList());
```

`map()`传入`String.toUpperCase()`方法将流中的元素替换为大写，`collect()`传入`Collectors.toList()`方法将流转换为list集合存储。

```java
List<String> list = Arrays.asList("m,k,l,a", "1,3,5,7");
List<String> listNew = list.stream().flatMap(s -> {
    // 将每个元素转换成一个stream
    String[] split = s.split(",");
    Stream<String> s2 = Arrays.stream(split);
    return s2;
	}).collect(Collectors.toList());
```

`flatMap()`将流中每一个元素转换成stream然后连接成同一个流返回。
