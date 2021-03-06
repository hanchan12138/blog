---
title: CSS Masonry Layouts（1）—— multi-columns
date: 2017-04-24
---

## CSS Masonry Layouts（1）—— multi-columns

Masonry Layouts —— 瀑布流布局，核心是一个网格布局，每行包含的内容列表的高度是不可控的，并且，每个内容列表呈堆栈状排列。称作瀑布流，最关键的是——**各堆栈间没有多余的间距差**（将整个布局看做一个大的容器，最新进来的内容元素，永远在高度最短的那一列上）。具体如下图所示：

![](https://cdn.int64ago.org/srwap2bn.jpeg)

<!-- more -->

注：上图是 二次元社区GACHA 的插画频道，该瀑布流插件基于NEJ，交互要求是：**每一列固定宽度，根据页面大小显示不同的列，页面宽度，重排，预加载一屏，滚动加载**。设计的思路是：

1. **计算列数**：根据当前页面（容器）宽度，计算一行可以排放几个元素
2. **初始堆栈**：建立列堆栈数组，用于存放各个列堆栈的高度
3. **最小索引**：得到列堆栈数组中的最小值和其对应索引
4. **元素定位**：所有元素 ```absolute``` 定位，设置后续加入元素的 ```top``` 值为列堆栈数组中的最小值，同时设置它的 ```left``` 值为该索引对应列的 left 值
5. **更新堆栈**：更新列堆栈数组中该索引下元素的值( no.4 中的 ```top``` 值加元素本身的的高度)

* 注：上面略去了对数据的预处理（包括图片的等比缩放以及属性的二次处理等）以及滚动加载的处理过程，对于预加载一屏，由于元素高度的不可控性，只能根据瀑布流元素的平均高度行预估计，然后推算出一屏需要元素的范围，然后请求相应数量。

简化版应该是下面这样的：

![](https://cdn.int64ago.org/r4oy5tyl.jpg)

但是，这样处理，表现层的东西依赖 ```javaScript``` 来处理，有点多余（当时确实是唯一的办法），能不能用样式来搞定呢？

首先，会想到 ```float``` 或者 ```inline-block``` ，但是它们都没办法很好的控制列表之间的间距。最终得到的效果就像下面这样：

![](https://cdn.int64ago.org/d26yfnkb.jpeg)

那，除此之外，就没有单纯用 ```css``` 可以搞定的方法了吗？近几年，```css``` 的技术更新频繁，出现了很多新的布局方法：```multi-columns```、```Flexbox```、```Grid```。用上面提到的布局方法能否实现瀑布流呢？

### multi-columns

```multi-columns``` 产生之初，是用来实现文本多列排列，类似报纸、杂志的文本排列方式。```multi-columns``` 的**列**布局方式跟瀑布流的有些类似：

[demo](https://codepen.io/realign/full/ybeBMm/)

三列：

![](https://cdn.int64ago.org/8tds2ivs.jpeg)

四列：

![](https://cdn.int64ago.org/fimtkz9y.jpeg)

![](https://cdn.int64ago.org/p7qw71k4.jpeg)

猛一看很完美。但是，与使用 js 实现对比之下：

* ```multi-columns``` 的元素是纵向排列的（在对元素先后次序有要求的情况下不理想）
* 随着容器宽度变化，明显发现出现比较大的空白区域（从三列到四列）

第一个问题，```multi-columns``` 的这种布局方式，决定了只能纵向排列；对于第二个问题:

### multi-columns 简略布局思想

首先，只有在**多列元素集含有块级元素、并且避免在元素内部断行并产生新列**的时候，才会涉及到布局，上面的瀑布流例子，要是不避免在元素内部断行并产生新列，将会是这样的：

![](https://cdn.int64ago.org/zkfzanl.jpeg)

只有将属性 ```break-inside``` 设置为 ```avoid```，才会有块级元素的效果。

回到上面说到的空白区域的问题，```multi-columns``` 的整体布局会受到几个因素的影响：

![](https://cdn.int64ago.org/vfhgqllp.jpeg)

总体而言，优先级：自身属性 > 容器属性。

当容器的高度小于按照 ```column-count``` 布局后的列的高度，会发生元素断裂、跨列的现象；当容器的高度大于按照 ```column-count``` 布局后的列的高度，单个列的高度会由最优布局算法生成。

所谓最优布局算法，简单来说，就是自适应：

* 首先取 ```column-count``` ，如果元素（block）个数不超过 ```column-count``` ，布局成一横排；

![](https://cdn.int64ago.org/mgd56d9.jpeg)

* 若是元素（block）个数超过 ```column-count``` ，首先在第一列增加 block 元素，而后以第一列的高度为标准，来填充后续各列（此时会发生列填充不满的情况）；

![](https://cdn.int64ago.org/d8ducmx.jpeg)

![](https://cdn.int64ago.org/db76hgs.jpeg)

* 若是之后元素中发现一个高度较高的**X元素**，对于布局的调整会以这个较高的**X元素**为分界线，不会将后面的低高度元素排列到**X元素**之前：

![](https://cdn.int64ago.org/bzd833en.jpeg)

就会出现较大空白区域的情况。
所以，出现空白区域的根本原因是：```multi-columns``` 布局的特点是**按列布局、顺序计算、顺序排列**，前面有较大空白区域，不会用后续元素去填补。

想要具体了解 ```multi-columns``` 布局，请参考：

* [CSS Writing Modes Level 3](https://www.w3.org/TR/css-writing-modes-3/#abstract-axes)
* [CSS3魔法堂：说说Multi-column Layout](http://www.cnblogs.com/fsjohnhuang/p/5412841.html)

***

### 总结：

```multi-columns``` 瀑布流布局，适用于：

1. 元素不存在优先级，或者优先级要求较低
2. 元素高度差异不明显（防止出现大片空白区域）
3. 不兼容低版本浏览器（最重要的）

除了 ```multi-columns```， ```Flexbox``` 以及 ```Grid``` 也可以运用到瀑布流布局中来。未完，待续。。。


