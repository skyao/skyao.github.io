title: logback中logger对象的创建过程源码分析
date: 2014-08-08 11:33:38
categories: Log
tags: [log,logback,logger]
---

这次我们继续详细研究一下logback，看看logback中是如何创建Logger对象的。

<!--more-->

首先看这段代码，基本所有用到slf4j的地方都有类似的东东：

	public class Class1 {
		private static Logger logger = LoggerFactory.getLogger(Class1.class);

		private void someMethod() {
			logger.debug("some log content");
		}
	}

我们现在要研究的就是LoggerFactory.getLogger(Class1.class)这句代码里面到底发生了什么事情。在上一个blog里面我们刚刚研究过这里logback是如何和slf4j绑定和初始化logback配置的，这次我们关注的是这个方法返回的logger对象是如何在logback中创建的。

在logback中负责生成logback的Logger对象的是类 LoggerContext：

	package ch.qos.logback.classic;

	public class LoggerContext extends ContextBase implements ILoggerFactory,LifeCycle {}

	public final Logger getLogger(final Class clazz) {
		return getLogger(clazz.getName());
	}

	public final Logger getLogger(final String name) {...}

这里的getLogger()方法有两个版本，从代码上看，如果一个权限定名为"a.b.c.D"的类，通过getLogger(D.class)获取logger，本质上就是调用getLogger("a.b.c.D")。这个方式是slf4j定义的，logback遵循。

重点看getLogger(String name)方法的实现，忽略一些细节，我们首先会发现getLogger()方法中使用cache来缓存所有已经创建了的Logger对象实例：

	private Map<String, Logger> loggerCache;

	public final Logger getLogger(final String name) {
		
	    Logger childLogger = (Logger) loggerCache.get(name);
	    // if we have the child, then let us return it without wasting time
	    if (childLogger != null) {
	      return childLogger;
	    }
		......
		childLogger = logger.createChildByName(childName);
        loggerCache.put(childName, childLogger);
	}

这个cache的存在可以优化Logger实例创建的效率，所以对于下面的两种Logger创建方式，其实性能差距不大(只多一个很小的从cache中取对象的开销)。

	private static Logger logger = LoggerFactory.getLogger(Class1.class);
	private Logger logger = LoggerFactory.getLogger(Class1.class);

继续看getLogger()方法的时候，如果cache没有命中，那么就需要创建对应的实例：

    String childName;
    while (true) {
      int h = LoggerNameUtil.getSeparatorIndexOf(name, i);
      if (h == -1) {
        childName = name;
      } else {
        childName = name.substring(0, h);
      }
      // move i left of the last point
      i = h + 1;
      synchronized (logger) {
        childLogger = logger.getChildByName(childName);
        if (childLogger == null) {
          childLogger = logger.createChildByName(childName);
          loggerCache.put(childName, childLogger);
          incSize();
        }
      }
      logger = childLogger;
      if (h == -1) {
        return childLogger;
      }
    }

这里有个特殊处理，logback会从root开始依次检查并创建直到当前name的各层Logger实例。我们来看看Logger对象的实现：

	public final class Logger implements org.slf4j.Logger {
		private String name;
		transient private Logger parent;
		transient private List<Logger> childrenList;
	}

每个Logger对象都保存有自己的name，parent，和childrenList。其中的parent属性除了root logger之外都会指向当前logger的parent。

对于我们的例子，getLogger("a.b.c.D")，logback会依次检查以下各个Logger实例的是否存在，如果不存在则依次创建，并加入cache。

	a
	a.b
	a.b.c
	a.b.c.D

其中logger "a"的parent是root logger，"a.b"的parent是"a", 最后的"a.b.c.D"的parent是"a.b.c"。至此getLogger("a.b.c.D")方法完成了从root到当前"a.b.c.D" logger对象的所有相关的logger的创建。

总结一下，在logger对象的创建过程中，logback实际做了以下两个事情：

1. 创建了从root logger开始到当前logger的所有层次的logger
2. 所有的logger对象都被cache

