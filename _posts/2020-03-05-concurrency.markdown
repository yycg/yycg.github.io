---
layout:     post
title:      "并发"
subtitle:   ""
date:       2020-03-05 12:00:00
author:     "盈盈冲哥"
header-img: "img/fleabag.jpg"
mathjax: true
catalog: true
tags:
    - 学习
---

## 并发容器

- 并发容器

  **JDK 提供的并发容器总结**

  - ConcurrentHashMap: 线程安全的 HashMap
  - CopyOnWriteArrayList: 线程安全的 List，在读多写少的场合性能非常好，远远好于 Vector.
  - ConcurrentLinkedQueue: 高效的并发队列，使用链表实现。可以看做一个线程安全的 LinkedList，这是一个非阻塞队列。
  - BlockingQueue: 这是一个接口，JDK 内部通过链表、数组等方式实现了这个接口。表示阻塞队列，非常适合用于作为数据共享的通道。
  - ConcurrentSkipListMap: 跳表的实现。这是一个 Map，使用跳表的数据结构进行快速查找。

  **CopyOnWriteArrayList**

  在很多应用场景中，读操作可能会远远大于写操作。由于读操作根本不会修改原有的数据，因此对于每次读取都进行加锁其实是一种资源浪费。我们应该允许多个线程同时访问 List 的内部数据，毕竟读取操作是安全的。

  这和我们之前在多线程章节讲过 ReentrantReadWriteLock 读写锁的思想非常类似，也就是读读共享、写写互斥、读写互斥、写读互斥。JDK 中提供了 CopyOnWriteArrayList 类比相比于在读写锁的思想又更进一步。为了将读取的性能发挥到极致，CopyOnWriteArrayList 读取是完全不用加锁的，并且更厉害的是：写入也不会阻塞读取操作。只有写入和写入之间需要进行同步等待。这样一来，读操作的性能就会大幅度提升。那它是怎么做的呢？

  CopyOnWriteArrayList 类的所有可变操作（add，set 等等）都是通过创建底层数组的新副本来实现的。**当 List 需要被修改的时候，我并不修改原有内容，而是对原有数据进行一次复制，将修改的内容写入副本。写完之后，再将修改完的副本替换原来的数据，这样就可以保证写操作不会影响读操作了。**

  从 CopyOnWriteArrayList 的名字就能看出CopyOnWriteArrayList 是满足CopyOnWrite 的 ArrayList，所谓CopyOnWrite 也就是说：在计算机，如果你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉了。

  **读取操作没有任何同步控制和锁操作，理由就是内部数组 array 不会发生修改，只会被另外一个 array 替换，因此可以保证数据安全。**

  CopyOnWriteArrayList **写入操作 add() 方法在添加集合的时候加了锁，保证了同步，避免了多线程写的时候会 copy 出多个副本出来。**

  **ConcurrentLinkedQueue**

  Java 提供的线程安全的 Queue 可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是 BlockingQueue，非阻塞队列的典型例子是 ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。 阻塞队列可以通过加锁来实现，非阻塞队列可以通过 CAS 操作实现。

  从名字可以看出，ConcurrentLinkedQueue这个队列使用链表作为其数据结构．ConcurrentLinkedQueue 应该算是在高并发环境中性能最好的队列了。它之所有能有很好的性能，是因为其内部复杂的实现。

  ConcurrentLinkedQueue 内部代码我们就不分析了，大家知道 **ConcurrentLinkedQueue 主要使用 CAS 非阻塞算法**来实现线程安全就好了。

  ConcurrentLinkedQueue 适合在对性能要求相对较高，同时对队列的读写存在多个线程同时进行的场景，即如果对队列加锁的成本较高则适合使用无锁的 ConcurrentLinkedQueue 来替代。

  **BlockingQueue**

  上面我们己经提到了 ConcurrentLinkedQueue 作为高性能的非阻塞队列。下面我们要讲到的是阻塞队列——BlockingQueue。阻塞队列（BlockingQueue）被广泛使用在“生产者-消费者”问题中，其原因是 BlockingQueue 提供了可阻塞的插入和移除的方法。当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止。

  BlockingQueue 是一个接口，继承自 Queue，所以其实现类也可以作为 Queue 的实现来使用，而 Queue 又继承自 Collection 接口。

  下面主要介绍一下:ArrayBlockingQueue、LinkedBlockingQueue、PriorityBlockingQueue，这三个 BlockingQueue 的实现类。

  - ArrayBlockingQueue

    ArrayBlockingQueue 是 BlockingQueue 接口的有界队列实现类，底层采用数组来实现。ArrayBlockingQueue 一旦创建，容量不能改变。其并发控制采用可重入锁来控制，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。当队列容量满时，尝试将元素放入队列将导致操作阻塞;尝试从一个空队列中取一个元素也会同样阻塞。

    ArrayBlockingQueue 默认情况下不能保证线程访问队列的公平性，所谓公平性是指严格按照线程等待的绝对时间顺序，即最先等待的线程能够最先访问到 ArrayBlockingQueue。而非公平性则是指访问 ArrayBlockingQueue 的顺序不是遵守严格的时间顺序，有可能存在，当 ArrayBlockingQueue 可以被访问时，长时间阻塞的线程依然无法访问到 ArrayBlockingQueue。如果保证公平性，通常会降低吞吐量。

  - LinkedBlockingQueue

    LinkedBlockingQueue 底层基于单向链表实现的阻塞队列，可以当做无界队列也可以当做有界队列来使用，同样满足 FIFO 的特性，与 ArrayBlockingQueue 相比起来具有更高的吞吐量，为了防止 LinkedBlockingQueue 容量迅速增，损耗大量内存。通常在创建 LinkedBlockingQueue 对象时，会指定其大小，如果未指定，容量等于 Integer.MAX_VALUE。

  - PriorityBlockingQueue

    PriorityBlockingQueue 是一个支持优先级的无界阻塞队列。默认情况下元素采用自然顺序进行排序，也可以通过自定义类实现 compareTo() 方法来指定元素排序规则，或者初始化时通过构造器参数 Comparator 来指定排序规则。

    PriorityBlockingQueue 并发控制采用的是 ReentrantLock，队列为无界队列（ArrayBlockingQueue 是有界队列，LinkedBlockingQueue 也可以通过在构造函数中传入 capacity 指定队列最大的容量，但是 PriorityBlockingQueue 只能指定初始的队列大小，后面插入元素的时候，如果空间不够的话会自动扩容）。

    简单地说，它就是 PriorityQueue 的线程安全版本。不可以插入 null 值，同时，插入队列的对象必须是可比较大小的（comparable），否则报 ClassCastException 异常。它的插入操作 put 方法不会 block，因为它是无界队列（take 方法在队列为空的时候会阻塞）。
  
  **ConcurrentSkipListMap**

  使用跳表实现 Map 和使用哈希算法实现 Map 的另外一个不同之处是：哈希并不会保存元素的顺序，而跳表内所有的元素都是排序的。因此在对跳表进行遍历时，你会得到一个有序的结果。所以，如果你的应用需要有序性，那么跳表就是你不二的选择。JDK 中实现这一数据结构的类是 ConcurrentSkipListMap。

