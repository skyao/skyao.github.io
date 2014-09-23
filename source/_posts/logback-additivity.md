title: logback中additivity属性源码分析
date: 2014-09-23 11:38:25
categories: Log
tags: [log,logback]
---

在前面研究logback打印日志内容的过程中，我们跳过了不少细节。现在回头来逐个看看，先看看logger的additivity属性。

<!--more-->

我们先来看代码。

## 源代码

### additivity定义

additivity是Logger对象的一个基本属性，在类 Logger 中定义如下：

	public final class Logger implements org.slf4j.Logger, LocationAwareLogger,
    		AppenderAttachable<ILoggingEvent>, Serializable {

		transient private boolean additive = true;

### 配置方式

logger的additivity属性可以在logback中配置, 缺省值为true（请注意上面代码中additive 默认初始化为true）。

	<configuration>
	  ......
	  <logger name="chapters.configuration.Foo" additivity="false">
	    <appender-ref ref="FILE" />
	  </logger>
	
	  <root level="debug">
	    <appender-ref ref="STDOUT" />
	  </root>
	</configuration>

### 读取配置

logback在读取logback.xml文件配置时，会读取logger的additivity属性并设置。

	public static final String ADDITIVITY_ATTRIBUTE = "additivity";

    String additivityStr =  ec.subst(attributes.getValue(ActionConst.ADDITIVITY_ATTRIBUTE));
    if (!OptionHelper.isEmpty(additivityStr)) {
      boolean additive = OptionHelper.toBoolean(additivityStr, true);
      addInfo("Setting additivity of logger [" + loggerName + "] to "
          + additive);
      logger.setAdditive(additive);
    }

注意OptionHelper.toBoolean(additivityStr, true) 这句代码，如果additivity属性没有设置，则会使用默认值true。这里行为和Logger中additive 设置为 true 的行为是一致的。

### 打印日志时的代码

这里是关键代码了，在callAppenders()方法，addtivie属性被使用到：

	public void callAppenders(ILoggingEvent event) {
		int writes = 0;
		for (Logger l = this; l != null; l = l.parent) {
		  writes += l.appendLoopOnAppenders(event);
		  if (!l.additive) {
		    break;
		  }
		}
	}

## 代码分析

我们之前在"[logback中logger对象的创建过程源码分析](http://skyao.github.io/2014/08/08/logback-logger-creation/)"中研究过，当我们通过getLogger(“a.b.c.D”)这样的代码来创建logger对象时，实际logback会创建从root开始到当前logger对象的每一级logger，从parent到children顺序是这样的：

	root logger
	a
	a.b
	a.b.c
	a.b.c.D

而在执行logger.debug("***")做日志输出时，如上面callAppenders()方法的代码所示，logback会从当前logger开始游历整个层级结构：

	for (Logger l = this; l != null; l = l.parent)

通过parent属性依次向上游历，直到root logger（只有root logger的parent为null）。这个是logback的默认行为，如果additive设置为false，则打破这个行为：

	if (!l.additive) {
		break;
	}

## 理解 additivity

要理解logger的additivity属性，最好和logger的 Named Hierarchy （命名层级）联系起来。

Logback是是这样介绍Named Hierarchy的
	
*A logger is said to be an ancestor of another logger if its name followed by a dot is a prefix of the descendant logger name. A logger is said to be a parent of a child logger if there are no ancestors between itself and the descendant logger.*

再结合logger对象的生成方式和日志打印时对logger和appender的调用，就很清晰了：logback是通过创建整个层级的的多个logger实例来实现这个命名层级。然后在日志打印时，游历整个命名层级的所有logger，将日志内容输出到这些logger配置的appender来实现最终的日志打印。

![logback-process.png](/images/logback-process/logback-process.png)

additivity属性可以用来提前退出这个logger的多层级游历，从而使得在这个层级结构中，处于底层的logger可以有机会定义自己特殊的行为并且跳过上层logger。配合logger的leve和appender设置，可以组合出非常有弹性的方案。