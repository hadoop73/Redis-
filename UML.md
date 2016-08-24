---
title: UML 
tags: 
grammar_cjkRuby: true
---

[UML类图几种关系的总结][1]


**泛化（Generalization）**

子类是父类的一种
实线,三角箭头

![enter description here][2]


**实现（Realization）**
类实现一个接口
虚线,三角箭头

![enter description here][3]


** 关联（Association)**
拥有关系,如老师和学生
成员变量
实线,普通箭头

![enter description here][4]



**聚合（Aggregation）**
整体与部分,且部分可以离开整体而单独存在,如车和轮胎
成员变量
空心菱形,实线,普通箭头

![enter description here][5]



**组合(Composition)**
是整体与部分的关系，但部分不能离开整体而单独存在。如公司和部门
成员变量
实心菱形,实现,普通箭头


![enter description here][6]


**依赖(Dependency)**
使用关系,一个类需要另一个类协助
局部变量、方法的参数或者对静态方法的调用
虚线,普通箭头

![enter description here][7]


  [1]: http://www.open-open.com/lib/view/open1328059700311.html
  [2]: ./images/1471957885361.jpg "1471957885361.jpg"
  [3]: ./images/1471957897253.jpg "1471957897253.jpg"
  [4]: ./images/1471957924870.jpg "1471957924870.jpg"
  [5]: ./images/1471957940999.jpg "1471957940999.jpg"
  [6]: ./images/1471957956560.jpg "1471957956560.jpg"
  [7]: ./images/1471957965699.jpg "1471957965699.jpg"