- 如何线程安全地遍历List：Vector、CopyOnWriteArrayList

  > [https://blog.csdn.net/xiao__gui/article/details/51050793](https://blog.csdn.net/xiao__gui/article/details/51050793)

  **Vector**

  Vector和ArrayList类似，是长度可变的数组，与ArrayList不同的是，Vector是线程安全的，它给几乎所有的public方法都加上了synchronized关键字。由于加锁导致性能降低，在不需要并发访问同一对象时，这种强制性的同步机制就显得多余，所以现在Vector已被弃用。

  **HashTable**
  
  HashTable和HashMap类似，不同点是HashTable是线程安全的，它给几乎所有public方法都加上了synchronized关键字，还有一个不同点是HashTable的K，V都不能是null，但HashMap可以，它现在也因为性能原因被弃用了。

  **Collections包装方法**
  
  Vector和HashTable被弃用后，它们被ArrayList和HashMap代替，但它们不是线程安全的，所以Collections工具类中提供了相应的包装方法把它们包装成线程安全的集合。

  ```java
  List<E> synArrayList = Collections.synchronizedList(new ArrayList<E>());

  Set<E> synHashSet = Collections.synchronizedSet(new HashSet<E>());

  Map<K,V> synHashMap = Collections.synchronizedMap(new HashMap<K,V>());
  ```

  Collections针对每种集合都声明了一个线程安全的包装类，在原集合的基础上添加了锁对象，集合中的每个方法都通过这个锁对象实现同步。

  **ConcurrentHashMap**
  
  ConcurrentHashMap和HashTable都是线程安全的集合，它们的不同主要是加锁粒度上的不同。HashTable的加锁方法是给每个方法加上synchronized关键字，这样锁住的是整个Table对象。而ConcurrentHashMap是更细粒度的加锁。

  在JDK1.8之前，ConcurrentHashMap加的是分段锁，也就是Segment锁，每个Segment含有整个table的一部分，这样不同分段之间的并发操作就互不影响。

  JDK1.8对此做了进一步的改进，它取消了Segment字段，直接在table元素上加锁，实现对每一行进行加锁，进一步减小了并发冲突的概率。

  **CopyOnWriteArrayList和CopyOnWriteArraySet**

  它们是加了写锁的ArrayList和ArraySet，锁住的是整个对象，但读操作可以并发执行

  除此之外还有ConcurrentSkipListMap、ConcurrentSkipListSet、ConcurrentLinkedQueue、ConcurrentLinkedDeque等，至于为什么没有ConcurrentArrayList，原因是无法设计一个通用的而且可以规避ArrayList的并发瓶颈的线程安全的集合类，只能锁住整个list，这用Collections里的包装类就能办到。

- 为什么java.util.concurrent 包里没有并发的ArrayList实现？

  > [http://ifeve.com/why-is-there-not-concurrent-arraylist-in-java-util-concurrent-package/](http://ifeve.com/why-is-there-not-concurrent-arraylist-in-java-util-concurrent-package/)

  在java.util.concurrent包中没有加入并发的ArrayList实现的主要原因是：**很难去开发一个通用并且没有并发瓶颈的线程安全的List。**

  **像ConcurrentHashMap这样的类的真正价值（The real point / value of classes）并不是它们保证了线程安全。而在于它们在保证线程安全的同时不存在并发瓶颈。举个例子，ConcurrentHashMap采用了锁分段技术和弱一致性的Map迭代器去规避并发瓶颈。**

  所以问题在于，像“Array List”这样的数据结构，你不知道如何去规避并发的瓶颈。**拿contains() 这样一个操作来说，当你进行搜索的时候如何避免锁住整个list？**

  另一方面，Queue 和Deque (基于Linked List)有并发的实现是因为他们的接口相比List的接口有更多的限制，这些限制使得实现并发成为可能。

  CopyOnWriteArrayList是一个有趣的例子，它规避了只读操作（如get/contains）并发的瓶颈，但是它为了做到这点，在修改操作中做了很多工作和修改可见性规则。 此外，修改操作还会锁住整个List，因此这也是一个并发瓶颈。所以从理论上来说，CopyOnWriteArrayList并不算是一个通用的并发List。

- ConcurrentHashMap和HashTable的区别

  - **底层数据结构：** JDK1.7的 ConcurrentHashMap 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。HasTable 和 JDK1.8 之前的 HashMap 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；

  - **实现线程安全的方式（重要）：** 

    ① **在JDK1.7的时候，ConcurrentHashMap（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 

    > ConcurrentHashMap 类中包含两个静态内部类 HashEntry 和 Segment。HashEntry 用来封装映射表的键 / 值对；Segment 用来充当锁的角色，每个 Segment 对象守护整个散列映射表的若干个桶。每个桶是由若干个 HashEntry 对象链接起来的链表。一个 ConcurrentHashMap 实例中包含由若干个 Segment 对象组成的数组。HashEntry 用来封装散列映射表中的键值对。**在 HashEntry 类中，key，hash 和 next 域都被声明为 final 型，value 域被声明为 volatile 型。**
    >
    >  在ConcurrentHashMap 中，在散列时如果产生“碰撞”，将采用“分离链接法”来处理“碰撞”：把“碰撞”的 HashEntry 对象链接成一个链表。**由于 HashEntry 的 next 域为 final 型，所以新节点只能在链表的表头处插入。**

    **到了 JDK1.8 的时候已经摒弃了Segment的概念，则是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6以后 对 synchronized锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本；

    ② **HashTable(同一把锁)** :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

  - HashTable

    ![HashTable全表锁](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/HashTable%E5%85%A8%E8%A1%A8%E9%94%81.png)

  - JDK1.7的ConcurrentHashMap

    ![JDK1.7的ConcurrentHashMap](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/ConcurrentHashMap%E5%88%86%E6%AE%B5%E9%94%81.jpg)

  - JDK1.8的ConcurrentHashMap（TreeBin: 红黑二叉树节点 Node: 链表节点）

    ![JDK1.8的ConcurrentHashMap](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/JDK1.8-ConcurrentHashMap-Structure.jpg)

- ConcurrentHashmap读操作加锁吗，不加锁volatile修饰共享变量，为什么volatile能实现读操作线程安全

  不加锁，volatile能够保证内存可见性。

  当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。

  当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效，线程接下来将从主内存中读取共享变量。

- ConcurrentHashMap如何做到get操作不加锁？

  get方法中将要使用的共享变量都定义成volatile类型，定义成volatile的变量，只能被单线程写，但能被多个线程同时读，在线程之间保持可见性，保证不会读到过期的值。在get操作中只需要读不需要写共享变量get和value，所以可以不用加锁，之所以不会读到过期的值，是因为JMM的happen before规则，对volatile字段的写先于读。

- **ConcurrentHashMap如何在resize中并发的插入、删除和查找**

  利用CAS+synchronized实现node节点粒度的并发

  resize的时候单线程构建一个nextTable（2倍原容量）、多线程扩容
  
  put的时候如果检测到需要插入的位置被forward节点占有，就帮助扩容、如果检测到的节点非空且不是forward节点，对节点加syn锁，进行节点插入
  
  get的时候不加锁，可多线程查找
  
  remove的时候如果检测到需要删除的位置被forward节点占有，就帮助扩容、如果不是，则对节点加syn锁，进行节点删除

- concurrentHashmap的size实现

  > [https://juejin.im/post/5ae75584f265da0b873a4810](https://juejin.im/post/5ae75584f265da0b873a4810)
  
  JDK 8 推荐使用mappingCount 方法，因为这个方法的返回值是 long 类型，不会因为 size 方法是 int 类型限制最大值（size 方法是接口定义的，不能修改）。

  在没有并发的情况下，使用一个 baseCount volatile 变量就足够了，当并发的时候，CAS 修改 baseCount 失败后，就会使用 CounterCell 类了，会创建一个这个对象，通常对象的 volatile value 属性是 1。在计算 size 的时候，会将 baseCount 和 CounterCell 数组中的元素的 value 累加，得到总的大小，但这个数字仍旧可能是不准确的。

- 快速失败(fail-fast)和安全失败(fail-safe)的区别

  > [https://blog.csdn.net/qq_31780525/article/details/77431970](<https://blog.csdn.net/qq_31780525/article/details/77431970>)

  - **在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。**

    - 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就返回遍历；否则抛出异常，终止遍历。

    - 注意：这里异常的抛出条件是检测到 modCount!=expectedmodCount 这个条件。如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。

    - 场景：**java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改）。**

  - **采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。**

    - 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

    - 缺点：**基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。**

    - 场景：**java.util.concurrent包下的容器都是安全失败，可以在多线程下并发使用，并发修改。**

  - 快速失败和安全失败是对迭代器而言的。 

    - 快速失败：当在迭代一个集合的时候，如果有另外一个线程在修改这个集合，就会抛出ConcurrentModification异常，java.util下都是快速失败。
    - 安全失败：在迭代时候会在集合二层做一个拷贝，所以在修改集合上层元素不会影响下层。在java.util.concurrent下都是安全失败

---

- List、Set、Map是否继承自Collection接口？

  List、Set 是，Map 不是。Map是键值对映射容器，与List和Set有明显的区别，而Set存储的零散的元素且不允许有重复元素（数学中的集合也是如此），List是线性结构的容器，适用于按数值索引访问元素的情形。

- 常用集合类以及主要方法

  > [https://blog.csdn.net/zhj870975587/article/details/50996811](<https://blog.csdn.net/zhj870975587/article/details/50996811>)

  > 若要检查Collection中的元素，可以使用foreach进行遍历，也可以使用迭代器，Collection支持iterator()方法，通过该方法可以访问Collection中的每一个元素。Set和List是由Collection派生的两个接口。

  - **Collection接口**
    - **List接口**：LinkedList类、ArrayList类
    - Vector类
    - Stack类
    - **Set接口**：HashSet类、TreeSet类
    - Queue类

  - **Map接口**：HashTable类、HashMap类、TreeMap类、LinkedHashMap类

- Collection 和 Collections的区别

  - Collection是集合类的上级接口，继承与他的接口主要有Set 和List.
  - Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。

- Iterator和ListIterator的区别

  - Iterator可用来遍历Set和List集合，但是ListIterator只能用来遍历List。Iterator对集合只能是前向遍历，ListIterator既可以前向也可以后向。
  - ListIterator实现了Iterator接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。

- 什么是迭代器？

  - Iterator提供了统一遍历操作集合元素的统一接口，Collection接口实现Iterable接口，每个集合都通过实现Iterable接口中iterator()方法返回Iterator接口的实例，然后对集合的元素进行迭代操作.
  - 有一点需要注意的是：在迭代元素的时候不能通过集合的方法删除元素，否则会抛出ConcurrentModificationException 异常. 但是可以通过Iterator接口中的remove()方法进行删除.

- 为什么集合类没有实现Cloneable和Serializable接口？

  - 克隆(cloning)或者是序列化(serialization)的语义和含义是跟具体的实现相关的。因此，应该由集合类的具体实现来决定如何被克隆或者是序列化。
  - 实现Serializable序列化的作用：将对象的状态保存在存储媒体中以便可以在以后重写创建出完全相同的副本；按值将对象从一个从一个应用程序域发向另一个应用程序域。
    实现 Serializable接口的作用就是可以把对象存到字节流，然后可以恢复。所以你想如果你的对象没有序列化，怎么才能进行网络传输呢？要网络传输就得转为字节流，所以在分布式应用中，你就得实现序列化。如果你不需要分布式应用，那就没必要实现实现序列化。

## 并发

- 多线程和java.util.concurrent并发包总结

  > [https://blog.csdn.net/jiyiqinlovexx/article/details/51175975](https://blog.csdn.net/jiyiqinlovexx/article/details/51175975)

  一、描述线程的类：Runable和Thread都属于java.lang包

  二、内置锁synchronized属于jvm关键字，内置条件队列操作接口Object.wait()/notify()/notifyAll()属于java.lang包

  二、提供内存可见性和防止指令重排的volatile属于jvm关键字

  四、而java.util.concurrent包(J.U.C)中包含的是java并发编程中有用的一些工具类，包括几个部分：

  1、collections部分：散落在java.util.concurrent包中，提供并发容器相关功能；

  2、atomic部分：包含在java.util.concurrent.atomic包中，提供原子变量类相关的功能，是构建非阻塞算法的基础；
  
  3、locks部分：包含在java.util.concurrent.locks包中，提供显式锁(互斥锁和速写锁)相关功能；

  4、tools部分：散落在java.util.concurrent包中，提供同步工具类，如信号量、闭锁、栅栏等功能；

  5、executor部分：散落在java.util.concurrent包中，提供线程池相关的功能；

- synchronized关键字

  - 说一说自己对于 synchronized 关键字的了解

    synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。

    另外，在 Java 早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。庆幸的是在 Java 6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了。JDK1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

  - 说说自己是怎么使用 synchronized 关键字，在项目中用到了吗

    synchronized关键字最主要的三种使用方式：

    **修饰实例方法**: 作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁

    **修饰静态方法**: 也就是给当前类加锁，会作用于类的所有对象实例，因为静态成员不属于任何一个实例对象，是类成员（ static 表明这是该类的一个静态资源，不管new了多少个对象，只有一份）。所以如果一个线程A调用一个实例对象的非静态 synchronized 方法，而线程B需要调用这个实例对象所属类的静态 synchronized 方法，是允许的，不会发生互斥现象，**因为访问静态 synchronized 方法占用的锁是当前类的锁，而访问非静态 synchronized 方法占用的锁是当前实例对象锁。**

    **修饰代码块**: 指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

    总结： **synchronized 关键字加到 static 静态方法和 synchronized(class)代码块上都是是给 Class 类上锁。synchronized 关键字加到实例方法上是给对象实例上锁。**尽量不要使用 synchronized(String a) 因为JVM中，字符串常量池具有缓存功能！

  - 讲一下 synchronized 关键字的底层原理

    synchronized 关键字底层原理属于 JVM 层面。

    ① synchronized 同步语句块的情况

    ```java
    public class SynchronizedDemo {
        public void method() {
            synchronized (this) {
                System.out.println("synchronized 代码块");
            }
        }
    }
    ```

    synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。 当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

    ② synchronized 修饰方法的的情况

    ```java
    public class SynchronizedDemo2 {
        public synchronized void method() {
            System.out.println("synchronized 方法");
        }
    }
    ```

    synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

  - 说说 JDK1.6 之后的synchronized 关键字底层做了哪些优化，可以详细介绍一下这些优化吗

    JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

    锁主要存在四种状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

  - 谈谈 synchronized和ReentrantLock 的区别

    ① 两者都是可重入锁

    两者都是可重入锁。“可重入锁”概念是：自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。

    ② synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API

    synchronized 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 synchronized 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。

    ReentrantLock 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

    ③ ReentrantLock 比 synchronized 增加了一些高级功能

    相比synchronized，ReentrantLock增加了一些高级功能。主要来说主要有三点：①等待可中断；②可实现公平锁；③可实现选择性通知（锁可以绑定多个条件）

    ReentrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。

    ReentrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。 

    ReentrantLock默认情况是非公平的，可以通过 ReentrantLock类的ReentrantLock(boolean fair)构造方法来制定是否是公平的。

    synchronized关键字与wait()和notify()/notifyAll()方法相结合可以实现等待/通知机制，ReentrantLock类当然也可以实现，但是需要借助于Condition接口与newCondition() 方法。Condition是JDK1.5之后才有的，它具有很好的灵活性，**比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。** 在使用notify()/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合Condition实例可以实现“选择性通知” ，这个功能非常重要，而且是Condition接口默认提供的。而synchronized关键字就相当于整个Lock对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而Condition实例的signalAll()方法 只会唤醒注册在该Condition实例中的所有等待线程。

    如果你想使用上述功能，那么选择ReentrantLock是一个不错的选择。

    ④ 性能已不是选择标准

  - synchronized与java.util.concurrent.locks.Lock的相同之处和不同之处

    > [https://blog.csdn.net/qq838642798/article/details/65441415](<https://blog.csdn.net/qq838642798/article/details/65441415>)

    ReenTrantLock可重入锁（和synchronized的区别）总结

    **可重入性：**

    从名字上理解，ReenTrantLock的字面意思就是再进入的锁，其实synchronized关键字所使用的锁也是可重入的，两者关于这个的区别不大。两者都是同一个线程每进入一次，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。

    **锁的实现：**

    **Synchronized是依赖于JVM实现的，而ReenTrantLock是JDK实现的**，有什么区别，说白了就类似于操作系统来控制实现和用户自己敲代码实现的区别。前者的实现是比较难见到的，后者有直接的源码可供阅读。

    **性能的区别：**

    在Synchronized优化以前，synchronized的性能是比ReenTrantLock差很多的，但是自从Synchronized引入了偏向锁，轻量级锁（自旋锁）后，两者的性能就差不多了，在两种方法都可用的情况下，官方甚至建议使用synchronized，其实synchronized的优化我感觉就借鉴了ReenTrantLock中的CAS技术。都是试图在用户态就把加锁问题解决，避免进入内核态的线程阻塞。

    **功能区别：**

    便利性：很明显**Synchronized的使用比较方便简洁，并且由编译器去保证锁的加锁和释放，而ReenTrantLock需要手工声明来加锁和释放锁，为了避免忘记手工释放锁造成死锁，所以最好在finally中声明释放锁。**

    锁的细粒度和灵活度：很明显ReenTrantLock优于Synchronized

    **ReenTrantLock独有的能力：**

    1. **ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。**

    1. **ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的诸线程，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。**

    1. ReenTrantLock提供了一种能够**中断等待锁**的线程的机制，通过lock.lockInterruptibly()来实现这个机制。

    **ReenTrantLock实现的原理：**

    在网上看到相关的源码分析，本来这块应该是本文的核心，但是感觉比较复杂就不一一详解了，简单来说，ReenTrantLock的实现是一种自旋锁，通过循环调用CAS操作来实现加锁。它的性能比较好也是因为避免了使线程进入内核态的阻塞状态。**想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。**

    **什么情况下使用ReenTrantLock：**

    答案是，如果你需要实现ReenTrantLock的三个独有功能时。

- volatile关键字

  **讲一下Java内存模型**

  在 JDK1.2 之前，Java的内存模型实现总是从主存（即共享内存）读取变量，是不需要进行特别的注意的。而在当前的 Java 内存模型下，线程可以把变量保存本地内存（比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。

  ![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/%E6%95%B0%E6%8D%AE%E4%B8%8D%E4%B8%80%E8%87%B4.png)

  要解决这个问题，就需要把变量声明为volatile，这就指示 JVM，这个变量是不稳定的，每次使用它都到主存中进行读取。

  说白了， volatile 关键字的主要作用就是保证变量的可见性然后还有一个作用是防止指令重排序。

  ![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/volatile%E5%85%B3%E9%94%AE%E5%AD%97%E7%9A%84%E5%8F%AF%E8%A7%81%E6%80%A7.png)

  **说说 synchronized 关键字和 volatile 关键字的区别**

  synchronized关键字和volatile关键字比较

  volatile关键字是线程同步的轻量级实现，所以volatile性能肯定比synchronized关键字要好。但是volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块。synchronized关键字在JavaSE1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升，实际开发中使用 synchronized 关键字的场景还是更多一些。

  多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会发生阻塞。

  volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。

  volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性。

- ThreadLocal

  **ThreadLocal简介**

  通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。如果想实现每一个线程都有自己的专属本地变量该如何解决呢？ JDK中提供的ThreadLocal类正是为了解决这样的问题。 ThreadLocal类主要解决的就是让每个线程绑定自己的值，可以将ThreadLocal类形象的比喻成存放数据的盒子，盒子中可以存储每个线程的私有数据。

  如果你创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的本地副本，这也是ThreadLocal变量名的由来。他们可以使用 get() 和 set() 方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题。

  再举个简单的例子：

  比如有两个人去宝屋收集宝物，这两个共用一个袋子的话肯定会产生争执，但是给他们两个人每个人分配一个袋子的话就不会出现这样的问题。如果把这两个人比作线程的话，那么ThreadLocal就是用来避免这两个线程竞争的。

  **ThreadLocal原理**

  最终的变量是放在了当前线程的 ThreadLocalMap 中，并不是存在 ThreadLocal 上，ThreadLocal 可以理解为只是ThreadLocalMap的封装，传递了变量值。 ThrealLocal 类中可以通过Thread.currentThread()获取到当前线程对象后，直接通过getMap(Thread t)可以访问到该线程的ThreadLocalMap对象。

  每个Thread中都具备一个ThreadLocalMap，而ThreadLocalMap可以存储以ThreadLocal为key的键值对。 比如我们在同一个线程中声明了两个 ThreadLocal 对象的话，会使用 Thread内部都是使用仅有那个ThreadLocalMap 存放数据的，**ThreadLocalMap的 key 就是 ThreadLocal对象，value 就是 ThreadLocal 对象调用set方法设置的值。**

  **ThreadLocal 内存泄露问题**

  ThreadLocalMap 中使用的 **key 为 ThreadLocal 的弱引用,而 value 是强引用**。所以，**如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉**。这样一来，ThreadLocalMap 中就会出现key为null的Entry。假如我们不做任何措施的话，value 永远无法被GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。使用完 ThreadLocal方法后 最好手动调用remove()方法。

  **弱引用介绍**

  如果一个对象只具有弱引用，那就类似于可有可无的生活用品。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。

  弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

  **Java的四种引用类型**

  上面的分析可知，无论是通过引用计数还是可达性分析的判断都用到了引用，那么引用是否可以被回收就至关重要了，如果一个引用要么可以被回收，要么就不能被回收，那对于一些“可回收”的对象就无能无力了，jdk1.2之后扩充了引用的概念，将引用分为强引用（Strong Reference），软引用（Soft Reference），弱引用（Weak Reference），虚引用（Phantom Reference），四种引用引用的强度依次逐渐减弱。

  **强引用**：程序中的普通对象赋值就是强引用，只要引用还在垃圾回收器就永远不会回收被引用的对象。

  **软引用**：描述还有用但并非必须的对象，在系统将要发生内存溢出异常之前，将会把这些对象放入回收范围内进行二次回收，如果还没有足够内存，才抛出异常。

  **弱引用**：也是用来描述非必须对象，强度更弱，弱引用关联的对象只能生存到下一次垃圾收集发生之前，无论内存是否足够都会被回收掉。

  **虚引用**：一个对象是否有虚引用的存在，不会对其生存时间产生任何影响，也无法通过虚引用获取对象实例，虚引用的唯一一个目的就是能在对象被回收时收到一个系统通知。

- 线程池

  - 使用线程池的好处

    池化技术相比大家已经屡见不鲜了，线程池、数据库连接池、Http 连接池等等都是对这个思想的应用。池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。

    线程池提供了一种限制和管理资源（包括执行一个任务）。 每个线程池还维护一些基本统计信息，例如已完成任务的数量。

    这里借用《Java 并发编程的艺术》提到的来说一下使用线程池的好处：

    - **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
    - **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
    - **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

  - Executors 返回线程池对象的弊端如下：

    - FixedThreadPool 和 SingleThreadExecutor ： 允许请求的队列长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致OOM。
    - CachedThreadPool 和 ScheduledThreadPool ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致OOM。

  - 内存足够的情况下，线程池里的线程越多越好吗

    也不是线程越多，程序的性能就越好，因为线程之间的调度和切换也会浪费CPU时间。

  - 核心线程池ThreadPoolExecutor内部参数

    ThreadPoolExecutor 3 个最重要的参数：

    - **corePoolSize : 核心线程数定义了最小可以同时运行的线程数量。**
    - **maximumPoolSize : 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。**
    - **workQueue: 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，信任就会被存放在队列中。**

    ThreadPoolExecutor其他常见参数:

    - **keepAliveTime:当线程池中的线程数量大于 corePoolSize 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁；**
    - unit : keepAliveTime 参数的时间单位。
    - threadFactory :executor 创建新线程的时候会用到。
    - **handler :饱和策略。**

    ![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%90%84%E4%B8%AA%E5%8F%82%E6%95%B0%E7%9A%84%E5%85%B3%E7%B3%BB.jpg)

    ThreadPoolExecutor 饱和策略定义:

    **如果当前同时运行的线程数量达到最大线程数量并且队列也已经被放满了时，ThreadPoolTaskExecutor 定义一些策略:**

    - **ThreadPoolExecutor.AbortPolicy：抛出 RejectedExecutionException来拒绝新任务的处理。**
    - ThreadPoolExecutor.CallerRunsPolicy：调用执行自己的线程运行任务，也就是直接在调用execute方法的线程中运行(run)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。另外，这个策略喜欢增加队列容量。如果您的应用程序可以承受此延迟并且你不能任务丢弃任何一个任务请求的话，你可以选择这个策略。
    - **ThreadPoolExecutor.DiscardPolicy： 不处理新任务，直接丢弃掉。**
    - **ThreadPoolExecutor.DiscardOldestPolicy： 此策略将丢弃最早的未处理的任务请求。**

  - 线程池的运行流程，使用参数以及方法策略

    ![img](/img/post/线程池.jpg)

    ![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/%E5%9B%BE%E8%A7%A3%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86.png)

    > [https://blog.csdn.net/u011240877/article/details/73440993](https://blog.csdn.net/u011240877/article/details/73440993)

    1. 当前池中线程比核心数少，新建一个线程执行任务

    2. 核心池已满，但任务队列未满，添加到队列中

    3. 核心池已满，队列已满，试着创建一个新线程

  - 实现Runnable接口和Callable接口的区别

    Runnable自Java 1.0以来一直存在，但Callable仅在Java 1.5中引入,目的就是为了来处理Runnable不支持的用例。**Runnable 接口不会返回结果或抛出检查异常，但是Callable 接口可以。所以，如果任务不需要返回结果或抛出异常推荐使用 Runnable 接口，这样代码看起来会更加简洁。**

    工具类 Executors 可以实现 Runnable 对象和 Callable 对象之间的相互转换。（Executors.callable（Runnable task）或 Executors.callable（Runnable task，Object resule））。

  - 执行execute()方法和submit()方法的区别是什么呢？

    1. **execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否**；
    2. **submit()方法用于提交需要返回值的任务。线程池会返回一个 Future 类型的对象，通过这个 Future 对象可以判断任务是否执行成功**，并且可以通过 Future 的 get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用 get（long timeout，TimeUnit unit）方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

  - 线程池都有哪些状态

    1.**RUNNING**：这是最正常的状态，接受新的任务，处理等待队列中的任务。线程池的初始化状态是RUNNING。线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0。

    2.**SHUTDOWN**：不接受新的任务提交，但是会**继续处理等待队列中的任务**。**调用线程池的shutdown()方法时，线程池由RUNNING -> SHUTDOWN。**

    3.**STOP**：不接受新的任务提交，**不再处理等待队列中的任务，中断正在执行任务的线程**。**调用线程池的shutdownNow()方法时，线程池由(RUNNING or SHUTDOWN ) -> STOP。**

    4.**TIDYING**：所有的任务都销毁了，workCount 为 0，线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()。因为terminated()在ThreadPoolExecutor类中是空的，所以用户想在线程池变为TIDYING时进行相应的处理；可以通过重载terminated()函数来实现。 

        当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。

        当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

    5.**TERMINATED**：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。

- Atomic 原子类

  - 介绍一下Atomic 原子类

    Atomic 翻译成中文是原子的意思。在化学上，我们知道原子是构成一般物质的最小单位，在化学反应中是不可分割的。在我们这里 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

    所以，所谓原子类说简单点就是具有原子/原子操作特征的类。

  - JUC 包中的原子类是哪4类?

    **基本类型**

    使用原子的方式更新基本类型

    - AtomicInteger：整形原子类
    - AtomicLong：长整型原子类
    - AtomicBoolean：布尔型原子类
    
    **数组类型**

    使用原子的方式更新数组里的某个元素

    - AtomicIntegerArray：整形数组原子类
    - AtomicLongArray：长整形数组原子类
    - AtomicReferenceArray：引用类型数组原子类
    
    **引用类型**

    - AtomicReference：引用类型原子类
    - AtomicStampedReference：原子更新引用类型里的字段原子类
    - AtomicMarkableReference ：原子更新带有标记位的引用类型
    
    **对象的属性修改类型**

    - AtomicIntegerFieldUpdater：原子更新整形字段的更新器
    - AtomicLongFieldUpdater：原子更新长整形字段的更新器
    - AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。

  - 能不能给我简单介绍一下 AtomicInteger 类的原理

    AtomicInteger 线程安全原理简单分析

    AtomicInteger 类的部分源码：

    ```java
    // setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
    ```
    
    AtomicInteger 类主要利用 **CAS (compare and swap) + volatile** 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

    **CAS的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址，返回值是 valueOffset。**另外 value 是一个volatile变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。

- AQS

  **AQS介绍**
  
  AQS 的全称为（AbstractQueuedSynchronizer），这个类在 java.util.concurrent.locks 包下面。

  AQS 是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 ReentrantLock，Semaphore，其他的诸如 ReentrantReadWriteLock，SynchronousQueue，FutureTask(jdk1.7) 等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器。

  **AQS原理**

  AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

  ![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/Java%20%E7%A8%8B%E5%BA%8F%E5%91%98%E5%BF%85%E5%A4%87%EF%BC%9A%E5%B9%B6%E5%8F%91%E7%9F%A5%E8%AF%86%E7%B3%BB%E7%BB%9F%E6%80%BB%E7%BB%93/CLH.png)

  **AQS 定义两种资源共享方式**

  1)Exclusive（独占）

  只有一个线程能执行，如 **ReentrantLock**。又可分为公平锁和非公平锁,ReentrantLock 同时支持两种锁,下面以 ReentrantLock 对这两种锁的定义做介绍：

  - **公平锁：按照线程在队列中的排队顺序，先到者先拿到锁**
  - **非公平锁：当线程要获取锁时，先通过两次 CAS 操作去抢锁，如果没抢到，当前线程再加入到队列中等待唤醒。**

  公平锁和非公平锁只有两处不同：

  1. 非公平锁在调用 lock 后，首先就会调用 CAS 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了。
  2. 非公平锁在 CAS 失败后，和公平锁一样都会进入到 tryAcquire 方法，在 tryAcquire 方法中，如果发现锁这个时候被释放了（state == 0），非公平锁会直接 CAS 抢锁，但是公平锁会判断等待队列是否有线程处于等待状态，如果有则不去抢锁，乖乖排到后面。

  公平锁和非公平锁就这两点区别，如果这两次 CAS 都不成功，那么后面非公平锁和公平锁是一样的，都要进入到阻塞队列等待唤醒。

  相对来说，非公平锁会有更好的性能，因为它的吞吐量比较大。当然，非公平锁让获取锁的时间变得更加不确定，可能会导致在阻塞队列中的线程长期处于饥饿状态。

  2)Share（共享）

  多个线程可同时执行，如 Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。

  ReentrantReadWriteLock 可以看成是组合式，因为 ReentrantReadWriteLock 也就是读写锁允许多个线程同时对某一资源进行读。

  不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS 已经在上层已经帮我们实现好了。

  **Semaphore(信号量)-允许多个线程同时访问**

  synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。

  执行 acquire 方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个 release 方法增加一个许可证，这可能会释放一个阻塞的 acquire 方法。然而，其实并没有实际的许可证这个对象，Semaphore 只是维持了一个可获得许可证的数量。 Semaphore 经常用于限制获取某种资源的线程数量。

  Semaphore 有两种模式，公平模式和非公平模式。

  - 公平模式： 调用 acquire 的顺序就是获取许可证的顺序，遵循 FIFO；
  - 非公平模式： 抢占式的。

  **CountDownLatch （倒计时器）**

  CountDownLatch 是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行。在 Java 并发中，countdownlatch 的概念是一个常见的面试题，所以一定要确保你很好的理解了它。

  - CountDownLatch 的三种典型用法

    ① 某一线程在开始运行前等待 n 个线程执行完毕。将 CountDownLatch 的计数器初始化为 n ：new CountDownLatch(n)，每当一个任务线程执行完毕，就将计数器减 1 countdownlatch.countDown()，当计数器的值变为 0 时，在CountDownLatch上 await() 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

    ② 实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 CountDownLatch 对象，将其计数器初始化为 1 ：new CountDownLatch(1)，多个线程在开始执行任务前首先 coundownlatch.await()，当主线程调用 countDown() 时，计数器变为 0，多个线程同时被唤醒。

    ③ 死锁检测：一个非常方便的使用场景是，你可以使用 n 个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。
  
  - CountDownLatch 的不足

    CountDownLatch 是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 CountDownLatch 使用完毕后，它不能再次被使用。
  
  **CyclicBarrier(循环栅栏)**

  CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。

  CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier 默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。

  CyclicBarrier 的应用场景

  CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个 Excel 保存了用户所有银行流水，每个 Sheet 保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个 sheet 里的银行流水，都执行完之后，得到每个 sheet 的日均银行流水，最后，再用 barrierAction 用这些线程的计算结果，计算出整个 Excel 的日均银行流水。

  **CyclicBarrier 和 CountDownLatch 的区别**

  下面这个是国外一个大佬的回答：

  CountDownLatch 是计数器，只能使用一次，而 CyclicBarrier 的计数器提供 reset 功能，可以多次使用。但是我不那么认为它们之间的区别仅仅就是这么简单的一点。我们来从 jdk 作者设计的目的来看，javadoc 是这么描述它们的：

  > CountDownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.(**CountDownLatch: 一个或者多个线程，等待其他多个线程完成某件事情之后才能执行；**) CyclicBarrier : A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point.(**CyclicBarrier : 多个线程互相等待，直到到达同一个同步点，再继续一起执行。**)

  对于 CountDownLatch 来说，重点是“一个线程（多个线程）等待”，而其他的 N 个线程在完成“某件事情”之后，可以终止，也可以等待。而对于 CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。

  CountDownLatch 是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而 CyclicBarrier 更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。

  **ReentrantLock 和 ReentrantReadWriteLock**

  ReentrantLock 和 synchronized 的区别在上面已经讲过了这里就不多做讲解。另外，需要注意的是：读写锁 ReentrantReadWriteLock 可以保证多个线程可以同时读，所以在读操作远大于写操作的时候，读写锁就非常有用了。

  **CyclicBarrier和CountDownLatch区别**

  > [https://blog.csdn.net/tolcf/article/details/50925145](<https://blog.csdn.net/tolcf/article/details/50925145>)

  | CountDownLatch                                               | CyclicBarrier                                                |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 减计数方式                                                   | 加计数方式                                                   |
  | 计算为0时释放所有等待的线程                                  | 计数达到指定值时释放所有等待线程                             |
  | 计数为0时，无法重置                                          | 计数达到指定值时，计数置为0重新开始                          |
  | **调用countDown()方法计数减一，调用await()方法只进行阻塞，对计数没任何影响** | **调用await()方法计数加1，若加1后的值不等于构造方法的值，则线程阻塞** |
  | 不可重复利用                                                 | 可重复利用                                                   |

  **如何让多个执行速度不同的线程跑到一个点上（同步）**

  **CyclicBarrier**

  **用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行**
  
  和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

  CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它
  才叫做循环屏障。

  **CountDownLatch**

  **用来控制一个线程等待多个线程。**

  维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

  **Semaphore**

  Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

- 悲观锁和乐观锁

  > [https://snailclimb.gitee.io/javaguide/#/docs/essential-content-for-interview/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87%E4%B9%8B%E4%B9%90%E8%A7%82%E9%94%81%E4%B8%8E%E6%82%B2%E8%A7%82%E9%94%81?id=2-cas%e7%ae%97%e6%b3%95](https://snailclimb.gitee.io/javaguide/#/docs/essential-content-for-interview/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87%E4%B9%8B%E4%B9%90%E8%A7%82%E9%94%81%E4%B8%8E%E6%82%B2%E8%A7%82%E9%94%81?id=2-cas%e7%ae%97%e6%b3%95)

  **悲观锁**

  **总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁**（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之su前先上锁。Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现。

  **乐观锁**

  **总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁**，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用**版本号机制和CAS算法**实现。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中java.util.concurrent.**atomic**包下面的**原子变量类**就是使用了乐观锁的一种实现方式**CAS**实现的。

  **两种锁的使用场景**

  从上面对两种锁的介绍，我们知道两种锁各有优缺点，不可认为一种好于另一种，**像乐观锁适用于写比较少的情况下（多读场景）**，即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以**一般多写的场景下用悲观锁就比较合适**。

  **乐观锁常见的两种实现方式**

  > 乐观锁一般会使用版本号机制或CAS算法实现。

  1. 版本号机制

      一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

      举一个简单的例子： 假设数据库中帐户信息表中有一个 version 字段，当前值为 1 ；而当前帐户余额字段（ balance ）为 $100 。

      1. 操作员 A 此时将其读出（ version=1 ），并从其帐户余额中扣除 $50（ $100-$50 ）。
      2. 在操作员 A 操作的过程中，操作员B 也读入此用户信息（ version=1 ），并从其帐户余额中扣除 $20 （ $100-$20 ）。
      3. 操作员 A 完成了修改工作，将数据版本号加一（ version=2 ），连同帐户扣除后余额（ balance=$50 ），提交至数据库更新，此时由于提交数据版本大于数据库记录当前版本，数据被更新，数据库记录 version 更新为 2 。
      4. 操作员 B 完成了操作，也将版本号加一（ version=2 ）试图向数据库提交数据（ balance=$80 ），但此时比对数据库记录版本时发现，操作员 B 提交的数据版本号为 2 ，数据库记录当前版本也为 2 ，不满足 “ 提交版本必须大于记录当前版本才能执行更新 “ 的乐观锁策略，因此，操作员 B 的提交被驳回。

      这样，就避免了操作员 B 用基于 version=1 的旧数据修改的结果覆盖操作员A 的操作结果的可能。

  2. CAS算法

      即compare and swap（比较与交换），是一种有名的无锁算法。无锁编程，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization）。CAS算法涉及到三个操作数

      - 需要读写的内存值 V
      - 进行比较的值 A
      - 拟写入的新值 B
      
      当且仅当 V 的值等于 A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作（比较和替换是一个原子操作）。一般情况下是一个自旋操作，即不断的重试。

      > [https://blog.csdn.net/qq_34337272/article/details/81252853](https://blog.csdn.net/qq_34337272/article/details/81252853)
      
      自旋锁（spinlock）：是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。

      ```java
      public class SpinLock {
          private AtomicReference<Thread> cas = new AtomicReference<Thread>();
          public void lock() {
              Thread current = Thread.currentThread();
              // 利用CAS
              while (!cas.compareAndSet(null, current)) {
                  // DO nothing
              }
          }
          public void unlock() {
              Thread current = Thread.currentThread();
              cas.compareAndSet(current, null);
          }
      }
      ```

      > [https://blog.csdn.net/bohu83/article/details/51124065](https://blog.csdn.net/bohu83/article/details/51124065)

      ```java
      /** 
      * Atomically sets the value to the given updated value 
      * if the current value {@code ==} the expected value. 
      * 
      * @param expect the expected value 
      * @param update the new value 
      * @return true if successful. False return indicates that 
      * the actual value was not equal to the expected value. 
      */  
      public final boolean compareAndSet(int expect, int update) {  
          return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  
      } 
      ```

  **乐观锁的缺点**

  > ABA 问题是乐观锁一个常见的问题

  1. **ABA 问题**

      如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的 "ABA"问题。

      JDK 1.5 以后的 AtomicStampedReference 类就提供了此种能力，其中的 compareAndSet 方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

  2. **循环时间长开销大**

      **自旋CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。** 如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

  3. **只能保证一个共享变量的原子操作**

      CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。但是从 JDK 1.5开始，提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作.所以我们可以使用锁或者利用AtomicReference类把多个共享变量合并成一个共享变量来操作。
  
  **CAS与synchronized的使用情景**

  > **简单的来说CAS适用于写比较少的情况下（多读场景，冲突一般较少），synchronized适用于写比较多的情况下（多写场景，冲突一般较多）**

  1. 对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。
  2. 对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

  补充： Java并发编程这个领域中synchronized关键字一直都是元老级的角色，很久之前很多人都会称它为 “重量级锁” 。但是，在JavaSE 1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的 偏向锁 和 轻量级锁 以及其它各种优化之后变得在某些情况下并不是那么重了。synchronized的底层实现主要依靠 Lock-Free 的队列，基本思路是 自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。在线程冲突较少的情况下，可以获得和CAS类似的性能；而线程冲突严重的情况下，性能远高于CAS。

- 多线程

  - 什么是线程和进程?

    **何为进程?**

    进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

    在 Java 中，当我们启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称主线程。

    **何为线程?**
    
    线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享进程的堆和方法区资源，但每个线程有自己的程序计数器、虚拟机栈和本地方法栈，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

  - 程序计数器为什么是私有的?

    程序计数器主要有下面两个作用：

    字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。

    在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

    需要注意的是，如果执行的是 native 方法，那么程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址。

    所以，程序计数器私有主要是为了线程切换后能恢复到正确的执行位置。

  - 虚拟机栈和本地方法栈为什么是私有的?

    虚拟机栈： 每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。

    本地方法栈： 和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

    所以，为了保证线程中的局部变量不被别的线程访问到，虚拟机栈和本地方法栈是线程私有的。

  - 为什么要使用多线程呢?

    先从总体上来说：

    从计算机底层来说： 线程可以比作是轻量级的进程，是程序执行的最小单位,线程间的切换和调度的成本远远小于进程。另外，多核 CPU 时代意味着多个线程可以同时运行，这减少了线程上下文切换的开销。

    从当代互联网发展趋势来说： 现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

    再深入到计算机底层来探讨：

    单核时代： 在单核时代多线程主要是为了提高 CPU 和 IO 设备的综合利用率。举个例子：当只有一个线程的时候会导致 CPU 计算时，IO 设备空闲；进行 IO 操作时，CPU 空闲。我们可以简单地说这两者的利用率目前都是 50%左右。但是当有两个线程的时候就不一样了，当一个线程执行 CPU 计算时，另外一个线程可以进行 IO 操作，这样两个的利用率就可以在理想情况下达到 100%了。

    多核时代: 多核时代多线程主要是为了提高 CPU 利用率。举个例子：假如我们要计算一个复杂的任务，我们只用一个线程的话，CPU 只会一个 CPU 核心被利用到，而创建多个线程就可以让多个 CPU 核心被利用到，这样就提高了 CPU 的利用率。

  - 线程的生命周期和状态

    **Java线程的状态：新建New（线程被构建）、可运行Runnable（运行中Running、就绪Ready）、阻塞Blocked（线程阻塞于锁）、等待Waiting（当前线程需要等待其他线程的消息）、超时等待TimeWaiting（在等待状态的基础上增加了超时限制，在指定时间自动回到Runnable）、终止Terminated**

    ![Java线程的状态](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

    ![Java线程状态变迁](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%20%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E8%BF%81.png)

    由上图可以看出：线程创建之后它将处于 NEW（新建） 状态，调用 start() 方法后开始运行，线程这时候处于 READY（可运行） 状态。可运行状态的线程获得了 CPU 时间片（timeslice）后就处于 RUNNING（运行） 状态。

    > 操作系统隐藏 Java 虚拟机（JVM）中的 RUNNABLE 和 RUNNING 状态，它只能看到 RUNNABLE 状态，所以 Java 系统一般将这两个状态统称为 RUNNABLE（运行中） 状态 。

    当线程执行 wait()方法之后，线程进入 WAITING（等待） 状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而 TIME_WAITING(超时等待) 状态相当于在等待状态的基础上增加了超时限制，比如通过 sleep（long millis）方法或 wait（long millis）方法可以将 Java 线程置于 TIMED WAITING 状态。当超时时间到达后 Java 线程将会返回到 RUNNABLE 状态。当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到 BLOCKED（阻塞） 状态。线程在执行 Runnable 的run()方法之后将会进入到 TERMINATED（终止） 状态。
  
  - 什么是上下文切换?

    多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。

    概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换。**

    上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。

    Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。

  - 说说 sleep() 方法和 wait() 方法区别和共同点?

    两者最主要的区别在于：**sleep 方法没有释放锁，而 wait 方法释放了锁**。
    两者都可以暂停线程的执行。

    Wait 通常被用于线程间交互/通信，sleep 通常被用于暂停执行。

    wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify() 或者 notifyAll() 方法。sleep() 方法执行完成后，线程会自动苏醒。或者可以使用 wait(long timeout)超时后线程会自动苏醒。
    
  - 为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？

    new 一个 Thread，线程进入了新建状态;调用 start() 方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。 而直接执行 run() 方法，会把 run 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

    总结： 调用 start 方法方可启动线程并使线程进入就绪状态，而 run 方法只是 thread 的一个普通方法调用，还是在主线程里执行。

- 创建线程有几种不同的方式？你喜欢哪一种？为什么？

  > [https://blog.csdn.net/u012973218/article/details/51280044](<https://blog.csdn.net/u012973218/article/details/51280044>)

  1. 继承Thread类创建线程类

     ```java
     public class FirstThreadTest extends Thread {  
         int i = 0;  
         //重写run方法，run方法的方法体就是现场执行体  
         public void run() {  
             for(;i<100;i++) {  
                 System.out.println(getName()+"  "+i);  
             }  
         }  
         public static void main(String[] args) {  
             for(int i = 0;i< 100;i++) {  
                 System.out.println(Thread.currentThread().getName()+"  : "+i);  
                 if(i==20) {  
                     new FirstThreadTest().run();  
                     new FirstThreadTest().run();  
                 }  
             }  
         }   
     }
     ```

  2. 通过Runable接口创建线程类

     ```java
     public class RunnableThreadTest implements Runnable {  
         private int i;  
         public void run() {  
             for(i = 0;i <100;i++) {  
                 System.out.println(Thread.currentThread().getName()+" "+i);  
             }  
         }  
         public static void main(String[] args) {  
             for(int i = 0;i < 100;i++) {  
                 System.out.println(Thread.currentThread().getName()+" "+i);  
                 if(i==20) {  
                     RunnableThreadTest rtt = new RunnableThreadTest();  
                     new Thread(rtt,"新线程1").start();  
                     new Thread(rtt,"新线程2").start();  
                 }  
             }  
         }  
     }
     ```

  3. 通过Callable接口和FutureTask对象创建线程

     a. 创建Callable接口的实现类，并实现call()方法；

     b. 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callback对象的call()方法的返回值；

     c. 使用FutureTask对象作为Thread对象的target创建并启动新线程；

     d. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值。

     ```java
     import java.util.concurrent.Callable;  
     import java.util.concurrent.ExecutionException;  
     import java.util.concurrent.FutureTask;  
       
     public class CallableThreadTest implements Callable<Integer> {  
       
         public static void main(String[] args) {  
             CallableThreadTest ctt = new CallableThreadTest();  
             FutureTask<Integer> ft = new FutureTask<Integer>(ctt);  
     //        Thread thread = new Thread(ft,"有返回值的线程");
     //        thread.start();
             for(int i = 0;i < 100;i++) {  
                 System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);  
                 if(i==20) {  
                     new Thread(ft,"有返回值的线程").start();  
                 }  
             }  
             try {  
                 System.out.println("子线程的返回值："+ft.get());  
             } catch (InterruptedException e) {  
                 e.printStackTrace();  
             } catch (ExecutionException e) {  
                 e.printStackTrace();  
             }  
         }  
       
         @Override  
         public Integer call() throws Exception {  
             int i = 0;  
             for(;i<100;i++) {  
                 System.out.println(Thread.currentThread().getName()+" "+i);  
             }  
             return i;  
         }  
     } 
     ```

  4. 通过线程池创建线程

     ```java
     import java.util.concurrent.ExecutorService;
     import java.util.concurrent.Executors;
     
     public class ThreadPool {
     	/* POOL_NUM */
     	private static int POOL_NUM = 10;
     	
     	/**
     	 * Main function
     	 */
     	public static void main(String[] args) {
     		ExecutorService executorService = Executors.newFixedThreadPool(5);
     		for(int i = 0; i<POOL_NUM; i++) {
     			RunnableThread thread = new RunnableThread();
     			executorService.execute(thread);
     		}
     	}
     }
      
     class RunnableThread implements Runnable {
     	private int THREAD_NUM = 10;
     	public void run() {
     		for(int i = 0; i<THREAD_NUM; i++) {
     			System.out.println("线程" + Thread.currentThread() + " " + i);
     		} 
     	}
     }
     ```

- Java多线程回调是什么意思？

  > [https://blog.csdn.net/wenzhi20102321/article/details/52512536](<https://blog.csdn.net/wenzhi20102321/article/details/52512536>)

  所谓回调，就是客户程序C调用服务程序S中的某个方法A，然后S又在某个时候反过来调用C中的某个方法B，对于C来说，这个B便叫做回调方法。

  - 回答者(S)

  ```java
  package com.xykj.thread;
  public class XiaoZhang extends Thread {
      // 回答1+1，很简单的问题不需要线程
      public int add(int num1, int num2) {
         return num1 + num2;
      }
   
      // 重写run方法
      @Override
      public void run() {
         // 回答地球为什么是圆的
         askquestion();
         super.run();
      }
   
      // 回调接口的创建，里面要有一个回调方法
      //回调接口什么时候用呢？这个思路是最重要的   
      public static interface CallPhone {
         public void call(String question);
      }
   
      // 回调接口的对象
      CallPhone callPhone;
   
      // 回答地球为什么是圆的
      private void askquestion() {
         System.err.println("开始查找资料！");
         try {
             sleep(3000);// 思考3天
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
         // 把答案返回到回调接口的call方法里面
         if (callPhone!=null) {//提问者实例化callPhone对象，相当于提问者已经告诉我，我到时用什么方式回复答案
             //这个接口的方法实现是在提问者的类里面
             callPhone.call("知道了，！！！~~~~百度有啊");
         }     
      }
  }
  ```

  - 提问者(C)

  ```java
  package com.xykj.thread;
  import com.xykj.thread.XiaoZhang.CallPhone;
  public class MainClass {
      /**
       * java回调方法的使用
       * 实际操作时的步骤：（以本实例解释）
       * 1.在回答者的类内创建回调的接口
       * 2.在回答者的类内创建回调接口的对象，
       * 3.在提问者类里面实例化接口对象，重写接口方法
       * 2.-3.这个点很重要，回调对象的实例化，要在提问者的类内实例化，然后重写接口的方法
       * 相当于提问者先把一个联络方式给回答者，回答者找到答案后，通过固定的联络方式，来告诉提问者答案。
       * 4.调用开始新线程的start方法
       * 5.原来的提问者还可以做自己的事
       * */
      public static void main(String[] args) {
         // 小王问小张1+1=？，线程同步
         XiaoZhang xiaoZhang = new XiaoZhang();
         int i = xiaoZhang.add(1, 1);//回答1+1的答案
   
         // 问小张地球为什么是圆的？回调方法的使用
         //这相当于先定好一个返答案的方式，再来执行实际操作
        
         // 实例化回调接口的对象
         CallPhone phone = new CallPhone() {
             @Override
             public void call(String question) {
                //回答问题者，回答后，才能输出答案
                System.err.println(question);
             }
         };
        
         //把回调对象赋值给回答者的回调对象，回答问题者的回调对象才能回答问题
         xiaoZhang.callPhone = phone;
        
         System.out.println("交代完毕！");
         //相关交代完毕之后再执行查询操作
         xiaoZhang.start();
        
         //小王做自己的事！
         System.out.println("小王做自己的事！");
      }
   
  }
  ```

- 线程间通信

  > [https://www.cnblogs.com/linyufeng/p/9671844.html](https://www.cnblogs.com/linyufeng/p/9671844.html)

  > Java并发编程的艺术 P96

  - wait/notify
  - volatile
  - join
  - CountDownLatch/CyclicBarrier

- 协程

  > [https://www.cnblogs.com/lxmhhy/p/6041001.html](https://www.cnblogs.com/lxmhhy/p/6041001.html)

  **协程是一种用户态的轻量级线程，协程的调度完全由用户控制。**协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量，所以上下文的切换非常快。

- 当一个线程进入一个对象的synchronized方法A之后，其它线程是否可进入此对象的synchronized方法B？

  不能。其它线程只能访问该对象的非同步方法，同步方法则不能进入。因为非静态方法上的synchronized修饰符要求执行方法时要获得对象的锁，如果已经进入A方法说明对象锁已经被取走，那么试图进入B方法的线程就只能在等锁池（注意不是等待池哦）中等待对象的锁。

- 线程同步和线程调度的相关方法

  - wait()：使一个线程处于等待（阻塞）状态，并且释放所持有的对象的锁；
  - sleep()：使一个正在运行的线程处于睡眠状态，是一个静态方法，调用此方法要处理InterruptedException异常；
  - notify()：唤醒一个处于等待状态的线程，当然在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且与优先级无关；
  - notityAll()：唤醒所有处于等待状态的线程，该方法并不是将对象的锁给所有线程，而是让它们竞争，只有获得锁的线程才能进入就绪状态；
  - 通过Lock接口提供了显式的锁机制（explicit lock），增强了灵活性以及对线程的协调。Lock接口中定义了加锁（lock()）和解锁（unlock()）的方法，同时还提供了newCondition()方法来产生用于线程之间通信的Condition对象；此外，Java 5还提供了信号量机制（semaphore），信号量可以用来限制对某个共享资源进行访问的线程的数量。在对资源进行访问之前，线程必须得到信号量的许可（调用Semaphore对象的acquire()方法）；在完成对资源的访问后，线程必须向信号量归还许可（调用Semaphore对象的release()方法）。

- 线程的sleep()方法和yield()方法有什么区别？

  ① sleep()方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行的机会；yield()方法只会给相同优先级或更高优先级的线程以运行的机会；

  ② 线程执行sleep()方法后转入阻塞（blocked）状态，而执行yield()方法后转入就绪（ready）状态；

  ③ sleep()方法声明抛出InterruptedException，而yield()方法没有声明任何异常；

  ④ sleep()方法比yield()方法（跟操作系统CPU调度相关）具有更好的可移植性。

  > [https://www.jianshu.com/p/25e959037eed](<https://www.jianshu.com/p/25e959037eed>)

  Java中wait、sleep的区别或者Java中sleep、yield的区别是Java面试或者多线程面试中最常问的问题之一。在这3个在Java中能够用来暂停线程的方法中，sleep()和yield()方法是定义在Thread类中，而wait()方法是定义在Object类中的， 这也是面试中常问的一个问题。

  wait()和sleep()的关键的区别在于，**wait()是用于线程间通信的，而sleep()是用于短时间暂停当前线程。**更加明显的一个区别在于，**当一个线程调用wait()方法的时候，会释放它锁持有的对象的管程和锁，但是调用sleep()方法的时候，不会释放他所持有的管程。**

  回到**yield()方法**上来，与wait()和sleep()方法有一些区别，它**仅仅释放线程所占有的CPU资源，从而让其他线程有机会运行，但是并不能保证某个特定的线程能够获得CPU资源。谁能获得CPU完全取决于调度器，在有些情况下调用yield方法的线程甚至会再次得到CPU资源。所以，依赖于yield方法是不可靠的，它只能尽力而为。**

  - Java中wait和sleep的区别

    wait和sleep的主要区别是调用wait方法时，线程在等待的时候会释放掉它所获得的monitor，但是调用Thread.sleep()方法时，线程在等待的时候仍然会持有monitor或者锁。另外，Java中的wait方法应在同步代码块中调用，但是sleep方法不需要。
    **另一个区别是Thread.sleep()方法是一个静态方法，作用在当前线程上；但是wait方法是一个实例方法，并且只能在其他线程调用本实例的notify()方法时被唤醒。**另外，使用sleep方法时，被暂停的线程在被唤醒之后会立即进入就绪态（Runnable state)，但是使用wait方法的时候，被暂停的线程会首先获得锁（译者注：阻塞态），然后再进入就绪态。所以，根据你的需求，如果你需要暂定你的线程一段特定的时间就使用sleep()方法，如果你想要实现线程间通信就使用wait()方法。
    下面列出Java中wait和sleep方法的区别：

    1. wait只能在同步（synchronize）环境中被调用，而sleep不需要。详见[Why to wait and notify needs to call from synchronized method](https://link.jianshu.com/?t=http%3A%2F%2Fjavarevisited.blogspot.com%2F2011%2F05%2Fwait-notify-and-notifyall-in-java.html)
    2. 进入wait状态的线程能够被notify和notifyAll线程唤醒，但是进入sleeping状态的线程不能被notify方法唤醒。
    3. wait通常有条件地执行，线程会一直处于wait状态，直到某个条件变为真。但是sleep仅仅让你的线程进入睡眠状态。
    4. wait方法在进入wait状态的时候会释放对象的锁，但是sleep方法不会。
    5. wait方法是针对一个被同步代码块加锁的对象，而sleep是针对一个线程。更详细的讲解可以参考《Java核心技术卷1》，里面介绍了如何使用wait和notify方法。

  - yield和sleep的区别

    yield和sleep的区别主要是，**yield方法会临时暂停当前正在执行的线程，来让有同样优先级的正在等待的线程有机会执行。如果没有正在等待的线程，或者所有正在等待的线程的优先级都比较低，那么该线程会继续运行。**执行了yield方法的线程什么时候会继续运行由线程调度器来决定，不同的厂商可能有不同的行为。**yield方法不保证当前的线程会暂停或者停止，但是可以保证当前线程在调用yield方法时会放弃CPU。**
    在Java中Sleep方法有两个， 一个只有一个毫秒参数，另一个有毫秒和纳秒两个参数。

    ```java
    sleep(long millis)
    ```

    or

    ```java
    sleep(long millis, int nanos)
    ```

    会让当前执行的线程sleep指定的时间。

    下面这张图很好地展示了在调用wait、sleep、yield方法的时候，线程状态如何转换。

    ![img](https://upload-images.jianshu.io/upload_images/66827-780462c52b8f5a83.png?imageMogr2/auto-orient/strip\|imageView2/2/w/1100/format/webp)

    Java中sleep方法的几个注意点：

    1. Thread.sleep()方法用来暂停线程的执行，将CPU放给线程调度器。
    2. Thread.sleep()方法是一个静态方法，它暂停的是当前执行的线程。
    3. Java有两种sleep方法，一个只有一个毫秒参数，另一个有毫秒和纳秒两个参数。
    4. 与wait方法不同，sleep方法不会释放锁
    5. 如果其他的线程中断了一个休眠的线程，sleep方法会抛出Interrupted Exception。
    6. 休眠的线程在唤醒之后不保证能获取到CPU，它会先进入就绪态，与其他线程竞争CPU。
    7. 有一个易错的地方，当调用t.sleep()的时候，会暂停线程t。这是不对的，因为Thread.sleep是一个静态方法，它会使当前线程而不是线程t进入休眠状态。

    这就是java中的sleep方法。我们已经看到了java中sleep、wait以及yield方法的区别。总之，记住sleep和yield作用于当前线程。

- 第一个 问题：Java中有几种方法可以实现一个线程？

  第二个问题：用什么关键字修饰同步方法?  

  第三个问题：stop()和suspend()方法为何不推荐使用，请说明原因？

  > [https://blog.csdn.net/Amen_Wu/article/details/54025804](<https://blog.csdn.net/Amen_Wu/article/details/54025804>)

  - **多线程有两种实现方法，分别是继承Thread类与实现Runnable接口。**

  - **用synchronized关键字修饰同步方法。（同步的实现方面有两种，分别是synchronized, wait与notify.）**

  - **反对使用stop()，是因为它不安全。它会解除由线程获取的所有锁定**，而且如果对象处于一种不连贯状态，那么其他线程能在那种状态下检查和修改它们。结果很难检查出真正的问题所在。

    **suspend()方法容易发生死锁。调用suspend()的时候，目标线程会停下来，但却仍然持有在这之前获得的锁定。**此时，其他任何线程都不能访问锁定的资源，除非被”挂起”的线程恢复运行。对任何线程来说，如果它们想恢复目标线程，同时又试图使用任何一个锁定的资源，就会造成死锁。

    所以不应该使用suspend()，而应在自己的Thread类中置入一个标志，指出线程应该活动还是挂起。若标志指出线程应该挂起，便用 wait()命其进入等待状态。若标志指出线程应当恢复，则用一个notify()重新启动线程。

- 启动一个线程是用run()还是start()?

  > [https://blog.csdn.net/wang_xing1993/article/details/70257475](<https://blog.csdn.net/wang_xing1993/article/details/70257475>)

  启动线程肯定要用start()方法。当用start()开始一个线程后，线程就进入就绪状态，使线程所代表的虚拟处理机处于可运行状态，这意味着它可以由JVM调度并执行。这并不意味着线程就会立即运行。当cpu分配给它时间时，才开始执行run()方法(如果有的话)。start()是方法,它调用run()方法.而run()方法是你必须重写的. run()方法中包含的是线程的主体。

- 在监视器(Monitor)内部，是如何做到线程同步的？在程序又应该做哪种级别的同步呢？

  > [https://www.nowcoder.com/questionTerminal/26fc16a2a85e49a5bd5fc2b5759dbbc2](https://www.nowcoder.com/questionTerminal/26fc16a2a85e49a5bd5fc2b5759dbbc2)

  在 java 虚拟机中, 每个对象( Object 和 class )通过某种逻辑关联监视器,每个监视器和一个对象引用相关联, 为了实现监视器的互斥功能, 每个对象都关联着一把锁。
 
  一旦方法或者代码块被 synchronized 修饰, 那么这个部分就放入了监视器的监视区域, 确保一次只能有一个线程执行该部分的代码, 线程在获取锁之前不允许执行该部分的代码。
  
  另外 java 还提供了显式监视器( Lock )和隐式监视器( synchronized )两种锁方案。

- 同步方法和同步代码块的区别是什么？

  同步方法默认用this或者当前类class对象作为锁；
  同步代码块可以选择以什么来加锁，比同步方法要更细颗粒度，我们可以选择只同步会发生同步问题的部分代码而不是整个方法。

- 什么是生产者消费者模式？

  ![img](https://uploadfiles.nowcoder.com/images/20180925/308572_1537880635592_7142B8354CA8A352B2B805F997C71549)

  生产者消费者问题是线程模型中的经典问题：生产者和消费者在同一时间段内共用同一存储空间，生产者向空间里生产数据，而消费者取走数据。

- 线程安全有哪些实现方式

  1、同步方案：互斥同步（阻塞同步）：**synchronized和lock**

  非阻塞同步（乐观锁）：**CAS**

  2、无同步方案：不可变对象（本身就是线程安全的，因为不能修改。比如final、String、enum（枚举））

  栈封闭（多线程访问一个方法的局部变量，不会有问题，因为虚拟机栈是线程私有的）

  线程本地存储（**ThreadLocal**，线程局部变量）

  可重入代码（纯代码，可以在代码执行的任何时刻中断，转而去执行其他代码，返回之后，不会出现程序错误）

  （特征：不依赖堆等共享资源，用到的状态变量全部由参数传入，不调用不可重入的代码）

- 实现多线程同步的方法

  > [https://www.cnblogs.com/xhjt/p/3897440.html](<https://www.cnblogs.com/xhjt/p/3897440.html>)

  1. **同步方法**

     即有synchronized关键字修饰的方法。由于java的每个对象都有一个内置锁，当用此关键字修饰方法时，内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。

     代码如：

     ```java
     public synchronized void save(){}
     ```

     注： synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类

  2. **同步代码块**

     即有synchronized关键字修饰的语句块。被该关键字修饰的语句块会自动被加上内置锁，从而实现同步。

     代码如：

     ```java
     synchronized(object){}
     ```

     注：同步是一种高开销的操作，因此应该尽量减少同步的内容。通常没有必要同步整个方法，使用synchronized代码块同步关键代码即可。

     代码实例：

     ```java
     package com.xhj.thread;
     
     /**
       * 线程同步的运用
       */
     public class SynchronizedThread {
     
         class Bank {
     
             private int account = 100;
     
             public int getAccount() {
                 return account;
             }
     
             /**
               * 用同步方法实现
               * 
               * @param money
               */
             public synchronized void save(int money) {
                 account += money;
             }
     
             /**
               * 用同步代码块实现
               * 
               * @param money
               */
             public void save1(int money) {
                 synchronized (this) {
                     account += money;
                 }
             }
         }
     
         class NewThread implements Runnable {
             private Bank bank;
     
             public NewThread(Bank bank) {
                 this.bank = bank;
             }
     
             @Override
             public void run() {
                 for (int i = 0; i < 10; i++) {
                     // bank.save1(10);
                     bank.save(10);
                     System.out.println(i + "账户余额为：" + bank.getAccount());
                 }
             }
     
         }
     
         /**
           * 建立线程，调用内部类
           */
         public void useThread() {
             Bank bank = new Bank();
             NewThread new_thread = new NewThread(bank);
             System.out.println("线程1");
             Thread thread1 = new Thread(new_thread);
             thread1.start();
             System.out.println("线程2");
             Thread thread2 = new Thread(new_thread);
             thread2.start();
         }
     
         public static void main(String[] args) {
             SynchronizedThread st = new SynchronizedThread();
             st.useThread();
         }
     
     }
     ```

  3. 使用特殊域变量(**volatile**)实现线程同步

     a.volatile关键字为域变量的访问提供了一种免锁机制，

     b.使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新，

     c.因此每次使用该域就要重新计算，而不是使用寄存器中的值

     d.volatile不会提供任何原子操作，它也不能用来修饰final类型的变量

     例如：在上面的例子当中，只需在account前面加上volatile修饰，即可实现线程同步。

     代码实例：

     ```java
     //只给出要修改的代码，其余代码与上同
     class Bank {
         //需要同步的变量加上volatile
         private volatile int account = 100;
     
         public int getAccount() {
             return account;
         }
         //这里不再需要synchronized 
         public void save(int money) {
             account += money;
         }
     }
     ```

     注：多线程中的非同步问题主要出现在对域的读写上，如果让域自身避免这个问题，则就不需要修改操作该域的方法。用final域，有锁保护的域和volatile域可以避免非同步的问题。

  4. 使用**重入锁**实现线程同步

     在JavaSE5.0中新增了一个java.util.concurrent包来支持同步。ReentrantLock类是可重入、互斥、实现了Lock接口的锁，它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力。

     **ReentrantLock类**的常用方法有：

     - ReentrantLock() : 创建一个ReentrantLock实例
     - lock() : 获得锁
     - unlock() : 释放锁

     注：ReentrantLock()还有一个可以创建公平锁的构造方法，但由于能大幅度降低程序运行效率，不推荐使用。

     例如：在上面例子基础上，修改后的代码为：

     代码实例：

     ```java
     //只给出要修改的代码，其余代码与上同
     class Bank {
     
         private int account = 100;
         //需要声明这个锁
         private Lock lock = new ReentrantLock();
         public int getAccount() {
             return account;
         }
         //这里不再需要synchronized 
         public void save(int money) {
             lock.lock();
             try{
                 account += money;
             }finally{
                 lock.unlock();
             }
         }
     }
     ```

     注：关于Lock对象和synchronized关键字的选择：

     a.最好两个都不用，使用一种java.util.concurrent包提供的机制，能够帮助用户处理所有与锁相关的代码。

     b.如果synchronized关键字能满足用户的需求，就用synchronized，因为它能简化代码

     c.如果需要更高级的功能，就用ReentrantLock类，此时要注意及时释放锁，否则会出现死锁，通常在finally代码释放锁

  5. 使用局部变量实现线程同步

     **如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本，副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。**

     **ThreadLocal 类**的常用方法

     - ThreadLocal() : 创建一个线程本地变量
     - get() : 返回此线程局部变量的当前线程副本中的值
     - initialValue() : 返回此线程局部变量的当前线程的"初始值"
     - set(T value) : 将此线程局部变量的当前线程副本中的值设置为value

     例如：在上面例子基础上，修改后的代码为：

     代码实例：

     ```java
     //只改Bank类，其余代码与上同
     public class Bank{
         //使用ThreadLocal类管理共享变量account
         private static ThreadLocal<Integer> account = new ThreadLocal<Integer>(){
             @Override
             protected Integer initialValue(){
                 return 100;
             }
         };
         public void save(int money){
             account.set(account.get()+money);
         }
         public int getAccount(){
             return account.get();
         }
     }
     ```

     注：ThreadLocal与同步机制

     a.**ThreadLocal**与**同步机制**都是为了解决多线程中相同变量的访问冲突问题。

     b.前者采用以"**空间换时间**"的方法，后者采用以"**时间换空间**"的方式

  6. 使用**阻塞队列**实现线程同步

     前面5种同步方式都是在底层实现的线程同步，但是我们在实际开发当中，应当尽量远离底层结构。 
     使用javaSE5.0版本中新增的java.util.concurrent包将有助于简化开发。 

     本小节主要是使用**LinkedBlockingQueue<E>**来实现线程的同步 。LinkedBlockingQueue<E>是一个基于已连接节点的，范围任意的blocking queue。 队列是先进先出的顺序（FIFO），关于队列以后会详细讲解~ 

     **LinkedBlockingQueue 类常用方法** 

     - LinkedBlockingQueue() : 创建一个容量为Integer.MAX_VALUE的LinkedBlockingQueue 
     - put(E e) : 在队尾添加一个元素，如果队列满则阻塞 
     - size() : 返回队列中的元素个数 
     - take() : 移除并返回队头元素，如果队列空则阻塞 

     **代码实例：** 实现商家生产商品和买卖商品的同步

     ```java
     package com.xhj.thread;
     
     import java.util.Random;
     import java.util.concurrent.LinkedBlockingQueue;
     
     /**
      * 用阻塞队列实现线程同步 LinkedBlockingQueue的使用
      */
     public class BlockingSynchronizedThread {
         /**
          * 定义一个阻塞队列用来存储生产出来的商品
          */
         private LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<Integer>();
         /**
          * 定义生产商品个数
          */
         private static final int size = 10;
         /**
          * 定义启动线程的标志，为0时，启动生产商品的线程；为1时，启动消费商品的线程
          */
         private int flag = 0;
     
         private class LinkBlockThread implements Runnable {
             @Override
             public void run() {
                 int new_flag = flag++;
                 System.out.println("启动线程 " + new_flag);
                 if (new_flag == 0) {
                     for (int i = 0; i < size; i++) {
                         int b = new Random().nextInt(255);
                         System.out.println("生产商品：" + b + "号");
                         try {
                             queue.put(b);
                         } catch (InterruptedException e) {
                             // TODO Auto-generated catch block
                             e.printStackTrace();
                         }
                         System.out.println("仓库中还有商品：" + queue.size() + "个");
                         try {
                             Thread.sleep(100);
                         } catch (InterruptedException e) {
                             // TODO Auto-generated catch block
                             e.printStackTrace();
                         }
                     }
                 } else {
                     for (int i = 0; i < size / 2; i++) {
                         try {
                             int n = queue.take();
                             System.out.println("消费者买去了" + n + "号商品");
                         } catch (InterruptedException e) {
                             // TODO Auto-generated catch block
                             e.printStackTrace();
                         }
                         System.out.println("仓库中还有商品：" + queue.size() + "个");
                         try {
                             Thread.sleep(100);
                         } catch (Exception e) {
                             // TODO: handle exception
                         }
                     }
                 }
             }
         }
     
         public static void main(String[] args) {
             BlockingSynchronizedThread bst = new BlockingSynchronizedThread();
             LinkBlockThread lbt = bst.new LinkBlockThread();
             Thread thread1 = new Thread(lbt);
             Thread thread2 = new Thread(lbt);
             thread1.start();
             thread2.start();
         }
     }
     ```

     注：BlockingQueue<E>定义了阻塞队列的常用方法，尤其是三种添加元素的方法，我们要多加注意，当队列满时：add()方法会抛出异常，offer()方法返回false，put()方法会阻塞。

  7. 使用**原子变量**实现线程同步

     需要使用线程同步的根本原因在于对普通变量的操作不是原子的。那么什么是原子操作呢？原子操作就是指将读取变量值、修改变量值、保存变量值看成一个整体来操作，即-这几种行为要么同时完成，要么都不完成。在java的**util.concurrent.atomic包中提供了创建了原子类型变量的工具类**，使用该类可以简化线程同步。其中**AtomicInteger** 表可以用原子方式更新int的值，可用在应用程序中(如以原子方式增加的计数器)，但不能用于替换Integer；可扩展Number，允许那些处理机遇数字类的工具和实用工具进行统一访问。

     **AtomicInteger类常用方法：**

     - AtomicInteger(int initialValue) : 创建具有给定初始值的新的AtomicInteger
     - addAddGet(int dalta) : 以原子方式将给定值与当前值相加
     - get() : 获取当前值

     **代码实例：**只改Bank类，其余代码与上面第一个例子同

     ```java
     class Bank {
         private AtomicInteger account = new AtomicInteger(100);
     
         public AtomicInteger getAccount() {
             return account;
         }
     
         public void save(int money) {
             account.addAndGet(money);
         }
     }
     ```

     **补充--原子操作主要有：**

     对于引用变量和大多数原始变量(long和double除外)的读写操作；

     对于所有使用volatile修饰的变量(包括long和double)的读写操作。

- Java终止线程的几种方式

  > [https://www.cnblogs.com/yongdaimi/p/12074507.html](https://www.cnblogs.com/yongdaimi/p/12074507.html)

- 什么是线程安全

  如果一段代码可以保证多个线程访问的时候正确操作共享数据，那么它是线程安全的

- 多线程中的i++线程安全吗？请简述一下原因？

  不安全。i++不是原子性操作。i++分为读取i值，对i值加一，再赋值给i++，执行期中任何一步都是有可能被其他线程抢占的。

- synchronized的可重入怎么实现

  > 《Java并发编程实践》

  **“重入”意味这获取锁的操作的粒度是“线程”，而不是“调用”。**重入的一种**实现方法**是，为每个锁关联一个获取计数值和一个所有者线程。当计数值为0时，这个锁就被认为是没有被任何线程持有。当线程请求一个未被持有的锁时，JVM将记下锁的持有者，并且将获取计数值置为1. 如果同一个线程再次获取这个锁，计数值将递增，而当线程退出同步代码块时，计数器会相应地递减。当计数值为0时，这个锁将被释放。

  > [https://www.jianshu.com/p/5379356c648f](<https://www.jianshu.com/p/5379356c648f>)

  若一个程序或子程序可以“**在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错**”，则称其为可重入（reentrant或re-entrant）的。即**当该子程序正在运行时，执行线程可以再次进入并执行它**，仍然获得符合设计时预期的结果。与多线程并发执行的线程安全不同，可重入强调对单个线程执行时重新进入同一个子程序仍然是安全的。

- 《Java并发编程实战》摘录

  - 第二章 线程安全性

    - **安全性**的含义是“**永远不发生糟糕的事情**”，而**活跃性**则关注于另一个目标，即“**某件正确的事情最终会发生**”。

    - **当多个线程访问某个类时**，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，**这个类都能表现出正确的行为**，那么就称**这个类是线程安全的**。

    - 无状态对象一定是线程安全的。

    - 当某个计算的正确性取决于多个线程的交替执行时序时，就会发生**竞争条件**(race condition)。换句话说，就是正确的结果要取决于运气。最常见的竞争条件类型就是“**先检查后执行**(check-then-act)”操作，即通过一个可能失效的观测结果来决定下一步的动作。

    - 假定有两个操作A和B，如果从执行A的线程来看，当另一个线程执行B时，要么将B全部执行完，要么完全不执行B，那么A和B对彼此来说时**原子的**。原子操作是指，对于访问同一个状态的所有操作（包括该操作本身）来说，这个操作是一个以原子方式执行的操作。

    - 在java.util.concurrent.atomic包中包含了一些原子变量类，用于实现在数值和对象引用上的原子状态转换。通过用AtomicLong来代替long类型的计数器，能够确保所有对计数器状态的访问操作都是原子的。

    - 要保持状态的一致性，就需要在单个原子操作中更新所有相关的状态变量。

    - **“重入”意味这获取锁的操作的粒度是“线程”，而不是“调用”。**重入的一种**实现方法**是，为每个锁关联一个获取计数值和一个所有者线程。当计数值为0时，这个锁就被认为是没有被任何线程持有。当线程请求一个未被持有的锁时，JVM将记下锁的持有者，并且将获取计数值置为1. 如果同一个线程再次获取这个锁，计数值将递增，而当线程退出同步代码块时，计数器会相应地递减。当计数值为0时，这个锁将被释放。

      在以下代码中，子类改写了父类的synchronized方法，然后调用父类的方法，此时如果没有可重入的锁，那么这段代码将产生死锁。

      ```java
      public class Widget {
          public synchronized void doSomething() {
              ...
          }
      }
      
      public class LoggingWiget extends Widget {
          public synchronized void doSomething() {
              System.out.println(toString() + ": calling doSomething");
              super.doSomething();
          }
      }
      ```

    - 一种常见的错误是认为，只有在写入共享变量时才需要使用同步，然而事实并非如此。

    - 对于可能被多个线程同时访问的可变状态变量，在访问它时都需要持有同一个锁，在这种情况下，我们称状态是由这个锁保护的。**每个共享的和可变的变量都应该只由一个锁来保护，从而使维护人员知道是哪一个锁。对于包含多个变量的不变性条件，其中涉及的所有变量都需要由同一个锁来保护。**

    - Servlet中一个同步代码块负责保护判断是否只需返回缓存结果的“先检查后执行”操作序列，另一个同步代码块则负责确保对缓存的数值和引述分解结果进行同步更新。

      ```java
      @ThreadSafe
      public class CacheFactorizer implments Servlet {
          @GuardedBy("this") private BigInteger lastNumber;
          @GuardedBy("this") private BigInteger[] lastFactors;
          @GuardedBy("this") private long hits;
          @GuardedBy("this") private long cacheHits;
          
          public synchronized long getHits() { return hits; }
          public synchronized double getCacheHitRatio() {
              return (double) cacheHits / (double) hits;
          }
          
          public void service(ServletRequest req, ServletResponse resq) {
              BigInteger i = extractFromRequest(req);
              BigInteger[] factors = null;
              synchronized (this) {
                  ++hits;
                  if (i.equals(lastNumber)) {
                      ++cacheHits;
                      factors = lastFactors.clone();
                  }
              }
              if (factors == null) {
                  factors = factor(i);
                  synchronized (this) {
                      lastNumber = i;
                      lastFactors = factors.clone();
                  }
              }
              encodeIntResponse(resp, factors);
          }
      }
      ```

      在简单性与性能之间存在着相互制约因素。

      **当执行时间较长的计算或者可能无法快速完成的操作时（例如，网络I/O或控制台I/O），一定不要持有锁。**

  - 第三章 对象的共享

    - 在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行一些意向不到的调整。在缺乏足够同步的多线程程序中，要想对内存操作的执行顺序进行判断，几乎无法得出正确的结论。在缺乏同步的程序中可能产生错误结果的一种情况是**失效数据**。仅对set方法进行同步是不够的，调用get的线程仍然会看到失效值。

    - 当线程在没有同步的情况下读取变量时，可能会得到一个失效值，但至少这个值是由之前某个线程设置的值，而不是一个随机值。这种安全性保证也被称为最低安全性(out-of-thin-air safety)。最低安全性适用于绝大多数变量，但是存在一个例外：非volatile类型的64位数值变量（double和long）。Java内存模型要求，变量的读取操作和写入操作都必须时原子操作，但对于非volatile类型的long和double变量，JVM允许将64位读操作或写操作分解为两个32位的操作。当读取一个非volatile类型的long时，如果对该变量的读操作和写操作在不同的线程中执行，那么很可能会读取到某个值的高32位和另一个值的低32位。

    - 在访问某个共享且可变的变量时要求所有线程在同一个锁上同步，就是为了确保某个线程写入该变量的值对于其他线程来说时可见的。**加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，所有执行读操作或者写操作的线程都必须在同一个锁上同步。**

    - **Java语言提供了一种稍弱的同步机制，即volatile变量，用来确保变量的更新操作通知到其他线程。在读取volatile类型的变量时总会返回最新写入的值。在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比synchronized关键字更轻量级的同步机制。**

    - volatile变量对可见性的影响比volatile变量本身更为重要。当线程A首先写入一个volatile变量并且线程B随后读取该变量时，在写入volatile变量之前对A可见的所有变量的值，在B读取了volatile变量后，对B也是可见的。因此，从内存可见性的角度来看，写入volatile变量相当于退出同步代码块，而读取volatile变量就相当于进入同步代码块。

    - **加锁机制既可以确保可见性又可以确保原子性，而volatile变量只能确保可见性。**虽然volatile变量很方便，但也存在一些局限性，在使用时要非常小心。例如，volatile的语义不足以确保递增操作(count++)的原子性，除非你能确保只有一个线程对变量执行写操作。

    - 当且仅当满足以下所有条件时，才应该使用volatile变量：

      - 对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值。
      - 该变量不会与其他状态一起纳入不变性条件中。
      - 在访问变量时不需要加锁。

    - volatile变量的典型用法：检查某个状态标记以判断是否退出循环。为了使这个示例能正确执行，asleep必须为volatile变量。否则，当asleep被另一个线程修改时，执行判断的线程却发现不了。

      ```java
      volatile boolean asleep;
      ...
          while (!asleep)
              countSomeSheep();
      ```

- 线程池如何保证线程不被销毁

  > [https://blog.csdn.net/lbh199466/article/details/102700780](https://blog.csdn.net/lbh199466/article/details/102700780)

  如果队列中没有任务时，核心线程会一直阻塞在获取任务的方法，直到返回任务。