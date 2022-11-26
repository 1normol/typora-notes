# CompletableFuture



## Future

​	在JDK8之前，要想使用多线程处理带返回值的任务，只能使用callable得到Future对象，Future对象可以监视目标线程调用call的情况，通过get方法阻塞获取结果。

### Futrue常用API

- cancel，取消Callable的执行，当Callable还没有完成时
- get，获得Callable的返回值
- isCanceled，判断是否取消了
- isDone，判断是否完成

### Futrue优缺点

- 一定程度上能够让线程池内的任务异步执行
- 传统回调最大的问题就是不能将控制流分离到不同的事件处理器中。例如主线程等待各个异步执行的线程返回的结果来做下一步操作，则必须阻塞在future.get()的地方等待结果返回。这时候又变成同步了

## CompletableFuture介绍

​	CompletableFuture在JDK8引入，它实现了Future和CompletionStage接口，在保留了Future的优点的同时，弥补了起不足。

![image-20221116141337142](C:\Users\35541\AppData\Roaming\Typora\typora-user-images\image-20221116141337142.png)

## CompletableFuture常用API

### 1. 创建异步任务的方法

- 提供**runAsync** 和 **supplyAsync**方法来创建一个异步任务，

- runAsync方法不支持返回值。

- supplyAsync可以支持返回值。

  ```java
  public static CompletableFuture<Void> runAsync(Runnable runnable)
  public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
  public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
  public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
  ```

#### Demo示例

```java
//使用runAsync，无返回值。        
CompletableFuture.runAsync(()-> System.out.println(1+1));
//使用supplyAsync，有返回值
NetMall mall = CompletableFuture.supplyAsync(() -> new NetMall("京东")).get();
System.out.println(mall);
```



### 2. 计算结果完成时的回调方法

- 当一个**completableFuture**完成时，提供**whenComplete**方法，继续执行特定的Action。

- **whenComplete**将在执行当前任务的线程继续执行接下来的action。
- **whenCompleteAsync**后者将把接下来的action继续提交给线程池执行。

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```

#### Demo示例

```java
       Person person = new Person("小宝", 22, null);
        ExecutorService threadPool = Executors.newSingleThreadExecutor();
        Person newPerson = CompletableFuture.supplyAsync(() -> {
                    System.out.println("当前线程是:" + Thread.currentThread().getName());
                    return PersonUtil.playGame(person);
                })
                .whenComplete((person1, throwable) -> {
                    System.out.println("当前线程是:" + Thread.currentThread().getName());
                    PersonUtil.takeShower(person1);
                }).get();

        System.out.println(newPerson);
        //指定whenCompleteAsync执行的线程池，可以看到action在我们自定义的线程池中执行
        Person newPerson1 = CompletableFuture.supplyAsync(() -> {
            System.out.println("当前线程是:" + Thread.currentThread().getName());
            return PersonUtil.playGame(person);
        }).whenCompleteAsync((person1, throwable) -> {
            PersonUtil.takeShower(person1);
            System.out.println("当前线程是:" + Thread.currentThread().getName());
        }, threadPool).get();
        System.out.println(newPerson1);
```



### 3. thenApply方法

- 当一个线程依赖另一个线程的结果时，可以使用thenApplay使这两个线程串行化

- Function<? super T,? extends U> T：上一个任务返回结果的类型 U：当前任务的返回值类型

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```

#### Demo示例

```java
     Person person = CompletableFuture
                .supplyAsync(() -> new Person("小宝", 22, null))
                .thenApply(person1 -> {
                    person1.setState("阳光向上的");
                    return person1;
                })
                .get();
        System.out.println(person);
```

### 4. handle方法

- handler和thenApply方法处理方式和用途基本一样。
- handler在任务完成之后再执行，可以处理带异常的任务。thenApply只能执行正常的任务，任务出现异常不会继续执行。

```java
public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn,Executor executor);
```

#### Demo示例

```java
      Person person = CompletableFuture
                .supplyAsync(() -> new Person("小宝", 22, null))
                .handle((person1, throwable) -> {
                    if (throwable == null) {
                        person1.setState("健康的");
                        return person1;
                    } else {
                        System.out.println(throwable.getMessage());
                        return null;
                    }
                })
                .get();
        System.out.println(person);
```



### 5. thenAccept 消费处理结果

- 接收上一个任务的结果，并消费，无返回结果。

```java
public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action,Executor executor);
```

#### Demo示例

```java
   CompletableFuture.supplyAsync(() -> new Person("小张", 0, "开心的"))
                .thenAccept(person -> {
                    person.setAge(18);
                    System.out.println(person);
                });
```



### 6. thenRun方法

- 与thenAccept方法相似，但不依赖上一个任务的结果。

```java
public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action,Executor executor);
```

#### Demo示例

```java
     Person person = new Person("小张", 11, null);
        CompletableFuture.supplyAsync(() -> PersonUtil.sleep(person))
                .thenRun(() -> System.out.println("run-run-run"));
```

### 7. thenCombine合并任务

