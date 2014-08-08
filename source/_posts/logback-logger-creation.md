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

这里的getLogger()方法有两个版本，从代码上看，如果一个权限定名为"a.b.c.D"的类，通过getLogger(D.class)获取logger，本质上就是调用getLogger("a.b.c.D")。这个方式是slf4j定义的，logback只是简单遵循。

重点看getLogger(String name)方法的实现，忽略一些细节

	private Map<String, Logger> loggerCache;

	public final Logger getLogger(final String name) {
		
	    Logger childLogger = (Logger) loggerCache.get(name);
	    // if we have the child, then let us return it without wasting time
	    if (childLogger != null) {
	      return childLogger;
	    }
	}
