---
title: jquery简单代码实现相邻元素位置互换
tags: 
 - jquery
comments: true
date: 2018-05-06 00:34:14
---
### 场景
在页面上经常会用到相邻位置的同级元素要互换位置，我们可以用几行代码实现这样的效果。下面的方法中可能会有一些方法名的混淆。我尝试说明。
### 方法
#### 查找方法
* prev() 获得匹配元素集合中每个元素紧邻的前一个同胞元素，通过选择器进行筛选是可选的。
```
.prev(selector)
```
* next() 获得匹配元素集合中每个元素紧邻的同胞元素。如果提供选择器，则取回匹配该选择器的下一个同胞元素。
```
.next(selector)
```
#### 移动方法
* before() 方法在被选元素前插入指定的内容。
```
$(selector).before(content)
```
* after() 方法在被选元素后插入指定的内容。
```
$(selector).after(content)
```
### 左移、上移
```
var $cur = $(obj);
var $prev = $cur.prev(selector);
$prev.before($cur);
```
为方便理解，我们把代码翻译成人类语言，$prev.before($cur)这句的意思是把当前元素移动到前序元素之前，所以能实现左移、上移。
### 右移、下移
```
var $cur = $(obj);
var $next = $cur.next(selector);
$next.after($cur);
```
$next.after($cur)这句的意思是把当前元素移动到后序元素之后。
