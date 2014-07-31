title: logback初始化过程源码分析
date: 2014-07-23 15:48:53
categories: Log
tags: [log,logback]
---

上一篇Blog中研究了一下Slf4j是如何在运行时和底层的log api实现做绑定。当时举的例子是slf4j-simple和log4j. 

这次我们来详细研究一下logback，顺便看看logback是怎么完成初始化配置的。

<!--more-->

# 和Slf4j绑定

logback和slf4j绑定的方式遵循slf4j的标准方式。

在logback-classic的jar包下，可以找到logback自己实现的org.slf4j.impl.StaticLoggerBinder。

StaticLoggerBinder是标准的sigleton模式。实现了slf4j要求的getSingleton()方法和getLoggerFactory()方法。

	private static StaticLoggerBinder SINGLETON = new StaticLoggerBinder();

	public static StaticLoggerBinder getSingleton() {
		return SINGLETON;
	}

	public ILoggerFactory getLoggerFactory() {
		......
		return contextSelectorBinder.getContextSelector().getLoggerContext();
	}

我们先忽略getLoggerFactory()方法的实现细节，只看返回的结果，发现logback中实现ILoggerFactory的类是LoggerContext。

	public class LoggerContext extends ContextBase implements ILoggerFactory,
        LifeCycle {
		......

		public final Logger getLogger(final String name) {
			......
		}
	}

getLogger()方法负责创建具体的logback的Logger对象，里面有缓冲，也有logback独特的自动创建从root到当前logger的中间各级logger，后面另外再来细看。

和slf4j的绑定就这么简单的完成了。

# logback初始化

## 概况

重点来看logback的初始化，回到类StaticLoggerBinder:

	private boolean initialized = false;

	static {
		SINGLETON.init();
	}

	void init() {
		......
		initialized = true;
	}

static代码段中执行init()方法做logback的初始化，initialized 属性保存初始化结果。在init()方法中如果初始化成功，则设置initialized=true。 

getLoggerFactory()方法中，如果发现初始化失败，就返回默认的defaultLoggerContext。否则就是用当前ContextSelector提供的LoggerContext。

	public ILoggerFactory getLoggerFactory() {
		if (!initialized) {
			return defaultLoggerContext;
		}
		
		if (contextSelectorBinder.getContextSelector() == null) {
			throw new IllegalStateException(
				"contextSelector cannot be null. See also " + NULL_CS_URL);
		}
		return contextSelectorBinder.getContextSelector().getLoggerContext();
	}

## 初始化过程

来看StaticLoggerBinder的init()方法的细节：

	void init() {
	    try {
	      try {
	        new ContextInitializer(defaultLoggerContext).autoConfig();
	      } catch (JoranException je) {
	        Util.report("Failed to auto configure default logger context", je);
	      }
	      // logback-292
	      if(!StatusUtil.contextHasStatusListener(defaultLoggerContext)) {
	        StatusPrinter.printInCaseOfErrorsOrWarnings(defaultLoggerContext);
	      }
	      contextSelectorBinder.init(defaultLoggerContext, KEY);
	      initialized = true;
	    } catch (Throwable t) {
	      // we should never get here
	      Util.report("Failed to instantiate [" + LoggerContext.class.getName()
	          + "]", t);
	    }
	}

“new ContextInitializer(defaultLoggerContext).autoConfig();” 这里是做defaultLoggerContext的自动配置。完成后会调用contextSelectorBinder.init()做Context Selector的绑定，最后设置initialized = true。

## Context Selector

