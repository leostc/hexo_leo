---
layout: post
title: "实现类google app的Material Design切换UIViewController"
date: 2015-04-06 23:13:10 +0800
comments: true
categories: iOS
---
## 前言
重新回到iOS开发的大部队中，以后博客的内容应该会以iOS居多。 
## Material Design 
质感设计，Google2014年发布的全平台设计语言规范。公司的设计师在参考了Google的iOS App的设计，在我们的新iOS产品中借鉴了该交互方式。
下面是我录的Google App的几个页面切换的效果，点击按钮后，按钮放大到全屏再消失，再进入下一个页面。
<iframe height=498 width=510 src="http://player.youku.com/embed/XOTI3NjgyMjI0" frameborder=0 allowfullscreen></iframe>

## 前人的轮子
于是我先从github上找寻了下相关的代码，发现有人实现了两个UIView之间的切换的动画效果：
[https://github.com/moqod/ios-material-design](https://github.com/moqod/ios-material-design)
但是，为了在工程比较优雅的实现，最好改为两个UIViewController之间的切换。

## 改造
通过Category，为UIViewController添加了4个切换的方法：

```objectivec
/**
 *  Present View Controller with Material Design
 *
 *  @param viewController presented View Controller
 *  @param view           view tapped, to calculate the point that animation starts
 *  @param color          animation color, if nil, will use viewController's background color
 *  @param animated       animated or not
 *  @param completion     completion block
 */
- (void)presentLHViewController:(UIViewController *)viewController
                        tapView:(UIView *)view
                          color:(UIColor *)color
                       animated:(BOOL)animated
                     completion:(void (^)(void))completion;

/**
 *  Dismiss View Controller with Material Design
 *
 *  @param view       view tapped, if nil, will use the presenting view controller's point that animation starts
 *  @param color      animation color, if nil, will use viewController's background color
 *  @param animated   animated or not
 *  @param completion completion block
 */
- (void)dismissLHViewControllerWithTapView:(UIView *)view
                                     color:(UIColor*)color
                                  animated:(BOOL)animated
                                completion:(void (^)(void))completion;

/**
 *  Push View Controller in UINavigationController with Material Design
 *
 *  @param viewController presented View Controller
 *  @param view           view tapped, to calculate the point that animation starts
 *  @param color          animation color, if nil, will use viewController's background color
 *  @param animated       animated or not
 *  @param completion     completion block
 */
- (void)pushLHViewController:(UIViewController *)viewController
                     tapView:(UIView *)view
                       color:(UIColor*)color
                    animated:(BOOL)animated
                  completion:(void (^)(void))completion;

/**
 *  Pop View Controller in UINavigationController with Material Design
 *
 *  @param view       view tapped, if nil, will use the presenting view controller's point that animation starts
 *  @param color      animation color, if nil, will use viewController's background color
 *  @param animated   animated or not
 *  @param completion completion block
 */
- (void)popLHViewControllerWithTapView:(UIView *)view
                                 color:(UIColor*)color
                              animated:(BOOL)animated
                            completion:(void (^)(void))completion;
```

进一步实现:
1，获取到点击的按钮的中心点
2，根据点击坐标及颜色创建一个CAShapeLayer
3，创建由小到大或者大到小的CABasicAnimation，添加在CAShapeLayer，并执行
4，将CAShapeLayer添加到view上，并在合适的时机添加或者移除ViewController
另外，考虑到dimiss或pop的时候，动画终结点可能会回到前一个页面的点击点，因此用runtime在category中添加了一个变量用来记录每一次present或push的动画起始点。

## 源码
按照惯例，源码就不贴了，直接放github上，欢迎加星及修改哈[LHMaterialDesign](https://github.com/leostc/LHMaterialDesign)