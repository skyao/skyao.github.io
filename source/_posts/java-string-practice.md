title: java中String和byte数组转换的小技巧
date: 2016-01-13 23:37:46
categories: Java
tags: [java]
---

今日看公司代码时发现，在string和byte数组转换的过程中，大量的无聊try catch。所以写了本文，作一个java基本编程知识的小科普。

<!--more-->

分享一个java编程的小技巧，简单实用。

# 建议

其实内容就一句话：

> **在做String和byte[]的相互转换时，请使用StandardCharsets.UTF_8来替代"utf-8"**

解释一下，通常我们代码是这样写:

```java
String string = new String(bytes, "utf-8");
byte[] bytes = string.getBytes("utf-8");
```

请换成下面这个写法：

```java
String string = new String(bytes, StandardCharsets.UTF_8);
byte[] bytes = string.getBytes(StandardCharsets.UTF_8);
```

注： 其实大家看到这里就可以打住了

# 问题解释

第一个写法功能当然没问题，但是代码写完之后，IDE一定会立刻提醒你，这里会抛出UnsupportedEncodingException。

而最要命的是，UnsupportedEncodingException是继承自类Exceptio。

这是一个checked exception， 这是一个checked exception， 这是一个checked exception！

JDK的源代码如下：

```java
public String(byte bytes[], String charsetName)
        throws UnsupportedEncodingException {
    this(bytes, 0, bytes.length, charsetName);
}
```

这意味着我们要不加上try catch，要不就要在方法上显示申明要抛出异常。
而申明抛出UnsupportedEncodingException异常绝对不是一个好注意，鬼都知道这里这个UTF-8一定不会
unsupported，把这个麻烦扔给调用者绝对是一个不负责任的行为。

所以我在代码中看到大量的类似代码：

```java
try {
    String json = new String(data, "utf-8");
    // 此处略去××字
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}
```

# 解决方法

这个问题由来已久，java社区解决这个问题的方式也很早就有，没有记错的话 apache commons大概十年前就提供了
方案，注意jdk中提供的另外一个不抛UnsupportedEncodingException的构造函数：

```java
public String(byte bytes[], Charset charset) {
    this(bytes, 0, bytes.length, charset);
}
```

和前一个构造函数的差别就是这里直接输入了Charset对象，不需要做一次从string到Charset
的转化（这里才是UnsupportedEncodingException抛出的根源）。而我们日常要用到的charset是非常
有限的，因此只要简单列常来最常用的几个就好了。

JDK7之后，java引入了java.nio.charset.StandardCharsets来做charset预定义：

```java
public final class StandardCharsets {
    public static final Charset US_ASCII = Charset.forName("US-ASCII");
    public static final Charset ISO_8859_1 = Charset.forName("ISO-8859-1");
    public static final Charset UTF_8 = Charset.forName("UTF-8");
	......
}
```

在jdk之前，很多基本类库都提供类似的功能，比如大家熟悉的apache commons,这个是最早提供也是用的最多的的：

```java
org.apache.commons.codec.Charsets

    /**
     * @see <a href="http://docs.oracle.com/javase/6/docs/api/java/nio/charset/Charset.html">Standard charsets</a>
     * @deprecated Use Java 7's {@link java.nio.charset.StandardCharsets}
     */
    @Deprecated
    public static final Charset UTF_8 = Charset.forName(CharEncoding.UTF_8);
```

注意上面的注释，现在apache已经Deprecated 它了，建议大家用StandardCharsets。



