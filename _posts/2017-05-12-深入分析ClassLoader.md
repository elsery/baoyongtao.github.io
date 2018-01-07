---
layout: post
title: "深入分析ClassLoader"
date: 2017-05-12 
description: "java，jvm"
description: JVM
--- 


## why？

ClassLoader，即[Java]类加载器，主要作用是将class加载到JVM内，同时它还要考虑class由谁来加载。在说java的类加载机制之前，还是像前面的博客一样，先说说为什么要知道java的类加载机制。个人认为主要有以下几个原因：

- **按需加载**。JVM启动时不能确定我要加载哪些东西，或者有些类非常大，我只希望用到它时再加载，并非一次性加载所有的class，所以这时候了解了加载机制就可以按需加载了。
- **类隔离**。比如web容器中部署多个应用，应用之间互相可能会有冲突，所以希望尽量隔离，这里可能就要分析各个应用加载的资源和加载顺序之间的冲突，针对这些冲突再自己定些规则，让它们能够愉快地玩耍。
- **资源回收**。如果你不了解java是如何加载资源的，又怎么理解java是如何回收资源的？

## what？

一般说到java的类加载机制，都要说到“双亲委派模型”（其实个人很不理解为什么叫“双亲”，其实英文叫“parent”）。使用这种机制，可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。JVM根据 **`类名+包名+ClassLoader实例ID` **来判定两个类是否相同，是否已经加载过（所以这里可以略微扩展下，可以通过创建不同的classloader实例来实现类的热部署）。 
有个图很形象（来源见参考资料）。

![ClassLoader]

**注意：如果想形象地看到java的类加载顺序，可以在运行java的时候加个启动参数，即`java –verbose`**

下面结合上图来详细介绍下java的类加载过程。

- **BootStrapClassLoader**。它是最顶层的类加载器，是由C++编写而成, 已经内嵌到JVM中了。在JVM启动时会初始化该ClassLoader，它主要用来读取Java的核心类库JRE/lib/rt.jar中所有的class文件，这个jar文件中包含了java规范定义的所有接口及实现。
- **ExtensionClassLoader**。它是用来读取Java的一些扩展类库，如读取JRE/lib/ext/*.jar中的包等（**这里要注意，有些版本的是没有ext这个目录的**）。
- **AppClassLoader**。它是用来读取CLASSPATH下指定的所有jar包或目录的类文件，一般情况下这个就是程序中默认的类加载器。
- **CustomClassLoader**。它是用户自定义编写的，它用来读取指定类文件 。基于自定义的ClassLoader可用于加载非Classpath中（如从网络上下载的jar或二进制）的jar及目录、还可以在加载前对class文件优一些动作，如解密、编码等。

很多资料和文章里说，`ExtClassLoader`的父类加载器是`BootStrapClassLoader`，其实这里省掉了一句话，容易造成很多新手（比如我）的迷惑。严格来说，`ExtClassLoader`的父类加载器是`null`，只不过在默认的ClassLoader 的 `loadClass` 方法中，当parent为`null`时，是交给`BootStrapClassLoader`来处理的，而且`ExtClassLoader` 没有重写默认的`loadClass`方法，所以，`ExtClassLoader`也会调用`BootStrapLoader`类加载器来加载，这就导致*“BootStrapClassLoader具备了ExtClassLoader父类加载器的功能”*。看一下下面的代码就很容易理解上面这一大段话了。

    /**
     * 查看父类加载器
     */
     private static void test1() {
        ClassLoader appClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println("系统类装载器:" + appClassLoader);
        ClassLoader extensionClassLoader = appClassLoader.getParent();
        System.out.println("系统类装载器的父类加载器——扩展类加载器:" + extensionClassLoader);
        ClassLoader bootstrapClassLoader = extensionClassLoader.getParent();
        System.out.println("扩展类加载器的父类加载器——引导类加载器:" + bootstrapClassLoader);
    }

可以看出`ExtensionClassLoader`的`parent`为`null`。

## 三个重要的方法

查看classloader的源码可以发现三个重要的方法:

