---
title: runtime总结
date: 2019-10-31 20:00:01
tag: iOS
categories: 学习笔记
---

#### 前言

​    网上关于runtime的文章有很多，但是却几乎都是在讲怎么用，并没有讲清楚原理。所以在学习了很多关于runtime的知识之后，打算自己总结一下。

#### runtime是什么？

​    首先，runtime是明确实体是一个库，名字叫runtime，![image-20191031200820268](/Users/jackfrow/Library/Application Support/typora-user-images/image-20191031200820268.png)。

#### 有什么用？

   在OC中，95%的代码都是用C语言写的，剩下的就是一个runtime，说到底，oc的方法调用到最后都是调用一个函数，但是通过runtime这个系统库，它把寻找函数调用的通过封装到了runtime里面，于是OC就从C语言多了对象的概念，有了发消息的概念，说到底，就是让我们在编程时，有了不同的编程思维感受，能够更加方便的完成任务。