# Java基础



## 集合

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java-collection-hierarchy.png)



- Collection是集合



### List

#### ArrayList

##### 数据结构

- ArrayList底层数据结构是数组，与Java数组相比，它能够实现动态增长。可以理解为动态数组。

##### 特性：

- ArrayList随机查询效率高，可以通过数组下标快速定位。查询复杂度为O(1)。
- ArrayList在插入元素时，如果不指定元素下标（尾插），效率较高，时间复杂度为O(1)，源码中只需要确认数组容量之后即可插入。（ensureCapitityInternal）。指定元素下标时（头插或中间插入），需要额外调用native方法copy一份新的数组，并将原来的数组放入。
- ArrayList修改元素时，可以根据index快速定位下标，时间复杂度为O（1）。
- ArrayList扩容，默认情况下，ArrayList是懒加载的，使用时才会初始化构造方法，默认容量为10。每次扩容，grow方法的扩容大小是原数组+原数组>>1。即扩容到原来的1.5倍。
- ArrayList删除元素时不会改变数组容量大小，如果需要缩小数组容量，需要调用trimTosize方法（）

#### LinkedList

##### 数据结构

- LinkedList底层数据结构是双向链表

##### 特性：

- LinkedList随即查询效率极低，getFirst和getLast时间复杂度为O(1)，如果要指定索引来查询，只能通过遍历链表的方式来查询。遍历链表时，根据链表大小判断从头遍历还是从尾遍历，最坏情况下需要遍历n/2次才能获取元素。
- LinkedList在添加元素时，添加头部和尾部指针复杂度都为O(1)，添加指定index的元素时，不需要调用native方法复制数组。只需要根据index遍历链表获取该结点的原节点，将新节点插入原节点，交换指针。

#### Vector

##### 数据结构

- 和ArrayList一样，所有读写方法使用syncornized修饰。

##### 特性：

- Vector可以理解为ArrayList的线程安全版本，但是锁的粒度太大，对于读多写少的场景，更推荐使用CopyOnwirteArrayList

#### CopyOnWriteArrayList（写时复制）

##### 数据结构

- 使用ReentrantLock加写锁。内部还维护了一个由volatile修饰的数组。

##### 特性

- Cow内部的数组由volatile修饰，对于写操作，拿到锁后，将原来的数组调用copyof方法复制到新数组中。
- Cow读操作并不加锁，这也是他优于Vector的一个因素。

#### 快速失败与安全失败

- 快速失败和安全失败是一种思想，属于系统设计范畴。

##### 快速失败（fail-fast）

- 系统运行中，如果有错误，则立即停止。
- 集合就是快速失败的思想。

###### 例子

- 用迭代器遍历一个集合对象时，如果遍历途中有别的线程（或自己）修改了元素，则会抛出ConcurrentModificationException。
- 原理是迭代器遍历集合的时候，会保存一个modCount的值，每当迭代器遍历下一个元素之前，都会检查modCount的值是否变化，如果发生变化，则抛出ConcurrentModificationException。

##### 安全失败（fail-safe）

- 系统运行中，如果有错误，它会忽略错误（但是会有地方记录下来），并不会停止，仍然继续运行。
- 并发包下的集合是安全失败的思想。

###### 例子

- 并发包下的集合，迭代器访问会先拷贝一份原有的集合，再拷贝的集合上进行遍历，所以遍历不会抛出ConcurrentModificationException。

### Map

#### HashMap

##### 数据结构

- JDK1.7时，采用数组+链表。
- JDK1.8时，采用数组+链表+红黑树

##### 特性

###### 查询

- HashMap查询效率：理想情况下，当key均匀分布在hash表的桶位上，且没有使用链表，查询复杂度为O(1),当hash表桶里有链表元素时，最长的时间复杂度为O(n)。当链表升级到红黑树时，查询复杂度为O(logn)

###### 新增(put):

- 检查当前hash表是否做过初始化，若没有，进行初始化，初始容量为16。
- 计算hash值，hashCode与自身>>16位做异或运算。主要是为了hash值的离散性（尽可能减少hash碰撞）。
- 为什么要使用异或运算？如下，可以看到，使用and运算时，为1的概率为25%,or运算时，为1的概率为75。只有当异或运算时，概率才能均匀分布。

```shell
and运算
1&1=1; true&&true=true;
1&0=0; true&&false=false;
0&1=0; false&&true=false;
0&0=0; false&&false=false
or运算
1or1=1;
1or0=1;
0or1=1;
0or0=0;
^运算（异或运算）
1^1=1;
1^0=0;
0^1=0;
0^0=1;
```

- 根据hash值&table.length-1得到桶的位置。
- 桶上若无key，直接插入。桶上有值时，调用equals方法比较元素是否为桶上的某个key。需要看当前桶上的数据结构是链表还是红黑树来进行分析。如果是链表，则将其插在链表的尾部，插入完毕后，检查链表长度是否需要树化，若链表长度>=8，且hash表容量>64，则进行树化，如果链表长度>=8但hash表容量<64，则对hash表进行扩容，扩容后将重新计算hash值并将元素重新分配桶位。如果是红黑树，则将元素插入到红黑树中。

#### ConcurrentHashMap

##### 数据结构

- JDK1.7时，ConcurrentHashMap采用segment+hashEntry，可以理解为在segment数组里又包了一层hashmap。JDK1.8时，采用数组+链表+红黑树。
- JDK1.7时，由于segment继承了ReentrantLock，最大容量为16，也就是说最多支持16并发。JDK1.8时，锁的粒度精确到了hash桶，并发容量取决于当前hash桶的容量。

##### 特性

###### 新增（JDK1.7）

- JDK1.7时，先根据hash算法定位到segment数组。
- 判断当前位置有没有被初始化，若无，则初始化。
- 调用segment的put方法，获取锁，如果失败，则自旋获取锁，成功获取锁后再hash定位到具体的hash桶位。
- 接下来就和hashmap的插入差不多。

###### 新增（JDK1.8）

- JDK1.8时，先根据hash算法定位到hash桶。
- 判断是否需要初始化。若需要，则初始化。
- 定位到Node节点，拿到首节点f，并判断。
  - 如果首先点为null,则CAS添加元素。
  - 如果当前位置的hashcode == moved == -1那说明别的线程正在进行扩容。帮助扩容
  - 如果上述都不满足，使用synchronized加锁，遍历添加元素，和hashmap类似。（根据链表或红黑树）

###### 查询

- 和HashMap基本一样。

#### HashTable

- HashMap的线程安全版本，读写都使用synchornized方法修饰，锁粒度大，不推荐使用。
