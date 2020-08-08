---
layout:     post
title:      "设计模式"
subtitle:   ""
date:       2020-03-05 12:00:00
author:     "盈盈冲哥"
header-img: "img/fleabag.jpg"
mathjax: true
catalog: true
tags:
    - 学习
---

## 设计模式

> 大话设计模式

- 封装

  - **每个对象都包含它能进行操作所需要的所有信息，这个特性称为封装，因此对象不必依赖其他对象来完成自己的操作。**
  - **封装有很多好处，第一、良好的封装能够减少耦合，第二、类内部的实现可以自由地修改，第三、类具有清晰的对外接口。**

- 继承

  - **对象的继承代表了一种is-a的关系，如果两个对象A和B可以描述为B是A，则表明B可以继承A。**
  - 继承者还可以理解为是对被继承者的特殊化，因为它除了具备背继承者的特性外，还具备自己独有的个性。
  - 如果子类继承于父类，第一、子类拥有父类非private的属性和功能，第二、子类具有自己的属性和功能，即子类可以扩展父类没有的属性和功能，第三、子类还可以以自己的方式实现父类的功能（方法重写）。
  - 构造方法不能被继承，只能被调用
  - **不用继承的话，如果要修改功能，就必须在所有复杂的方法中修改，代码越多，出错的可能就越大，而继承的优点是，继承使得所有子类公共的部分都放在了父类，使得代码得到了共享，这就避免了重复，另外，继承可使得修改或扩展继承而来的实现都较为容易。**
  - **继承是有缺点的，那就是父类变，则子类不得不变。继承会破坏包装，父类实现细节暴露给子类。继承显然是一种类与类的强耦合的关系。**

- 多态

  - **多态表示不同的对象可以执行相同的动作，但要通过它们自己的实现代码来执行。**
  - **注意：第一、子类以父类的身份出现，第二、子类在工作时以自己的方式来实现，第三、子类以父类的身份出现时，子类特有的属性和方法不可以使用。**
  - **多态的原理是当方法被调用时，无论对象是否被转换为其父类，都只有位于对象继承链最末端的方法实现会被调用。也就是说，虚方法是按照其运行时类型而非编译时类型进行动态绑定调用的。**

- **单一职责原则：就一个类而言，应该仅有一个引起它变化的原因。**

  - 软件设计真正要做的许多内容，就是发现职责并把那些职责项目分离。
  - 如果你能够想到多于一个的动机去改变一个类，那么这个类就具有多于一个的职责。

- **开放-封闭原则：软件实体（类、模块、函数等等）对于扩展是开放的，对于更改是封闭的。**

  - 无论模块是多么的封闭，都会存在一些无法对之封闭的变化。既然不可能完全封闭，设计人员必须对于他设计的模块应该对哪种变化封闭做出选择。他必须先猜测出最有可能发生的变化种类，然后构造抽象来隔离那些变化。
  - 等待变化发生时立即采取行动。
  - **在我们最初编写代码时，假设变化不会发生。当发生变化时，我们就创建抽象来隔离以后发生的同类变化。**
  - **面对需求，对程序的改动时通过增加新代码进行的，而不是更改现有的代码。**

- **依赖倒转原则：A. 高层模块不应该依赖低层模块。两个都应该依赖抽象。B. 抽象不应该依赖细节。细节应该依赖抽象。**

  - 要针对接口编程，不要对实现编程。
  - 依赖倒转可以说是面向对象设计的标志，用哪种语言来编写程序不重要，如果编写时考虑的都是如何针对抽象编程而不是针对细节编程，即程序中所有的依赖关系都是终止于抽象类或者接口，那就是面向对象的设计，反之那就是过程化的设计了。

- **里氏替换原则：子类型必须能够替换掉它们的父类型**

