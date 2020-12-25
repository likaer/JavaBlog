---
title: Spring-Source-1
date: 2020-09-01 10:18:59
tags:
---



#Spring IOC容器

BeanFactory接口定义了最基本的IOC容器。

BeaFactory接口源码有一个常量FACTORY_BEAN_PREFIX，定义了用户使用容器时，可以通过在Bean前面加上转义符"&"来获取FactoryBean。

注意： 在Spring中，所有Bean都有BeanFactory来管理的，而FactoryBean则是一个能产生对象或者修饰对象的工厂类Bean