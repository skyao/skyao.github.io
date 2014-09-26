title: logback中的filter
date: 2014-09-23 16:14:47
categories: Log
tags: [log,logback]
---

在前面研究logback是如何打印日志内容的过程中，我们跳过了TurboFilter和Filter的内容。现在我们来看看这两个功能是如何工作的。

<!--more-->


## 介绍

在logger的level设置之外，logback提供Filter和TurboFilter两个功能，用于控制是否容许日志内容输出。

### Filter

Filter全限定名是 ch.qos.logback.core.filter.Filter，来自logback-core。用于实现自定义的事件过滤。简化后的代码如下：

	public abstract class Filter<E> extends ContextAwareBase implements LifeCycl {
		public abstract FilterReply decide(E event);
	}

decide()方法接受一个泛型event，通常情况下都是logback的LoggingEvent。

### TurboFilter

TurboFilter全限定名是 ch.qos.logback.classic.turbo.TurboFilter， 来自logback-classic。简化后的代码如下：

	public abstract class TurboFilter extends ContextAwareBase implements LifeCycle {
		public abstract FilterReply decide(Marker marker, Logger logger,
		  Level level, String format, Object[] params, Throwable t);
	}

TurboFilter的decide()方法和Filter不同，输入的参数不是event，而是和LoggingEvent密切相关的多个参数。

### FilterReply

两个filter的decide()方法都返回FilterReply，这个是过滤的结果。FilterReply是个枚举类型：

	public enum FilterReply {
	  DENY,
	  NEUTRAL,
	  ACCEPT;
	}

DENY拒绝，ACCEPT接受，NEUTRAL中立。



### 区别

## 代码分析

### log打印的全过程

### TurboFilter

### Filter

## 总结