- **loadClass**。classloader加载类的入口，此方法负责加载指定名字的类，ClassLoader的实现方法为先从已经加载的类中寻找，如没有则继续从父ClassLoader中寻找，如仍然没找到，则从BootstrapClassLoader中寻找，最后再调用findClass方法来寻找，如要改变类的加载顺序，则可覆盖此方法，如加载顺序相同，则可通过覆盖findClass来做特殊的处理，例如解密、固定路径寻找等，当通过整个寻找类的过程仍然未获取到Class对象时，则抛出ClassNotFoundException。如类需要resolve，则调用resolveClass进行链接。
- **findClass**。此方法直接抛出ClassNotFoundException，因此需要通过覆盖loadClass或此方法来以自定义的方式加载相应的类。
- **defineClass**。此方法负责将二进制的字节码转换为Class对象，这个方法对于自定义加载类而言非常重要，如二进制的字节码的格式不符合JVM Class文件的格式，抛出ClassFormatError；如需要生成的类名和二进制字节码中的不同，则抛出NoClassDefFoundError；如需要加载的class是受保护的、采用不同签名的或类名是以java.开头的，则抛出SecurityException；如需加载的class在此ClassLoader中已加载，则抛出LinkageError。

好，可能有人看了上面会疑惑，为什么上面说自定义classloader是需要重写findClass而不是loadClass或者其他方法。这里建议查看源码。

    protected Class<?> loadClass(String name, boolean resolve)
            throws ClassNotFoundException
        {
            synchronized (getClassLoadingLock(name)) {
                // First, check if the class has already been loaded
                Class c = findLoadedClass(name);
                if (c == null) {
                    long t0 = System.nanoTime();
                    try {
                        if (parent != null) {
                            c = parent.loadClass(name, false);
                        } else {
                            c = findBootstrapClassOrNull(name);
                        }
                    } catch (ClassNotFoundException e) {
                        // ClassNotFoundException thrown if class not found// from the non-null parent class loader
                    }
    
                    if (c == null) {
                        // If still not found, then invoke findClass in order// to find the class.long t1 = System.nanoTime();
                        c = findClass(name);
    
                        // this is the defining class loader; record the stats
                        sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                        sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                        sun.misc.PerfCounter.getFindClasses().increment();
                    }
                }
                if (resolve) {
                    resolveClass(c);
                }
                return c;
            }
        }

可以看到，JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的判断方法，当在loadClass方法中搜索不到类时，loadClass方法就会调用findClass方法来搜索类，所以我们只需重写该方法即可。

看完这一大串可能有人还是不理解，ok，那我们现在就动手写一个自己的ClassLoader，尽量包含上面三个方法。

## 自定义ClassLoader

先定义一个Person接口。

    public interface Person {
    	public void say();
    }

再定一个高富帅类实现这个接口

    public class HighRichHandsome implements Person {
    @Override
    public void say() {
            System.out.println("I don't care whether you are rich or not");
        }
    
    }

