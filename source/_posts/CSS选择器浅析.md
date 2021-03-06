---
title: CSS选择器浅析
date: 2019-8-26 10:58:41
subtitle: 
header-img: 
top: 
tags: 
- 选择器 
- 权重
categories: CSS
# cover: true
# coverImg: '../../img/11.jpg'
---
在CSS中，选择器是一种模式，用于选择需要添加样式的元素。在实际开发过程中，合理运用选择器会让开发更快捷，代码更简洁，甚至可以结合HTML去实现一些逆向选择的技巧。本文对CSS选择器中一些常用例子做了简要说明，并对一些理论做了解析

## 选择器分类

### 后代选择器
`.link li {}`
选择link类元素的后代元素中所有li标签

### 子选择器
`div > p {}`
选择div标签的所有p子元素，如果p元素为子元素的后代则无效

### 兄弟选择器
`.bro1 + .bro2 {}`
选择bro1类元素相邻且拥有相同父元素的bro2元素，注意css文档里bro2类必须在bro1类的后面

### 属性选择器
`a[tittle]{}`  
表示对有tittle属性的连接进行设置

### 伪类与伪元素

#### 伪类
* `:hover , :visited , :active , :focus`
分别表示鼠标停留时，已访问之后，鼠标点击时，获得焦点时

* `p:first-child , p:last-child`
分别表示任何一个元素的第一个p子元素和最后一个p子元素

* `p:nth-child(2)`
表示任何一个元素的第二个子元素p，如果第二个子元素不是p，则无效

* `p:nth-of-type(2)` 
表示任何一个元素的第二个p子元素,无论p是否为第二个子元素都生效

#### 伪元素
`::before , ::after`
分别表示在元素前面和后面插入内容

---

## 选择器权重

### 使用选择器遵守的原则
1. !important 优先级最高，但也会被权重高的important所覆盖
2. 内联样式总会覆盖外部样式表的任何样式
3. 如果两个权重不同的选择器作用在同一元素上，权重值高的覆盖权重低的
4. 如果两个相同权重的选择器作用在同一元素上，在文档后面的覆盖前面的

### 选择器权重值
1. 内联样式： 1000
2. ID选择器： 100
3. 类、伪类、属性选择器： 10
4. 元素、伪元素： 1

### 选择器使用建议
1. 不要使用 !important
2. 少用id选择器，只在必要时使用id增加权重
3. 避免层层嵌套使用选择器

***
## 关于父选择器
> CSS之所以不支持父选择器，是因为在加载css树时，如果支持父选择器，就会导致所有子元素加载完才能渲染父元素，而渲染是自上而下的，这样会严重影响渲染速度，这也是兄弟选择器不支持向上选择的原因

如何通过css实现父选择器呢？
可以参考张鑫旭的[如何在CSS中实现父选择器效果?](https://www.zhangxinxu.com/wordpress/2016/08/css-parent-selector/)