- **合成/聚合复用原则：尽量使用合成/聚合，尽量不要使用类继承**

  - 聚合表示一种弱的拥有关系，体现的是A对象可以包含B对象，但B对象不是A对象的一部分；合成则是一种强的拥有关系，体现了严格的部分和整体的关系，部分和整体的生命周期一样。
  - 优先使用对象的合成/聚合讲有助于保持每个类被封装，并被集中在单个任务上。这样类和类继承层次会保持较小规模，并且不太可能增长位不可控制的庞然大物。

- 简单工厂模式

- 工厂方法模式：定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。

  - 类图：P71
  - Product：定义工厂方法所创建的对象的接口
  - Creator：声明工厂方法，该方法返回一个Product类型的对象
  - ConcreteProduct：具体的产品，实现了Product接口
  - ConcreteCreator：重定义工厂方法以返回一个ConcreteProduct实例

- 抽象工厂模式：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

  - 类图：P150
  - AbstractFactory：抽象工厂接口，它里面应该包含所有的产品船舰的抽象方法
  - AbstractProduct：抽象产品，它们都有可能有两种不同的实现
  - ConcreteFactory1, 2：具体的工厂，创建具有特定实现的产品对象
  - ConcreteProduct1, 2：对两个抽象产品的具体分类的实现

- 策略模式：它定义了算法家族，分别封装起来，让他们之间可以互相替换，此模式让算法的变化，不会影响到使用算法的客户。

  - 类图：P23
  - Context：上下文类，维护一个对Strategy的引用
  - Strategy：策略类，定义所有支持的算法的公共接口
  - ConcreteStrategy：具体策略类，封装了具体的算法或行为，继承于Strategy

- 代理模式：为其他对象提供一种代理以控制对这个对象的访问。

  - 类图：P64
  - Subject：主体类，定义了RealSubject和Proxy的共用接口，这样就在任何使用RealSubject的地方都可以使用Proxy
  - RealSubject：真实实体类
  - Proxy：代理类，保存一个引用使得代理可以访问实体，并提供一个与Subject的接口相同的接口，这样代理就可以用来代替实体
  - 用途：代理模式其实就是在访问对象时引入一定程度的间接性，因为这种间接性，可以附加多种用途。

- 观察者模式（发布-订阅模式）：定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态发生变化时，会通知所有观察者对象，使他们能够自动更新自己。

  - 类图：P131
  - Subject：主体类，它把所有对观察者对象的引用保存在一个聚集中，每个对象都可以有任何数量的观察者。抽象主题提供一个接口，可以增加和删除观察者对象。
  - Observer：观察者对象，抽象观察者，为所有的具体观察者定义一个接口，在得到主体的通知时更新自己。
  - ConcreteSubject：具体主体
  - ConcreteObject：具体观察者

- 适配器模式：将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

  - 类图：P172
  - Target：客户所期待的接口。目标可以是具体的或抽象的类，也可以是接口。
  - Adapter：通过在内部保证一个Adaptee对象，把原接口转换成目标接口。
  - Adaptee：需要适配的类。

- 单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

- 桥接模式：将抽象部分与它的实现部分分离，使得它们都可以独立地变化。

  - 类图：P231
  - Abstract：抽象
  - Implementor：实现
  - RefinedAbstraction：被提炼的抽象
  - ConcreteImplementationA, B：具体实现
  - 实现系统可能有多角度分类，每一种分类都有可能变化，那么就把这种多角度分离出来让它们独立变化，减少它们之间的耦合。

> Java面试突击

> 设计模式比较常见的就是让你手写一个单例模式（注意单例模式的几种不同的实现方法）或者让你说一下某个常见的设计模式在你的项目中是如何使用的，另外面试官还有可能问你抽象工厂和工厂方法模式的区别、工厂模式的思想这样的问题。
>
> 建议把代理模式、观察者模式、（抽象）工厂模式好好看一下，这三个设计模式也很重要。