Context Selector 是logback提供的一个特殊功能，按照[官方文档](http://logback.qos.ch/manual/contextSelector.html) 的说法是用来解决Logging Separation (日志隔离)问题的。

	The chapter deals with a relatively difficult problem of providing a separate logging environment for multiple applications running on the same web or EJB container. 

	Logback provides a mechanism for dealing with multiple contexts, without corruption of data, nor collision between logger context instances.

Context Selector这个坎有点深，我们这次先跳过。

在ContextSelectorStaticBinder的init()方法，会根据系统属性logback.ContextSelector来选择使用具体的Context Selector。

默认没有设置时，使用DefaultContextSelector。

	public void init(LoggerContext defaultLoggerContext, Object key) {
    	......
	    String contextSelectorStr = OptionHelper
	        .getSystemProperty(ClassicConstants.LOGBACK_CONTEXT_SELECTOR);
	    if (contextSelectorStr == null) {
	      contextSelector = new DefaultContextSelector(defaultLoggerContext);
	    } else if (contextSelectorStr.equals("JNDI")) {
	      // if jndi is specified, let's use the appropriate class
	      contextSelector = new ContextJNDISelector(defaultLoggerContext);
	    } else {
	      contextSelector = dynamicalContextSelector(defaultLoggerContext,
	          contextSelectorStr);
	    }
	}

DefaultContextSelector类只是简单保存传入的defaultLoggerContext，以后直接用。

对于默认情况，我们可以忽略Context Selector相关的内容。

## 初始化细节

细细看看ContextInitializer的代码，看看下面这代码具体做了什么，这个是logback初始化配置的关键之处。

	new ContextInitializer(defaultLoggerContext).autoConfig();

autoConfig()方法的代码：

	public void autoConfig() throws JoranException {
		StatusListenerConfigHelper.installIfAsked(loggerContext);
		URL url = findURLOfDefaultConfigurationFile(true);
		if (url != null) {
		  configureByResource(url);
		} else {
		  BasicConfigurator.configure(loggerContext);
		}
	}

findURLOfDefaultConfigurationFile()中有我们最关心的内容：

	public URL findURLOfDefaultConfigurationFile(boolean updateStatus) {
		ClassLoader myClassLoader = Loader.getClassLoaderOfObject(this);
		URL url = findConfigFileURLFromSystemProperties(myClassLoader, updateStatus);
		if (url != null) {
		  return url;
		}
		
		url = getResource(GROOVY_AUTOCONFIG_FILE, myClassLoader, updateStatus);
		if (url != null) {
		  return url;
		}
		
		url = getResource(TEST_AUTOCONFIG_FILE, myClassLoader, updateStatus);
		if (url != null) {
		  return url;
		}
		
		return getResource(AUTOCONFIG_FILE, myClassLoader, updateStatus);
	}	

从这段代码我们可以看到logback获取自动配置文件的顺序，依次是：

1. 系统配置“logback.configurationFile”，这个参数通常是通过"java -Dlogback.configurationFile=***" 这个方式设置的
2. 资源文件logback.groovy
3. 资源文件logback-test.xml，这个对开发时做测试非常有帮助
4. 资源文件logback.xml

## 系统配置的多种支持

注意第一步，系统配置“logback.configurationFile”的值可以有多种方式，logback做的很细致啊

	private URL findConfigFileURLFromSystemProperties(ClassLoader classLoader, boolean updateStatus) {
		String logbackConfigFile = OptionHelper.getSystemProperty(CONFIG_FILE_PROPERTY);
		if (logbackConfigFile != null) {
		  URL result = null;
		  try {
		    result = new URL(logbackConfigFile);
		    return result;
		  } catch (MalformedURLException e) {
		    // so, resource is not a URL:
		    // attempt to get the resource from the class path
		    result = Loader.getResource(logbackConfigFile, classLoader);
		    if (result != null) {
		      return result;
		    }
		    File f = new File(logbackConfigFile);
		    if (f.exists() && f.isFile()) {
		      try {
		        result = f.toURI().toURL();
		        return result;
		      } catch (MalformedURLException e1) {
		      }
		    }
		  } finally {
		    if (updateStatus) {
		      statusOnResourceSearch(logbackConfigFile, classLoader, result);
		    }
		  }
		}
		return null;
	}

为了支持多种数据来源，logback先后做了三次尝试来试图读取logback.configurationFile参数所指定的文件来源：

1. new URL(logbackConfigFile)，看看是不是URL.
2. 如果URl解析失败，说明不是URL，尝试用resource来解析，Loader.getResource(logbackConfigFile, classLoader)
3. 如果还不行，再试试看是不是文件路径，new File(logbackConfigFile)

这意味着我们在使用时，logback.configurationFile的值是可以设置为URL/资源/文件三种方式中的任何一种，应该足以满足日常使用要求了。