好的，开胃菜结束，主角来了，MyClassLoader。

    import java.io.ByteArrayOutputStream;
    import java.io.IOException;
    import java.io.InputStream;
    
    public class MyClassLoader extends ClassLoader{
    		/* 
         * 覆盖了父类的findClass，实现自定义的classloader
         */    
         @Override
         public Class<?> findClass(String name) {byte[] bt = loadClassData(name);
            return defineClass(name, bt, 0, bt.length);
        }
    
        private byte[] loadClassData(String className) {
            InputStream is = getClass().getClassLoader().getResourceAsStream(
                    className.replace(".", "/") + ".class");
            ByteArrayOutputStream byteSt = new ByteArrayOutputStream();
            int len = 0;
            try {
                while ((len = is.read()) != -1) {
                    byteSt.write(len);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            return byteSt.toByteArray();
        }
    }
    

代码很简单，不解释了，最后在[测试]类LoaderTest里写个测试方法。

    /**
     * 父类classloader
     * @throws Exception
     */
     private static void test2() throws Exception{
        MyClassLoader loader = new MyClassLoader();
        Class<?> c = loader.loadClass("com.alibaba.classload.HighRichHandsome");
        System.out.println("Loaded by :" + c.getClassLoader());
    
        Person p = (Person) c.newInstance();
        p.say();
    
        HighRichHandsome man = (HighRichHandsome) c.newInstance();
        man.say();    
    }

main方法中调用这个方法即可。LoaderTest默认构造函数会设置`AppClassLoader`为`parent`， 测试时执行`test2()`方法会发现`HighRichHandsome`类是委托`AppClassLoader`加载的，所以AppClassLoader可以访问到，不会出错。

但是我们再想一下，如果我们直接加载，不委托给父类加载，会出现什么情况？

    /**
     * 自己的classloader加载
     * @throws Exception
     */
     private static void test3() throws Exception{
        MyClassLoader loader = new MyClassLoader();
        Class<?> c = loader.findClass("com.alibaba.classload.HighRichHandsome");
        System.out.println("Loaded by :" + c.getClassLoader());
    
        Person p = (Person) c.newInstance();
        p.say();
    
        //注释下面两行则不报错
        HighRichHandsome man = (HighRichHandsome) c.newInstance();
        man.say();    
    }
    

可以看到，悲剧的报错了。根据class loader命名空间规则，每个class loader都有自己唯一的命名空间，每个class loader 只能访问自己命名空间中的类，如果一个类是委托parent加载的，那么加载后，这个类就类似共享的，parent和child都可以访问到这个类，因为parent是不会委托child加载类的，所以child加载的类parent访问不到。简单来说，即**子加载器的命名空间包含了parent加载的所有类，反过来则不成立。** 本例中`LoaderTest`类是`AppClassLoader`加载的，所以其看不见由`MyClassLoader`加载的`HighRichHandsome`类，但Person接口是可以访问的，所以赋给Person类型不会出错。

## 一些小的知识点

**1.Class.forName()**

相信大家都写过连接[数据库](http://lib.csdn.net/base/mysql)的例子，基本上就是加载驱动，建立连接，创建请求，写prepareStatement，关闭连接之类的。在这里，有一段代码：

    public DbTest() {
        try {
            Class.forName("com.mysql.jdbc.Driver");// 加载驱动
            conn = DriverManager.getConnection(url, "root", "");// 建立连接
            stm = conn.createStatement(); // 创建请求
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    

我相信大家一开始的时候肯定都有些疑惑，就是Class.forName(“com.MySQL.jdbc.Driver”)，为什么加载驱动是Class.forName，而不是ClassLoader的loadClass？为什么这么写就可以加载驱动了呢？

其实`Class.forName()`是显示加载类，作用是要求JVM查找并加载指定的类，也就是说JVM会执行该类的静态代码段。查看`com.mysql.jdbc.Driver`源码可以发现里面有个静态代码块，在加载后，类里面的静态代码块就执行（主要目的是注册驱动，把自己注册进去），所以主要目的就是为了触发这个静态方法。

**2.Web容器加载机制**

其实web容器的加载机制这一点我说不好，因为没看过诸如tomcat之类的源码，但是根据自己使用感觉以及查了相关资料，可以简单介绍一下。一般服务器端都要求类加载器能够反转委派原则，也就是先加载本地的类，如果加载不到，再到parent中加载。这个得细看，我这里只知道这个概念，所以就不说了。JavaEE规范推荐每个模块的类加载器先加载本类加载的内容，如果加载不到才回到parent类加载器中尝试加载。

**3.重复加载与回收**

一个class可以被不同的class loader重复加载，但同一个class只能被同一个class loader加载一次。见下列代码：

    /**
     * 对象只加载一次，返回true
     */
     private static void test4() {
        ClassLoader c1 = LoaderTest.class.getClassLoader();
        LoaderTest loadtest = new LoaderTest();
        ClassLoader c2 = loadtest.getClass().getClassLoader();
        System.out.println("c1.equals(c2):"+c1.equals(c2));
    }

类的回收。说到最开始的why，其实java的回收机制我前面有篇博客说的比较详细，这里就插一句废话吧，***当某个classloader加载的所有类实例化的所有对象都被回收了，则该classloader会被回收。***
 