- 该方法会把两个CompletionStage 任务的结果合并导thenCombine处理

```java
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn,Executor executor);
```

#### Demo示例

```java
 public void calculatePriceByCompletableFuture() throws Exception {
		//getPrice方法为得到价格 getDisCount为得到折扣。combine合并这两个结果并计算真实价格
        CompletableFuture<PriceResult> pddFuture = CompletableFuture
                .supplyAsync(() -> getPrice("PDD"))
                .thenCombine(CompletableFuture.supplyAsync(() -> getDisCount("PDD")), (price, disCount) -> computeRealPriceTT("PDD", disCount, price));

        CompletableFuture<PriceResult> jdFuture = CompletableFuture
                .supplyAsync(() -> getPrice("JD"))
                .thenCombine(CompletableFuture.supplyAsync(() -> getDisCount("JD")), (price, disCount) -> computeRealPriceTT("JD", disCount, price));

        CompletableFuture<PriceResult> tbFuture = CompletableFuture
                .supplyAsync(() -> getPrice("TB"))
                .thenCombine(CompletableFuture.supplyAsync(() -> getDisCount("TB")), (price, disCount) -> computeRealPriceTT("TB", disCount, price));

        PriceResult result = Stream.of(pddFuture.get(), jdFuture.get(), tbFuture.get()).min(((o1, o2) -> o1.getPrice().compareTo(o2.getPrice()))).get();
        System.out.println("最低价格平台为" + result.getName() + "价格为" + result.getPrice());
```



### 8. thenAccotBoth方法对结果进行消费

- 当两个CompletionStage执行完成之后，thenAccopBoth将对结果进行消费。

```java
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action,     Executor executor);
```

#### Demo示例

```java
        CompletableFuture
                .supplyAsync(() -> getPrice("PDD"))
                .thenAcceptBoth(CompletableFuture.supplyAsync(() -> getDisCount("PDD")),
                        (price, disCount) -> System.out.println(price * disCount)).get();
```



### 9. applyToEither方法

- 根据两个CompletionStage操作所消耗的时间来决定，谁的时间快，就使用它的结果进行下一步操作。

#### Demo示例

```java
        CompletableFuture.supplyAsync(() -> {
            try {
                System.out.println("速度快的任务");
                Thread.sleep(1000);
                return 1;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).acceptEitherAsync(CompletableFuture.supplyAsync(() -> {
            try {
                System.out.println("速度慢的任务");
                Thread.sleep(2000);
                return 2;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }), integer -> System.out.println(integer * 3)).get();
```



### 10. accptToEither方法

- 和applyToEither方法效果类似，选用执行速度快的completionStage的结果进行下一步消费。



### 11. runAfterEither方法

- 两个completionStage，任何一个完成了都会进行下一步操作，下一步操作传参为runnable，不支持结果消费。
- runAfterEither方法，一个completionStage执行完成之后不会再执行另一个，而applyToEither两个completionStage都会执行。

#### Demo示例

```java
   @Test
    public void t10() throws ExecutionException, InterruptedException {
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("快的任务");
                return 1;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).runAfterEither(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
                System.out.println("慢的任务");
                return 2;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }), () -> {
            System.out.println("执行runnable任务");
        }).get();
    }
```

### 12. runAfterBoth方法

- 两个completionStage，都执行完了才会进行下一步操作。传入任务为runnable类型，不支持结果消费。

#### Demo示例

```java
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("快的任务");
                return 1;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).runAfterBoth(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
                System.out.println("慢的任务");
                return 2;
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }), () -> {
            System.out.println("执行runnable任务");
        }).get();
```



### 13. thenCompose方法

- 该方法运行两个completionStage拼接为流水线操作，一个completionStage可以在得到另一个completionStage结果之后进行下一步操作。

#### Demo示例

```java
     CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> getPrice("PDD"));
        Double price = future.thenCompose(integer -> CompletableFuture.supplyAsync(() -> getDisCount("PDD") * integer)).get();
        System.out.println(price);
```

## CompletableFuture实践

### 1.准备工作

- 模拟从JD,TB,PDD三个平台获取最低商品价格。

- 具体接口不实现，使用线程睡眠模拟耗时。

