[toc]



# JVM分代模型（年轻代、老年代、永久代）

1. 大部分对象存活周期极短
2. 少数对象是长期存活

## JVM分代模型

+ 年轻代：很快就要被回收的对象
+ 老年代：长期存在的对象
+ 永久代：保存类信息（JDK1.7）

### 对象在JVM中如何分配、流转

1. 大部分正常对象，都优先在新生代分配内存
2. 垃圾回收不是实时的，得满足一定的条件才会触发
   + 新生代内存不足，会触发一次新生代内存空间的垃圾回收（Minor GC / Young GC），会将年轻代里面没人引用的垃圾对象回收掉
3. 长期存活的对象会躲过多次垃圾回收
   + JVM规定，如果一个新生代对象，成功地在15次垃圾回收后，还没有被回收掉，就会被转移到老年代中去
   + 经历一次GC后存活下来的对象，年龄会+1
4. 老年代满了也会发生垃圾回收

**方法区会不会发生垃圾回收**

会，满足下面三个条件，方法区中的类会被回收

+ 该类的所有实例对象都会被回收
+ 加载该类的ClassLoader已经被回收
+ 该类的Class对象没有任何引用

## 课后问题

1. 每个线程都有自己的虚拟机栈，里面也有方法的局部变量表等数据，Java虚拟机栈需要垃圾回收吗？

   > 方法执行完，方法的栈帧直接出栈了，清理啥？？？

2. 你负责的系统中，短生存周期的对象特征有哪些，长生存周期的对象特征有哪些

   > 短： 方法里面的对象，长：Spring Service

# 设置JVM内存大小

1. -Xms : Java堆内存大小
2. -Xmx ： Java堆内存最大大小
3. -Xmn ：Java堆内存中新生代的空间大小，剩余空间为老年代的空间大小
4. -XX:PermSize : 永久代大小
5. -XX: MaxPermSize : 永久代最大大小
6. -Xss：每个线程的栈内存大小

> younggc:执行时间不超过50ms 执行不是很频繁不小于10s一次   fullgc:执行时间不超过1s 执行频率不低于10分钟

## 课后习题

Tomcat、Spring Boot部署启动系统的时候，JVM参数如何设置？

+ Tomcat 

  > Windows下，在文件/bin/catalina.bat，Unix下，在文件/bin/catalina.sh的前面，增加如下设置

+ Spring Boot

  > `java -Xms1024m -Xmx1024m -jar app.jar` 

对象的大小估算：

> 一般实体都是包装类型 对象头(4字节)+引用类型(4字节)+实例数据(integer:4 Long:8)+数据填充(将整个大小补齐为8的倍数)

# 第二周作业：你的线上系统是如何设置JVM参数的

非常简单，希望每个人按照案例里分析的那个过程，把你手头负责的系统的核心业务流程简化、抽象以及梳理出来，看看你们线上的真实负载情况，每秒钟多少请求？

然后根据你们的核心业务流程，看看每秒钟你负责的系统对内存使用的压力有多大？

假如你发现自己负责的系统很Low，没什么压力，那你可以尝试思考一下，如果你系统负载扩大100倍呢？此时对你内存使用压力有多大？

接着你再思考一下，就你的系统内存使用压力之下，目前你们线上机器是多大的堆内存？新生代多大？老年代多大？

如果没设置，可以自行百度默认的内存大小。然后分析一下，目前的这个内存配置，你们的垃圾回收有多频繁？

希望大家对自己手头的系统，严格的去分析一下，这个流程做下来，相信你对JVM的理解，对你负责的系统的理解，对线上系统的内存分配，都会提升一个理解的层次。



我们系统一般是提交一个表单，然后保存到数据库，会涉及到一些审核对象

+ 每秒多少请求？
+ 系统对内存使用的压力有多大？

> JDK1.7 的单机系统，堆内存初始大小和最大值都为2G内存，永久代初始大小128M，最大可扩容到512M

# 第二周答疑笔记

1. 既然栈帧存放了方法对应的局部变量的数据也包括了方法执行的其它相关信息，那为什么不把程序计算器那块记录执行的情况，也放在各个方法自己的栈帧里，而是要单独列一个程序计数器去存储呢？请教，谢谢

   > 这就是JVM设计者的设计思想了，因为程序计数器针对的是代码指令的执行，Java虚拟栈针对的是放方法的数据，一个是指令，一个是数据，分开设计

2. tomcat需要破坏双亲委派模型的原因：？？？

   (1)tomcat中的需要支持不同web应用依赖同一个第三方类库的不同版本，jar类库需要保证相互隔离；

   (2)同一个第三方类库的相同版本在不同web应用可以共享

   (3)tomcat自身依赖的类库需要与应用依赖的类库隔离 (3)jsp需要支持修改后不用重启tomcat即可生效 为了上面类加载隔离和类更新不用重启，定制开发各种的类加载器

3. 老师我上网查了一下资料，把问题弄明白了。Test.class是被加载了，但是并没有 执行初始化步骤。 

   课程中提到了类加载的时机，但是**没有提到类初始化的时机**，我把一直理解类的 加载->验证->准备->解析->初始化是一个连续的动作，以为类一旦加载必定 会立即初始化。

   补充类初始化的时机如下：

    1.当创建某个类的新实例时（如通过new或者反射，克隆，反序列化等）

    2.当调用某个类的静态方法时

    3.当使用某个类或接口的静态字段时

    4.调用Java API中的某些反射方法时，比如类Class中的方法，或者java.lang.reflect中的类的方法时

    5.当初始化某个子类时

    6.当虚拟机启动某个被标明为启动类的类（即包含main方法的那个类） 所以System.ou.println(Test.class)不满足上面6种情况，也就没有做初始化