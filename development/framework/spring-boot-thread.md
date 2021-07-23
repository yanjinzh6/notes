---
title: spring-boot-thread
date: 2021-03-21 11:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-boot-thread
---

https://my.oschina.net/u/3035165/blog/919492

# [Spring boot 守护线程](https://my.oschina.net/u/3035165/blog/919492)

原创

[Geeyu](https://my.oschina.net/u/3035165)

[Java](https://my.oschina.net/u/3035165? tab= newest&catalogId=5607358)

2017/06/12 16:18

阅读数 5.1K

由于项目需要, 在系统启动之后, 要新起一条线程一直跑, 用作监听器, 使用回调方法处理将要发生的事情, 处理时, 需要用到 JPA 的接口.

因此, 需要设置一条守护线程, 并且可以自动装配 Spring Bean, 采用第三个方法.

方案一

```
@SpringBootApplication
public class App {

	// 发射 App
	public static void main(String[] args) {

		Thread thread = new Thread();
		thread.setDaemon(true);
		thread.start();

		SpringApplication app = new SpringApplication(App.class);
		app.run(args);
	}

}
```

方案二

```
@Bean
class EventSubscriber implements DisposableBean, Runnable {

    private Thread thread;
    private volatile boolean someCondition;

    EventSubscriber() {
        this.thread = new Thread(this);
    }

    @Override
    public void run() {
        while(someCondition) {
            doStuff();
        }
    }

    @Override
    public void destroy() {
        someCondition = false;
    }

}
```

方案三

```
 @Component
public class Listener implements ApplicationListener<ContextRefreshedEvent>, Runnable {

    @Autowired
    IUserRepo userRepo;// 使用 Spring data jpa, 接口, 没办法实现, 只能让 Spring 自动装配

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        Listener listener = event.getApplicationContext().getBean(Listener.class);
        new Thread(this).start();
    }

    @Override
    public void run() {
        System.out.println("run run run...");
    }
}
``````
 @SpringBootApplication
public class App {

	// 发射 App
	public static void main(String[] args) {
		SpringApplication app = new SpringApplication(App.class);
		app.setListeners(new Listener());
		app.run(args);
	}

}
```

方案四

ApplicationRunner 

展开阅读全文

© 著作权归作者所有

举报

打赏

0 赞

1 收藏

微信 QQ 微博

分享

### 作者的其它热门文章

[Springboot 占用内存太大](https://my.oschina.net/u/3035165/blog/1584140 "Springboot 占用内存太大 ")

[Apache POI .xlsx 下拉框实现](https://my.oschina.net/u/3035165/blog/1530016 "Apache POI .xlsx 下拉框实现 ")

[@oneToMany 级联保存 多的一方自动保存外键](https://my.oschina.net/u/3035165/blog/1581774 "@oneToMany 级联保存 多的一方自动保存外键 ")

[SpringBoot 多模块 自动装载 (@Autowired)](https://my.oschina.net/u/3035165/blog/903790 "SpringBoot 多模块 自动装载 (@Autowired)")