- 准备实体类

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class PriceResult {
      private String name;
      private Integer oldPrice;
      private Double disCount;
      private Double price;
  }
  ```

- 准备工具类模拟耗时,模拟获取价格，折扣耗时。

  ```java
  public class ShopUtil {
      public static Double getDisCount(String type) {
          try {
              if ("PDD".equals(type)) {
                  Thread.sleep(900);
                  return 0.5;
              }
              if ("JD".equals(type)) {
                  Thread.sleep(1000);
                  return 0.9;
              }
              if ("TB".equals(type)) {
                  Thread.sleep(1100);
                  return 0.8;
              }
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          }
          throw new RuntimeException("不存在的类型");
      }
      
      public static Integer getPrice(String type) {
          try {
              if ("PDD".equals(type)) {
                  Thread.sleep(1000);
                  return 500;
              }
              if ("JD".equals(type)) {
                  Thread.sleep(1100);
                  return 600;
              }
              if ("TB".equals(type)) {
                  Thread.sleep(1000);
                  return 550;
              }
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          }
          throw new RuntimeException("不存在的类型");
      }
  ```

- 准备计算价格的工具类

  ```java
      public static PriceResult computeRealPrice(String type) {
          try {
              if ("PDD".equals(type)) {
                  PriceResult pdd = new PriceResult();
                  pdd.setName("PDD");
                  Integer price = getPrice(type);
                  pdd.setOldPrice(price);
                  System.out.println(Thread.currentThread().getName() + "获取PDD原价成功，" + "价格为" + price);
                  Double disCount = getDisCount(type);
                  pdd.setDisCount(disCount);
                  System.out.println(Thread.currentThread().getName() + "获取PDD折扣成功，" + "折扣为" + disCount);
                  pdd.setPrice(disCount * price);
                  System.out.println(Thread.currentThread().getName() + "PDD折扣后的价格为" + disCount * price);
                  return pdd;
              }
              if ("JD".equals(type)) {
                  PriceResult jd = new PriceResult();
                  jd.setName("JD");
                  Integer price = getPrice(type);
                  jd.setOldPrice(price);
                  System.out.println(Thread.currentThread().getName() + "获取JD原价成功，" + "价格为" + price);
                  Double disCount = getDisCount(type);
                  jd.setDisCount(disCount);
                  System.out.println(Thread.currentThread().getName() + "获取JD折扣成功，" + "折扣为" + disCount);
                  jd.setPrice(disCount * price);
                  System.out.println(Thread.currentThread().getName() + "JD折扣后的价格为" + disCount * price);
                  return jd;
              }
              if ("TB".equals(type)) {
                  PriceResult tb = new PriceResult();
                  tb.setName("TB");
                  Integer price = getPrice(type);
                  tb.setOldPrice(price);
                  System.out.println(Thread.currentThread().getName() + "获取TB原价成功，" + "价格为" + price);
                  Double disCount = getDisCount(type);
                  tb.setDisCount(disCount);
                  System.out.println(Thread.currentThread().getName() + "获取TB折扣成功，" + "折扣为" + disCount);
                  tb.setPrice(disCount * price);
                  System.out.println(Thread.currentThread().getName() + "TB折扣后的价格为" + disCount * price);
                  return tb;
              }
          } catch (Exception e) {
              throw new RuntimeException(e);
          }
          throw new RuntimeException("不存在的类型");
      }
  ```

### 2.常规串行执行查看结果

```java
    public void calculatePrice() throws Exception {
        PriceResult pdd = ShopServiceImpl.computeRealPrice("PDD");
        PriceResult jd = ShopServiceImpl.computeRealPrice("JD");
        PriceResult tb = ShopServiceImpl.computeRealPrice("TB");
        PriceResult result = Stream.of(pdd, jd, tb).min((o1, o2) -> o1.getPrice().compareTo(o2.getPrice())).get();
        System.out.println("最低价格平台为" + result.getName() + "价格为" + result.getPrice());
    }
```

结果如下：

```xml
main获取PDD原价成功，价格为500
main获取PDD折扣成功，折扣为0.5
mainPDD折扣后的价格为250.0
main获取JD原价成功，价格为600
main获取JD折扣成功，折扣为0.9
mainJD折扣后的价格为540.0
main获取TB原价成功，价格为550
main获取TB折扣成功，折扣为0.8
mainTB折扣后的价格为440.0
最低价格平台为PDD价格为250.0
串行执行costTime:6187
```

### 3.使用Future异步执行并查看结果

```java
    @Override
    public void calculatePriceByFuture() throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService threadPool = Executors.newCachedThreadPool();
        Future<PriceResult> pddFuture = threadPool.submit(() -> ShopServiceImpl.computeRealPrice("PDD"));
        Future<PriceResult> jdFuture = threadPool.submit(() -> ShopServiceImpl.computeRealPrice("JD"));
        Future<PriceResult> tbFuture = threadPool.submit(() -> ShopServiceImpl.computeRealPrice("TB"));
        PriceResult result = Stream.of(pddFuture.get(), jdFuture.get(), tbFuture.get()).min(Comparator.comparing(PriceResult::getPrice)).get();
        System.out.println("最低价格平台为" + result.getName() + "价格为" + result.getPrice());
    }
```

结果如下

```xml
pool-1-thread-3获取TB原价成功，价格为550
pool-1-thread-1获取PDD原价成功，价格为500
pool-1-thread-2获取JD原价成功，价格为600
pool-1-thread-1获取PDD折扣成功，折扣为0.5
pool-1-thread-1PDD折扣后的价格为250.0
pool-1-thread-2获取JD折扣成功，折扣为0.9
pool-1-thread-2JD折扣后的价格为540.0
pool-1-thread-3获取TB折扣成功，折扣为0.8
pool-1-thread-3TB折扣后的价格为440.0
最低价格平台为PDD价格为250.0
线程池+Future执行costTime:2130
```

