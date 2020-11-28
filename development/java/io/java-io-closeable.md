---
title: Closeable 接口
date: 2020-11-21 20:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', 'IO']
permalink: java-io-closeable
---

## 简介

Closeable 接口从 Java 5.0 开始引入, 其目的就是提供一个规范, 当使用资源的时候记得在用完调用 close 进行回收

一些占用操作系统资源的对象 (如文件, socket 句柄等) 操作都会实现 Closeable 接口

<!-- more -->

## 使用

实现统一规范的好处就是, 如果自行开发, 下面的 finally 中有可能出现 IOException 异常导致第二个 close 方法没有调用

```java
static void copy(String src, String dest)throws IOException {
  InputStream in = null;
  OutputStream out = null;
  try {
    in = new FileInputStream(src);
    out = new FileOutputStream(dest);
    byte[] buf = new byte[1024];
    int n;
    while ((n = in.read(buf)) >= 0) {
      out.write(buf, 0, n);
    }
  } finally {
    if (in != null) in.close();
    if (out != null) out.close();
  }
}
```

解决方法是将每个 close 调用都封装在 try catch 语句块中

```java

```java
static void copy(String src, String dest)throws IOException {
  InputStream in = null;
  OutputStream out = null;
  try {
    in = new FileInputStream(src);
    out = new FileOutputStream(dest);
    byte[] buf = new byte[1024];
    int n;
    while ((n = in.read(buf)) >= 0) {
      out.write(buf, 0, n);
    }
  } finally {
    closeIgnoringIOException(in);
    closeIgnoringIOException(out);
  }
}
private static void closeIgnoringIOException(Closeable c) {
  if (c != null) {
    try {
      c.close();
    } catch (IOException ex) { }
  }
}
```

## 带资源的 try 语句

从 Java 7.0 起引入了 `java.lang.AutoCloseable` 接口, 并且 `java.io.Closeable` 接口继承自它, 所以上述的资源类也都间接的实现该接口

带资源的 try 语句 (try-with-resources) 也是 Java 7.0 的新语法, 主要是为了更好的管理资源, 更准确的资源释放, 在使用 try-with-resources 语法创建的资源抛出异常后, JVM 会自动调用 close 方法进行资源释放, 正常退出 try-block 后也会调用 close 方法

```java
try ( /* 资源类对象的声明 */ ) {
  //可能有异常抛出的语句块
} catch (IOException e) {
  /* ... */
}
```

即使没有主动调用 close 方法, 在出现异常或者运行结束后, 都会对资源类对象按照声明的顺序逆序调用 close 方法, 既让代码更加精炼, 也减少了错漏的发生

```java
static void copy(String src, String dest) throws IOException {
  try (InputStream in = new FileInputStream(src);
    OutputStream out = new FileOutputStream(dest)){
    byte[] buf = new byte[1024];
    int n;
    while ((n = in.read(buf)) >= 0) {
      out.write(buf, 0, n);
    }
  } catch (Exception e) {
    log.debug("catch block: {}", e);
  } finally {
    log.debug("finally block");
  }
}
```

因为 InputStream 和 OutputStream 都间接实现了 AutoCloseable 接口, 所以在执行了 try-block 块后会先执行所有 try 方法中的资源声明实例的 close 方法

## 语法糖

使用 javap 解析相应的 class 文件后, 通过解析结果查表对照发现, 两段代码不同之处对应的字节码指令并不是此语法的含义, 是一种语法糖的处理, 在编译成 class 文件时已经由编译器 (javac) 处理了

反编译后结果如下

```java
static void copy(String src, String dest) {
    try {
        InputStream in = new FileInputStream(src);
        Throwable var3 = null;

        try {
            OutputStream out = new FileOutputStream(dest);
            Throwable var5 = null;

            try {
                byte[] buf = new byte[1024];

                int n;
                while((n = in.read(buf)) >= 0) {
                    out.write(buf, 0, n);
                }
            } catch (Throwable var44) {
                var5 = var44;
                throw var44;
            } finally {
                if (out != null) {
                    if (var5 != null) {
                        try {
                            out.close();
                        } catch (Throwable var43) {
                            var5.addSuppressed(var43);
                        }
                    } else {
                        out.close();
                    }
                }

            }
        } catch (Throwable var46) {
            var3 = var46;
            throw var46;
        } finally {
            if (in != null) {
                if (var3 != null) {
                    try {
                        in.close();
                    } catch (Throwable var42) {
                        var3.addSuppressed(var42);
                    }
                } else {
                    in.close();
                }
            }

        }
    } catch (Exception var48) {
        log.debug("catch block: {}", var48);
    } finally {
        log.debug("finally block");
    }

}
```

上面的处理使用了多个 finally 块保证了 out 和 in 两个资源实例能全部关闭

`Throwable.addSuppressed()` 方法将异常添加到当前异常的队列中, 当方法 read 或者 write 出现异常时 var5 变量便可以保存该异常和 `out.close()` 方法异常, var3 变量会保存该异常和 `in.close()` 方法异常

## 小结

使用 try-with-resources 语法声明的资源实例在执行过程中满足如下结果

- 无论是否抛出异常都会在 try-block 块执行完毕后调用所有在 try 方法中声明的资源实例的 close 方法
- 调用 close 方法的顺序与创建资源的顺序相反
- 执行完 close 方法后再执行 catch 块然后才是 finally 块

## 改进

在 Java 9.0 中对该语法进行改进, 具体参考 [JEP 213](https://openjdk.java.net/jeps/213)
