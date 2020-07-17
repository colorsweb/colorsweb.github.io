---
title: BFC
date: 2019-8-30 16:55:39
subtitle: 
header-img: 
top: 
tags: 
- BFC 
- margin重叠
categories: CSS
cover: true
coverImg: '../../img/21.jpg'
---
BFC全称是Block Formatting Context，即块格式化上下文。它是CSS2.1规范定义的，关于CSS渲染定位的一个概念。在实际开发过程中我们遇到的很多难以理解的坑实际上很多都是由于BFC造成的，例如margin重叠，元素高度塌陷等等，如果正确理清了BFC是怎么影响元素布局的，就能在开发中有更清晰的思路

### 如何触发BFC？

* 根元素或其它包含它的元素；
* 浮动 (元素的float不为none)；
* 绝对定位元素 (元素的position为absolute或fixed)；
* 行内块inline-blocks(元素的 display: inline-block)；
* 表格单元格(元素的display: table-cell，HTML表格单元格默认属性)；
* overflow的值不为visible的元素；
* 弹性盒 flex boxes (元素的display: flex或inline-flex)；

### BFC的作用
#### 内部块级盒会在垂直方向上按顺序排列
因为根元素为一个BFC，所以内部的块级盒都是垂直方向排列的，宽度默认为根元素的宽度
#### 浮动元素不叠加到BFC上
我们先看一个例子：
	
	<style>
	body{
		padding:0;
		margin:0;
	}
	.container{
		width: 300px;
		background: #eee;
	}
	.float{
		width:100px;
		height: 200px;
		float: left; 
		background: rgba(200,100,70,0.5);
	}
	.box{
		width: 200px;
		height: 100px;
		background: blue;
	}
	</style>

	<div class="container">
		<div class="float"></div>
		<div class="box"></div>
	</div>

![](/medias/bfc/1.png)
可以看到浮动元素和正常流的元素发生了重叠，原因是.box盒子并不是一个BFC
这时，我们让box触发BFC：

	.box{
		overflow: hidden;
	}

![](/medias/bfc/2.png)
两个盒子不再叠加了，验证了浮动元素不叠加到BFC上


#### 计算BFC的高度时，浮动子元素也参与计算
还是上面的例子，我们让父元素container触发BFC

	.container{
		overflow: hidden;
	}

![](/medias/bfc/3.png)
在父元素触发了BFC之后，计算高度时也把内部的浮动元素算进去了，当然，我们也可以在内部添加一个清除浮动的div来实现

#### 垂直方向的距离由margin决定(属于同一个BFC的两个相邻Box的margin会发生重叠)

	body{
		padding:0;
		margin:0;
	}
	.box1{
		width: 100px;
		height: 50px;
		background: blue;
		margin: 50px;
	}
	.box2{
		width: 100px;
		height: 50px;
		background: blue;
		margin: 50px;
	}
	<div class="box1"></div>
	<div class="box2"></div>
![](/medias/bfc/4.png)
虽然两个盒子都设置了50px的margin，但是实际的距离只有一个margin，但是这个并不重要，因为我们只需要知道两个盒子之间的距离就行了
子元素和父元素也会产生margin重叠：
	
	.box{
		width: 200px;
		height: 200px;
		background: blue;
	}
	.sonBox{
		width: 100px;
		height: 100px;
		background: red;
		margin-top: 50px;
	}
	<div class="box">
		<div class="sonBox"></div>
	</div>
![](/medias/bfc/5.png)
我们对子元素设置了50px的margin-top，但是却对父元素产生了效果，自身并没有位移，即使添加父元素的margin-top为50px，也不会发生改变，因为两者的margin重叠了
触发父元素的BFC：

	.box{
		overfloe: hidden;
	}
![](/medias/bfc/6.png)
此时子元素属于父元素的BFC中，而父元素则属于根元素的BFC中，margin重叠就修复了

#### 每个元素的margin-box的左边，与包含块border-box的左边相接触。即使存在浮动也是如此
我们先要清除什么是包含块？
> 包含块有可能是某个盒子的content-box，也有是某个盒子的padding-box。这取决于被包含块所包含的盒子的position属性

* 对于position为static或者relative的元素，包含块为它的父级元素的content-box
* 对于position为absolute的元素，包含块为离他最近的position不为static的祖先元素(没有就为body)的padding-box
* 对于position为fixed的元素，包含块为视口