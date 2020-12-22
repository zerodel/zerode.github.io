---
layout: post
title:  "R语言如何完成同一画布中多个图形的布局"
date:   2020-12-21
categories: [memo]
---


## R语言如何完成同一画布中多个图形的布局
本文将简要介绍如何在R语言中同时绘制多个图表. 

我们将分别对baseR 体系和ggplot体系下的实现方式进行简要的介绍. 


### base R
base R 环境就是R语言环境自带的绘图系统. 
比如以下的的代码,就是使用自带的绘图来绘制了`mtcars` 数据集中的`mpg`与`hp`之间关系的散点图. 

```R
data(mtcars)
plot(mpg~hp, data = mtcars)
```

在base R 的体系中,有以下三种方法来完成多个图形的布局. 

+ mfrow/nfcol 
+ layout()
+ fig

其中, mfrow/nfcol 和 fig 都是par函数的参数, 修改它们来调整R的图形系统.

#### mfrow/nfcol 参数

mfrow nfcol 这两个参数都是par函数用来建立图形矩阵的. 

它们设置的格式也是一样的. 

```R
mfrow=c(nrows, ncols)
nfcol=c(nrows, ncols)
```
两者的区别在于使用**mfrow**的时候, 接下来绘制的多个图表是*按行填充*进矩阵的. 
而**nfcol** 是*按列填充* ,

使用案例如下

```R
with(mtcars, {
opar <- par(no.readonly=TRUE)
par(mfrow=c(2,2))
plot(wt,mpg, main="Scatterplot of wt vs. mpg")
plot(wt,disp, main="Scatterplot of wt vs disp")
hist(wt, main="Histogram of wt")
boxplot(wt, main="Boxplot of wt")
par(opar)

})

```
以上的代码中, 

首先把R绘图系统的当前状态保存到`opar`中, 
然后用mfrow 设置一个2*2 的矩阵. 

逐个绘制四个图形后, 

再次使用opar来还原R绘图系统

#### layout()函数

和上面的mfrow不同, layout是一个单独的函数. 

```R
layout(matrix(c(1,1,2,3), 2, 2, byrow = TRUE))
```
上面的代码说明我们希望绘制3个图表, 记作1,2,3. 

这三个图表按行排列, 依次安排在2*2 矩阵中,
其中 c(1,1,2,3)表明了 图表1 需要占用按行排列的前两个格子. 也就是第一行. 

```R
with(mtcars, {
layout(matrix(c(1,1,2,3), 2, 2, byrow = TRUE))
hist(wt)
hist(mpg)
hist(disp)
})
```


#### fig参数

fig参数的设置格式如下:

```R
par(fig=c(x1, x2, y1, y2))
```
它设定了之后的图表在画布中占据的矩形区域. 
这个区域由 (x1, y1)作为左下角, (x2,y2)作为右上角. 

```R
opar <- par(no.readonly=TRUE)
par(fig=c(0, 0.8, 0, 0.8))            # 设置散点图
plot(mtcars$wt, mtcars$mpg,
     xlab="Miles Per Gallon",
     ylab="Car Weight")
par(fig=c(0, 0.8, 0.55, 1), new=TRUE)        # 在上方添加箱线图
boxplot(mtcars$wt, horizontal=TRUE, axes=FALSE)
par(fig=c(0.65, 1, 0, 0.8), new=TRUE)        # 在右侧添加箱线图
boxplot(mtcars$mpg, axes=FALSE)
mtext("Enhanced Scatterplot", side=3, outer=TRUE, line=-3)
par(opar)
```

注意由于fig 参数设置后会默认在新建的画布会绘图. 
需要在par函数中使用 new=T 来添加图表,而不是新建画布. 
	

### ggplot 

与R自带的绘图系统不同, ggplot绘图是建立在grid 绘图系统上的. 所以刚刚的方法不适用. 

而为了在ggplot的绘图中完成多个图表的同时绘制. 需要一些第三方包的协助. 
比如 `girdExtra` , `gtable` `ggpubr` 等等. 

这些包之间的关系如下:

![](/assets/img/ggplot_grid_egg.jpg)

而在这些包的帮助下, ggplot图形的布局功能也更强, 
不只是将多个图表摆在一起. 


接下来将简要介绍以下内容:

+ 图表的平铺
+ 图表的插入
+ 图表的对齐
+ 表格和图的混排

在进入这些具体的步骤之前,有一些概念需要提前了解一下. 

以上这些绘图的包都依赖于`grid`包,
而在`grid`包中, 图表是以图形对象(graphic object,grob)的形式存在的, 
并在某个 viewports 下展示在画布上. 

刚刚提及的各个包都是为grobs 和viewports 提供了更加人性化的接口. 

在ggplot2 包中`ggplotGrob`函数可以把图表转换成grob对象. 

#### 图表的平铺

假设你已经用ggplot绘制了一系列图表, 比如p1, p2, p3, p4

那么实现平铺的最简单的方法就是使用 `gridExtra::grid.arrange` 函数. 

```R
grid.arrange(p1,p2,p3,p4, nrowl=2)
```

如果需要对矩阵的尺寸进行微调,那么就需要用`ggplot2::ggplotGrob`函数来把图表转换成grobs的列表,并调整grid.arrange函数的更多参数. 

注意`grid.arrange`函数只是平铺图表, 并不对图表对齐. 如果需要对齐的话,需要另外的方法

#### 图表的插入
ggplot2中的 `annotation_custom`就可以在图表中插入一个小图表.

但注意, 这个函数的输入是一个grob. 



#### 图表的对齐

对齐的简单处理就是使用 ggplot 的facet_* 系列函数.
但facet只能在同一个数据集中按照某些分类变量来将这个数据集绘制成多个图表. 


##### 使用gtable实现
如果无法使用facet_*函数, 那么可以使用gtable中的 rbind cbind 函数来把 grob 组合成一个gtable. 
并使用 grid.draw 把这些组合好的图表绘制出来. 


```R
library(gtable)
g2 <- ggplotGrob(p2)
g3 <- ggplotGrob(p3)
g <- rbind(g2, g3, size = "first")
g$widths <- unit.pmax(g2$widths, g3$widths)
grid.newpage()
grid.draw(g)

```

上面的代码中, 首先把ggplot的图表转换成grob. 
之后使用 `gtable::rbind` 来组合成一个`gtable`. 
设置好相关的宽度后. 

使用`grid.newpage`建立新画布.
然后用`grid.draw` 来绘制图形. 

##### 使用ggarrange实现

除了gtable 中的方法, 也可以使用`egg`包或者 `ggpubr`中的`ggarrange`来排布图表

这个函数的使用类似`grid.arrange`函数. 


#### 表格与图表的混排

上文提及到 `ggplot2::ggplotGrob(x)` 可以把图表x 转换成 grob 格式. 

类似的 `gridExtra::tableGrob` 函数同样可以把一个表格转换成grob格式. 
这样就可以实现表格和图表的混排. 

比如:

```R
grid.arrange(
  tableGrob(mtcars[1:4, 1:4]),
  p2,
  ncol = 2,
  widths = c(1.5, 1),
  clip = FALSE
)
```


### 参考内容

[1] https://cran.r-project.org/web/packages/gridExtra/vignettes/arrangeGrob.html

[2] https://cran.r-project.org/web/packages/gridExtra/vignettes/arrangeGrob.html
