---
title: 设计模式七大原则
date: 2020-07-11 10:00:00
tags: '设计模式'
categories:
  - ['开发', '设计模式']
permalink: design-pattern-basic-law
mathjax: false
---

## 简介

面向对象的设计模式有七大基本原则：

- 开闭原则 (Open Closed Principle，OCP): 对扩展开放，对修改关闭
- 单一职责原则 (Single Responsibility Principle, SRP): 一个类只负责一个功能领域中的相应职责
- 里氏代换原则 (Liskov Substitution Principle，LSP): 所有引用基类的地方必须能透明地使用其子类的对象
- 依赖倒转原则 (Dependency Inversion Principle，DIP): 依赖于抽象，不能依赖于具体实现
- 接口隔离原则 (Interface Segregation Principle，ISP): 类之间的依赖关系应该建立在最小的接口上
- 合成/聚合复用原则 (Composite/Aggregate Reuse Principle，CARP): 尽量使用合成/聚合，而不是通过继承达到复用的目的
- 最少知识原则 (Least Knowledge Principle，LKP) 或者迪米特法则 (Law of Demeter，LOD): 一个软件实体应当尽可能少的与其他实体发生相互作用

其中: **单一职责原则、开闭原则、迪米特法则、里氏代换原则和接口隔离原则就是平常熟知的 SOLID**
