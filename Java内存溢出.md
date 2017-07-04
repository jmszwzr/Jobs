# Java内存溢出(java.lang.OutOfMemoryError)

[TOC]



## OutOfMemoryError异常分析

### 常见原因

1. 内存中加载的数据量过于庞大，如一次从数据库去取出过多数据；
2. 集合类中有对对象的引用，使用完后未清空，使得JVM不能回收；
3. 代码中存在死循环或循环产生过多重复的对象实体；
4. 使用的第三方软件的BUG；
5. 启动参数内存值设定过小；

### 错误提示

1. tomcat:java.lang.OutOfMemoryError:PermGen space
2. tomcat:java.lang.OutOfMemoryError:Java heap space
3. weblogic:Root cause of ServletException java.lang.OutOfMemoryError
4. resin:java.lang.OutOfMemoryError
5. java:java.lang.OutOfMemoryError

### 解决java.lang.OutOfMemoryError方法

#### 1、增加JVM的内存大小

1）在执行某个class文件时候，可以使用`java -Xmx256M aa.class`来设置运行`aa.class`时JVM所允许占用的最大内存为256M

2）对tomcat容器，可以在启动时对JVM设置内存限度。对tomcat，可以在catalina.bat中添加：

```
set CATALINA_OPTS=-Xms128M -Xmx256M
set JAVA_OPTS=-Xms128M -Xmx256M
```

或者把%CATALINA_OPTS%和%JAVA_OPTS%代替为-Xms128M -Xmx256M

3）对resin容器，同样可以在启动时对JVM设置内存限度。在bin文件夹下创建一个startup.bat文件，内容如下：

```
@echo off
call "httpd.exe"  "-Xms128M" "-Xmx256M"
:end
```

其中"-Xms128M"为最小内存，"-Xmx256M"为最大内存。

#### 2、优化程序，释放垃圾

​	主要包括避免死循环，应该及时释放各种资源：内存，数据库的各种连接，防止一次载入太多的数据。导致java.lang.OutOfMemoryError的根本原因是程序不健壮。

​	因此，从根本上解决Java内存溢出的唯一方法就是修改程序，及时地释放没用的对象，释放内存空间。

### Java代码导致OutOfMemoryError错误的解决

需要重点排查一下几点：

1. 检查代码中是否有死循环或递归调用。
2. 检查是否有大循环重复产生新对象实体。
3. 检查对数据库查询中，是否有一次获得全部数据的查询。一般来说，如果一次取十万条记录到内存，就可能引起内存溢出。这个问题比较隐蔽，在上线前，数据库中数据较少，不容易出问题，上线后，数据库中数据多了，一次查询就有可能引起内存溢出。因此对于数据库查询尽量采用分页的方式查询。
4. 检查List、Map等集合对象是否有使用完后，未清除的问题。List、Map等集合对象会始终存有对对象的引用，使得这些对象不能被GC回收。（数组、List、Map数据是否过大）
5. 全局变量是否过多。

#### tomcat中java.lang.OutOfMemoryError:PermGen space异常处理

**PermGen space**的全称是Permanent Generation space，是指内存的永久保存区域，这块内存主要是被JVM存放class和Meta信息的，class在被Loader时就会被放到PermGen space中，它和存放类实例（Instance）的Heap区域不同，GC（Garbage Collection）不会再主程序运行期对PermGen space进行清理，所以如果你的应用中有很多CLASS的话，就很可能出现PermGen space错误，这种错误常见在web服务器对JSP进行pre compile的时候。如果你的WEB APP下都用了大量的第三方jar，其大小超过了JVM默认的大小（4M），那么就会产生此错误信息了。

**解决方法：**手动设置MaxPermSize大小，修改TOMCAT_HOME/bin/catalina.sh，在

```
echo "Using CATALINA_BASE:   $CATALINA_BASE"
```

上面加入以下行：

```
JAVA_OPTS="-server -XX:PermSize=64M -XX:MaxPermSize=128m
```

> 建议：将相同的第三方jar文件移置到tomcat/shared/lib目录下，这样可以达到减少jar文档重复占用内存的目的。

### Java内存管理机制

​	在C++ 语言中，如果需要动态分配一块内存，程序员需要负责这块内存的整个生命周期。从申请分配、到使用、再到最后的释放。这样的过程非常灵活，但是却十分繁琐，程序员很容易由于疏忽而忘记释放内存，从而导致内存的泄露。 Java 语言对内存管理做了自己的优化，这就是垃圾回收机制。 Java 的几乎所有内存对象都是在堆内存上分配（基本数据类型除外），然后由 GC （ garbage collection）负责自动回收不再使用的内存。

​	上面是Java 内存管理机制的基本情况。但是如果仅仅理解到这里，我们在实际的项目开发中仍然会遇到内存泄漏的问题。也许有人表示怀疑，既然 Java 的垃圾回收机制能够自动的回收内存，怎么还会出现内存泄漏的情况呢？这个问题，我们需要知道 GC 在什么时候回收内存对象，什么样的内存对象会被 GC 认为是“不再使用”的。

​	Java中对内存对象的访问，使用的是引用的方式。在 Java 代码中我们维护一个内存对象的引用变量，通过这个引用变量的值，我们可以访问到对应的内存地址中的内存对象空间。在 Java 程序中，这个引用变量本身既可以存放堆内存中，又可以放在代码栈的内存中（与基本数据类型相同）。 GC 线程会从代码栈中的引用变量开始跟踪，从而判定哪些内存是正在使用的。如果 GC 线程通过这种方式，无法跟踪到某一块堆内存，那么 GC 就认为这块内存将不再使用了（因为代码中已经无法访问这块内存了）。

![](http://images2015.cnblogs.com/blog/330611/201609/330611-20160918171942953-1947458594.png)

​	通过这种有向图的内存管理方式，当一个内存对象失去了所有的引用之后，GC 就可以将其回收。反过来说，如果这个对象还存在引用，那么它将不会被 GC 回收，哪怕是 Java 虚拟机抛出 OutOfMemoryError 。





--------------

## 资料来源

- [Java 内存溢出（java.lang.OutOfMemoryError）的常见情况和处理方式总结](http://outofmemory.cn/c/java-outOfMemoryError)
- [java内存溢出示例(堆溢出、栈溢出)](http://www.cnblogs.com/panxuejun/p/5882424.html)
- [java 内存溢出 栈溢出的原因与排查方法](http://www.cnblogs.com/panxuejun/p/5882309.html)



























