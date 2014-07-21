title: slf4j初始化绑定源码分析
date: 2014-07-21 16:37:21
categories: Log
tags: [log,slf4j]
---

通过翻看源码研究一下 Slf4j 是如何在运行时绑定具体的log api实现。  


# 源码追踪

我们来看看slf4j的源代码，看当这段常见的写日志代码在第一次执行时，slf4j会如何工作

        Logger logger = LoggerFactory.getLogger(SomeClass.class);
        logger.debug("first log");

打开类 org.slf4j.LoggerFactory的源码，看到getLogger()方法的代码如下：

	public static Logger getLogger(String name) {
		ILoggerFactory iLoggerFactory = getILoggerFactory();
		return iLoggerFactory.getLogger(name);
	}

继续看 getILoggerFactory()方法的代码实现：

	static int INITIALIZATION_STATE = UNINITIALIZED;

	public static ILoggerFactory getILoggerFactory() {
		if (INITIALIZATION_STATE == UNINITIALIZED) {
			INITIALIZATION_STATE = ONGOING_INITIALIZATION;
			performInitialization();
		}
		    
		switch (INITIALIZATION_STATE) {
			case SUCCESSFUL_INITIALIZATION:
				return StaticLoggerBinder.getSingleton().getLoggerFactory();
		...
	}

这里涉及到一个名为INITIALIZATION_STATE 的静态变量，用来记录当前初始化的状态。默认是UNINITIALIZED，第一次调用getILoggerFactory()方法时，检查到INITIALIZATION_STATE == UNINITIALIZED，就会调用performInitialization()方法来进行初始化。

performInitialization()方法在初始化完成时，会设置INITIALIZATION_STATE为SUCCESSFUL_INITIALIZATION。这样后面的switch语句就会调用StaticLoggerBinder.getSingleton().getLoggerFactory()来返回需要的ILoggerFactory。

我们继续深入看performInitialization()方法的代码实现：

	private final static void performInitialization() {
		bind();
		if (INITIALIZATION_STATE == SUCCESSFUL_INITIALIZATION) {
			versionSanityCheck();
		}
	}

继续看bind()方法，忽略错误处理的代码：

	private final static void bind() {
	    try {
	      Set staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
	      reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
	      // the next line does the binding
	      StaticLoggerBinder.getSingleton();
	      INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
	      reportActualBinding(staticLoggerBinderPathSet);
	      emitSubstituteLoggerWarning();
	    } ......
	}

这里是关键代码了，findPossibleStaticLoggerBinderPathSet()方法用来查找当前classpath下可能的StaticLoggerBinder的实现，如果有多个的话，则reportMultipleBindingAmbiguity()和reportActualBinding()方法会在绑定前后打印相应的信息。

看findPossibleStaticLoggerBinderPathSet()里面干了什么：

	private static String STATIC_LOGGER_BINDER_PATH = "org/slf4j/impl/StaticLoggerBinder.class";

	private static Set findPossibleStaticLoggerBinderPathSet() {
	    // use Set instead of list in order to deal with  bug #138
	    // LinkedHashSet appropriate here because it preserves insertion order during iteration
	    Set staticLoggerBinderPathSet = new LinkedHashSet();
	    try {
	      ClassLoader loggerFactoryClassLoader = LoggerFactory.class
	              .getClassLoader();
	      Enumeration paths;
	      if (loggerFactoryClassLoader == null) {
	        paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
	      } else {
	        paths = loggerFactoryClassLoader
	                .getResources(STATIC_LOGGER_BINDER_PATH);
	      }
	      while (paths.hasMoreElements()) {
	        URL path = (URL) paths.nextElement();
	        staticLoggerBinderPathSet.add(path);
	      }
	    } catch (IOException ioe) {
	      Util.report("Error getting resources from path", ioe);
	    }
	    return staticLoggerBinderPathSet;
	}

忽略细节，findPossibleStaticLoggerBinderPathSet()方法其实就是通过classloader的getResources()方法找到所有的名为"org/slf4j/impl/StaticLoggerBinder.class"的resource。

我们再回来看bind()方法，其实真正绑定的代码只有一行

	import org.slf4j.impl.StaticLoggerBinder;

	private final static void bind() {
		......
	    // the next line does the binding
	    StaticLoggerBinder.getSingleton();
	    ......
	}

这里调用了StaticLoggerBinder.getSingleton()方法。我们看StaticLoggerBinder类的权限定名，恰好和findPossibleStaticLoggerBinderPathSet()方法中查找的一致。

StaticLoggerBinder类是从哪里来的？我们看代码的时候，可以发现在slf4j-api的源代码中，的确有对应的package和类存在。

![ImplClassInSlf4jApiSource.jpg](/images/slfj4-binding/ImplClassInSlf4jApiSource.jpg)

但是打开打包好的slf4j-api.jar，却发现根本没有这个implpackage。

![ImplClassNotInSlf4jApiJar.jpg](/images/slfj4-binding/ImplClassNotInSlf4jApiJar.jpg)

而且这个StaticLoggerBinder类的代码也明确说这个类不应当被打包到slf4j-api.jar：

	private StaticLoggerBinder() {
	    throw new UnsupportedOperationException("This code should have never made it into slf4j-api.jar");
	}

在slf4j-api项目的pom.xml文件中，我们可以找到下面的内容：

	<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-antrun-plugin</artifactId>
        <executions>
          <execution>
            <phase>process-classes</phase>
            <goals>
             <goal>run</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <tasks>
            <echo>Removing slf4j-api's dummy StaticLoggerBinder and StaticMarkerBinder</echo>
            <delete dir="target/classes/org/slf4j/impl"/>
          </tasks>
        </configuration>
	</plugin>

这里通过调用ant在打包为jar文件前，将package org.slf4j.impl和其下的class都删除掉了。

我们再来看，具体的log api实现要如何做才能和slf4j转载。

slf4j自带了一个极度简化的log实现slf4j-simple,这里我们可以找到slf4j-api需要的"org/slf4j/impl/StaticLoggerBinder.class":

![StaticLoggerBinderInSlf4jSimple.jpg](/images/slfj4-binding/StaticLoggerBinderInSlf4jSimple.jpg)

同样在slf4j-log4j12, slf4j-jkd14, slf4j-jcl的项目中可以找到类似的"org/slf4j/impl/StaticLoggerBinder.class"，这三个项目分别用于集成java社区最常见的几个log实现：log4j, jdk logging, apache common logging.

继续看回StaticLoggerBinder的代码，以slf4j-simple为例：

	  private final ILoggerFactory loggerFactory;
	  
	  private StaticLoggerBinder() {
	    loggerFactory = new SimpleLoggerFactory();
	  }
	  
	  public ILoggerFactory getLoggerFactory() {
	    return loggerFactory;
	  }

这里的getLoggerFactory()方法会返回slf4j-simple实现的SimpleLoggerFactory。

	public class SimpleLoggerFactory implements ILoggerFactory {

		ConcurrentMap<String, Logger> loggerMap;

		public Logger getLogger(String name) {
			Logger simpleLogger = loggerMap.get(name);
			if (simpleLogger != null) {
				return simpleLogger;
			} else {
				Logger newInstance = new SimpleLogger(name);
				Logger oldInstance = loggerMap.putIfAbsent(name, newInstance);
				return oldInstance == null ? newInstance : oldInstance;
			}
		}
	}

SimpleLoggerFactory实现slf4j定义的ILoggerFactory interface，getLogger()方法中负责创建SimpleLogger对象并返回(为了提高性能做了cache)。

类似的slf4j-log4j12中会返回Log4jLoggerFactory，而Log4jLoggerFactory中通过调用log4j的LogManager来创建log4j的Logger对象并通过Log4jLoggerAdapter类来包装为slf4j的Logger(adapter模式)。
	
	log4jLogger = LogManager.getLogger(name);
	Logger newInstance = new Log4jLoggerAdapter(log4jLogger);


# 过程总结


# 代码和类分析 

初始化流程中涉及到的类：

1. org.slf4j.Logger
2. org.slf4j.LoggerFactory
3. org.slf4j.ILoggerFactory

Logger和LoggerFactory是slf4j定义好的，业务代码通过LoggerFactory来创建Logger对象，并调用这个logger对象来写日志。业务代码在此时是无需知道（也无法知道）具体底层是哪个log api实现，从而摆脱对具体log api的依赖。

LoggerFactory通过ILoggerFactory来调用底层log api的实现来获取logger(已经包装为slf4j的Logger)。