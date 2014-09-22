title: logback中日志打印流程源码分析
date: 2014-09-11 11:57:09
categories: Log
tags: [log,logback]
---

继续研究logback，看看logback中是如何将日志信息打印出来的。

<!--more-->

## 代码分析

### Logger

以下面这件最简单的日志打印代码为例：

	logger.debug("hello world!");

先看一下代码实现：

	public void debug(String msg) {
    	filterAndLog_0_Or3Plus(FQCN, null, Level.DEBUG, msg, null, null);
	}

只是对filterAndLog_0_Or3Plus()方法的简单调用，和其他的info(),error()等方法的差别只是参数Level的取值。干货在filterAndLog_0_Or3Plus()方法中：

	private void filterAndLog_0_Or3Plus(final String localFQCN,
      final Marker marker, final Level level, final String msg,
      final Object[] params, final Throwable t) {

		// step1
		final FilterReply decision = loggerContext
		    .getTurboFilterChainDecision_0_3OrMore(marker, this, level, msg,
		        params, t);
		
		// step2
		if (decision == FilterReply.NEUTRAL) {
		  if (effectiveLevelInt > level.levelInt) {
		    return;
		  }
		} else if (decision == FilterReply.DENY) {
		  return;
		}
		
		//step3
		buildLoggingEventAndAppend(localFQCN, marker, level, msg, params, t);
	}

这里面有三个步骤：

1. 执行TurboFilter的处理逻辑
2. 根据TurboFilter返回的结果和Level信息做判断，如果不需要写日志则直接退出
3. 需要写日志了，调用buildLoggingEventAndAppend()方法来构建日志并写入。

TurboFilter的处理逻辑我们后面再来仔细研究，先假设当前配置中没有设置TurboFilter的最简单的情况。这种情况下，下面的这个逻辑被执行，用来检查当前logger配置的日志等级和传入的日志请求指定的日志等级，以决定是否需要打印这条日志：

	if (decision == FilterReply.NEUTRAL) {
		  if (effectiveLevelInt > level.levelInt) {
		    return;
		  }
		

如果检测结果是需要打印，则调用buildLoggingEventAndAppend()方法：

	private void buildLoggingEventAndAppend(final String localFQCN,
	  final Marker marker, final Level level, final String msg,
	  final Object[] params, final Throwable t) {
		LoggingEvent le = new LoggingEvent(localFQCN, this, level, msg, t, params);
		le.setMarker(marker);
		callAppenders(le);
	}

buildLoggingEventAndAppend()中先构建了LoggingEvent对象，然后调用callAppenders()方法：

	public void callAppenders(ILoggingEvent event) {
		int writes = 0;
		for (Logger l = this; l != null; l = l.parent) {
		  writes += l.appendLoopOnAppenders(event);
		  if (!l.additive) {
		    break;
		  }
		}
		// No appenders in hierarchy
		if (writes == 0) {
		  loggerContext.noAppenderDefinedWarning(this);
		}
	}

callAppenders()方法中从当前Logger开始，通过每个logger对象的parent属性依次向上游历所有的相关logger，直到root logger。如果遇到有某个logger的additive属性被设置为false，则中断这个向上的游历过程。additive属性的使用我们后面再来总结，这里先简单跳过。

### Appender

游历过程中，每个logger的appendLoopOnAppenders()方法将被调用，这个方法会返回其调用并使用过的appender的个数，writes变量会统计总数。游历结束后，会检查writes统计的结果，如果为0表明整个过程中都没有任何appender被使用，则会打印警告：
	
	final void noAppenderDefinedWarning(final Logger logger) {
		if (noAppenderWarning++ == 0) {
		  getStatusManager().add(
		          new WarnStatus("No appenders present in context [" + getName()
		                  + "] for logger [" + logger.getName() + "].", logger));
		}
	}

所以平时如果看到这个警告，就要注意检查对应logger的appender设置了。

继续看appendLoopOnAppenders()方法：

	transient private AppenderAttachableImpl<ILoggingEvent> aai;

	private int appendLoopOnAppenders(ILoggingEvent event) {
		if (aai != null) {
		  return aai.appendLoopOnAppenders(event);
		} else {
		  return 0;
		}
	}

这里涉及到AppenderAttachableImpl类的实例aai，这个AppenderAttachableImpl类也挺复杂的，涉及到并发和线程安全，后面依然会有单独的章节来细细研究它。我们先跳过它的细节，将它简化为一个appender list，这样看aai.appendLoopOnAppenders()方法中的代码就简单了：

	public int appendLoopOnAppenders(E e) {
		int size = 0;
		  for (Appender<E> appender : appenderList) {
		    appender.doAppend(e);
		    size++;
		  }
		return size;
	}

取出当前logger对象的所有appender(注意logger对象容许配置多个appender，虽然我们日常很少这样使用)，逐个调用每个appender的doAppend()方法。logback中appender的实现有很多种，我们以最简单的ConsoleAppender为例：

	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder>
		  <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
		</encoder>
	</appender>
	<root level="DEBUG">          
		<appender-ref ref="STDOUT" />
	</root>

这里有几个术语，几乎所有的appender都会使用到：

1. encoder 这个是logback从 0.9.19 版本之后引入的，替代原有的layout
2. layout 不再直接被appender使用，为了保持兼容，有一个LayoutWrappingEncoder 类将layout包装为encoder
3. pattern 用来指定日志打印的格式

上面例子中的日志内容将被pattern格式化，然后被encoder输出到控制台打印出来，这样 logger.debug("hello world!") 方法就完成了它的功能。

## 总结

最后，我们来总结一下logback打印日志的总体流程(在跳过若干细节)：

![logback-process.png](/images/logback-process/logback-process.png)

注意在这个过程中，有两次循环：

1. 从当前logger到root logger，按照parent属性依次游历
2. 调用logger对象的每个appender

