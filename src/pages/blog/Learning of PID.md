---
templateKey: blog-post
title: The Learning of PID
date: 2021-01-14T15:04:10.000Z
featuredpost: false
featuredimage: /img/flavor_wheel.jpg
description: PID学习
tags:
  - Control
---

## 一 · PID的基本认识

PID控制器**为比例单元（Proportional）、积分单元（Integral）和微分单元（Derivative）**组成，分别调节三项的系数来调节PID控制器。其主要用于**基本上呈线性、动态特性不随时间不变化**的系统。

---

## 二 · 调节PID实践的经验汇总：

![](https://gitee.com/cutice/my-blog-images/raw/master/images/PID2.jpg)

![](https://gitee.com/cutice/my-blog-images/raw/master/images/PID1.jpg)

---

## 三 · 调节pid基本步骤

 1：i和d置零，p给一个不大的数值 判断反馈符号是否正确 以云台为例，如果让云台偏移一个角度后开始震荡并且震荡发散（越振越大）则可以确定反馈符号反了 
如果反馈符号正常 则继续加大p直至出现震荡
出现震荡之后可以适当调低p（可以是70%左右，需根据实际情况来），但要保证电机仍然足够硬，即不会被手轻易拨动
随后调d d会一定程度上抑制p控制的幅度，并且使得电机对突然的变化比较敏感 调节d到云台符合运行要求即可

一般而言不需要i， i虽然可以消除一定稳态误差 但是可能会增加系统的不稳定性 pd控制器一般而言足够

上面的方式不绝对，需要根据实际情况出发。

---

## 四 · PID相关论文、资料

1. https://kns.cnki.net/KXReader/Detail?TIMESTAMP=637461657687949218&DBCODE=CJFD&TABLEName=CJFD2000&FileName=MOTO200003009&RESULT=1&SIGN=aMqEyFL9zLcLJm3Z6HvY7zVXrfk%3d

   PID参数先进整定方法综述

2. https://kns.cnki.net/KXReader/Detail?TIMESTAMP=637461656996240234&DBCODE=CJFD&TABLEName=CJFD2010&FileName=GXJM201007021&RESULT=1&SIGN=zWAuCuNMWwNAC%2fD1Eg5wBt5y6Dk%3d

   变结构PID在大型望远镜速度控制中的应用

3. https://zhuanlan.zhihu.com/p/129254220 PID算法解析与公式推导
4. https://blog.csdn.net/tingfenghanlei/article/details/85028677 PID控制详解
5. https://www.zhihu.com/question/293450508/answer/488607731 风控算法中双环PID的理解
6. https://zh.wikipedia.org/wiki/PID%E6%8E%A7%E5%88%B6%E5%99%A8 Wiki PID控制器

