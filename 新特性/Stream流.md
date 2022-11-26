## Stream流

### Stream流类型

#### 1.创建Stream流（开始管道）

​		显而易见，开始管道主要负责创建一个Stream流，可以基于一个数组，集合类型对象（List,Set,Map）创建出新的Stream流。

| API              | 功能说明                                         |
| ---------------- | ------------------------------------------------ |
| stream()         | 创建出一个新的Stream串行流对象                   |
| parallelStream() | 创建出一个可并行执行的Stream流对象               |
| Stream.of()      | 通过给定的一系列元素创建一个新的Stream串行流对象 |

#### 2.Stream流中间处理（中间管道）

​		中间管道负责对Stream流进行处理操作，需要注意的是，中间管道可以进行多次操作，即多次中间处理，它会返回一个新的对象。

| API        | 功能说明                                                     |
| ---------- | ------------------------------------------------------------ |
| filter()   | 按照条件过滤符合要求的元素，返回新的Stream流                 |
| map()      | 将传入元素转化为另一个类型对象，一对一的逻辑，返回新的Stream流 |
| flatmap()  | 将传入元素转化为另一个对象类型，一对多逻辑，一个对象可能会被转化为一个或多个新类型的元素，返回Stream流 |
| limit()    | 仅保留集合前面指定个数的元素，并返回新的Stream流             |
| skip()     | 跳过(忽略)集合前指定个数的元素，并返回新的Stream流           |
| concat()   | 将两个流的数据合并为一个新的流，并返回新的Stream流           |
| distinct() | 对Stream中的所有元素进行去重，并返回新的Stream流             |
| sorted()   | 对Stream中的元素按照指定规则进行排序，并返回新的Stream流     |
| peek()     | 对Stream中的每个元素进行遍历处理，并返回新的Stream流         |

#### 3.终止Stream流（终止管道）

​		终止管道负责将已经处理完的Stream流数据按照要求进行返回，终止管道操作之后，Stream流将会结束。

| API         | 功能说明                                                     |
| ----------- | ------------------------------------------------------------ |
| count()     | 返回Stream流处理后最终元素的个数                             |
| max()       | 返回经过Stream流中处理后的元素的最大值                       |
| min()       | 返回经过Stream流中处理后的元素的最大值                       |
| findFirst() | 返回Stream流中的第一个元素的Optional(找到第一个符合条件的元素就终止流处理并返回) |
| findAny()   | 找到任何一个符合条件的元素则退出流处理，在串行流中与findFirst相同，在并行流中比较高效，任何分片中找到符合条件的元素都会终止后续计算逻辑。 |
| anyMatch()  | 判断流中是否有符合条件的元素，返回一个boolean值              |
| allMatch()  | 判断流中是否所有的元素都符合条件，返回一个boolean值          |
| noneMatch   | 与allMatch()相反，判断流中是否所有元素都不符合条件，返回一个boolean值 |
| collect()   | 将流转换为指定的类型，传入Collectors.xxx指定类型             |
| toArray()   | 将流转换为数组                                               |
| iterator()  | 将流转换为Iterator对象                                       |
| foreach()   | 无返回值，对元素进行遍历执行指定逻辑                         |

### Stream流API使用

- 数据准备，准备实体类Person和Hobby类如下

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class Person {
      private String name;
      private Integer age;
      private List<Hobby> hobbyList;
  }
  ```

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class Hobby {
      private String hobbyName;
      private String hobbyContent;
  }
  
  ```

- 测试类准备如下数据

```java
  List<Hobby> hobbyList = Arrays.asList(
            new Hobby("play", "玩游戏玩游戏玩游戏"),
            new Hobby("watch", "看电视剧看电视剧"),
            new Hobby("exercise", "锻炼锻炼锻炼"));

    List<Hobby> hobbyList1 = Arrays.asList(
            new Hobby("play", "玩游戏玩游戏玩游戏"),
            new Hobby("work", "上班狂人"));

    List<Hobby> hobbyList2 = Arrays.asList(
            new Hobby("play", "玩游戏玩游戏玩游戏"),
            new Hobby("watch", "看电视剧看电视剧"));

    List<Person> personList = Arrays.asList(new Person("小明", 18, hobbyList),
            new Person("小王", 20, hobbyList1),
            new Person("小红", 16, hobbyList2));
    Person ming = new Person("小明", 18, hobbyList);
    Person wang = new Person("小王", 20, hobbyList1);
    Person hong = new Person("小红", 16, hobbyList2);
```



#### 开始管道

##### 1.集合直接创建Stream流

```
        List<Person> list = personList.stream().collect(Collectors.toList());
        list.forEach(people -> {
            System.out.println(people);
        });
```



##### 2.元素创建Stream流

```
   List<Person> list2 = Stream.of(ming, wang, hong).collect(Collectors.toList());
        list2.forEach(people -> {
            System.out.println(people);
        });
```



##### 3.创建并行流

```
        List<Person> list3 = personList.parallelStream().filter(person -> {
            System.out.println(Thread.currentThread().getName());
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            if (person.getAge() < 18) return false;
            return true;
        }).collect(Collectors.toList());
        list3.forEach(people -> {
            System.out.println(people);
        });
```

