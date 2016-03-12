---
layout: post
title: "AutoLayout进阶教程(1) "
date: 2015-06-07 21:43:17 +0800
comments: true
categories: iOS
---
## AutoLayout
入门的内容我就不讲了，网上也有不少。进阶教程的内容可能相对于读者您比较简单，但都是笔者工作中学习到的技巧，在此做个记录，还请多多包涵。

## 需求
在工作中往往我们会遇到这样的需求，对下图中的4个图片等距排列，并且在图片大小不变的情况下，每个图片的左右边距可以相对其父view扩大（本例讲的适配规则的实现方案可以基本满足于同类需求）

<img src="/images/2015-05-21-autolayoutjin-jie-jiao-cheng-1.jpg">
<img src="/images/2015-05-21-autolayoutjin-jie-jiao-cheng-6.jpg">

在本例中，4张图片的父view左右边距固定为20像素，每个图片宽高都是45个像素，通过图片的宽度我们就能配置好相应的适配规则了。

## 计算
<img src="/images/2015-05-21-autolayoutjin-jie-jiao-cheng-2.jpg">
上图为一个规则的属性配置项，我们可以看到有[First Item], [Relation], [Second Item], [Constant], [Priority], [Multiplier]等属性值可以配置，而它们之间有着紧密的关系，也是本例中最重要的地方。

它们之间的关系可以概括为一个式子：
```
[First Item] [Relation] [Second Item] * [Multiplier] + [Constant]
```

[Relation]有3个关系：等于、大于等于、小于等于。并且我们从图中可以注意到，[Multiplier]可以表示分数，用:来代替/号，即2:5就是5分之2的意思。
那么我们拿第一张图片bus来示范下如何设置好它的规则：
首先添加它的宽高规则为45，然后添加它在父view里的规则
<img src="/images/2015-05-21-autolayoutjin-jie-jiao-cheng-3.jpg">

这个时候添加的Center Horizontally In Container，是指图片bus和父view的水平中心线的距离，但添加的规则默认[Multiplier]为1，这可不是我们想要的，因为一旦对父view的宽度做拉伸，图片bus就会偏离我们的预期了。
<img src="/images/2015-05-21-autolayoutjin-jie-jiao-cheng-4.jpg">

因此我们选中图片bus之后，在右边的选项中可以看到添加在图片上的规则，双击Align Center X to: Superview，跳转之后的界面，我们就可以对该规则进行修改。为1，这可不是我们想要的，因为一旦对父view的宽度做拉伸，图片bus就会偏离我们的预期了。
<img src="/images/2015-05-21-autolayoutjin-jie-jiao-cheng-5.jpg">

单击First Item或者Second Item，选中最后一项Reverse First And Second Item，能对两者进行互换位置。
因为图片bus的宽度为45，每张图片的左右间距相等，所以
```
[bus.Center X] = 45/2 + (2 * [Superview.Center X] - 45 * 4)/5
```
即
```
[bus.Center X] = 2/5 * [Superview.Center X] - 27/2
```

因此我们只用将[Multiplier]设为2:5，[Constant]设为-13即可。

同理可计算得到另外三张图片的规则参数，依次设置后好，我们的适配工作就完成了。

## 代码
demo提交到github上了，地址：[LHAutoLayoutDemo](https://github.com/leostc/LHAutoLayoutDemo)

后续的教程demo应该都会放在同一仓库下面
