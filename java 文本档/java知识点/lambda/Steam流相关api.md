**Stream流究竟是什么？**

Stream就好像一个高级迭代器，但只能遍历一遍。Stream流是java8新提供给开发者的一组操作集合的Api，将要处理的元素集合看做一种流，留在管道中传输，并且可以在管道的节点进行处理，比如筛选，排序，聚合等。元素流在管道中经过中间操作的处理。最后由终端操作得到前面处理的结果。Stream流可以极大的提高开发效率，也可以使用它写出更加简洁明了的代码。

**创建流**

1. 调用集合的stream()方法或者parallelStream()方法创建流。
2. Stream类的静态of()方法创建流。

```
public class StreamTestDemo01 {
  public static void main(String[] args) {
    //
      List<String> sortStream = new ArrayList<>();

      //顺序流
      Stream<String> stream = sortStream.stream();
      //并行流
      Stream<String> parallelStream = sortStream.parallelStream();
      // of()方法创建
      Stream<String> stringStream = Stream.of(sortStream.toArray(new String[sortStream.size()]));
  }
```

**使用流**

```
public class StreamTestDemo03 {
  public static void main(String[] args) {
      //使用stream流筛选未及格学生名单

      List<Student> students = new ArrayList<>();

      List<Student> collect = students.stream().filter(one -> one.getScore() < 60).collect(Collectors.toList());

      System.out.println("collect = " + collect);
  }
}
```

**使用java**

```
public class StreamTestDemo04 {
  public static void main(String[] args) {
      //使用java筛选未及格学生名单

      List<Student> students = new ArrayList<>();
      
      List<Student> filterStudent = new ArrayList<>();

    for (Student student : students) {
      //
        if (student.getScore()<60){
            filterStudent.add(student);
        }
    }
    System.out.println(filterStudent);
  }
}
```

**流的基础知识**

**流的分类**

stream流分为顺序流和并行流，所谓顺序流就是按照顺序对集合中的元素进行处理，而并行流则是使用多线程同时对集合中多个元素进行处理，所以在使用并行流的时候就要注意线程安全的问题了。

**终端操作和中间操作**

终端操作会消费stream流，并且会产生一个结果，比如iterator()和spliterator()。如果一个stream流被消费过了，那它就不能被重用的。

中间操作会产生另一个流。需要注意的是中间操作不是立即发生的。而是当在中间操作创建的新流执行完终端操作后，中间操作指定的操作才会发生。流的中间操作还分无状态和有状态两种。

- 在无状态操作中，在处理流中的元素时，会对当前的元素进行单独处理。比如，过滤操作，因为每个元素都是单独被进行处理，所有它和流中的其他元素无关。
- 在有状态操作中，某个元素的处理可能依赖于其他元素。比如查找最小值，最大值，和排序，因为他们都依赖于其他的元素。

**流接口**

BaseStream接口

BaseStream接口是Stream流最基础的接口，它提供了所有流都可以使用的基本功能。baseStream是一个泛型接口，它有两个类型参数T和S，其中T指定了流中的元素的类型，S指定了具体流的类型，S也必定是Basetream流的子类。

- Iterator<T> iterator()：获取流的迭代器
- Spliterator spliterator()：获取流的spliterator。
- boolean isParallel(): 判断一个流是否是并行流，如果是则返回true，否则返回false。
- S sequential(): 基于调用流返回一个顺序流，如果调用流本身就是顺序流，则返回本身。
- S parallel():基于调用流，返回一个并行流，如果调用流本身就是并行流，则返回本身。
- S unorderd()： 基于调用流，返回一个无序流。
- S onClose(Runnable closeHandler)：返回一个新流，closeHandler指定了该流的关闭处理程序，当关闭该流时，将调用这个处理程序。
- void close()：从AutoCloseable继承来的，调用注册关闭处理程序，关闭调用流(很少会被使用到)

Stream接口

- Stream filter(Predicate predicate)：产生一个新流，其中包含调用流中满足predicate指定的谓词元素，即筛选符合条件的元素后重新生成一个新的流。(中间操作)
- Stream map(Function mapper)： 产生一个新流，对调用流中的元素应用mapper，新Stream流包含这些元素。(中间操作)
- IntStream mapToInt(ToIntFunction mapper)：对调用流中的元素应用mapper，产生包含这些元素的一个新IntStream流。(中间操作)

Stream的特性和优点：

- 无存储。Stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，java容器或I/O channel等。

- 为函数式编程而生。对Stream的任何修改都不会修改背后的数据源，比如对Stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新Stream。

- 惰式执行。Stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。

- 可消费性。Stream只能被“消费”一次，一旦遍历过就会失败，就像容器迭代器一样，想要再次遍历必须重新生成。

  ![15521192454583](C:\Users\HITO_JAVA5\Desktop\最近整理的文本\笔记\lambda\图片\15521192454583.jpg)

Stream的中间操作

filter

filter方法用于通过设置的条件过滤元素。以下代码片段使用filter方法过滤空字符串：

```
public class StreamTestDemo05 {
  public static void main(String[] args) {
      List<String> strings = Arrays.asList("Hollis", "", "HollisChuang", "H", "hollis");
      strings.stream().filter(string -> !string.isEmpty()).forEach(System.out::println);
      //Hollis, , HollisChuang, H, hollis

  }
}
```

map

map方法用于映射每个元素到对应的结果，以下代码片段使用map输出了元素的对应的平方数：

```
public class StreamTestDemo05 {
  public static void main(String[] args) {
      List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
      numbers.stream().map( i -> i*i).forEach(System.out::println);
      //9,4,4,9,49,9,25
  }
}
```

limit/skip

limit返回Stream的前面n个元素；skip则是扔掉前n个元素。以下代码片段使用limit方法保理4元素：

```
public class StreamTestDemo05 {
  public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
    numbers.stream().limit(4).forEach(System.out::println);
    // 3,2,2,3
  }
  }
```

sorted

sorted方法用于对流进行排序。以下代码片段使用sorted方法进行排序：

```
public class StreamTestDemo05 {
  public static void main(String[] args) {
      List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
      numbers.stream().sorted().forEach(System.out::println);
//2,2,3,3,3,5,7

  }
  }
```

distinct

distinct主要用来去重，以下代码片段使用distinct元素进行去重：

```
public class StreamTestDemo05 {
  public static void main(String[] args) {
      List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
      numbers.stream().distinct().forEach(System.out::println);
      //3,2,7,5
  }
  }
```

Stream的最终操作

Stream的中间操作得到的结果还是一个Stream，那么如何把一个Stream转换成我们需要的类型呢？比如计算出流中元素的个数，将流转换成集合。这就需要最终操作。

最终操作会消耗流，产生一个最终结果。也就是说，在最终操作之后，不能再次使用流，也不能在使用任何中间的操作。

forEach

Stream提供方法‘forEach’来迭代流中的每个数据。以下就是使用ForEach输出10个随机数：

```
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```

count

count用来统计流中元素的个数

```
List<String> strings = Arrays.asList("Hollis", "HollisChuang", "hollis","Hollis666", "Hello", "HelloWorld", "Hollis");
System.out.println(strings.stream().count());
//7
```

collect

collect就是一个规约操作，可以接受各种做法作为参数，将流中元素累积成一个汇总结果：

```
List<String> strings = Arrays.asList("Hollis", "HollisChuang", "hollis","Hollis666", "Hello", "HelloWorld", "Hollis");
strings  = strings.stream().filter(string -> string.startsWith("Hollis")).collect(Collectors.toList());
System.out.println(strings);
//Hollis, HollisChuang, Hollis666, Hollis
```

