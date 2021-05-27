---
title: 回流 (Reflow)和重绘 (Repaint)
date: 2019-01-24 23:14
tags:
  - DOM
  - 回流
  - 重绘
---

浏览器使用流式布局模型 (Flow Based Layout)。
浏览器会把 HTML 解析成 DOM，把 CSS 解析成 CSSOM，DOM 和 CSSOM 合并就产生了 Render Tree。
有了 RenderTree，我们就知道了所有节点的样式，然后计算他们在页面上的大小和位置，最后把节点绘制到页面上。
由于浏览器使用流式布局，对 Render Tree 的计算通常只需要遍历一次就可以完成，但 table 及其内部元素除外，他们可能需要多次计算，通常要花 3 倍于同等元素的时间，这也是为什么要避免使用 table 布局的原因之一。

### 我们要明白的是，页面的显示过程分为以下几个阶段：

- 生成 DOM 树(包括 display:none 的节点)

- 在 DOM 树的基础上根据节点的集合属性(margin, padding, width, height 等)生成 render 树(不包括 display:none，head 节点，但是包括 visibility:hidden 的节点)

- 在 render 树的基础上继续渲染颜色背景色等样式。

### 回流 (Reflow)

会导致回流的操作：
页面首次渲染
浏览器窗口大小发生改变
元素尺寸或位置发生改变
元素内容变化（文字数量或图片大小等等）
元素字体大小变化
添加或者删除可见的 DOM 元素
激活 CSS 伪类（例如：:hover）
查询某些属性或调用某些方法
一些常用且会导致回流的属性和方法：
clientWidth、clientHeight、clientTop、clientLeft
offsetWidth、offsetHeight、offsetTop、offsetLeft
scrollWidth、scrollHeight、scrollTop、scrollLeft
scrollIntoView()、scrollIntoViewIfNeeded()
getComputedStyle()
getBoundingClientRect()
scrollTo()

### 重绘 (Repaint)

当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility 等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。

#### 回流一定伴随着重绘，而重绘却可以单独出现

### 优化回流和重绘

- 用 transform 代替 top，left ，margin-top， margin-left... 这些位移属性

- 用 opacity 代替 visibility，但是要同时有 translate3d 或 translateZ 这些可以创建的图层的属性存在才可以阻止回流, 但是第二点经过我的实验，发现如果不加 transform: translateZ(0) 配合 opacity 的话还是会产生回流的，而只用 visibility 就只会产生重绘不会回流,而 opacity 加上 transform: translateZ/3d 这个属性之后便不会发生回流和重绘了

- 不要使用 js 代码对 dom 元素设置多条样式，选择用一个 className 代替之。

- 如果确实需要用 js 对 dom 设置多条样式那么可以将这个 dom 先隐藏，然后再对其设置

- 不要在循环内获取 dom 的样式例如：offsetWidth, offsetHeight, clientWidth, clientHeight... 这些。浏览器有一个回流的缓冲机制，即多个回流会保存在一个栈里面，当这个栈满了浏览器便会一次性触发所有样式的更改且刷新这个栈。但是如果你多次获取这些元素的实际样式，浏览器为了给你一个准确的答案便会不停刷新这个缓冲栈，导致页面回流增加。
  所以为了避免这个问题，应该用一个变量保存在循环体外。

- 不要使用 table 布局，因为 table 的每一个行甚至每一个单元格的样式更新都会导致整个 table 重新布局

- 动画的速度按照业务按需决定

- 对于频繁变化的元素应该为其加一个 transform 属性，对于视频使用 video 标签

- 必要时可以开启 GPU 加速，但是不能滥用
