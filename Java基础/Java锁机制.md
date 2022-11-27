# Java锁机制

## Java内存模型(Java Memory Model)：TODO

- JMM是一种规范，描述了java程序中的线程共享变量的访问规则。
- JMM规定所有共享变量都必须放在主内存，每个线程在操作共享变量的时候只能在自己的工作内存中完成。
- 不同线程不能直接读取对方工作副本中的变量。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpzhiaXUhn9W2XjuqeziaG1ibdvOgPyiaPib3U7oR6ZS77CqlAVp7BkTxS30UhDN1X6YJRfCGQadBP6xd9Q/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

### 交互协议

- Lock：作用于主内存，将变量标记为线程独占状态
- Read：作用于主内存，将变量从主内存运输到工作内存
- Load：作用于工作内存，把Read读到的变量装载到工作内存。
- Store：作用于工作内存，把工作内存的值传递到主内存
- Write：作用于主内存，把Store传递的值写入到主内存。
- Unlock：作用于主内存，将变量从锁定状态释放，释放后的变量才能被其他线程重新锁定。
- Use：**↓**
- Assign：**这俩待办，暂时理解不了。**

### 特性

- 原子性
- 可见性
- 有序性

## Volatile

### 基本概念

- Volatile是一个关键字，用来修饰变量，被Volatile修饰的变量能够保证可见性和有序性，但不能保证原子性。

### 原理

- Volatile保证通过MESI（缓存一致性协议）以及总线嗅探机制来保证可见性。

- Volatile通过禁止指定重排序来保证有序性。（通过插入内存屏障指令来禁止重排序）

#### 缓存一致性协议（MESI）

- 简答理解，当CPU写数据时，如果发现操作的数据是共享变量，即在其他CPU也存在工作副本，那么他会发出信号通知其他CPU将该变量的缓存设置为无效状态。因此当下次这个CPU需要读取该副本时，发现其无效则需重新到主存中读取。

#### 总线嗅探机制

- 每个处理器通过嗅探在总线上传播的数据来检查自己的值是否过期，当处理器发现自己缓存行的内存地址被修改，就会将该缓存行设置为无效。

#### 内存屏障

- ![内存屏障分类表](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/2/16320e796e1471c0~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

- ![volatile写插入内存屏障示意图](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/5/2/16320e796e03b351~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

- 在Volatile写之前插入StoreStore屏障，禁止Volatile和上面的普通写/读进行重排序，在Volatile写之后插入StoreLoad屏障，禁止Volatile写后面的Volatile读/写重排序。
- 在Volatile读之后插入LoadLoad屏障，禁止Volatile读与之后的普通读、Volatile读发生重排序，读之后插入LoadStore屏障，禁止Volatile读与之后的普通写发生重排序。

## synchronized

### 基本概念

- Synchronized是一个关键字，可以保证方法或代码块运行时，同一时刻只有一个方法或一个代码块进入到临界区。Synchronized的锁是通过JVM实现的。

### 特性

- Synchronized关键字可以保证原子性和可见性以及有序性。
- Synchronized锁是不公平锁，锁可重入，不可中断。

### 原理

- Jav对象头中的MakrWord记录锁标志状态，指向一个monitor对象，monitior对象中记录了锁的持有者（owner），阻塞队列（EntryList），等待队列（waitSet）。

- Synchronized通过JVM级别指令来完成加锁，依赖Monitor监视器来完成。
- 当Synchronized锁代码块时，会使用monitorenter和monitorexit来完成对代码块的加锁和解锁。当Sychronized修饰方法时，会标记该方法为ACC_SYNCHRONIZED，JVM会根据这个标记来判断是否是同步方法。







## Lock