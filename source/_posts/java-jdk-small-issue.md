title: 小处不可随便:谈jdk源代码中的一点小瑕疵
date: 2015-02-28 16:54:36
categories: Java
tags: [jdk]
---

今日工作中发现的一个小东西，jdk中的一点小瑕疵，虽然微小到不足以造成任何问题，但是的的确确的影响了我心中JDK原有高大上的印象......

<!--more-->

# 问题

在class ClassLoader 中，对parallelLockMap的使用，方法getClassLoadingLock()，源代码如下:

	private final ConcurrentHashMap<String, Object> parallelLockMap;

    protected Object getClassLoadingLock(String className) {
        Object lock = this;
        if (parallelLockMap != null) {
            Object newLock = new Object();
            lock = parallelLockMap.putIfAbsent(className, newLock);
            if (lock == null) {
                lock = newLock;
            }
        }
        return lock;
    }

这里有个小问题，我们忽略parallelLockMap == null的情况，会发现"Object newLock = new Object()"这行代码总是要执行的，如果className对应的lock已经存在于parallelLockMap中，那么就会很无谓的浪费这个新new出来的Object()。

# 简单修复

要修复这个问题，代码非常简单，简单先get一下，如果为空才new：

	protected Object getClassLoadingLock(String className) {
		Object lock = parallelLockMap.get(className);
		if (lock != null) {
			return lock;
		}
		
		lock = new Object();
		Object existLock = parallelLockMap.putIfAbsent(className, lock);
		return existLock != null ? existLock : lock;
    }

这样只有多线程同时发现lock == null 才会出现多次new Object()造成可能的浪费，之后就再不会有上述的浪费了。

# 分析

其实修复代码所体现的技巧非常的朴实，也谈不上什么高明。

而JDK源码所体现出来的大大咧咧，非常令人惊讶。当然我们知道这里由于是装载类，所以getClassLoadingLock()被调用的次数不会太多，这个浪费的情况不会太严重。另外new Object()是创建最简单的Object对象，消耗足够地。实际上这里的问题不会造成任何实际的影响，从实际角度可以忽略。

但是，不会造成实质后果，就是随便乱写的理由吗？

# 建议

JDK的这种写法是绝对不可取的，也千万不可能随意效仿，毕竟不是每个地方都可以这么幸运的因为浪费足够小而可以无视后果。

比如，使用ConcurrentHashMap的最经典的场景之一，实现一个简单的对象cache。如果我们需要保存的对象比较大或者不是简单new就可以生成（比如需要cpu计算或者访问数据库获取数据），而且有大量的读取操作。那么JDK的这种写法会坑死人的

# 总结

小处不可随便，这应该是任何一个敬业的程序员应该时刻坚持的品质和素养。

更何况是在JDK这样高大上的地方......