- Spring和JDK设计中用到的设计模式

  > [https://blog.csdn.net/TK_lTlei/article/details/101599074](https://blog.csdn.net/TK_lTlei/article/details/101599074)

  > [https://blog.csdn.net/wenjieyatou/article/details/80630685](https://blog.csdn.net/wenjieyatou/article/details/80630685)

  Spring中用到了那些设计模式

  - 简单工厂模式
  - 工厂方法
  - 单例模式
  - 代理模式
  - 观察者模式

  JDK中的设计模式

  - **Singleton（单例） Runtime**(https://www.cnblogs.com/fflower/p/10578437.html)
  - **Factory（静态工厂） Class.forName**
  - **Factory Method（工厂方法） Collection.iterator**
  - **Abstract Factory（抽象工厂） java.sql**
  - Prototype（原型） Object.clone
  - Adapter（适配器） java.io.InputStreamReader(InputStream)
  - **Proxy（代理） 动态代理**
  - Iterator（迭代器） Iterator
  - **Observer（观察者） Swing中的Listener**
  - Command（命令） Runnable

- Spring 框架中用到了哪些设计模式？

  - **工厂设计模式** : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
  - **代理设计模式** : Spring AOP 功能的实现。
  - **单例设计模式** : Spring 中的 Bean 默认都是单例的。
  - 模板方法模式 : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
  - 包装器设计模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
  - **观察者模式**: Spring 事件驱动模型就是观察者模式很经典的一个应用。
  - **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。
  - ......

- 单例模式

  > [https://blog.csdn.net/qq_34337272/article/details/80455972](https://blog.csdn.net/qq_34337272/article/details/80455972)

  - 单例模式简介

    **为什么要用单例模式呢？**

    在我们的系统中，有一些对象其实我们只需要一个，比如说：线程池、缓存、对话框、注册表、日志对象、充当打印机、显卡等设备驱动程序的对象。事实上，这一类对象只能有一个实例，如果制造出多个实例就可能会导致一些问题的产生，比如：程序的行为异常、资源使用过量、或者不一致性的结果。

    简单来说使用单例模式可以带来下面几个好处:

    对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
    由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

    **为什么不使用全局变量确保一个类只有一个实例呢？**

    我们知道全局变量分为静态变量和实例变量，静态变量也可以保证该类的实例只存在一个。

    只要程序加载了类的字节码，不用创建任何实例对象，静态变量就会被分配空间，静态变量就可以被使用了。

    但是，如果说这个对象非常消耗资源，而且程序某次的执行中一直没用，这样就造成了资源的浪费。**利用单例模式的话，我们就可以实现在需要使用时才创建对象，这样就避免了不必要的资源浪费。** 不仅仅是因为这个原因，在程序中我们要尽量避免全局变量的使用，大量使用全局变量给程序的调试、维护等带来困难。

  - 单例的模式的实现

    通常单例模式在Java语言中，有两种构建方式：

    - **饿汉方式。指全局的单例实例在类装载时构建。**
    - **懒汉方式。指全局的单例实例在第一次被使用时构建。**
    
    不管是那种创建方式，它们通常都存在下面几点相似处：

    - 单例类**必须要有一个 private 访问级别的构造函数，只有这样，才能确保单例不会在系统中的其他代码内被实例化;**
    - **uniqueInstance 成员变量和 getInstance 方法必须是 static 的。**

    **饿汉方式(线程安全)**

    ```java
    public class Singleton {
        //在静态初始化器中创建单例实例，这段代码保证了线程安全
        private ， Singleton uniqueInstance = new Singleton();
        //Singleton类只有一个构造方法并且是被private修饰的，所以用户无法通过new方法创建该对象实例
        private Singleton(){}
        public static Singleton getInstance(){
            return uniqueInstance;
        }
    }
    ```

    所谓 “饿汉方式” 就是说JVM在加载这个类时就马上创建此唯一的单例实例，不管你用不用，先创建了再说，如果一直没有被使用，便浪费了空间，典型的空间换时间，每次调用的时候，就不需要再判断，节省了运行时间。

    **懒汉式（非线程安全和synchronized关键字线程安全版本 ）**

    ```java
    public class Singleton {  
        private static Singleton uniqueInstance;  
        private Singleton (){}   
        //没有加入synchronized关键字的版本是线程不安全的
        public static Singleton getInstance() {
            //判断当前单例是否已经存在，若存在则返回，不存在则再建立单例
            if (uniqueInstance == null) {  
                uniqueInstance = new Singleton();  
            }  
            return uniqueInstance;  
        }  
    }
    ```

    所谓 “ 懒汉式” 就是说单例实例在第一次被使用时构建，而不是在JVM在加载这个类时就马上创建此唯一的单例实例。

    但是上面这种方式很明显是线程不安全的，如果多个线程同时访问getInstance()方法时就会出现问题。如果想要保证线程安全，一种比较常见的方式就是在getInstance() 方法前加上synchronized关键字，如下：

    ```java
    public static synchronized Singleton getInstance() {  
	      if (instance == null) {  
	          uniqueInstance = new Singleton();  
	      }  
	      return uniqueInstance;  
    } 
    ```

    我们知道synchronized关键字偏重量级锁。虽然在JavaSE1.6之后synchronized关键字进行了主要包括：为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升。

    但是在程序中每次使用getInstance() 都要经过synchronized加锁这一层，这难免会增加getInstance()的方法的时间消费，而且还可能会发生阻塞。我们下面介绍到的 双重检查加锁版本 就是为了解决这个问题而存在的。

    **懒汉式(双重检查加锁版本)**

    利用双重检查加锁（double-checked locking），首先检查是否实例已经创建，如果尚未创建，“才”进行同步。这样以来，只有一次同步，这正是我们想要的效果。

    ```java
    public class Singleton {
        //volatile保证，当uniqueInstance变量被初始化成Singleton实例时，多个线程可以正确处理uniqueInstance变量
        private volatile static Singleton uniqueInstance;
        private Singleton() {}
        public static Singleton getInstance() {
            //检查实例，如果不存在，就进入同步代码块
            if (uniqueInstance == null) {
                //只有第一次才彻底执行这里的代码
                synchronized(Singleton.class) {
                    //进入同步代码块后，再检查一次，如果仍是null，才创建实例
                    if (uniqueInstance == null) {
                        uniqueInstance = new Singleton();
                    }
                }
            }
            return uniqueInstance;
        }
    }
    ```

    很明显，这种方式相比于使用synchronized关键字的方法，可以大大减少getInstance() 的时间消费。

    我们上面使用到了volatile关键字来保证数据的可见性。

    **懒汉式（登记式/静态内部类方式）**

    静态内部实现的单例是懒加载的且线程安全。

    只有通过显式调用 getInstance 方法时，才会显式装载 SingletonHolder 类，从而实例化 instance（只有第一次使用这个单例的实例的时候才加载，同时不会有线程安全问题）。

    ```java
    public class Singleton {  
        private static class SingletonHolder {  
            private static final Singleton INSTANCE = new Singleton();  
        }  
        private Singleton (){}  
        public static final Singleton getInstance() {  
            return SingletonHolder.INSTANCE;  
        }  
    }
    ```

    **饿汉式（枚举方式）**

    这种实现方式还没有被广泛采用，但这是实现单例模式的最佳方法。 它更简洁，自动支持序列化机制，绝对防止多次实例化 （如果单例类实现了Serializable接口，默认情况下每次反序列化总会创建一个新的实例对象，关于单例与序列化的问题可以查看这一篇文章《单例与序列化的那些事儿》），同时这种方式也是《Effective Java 》以及《Java与模式》的作者推荐的方式。

    ```java
    public enum Singleton {
        //定义一个枚举的元素，它就是 Singleton 的一个实例
        INSTANCE;  
        
        public void doSomeThing() {  
          System.out.println("枚举方法实现单例");
        }  
    }
    ```

    使用方法：

    ```java
    public class ESTest {
        public static void main(String[] args) {
            Singleton singleton = Singleton.INSTANCE;
            singleton.doSomeThing();//output:枚举方法实现单例
        }
    }
    ```

    > 这种方法在功能上与公有域方法相近，但是它更加简洁，无偿提供了序列化机制，绝对防止多次实例化，即使是在面对复杂序列化或者反射攻击的时候。虽然这种方法还没有广泛采用，但是单元素的枚举类型已经成为实现Singleton的最佳方法。 —-《Effective Java 中文版 第二版》

- 工厂模式

  > [https://blog.csdn.net/qq_34337272/article/details/80472071](https://blog.csdn.net/qq_34337272/article/details/80472071)

  - 工厂模式介绍
  
    **工厂模式的定义**
    
    在基类中定义创建对象的一个接口，让子类决定实例化哪个类。工厂方法让一个类的实例化延迟到子类中进行。

    **工厂模式的分类**

    （1）简单工厂（Simple Factory）模式，又称静态工厂方法模式（Static Factory Method Pattern）。

    （2）工厂方法（Factory Method）模式，又称多态性工厂（Polymorphic Factory）模式或虚拟构造子（Virtual Constructor）模式；

    （3）抽象工厂（Abstract Factory）模式，又称工具箱（Kit 或Toolkit）模式。

    **工厂模式的例子**

    (1) Spring中通过getBean(“xxx”)获取Bean；

    (2) Java消息服务JMS中(下面以消息队列ActiveMQ为例子)

    ```java
    ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.25.155:61616");
    ```

    **为什么要用工厂模式**

    (1) 解耦 ：把对象的创建和使用的过程分开

    (2) 降低代码重复: 如果创建某个对象的过程都很复杂，需要一定的代码量，而且很多地方都要用到，那么就会有很多的重复代码。

    (3) 降低维护成本 ：由于创建过程都由工厂统一管理，所以发生业务逻辑变化，不需要找到所有需要创建对象的地方去逐个修正，只需要在工厂里修改即可，降低维护成本。

  - 简单工厂模式

    严格的说，简单工厂模式并不是23种常用的设计模式之一，它只算工厂模式的一个特殊实现。简单工厂模式在实际中的应用相对于其他2个工厂模式用的还是相对少得多，因为它只适应很多简单的情况。

    最重要的是它违背了我们在概述中说的 开放-封闭原则 （虽然可以通过反射的机制来避免，后面我们会介绍到） 。因为每次你要新添加一个功能，都需要在生switch-case 语句（或者if-else 语句）中去修改代码，添加分支条件。

  - 工厂方法模式

    工厂方法模式应该是在工厂模式家族中是用的最多模式，一般项目中存在最多的就是这个模式。

    工厂方法模式是简单工厂的进一步深化， **在工厂方法模式中，我们不再提供一个统一的工厂类来创建所有的对象，而是针对不同的对象提供不同的工厂。** 也就是说 每个对象都有一个与之对应的工厂 。

  - 抽象工厂模式

    在工厂方法模式中，其实我们有一个潜在意识的意识。那就是我们生产的都是同一类产品。**抽象工厂模式是工厂方法的仅一步深化，在这个模式中的工厂类不单单可以创建一种产品，而是可以创建一组产品。**

    抽象工厂是生产一整套有产品的（至少要生产两个产品)，这些产品必须相互是有关系或有依赖的，而工厂方法中的工厂是生产单一产品的工厂。

- 代理模式

  > [https://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern/index.html](https://www.ibm.com/developerworks/cn/java/j-lo-proxy-pattern/index.html)

- 观察者模式

  > [https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/observer.html](https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/observer.html)