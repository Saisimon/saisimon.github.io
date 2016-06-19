---
title:
  Hibernate中的SessionFactory
date:
  2015/7/7 18:03:34
categories: Hibernate
tags: [Hibernate,SessionFactory,Java]
---
![image](http://ww4.sinaimg.cn/mw690/7c2c72d3gw1f2ojtjgebvj21kw0fsjsw.jpg)
# SessionFactory -- 用来产生和管理Session的工厂类
通常情况下，每个应用只需要一个SessionFactory对象，可以使用单例模式来产生SessionFactory对象。

## SessionFactory的openSession()和getCurrentSession()方法的区别
获得sessionFactory:
```java
SessionFactory sessionFactory = null;
Configuration cfg = new Configuration().configure();
sessionFactory = cfg.buildSessionFactory(new StandardServiceRegistryBuilder().applySettings(cfg.getProperties()).build());
```
<!-- more -->
通过openSession()方法获得session:
​每次调用openSession()方法，都会产生新的Session对象。并且需要close()操作。
```java
Session session = sessionFactory.openSession();
Session session2 = sessionFactory.openSession();
System.out.println(session == session2); //false
```
​​通过getCurrentSession()方法获得session:
在Session对象commit之前，每次调用getCurrentSession()方法，都会产生同一个Session对象。
Session对象commit之后,自动关闭，调用getCurrentSession()方法，会产生新的Session对象。
```java
Session session = sessionFactory.getCurrentSession();
Session session2 = sessionFactory.getCurrentSession();
session.beginTransaction();
session.save(object);
session.getTransaction().commit();
Session session3 = sessionFactory.getCurrentSession();
System.out.println(session == session2);//true
System.out.println(session == session3);//false
```
