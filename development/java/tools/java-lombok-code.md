---
title: Lombok 优雅编码
date: 2020-07-19 15:00:00
tags: 'Java'
categories:
  - ['开发', 'Java']
permalink: java-lombok-code
---

## 简介

Lombok 是一种 Java 实用工具, 可以简化各种 Java 对象 (POJO), 通过注解可以实现例如无参构造函数, 各种 `Getter`, `Setter` 方法等

使用 Lombok 需要引入相应的依赖

```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
```

还需要安装好相应的 IDE 插件, 例如 IDEA 插件 `lombok plugin`

<!-- more -->

## 详细

- `@Getter`: 自动生成 `Getter` 方法
- `@Setter`: 自动生成 `Setter` 方法
- `@ToString`: 自动生成 `ToString` 方法
- `@EqualsAndHashCode`: 自动生成 `EqualsAndHashCode` 方法
- `@Data`: 包含 `@Getter` `@Setter` `@ToString` `@EqualsAndHashCode` `@NoArgsConstructor`
- `@AllArgsConstructor`: 生成一个包含所有变量的构造函数, 如果变量使用了 `@NonNull`, 会进行是否为空的校验, 该注解的作用域也是只有在实体类上, 参数的顺序与属性定义的顺序一致
- `@NoArgsConstructor`: 生成无参构造函数
  - `boolean force() default false;` 参数设置为 `true` 将初始化所有的参数为默认值, 否则编译错误
- `@RequiredArgsConstructor`: 生成一个包含常量 (final), 和标识了 `@NonNull` 的变量的构造函数
  - 设置 `String staticName() default "";` 参数会将原来的构造方法的访问修饰符将会变成私有的, 另外添加一个静态构造方法, 参数相同, 名字是设置的值, 访问修饰符为公有的
  - `AnyAnnotation[] onConstructor() default {};` 参数可以在构造函数参数前添加注解, 例如 `@RequiredArgsConstructor(onConstructor = @__(@Autowired))` 将在构造函数所有参数前添加 `@Autowired` 注解
  - `AccessLevel access() default lombok.AccessLevel.PUBLIC;` 参数设置构造函数的修饰符
- `@Slf4j`: 为类注入一个 `log` 的日志变量
- `@Log`: 为类注入一个 `log` 的日志变量, 使用 `java.util.logging.Logger`
- `@Builder`: `bulder` 模式构建对象
- `@Cleanup`: 自动化关闭流, 相当于 `jdk1.7` 的 `try with resource`
- `val`: 自动类型推导
- `@NonNull`: 自动添加 `if null` 判断, 并抛出 `NullPointerException` 异常
- `@SneakyThrows`: 在当前方法上调用, 不用显示的在方法名后面添加 `throw`
- `@Synchronized`: 方法中所有的代码都加入到一个代码块中, 默认静态方法使用的是全局锁, 普通方法使用的是对象锁, 也可以设置值来指定锁的对象名称

## 例子

```java
@Service
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
@Slf4j
public class UserServiceImpl implements IUserService {
   private final IUserRepository userRepository;

   // 生成
   public UserServiceImpl(@Autowired IUserRepository userRepository) {
     log.info("xxx");
     this.userRepository = userRepository;
   }
   ...
}
```

```java
@Cleanup
InputStream in = new FileInputStream(args[0]);
@Cleanup
OutputStream out = new FileOutputStream(args[1]);
```

```java
val example = new ArrayList<String>();
example.add("Hello, World!");

// 转换为
List<String> example = new ArrayList<String>();
example.add("Hello, World!");
```

```java
public NonNullExample(@NonNull Person person) {
  this.name = person.getName();
}

// 转换为
public NonNullExample(Person person) {
  if (person == null) {
    throw new NullPointerException("person");
  }
  this.name = person.getName();
}
```

```java
@SneakyThrows(Exception.class)
```

```java
private final Object lock = new Object();
@Synchronized("lock")
public void foo() {
    // Do something
}
```
