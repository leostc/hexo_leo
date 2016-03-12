---
layout: post
title: "实现对TextView的文字内容分段展示"
date: 2015-01-19 22:42:56 +0800
comments: true
categories: Android
---
## 关于TextView
本文利用到的最关键的一个函数是Android SDK的TextView中的getOffsetForPosition (float x, float y)，getOffsetForPosition的作用是通过传入一对坐标值，返回离该坐标值最近的字符的位置偏移量，即离该TextView的首字符的偏移量。Android官方文档对该API调用需要14以上，即Android4.0，其实我们可以将这段源代码直接从摘抄出来，直接调用，这样就可以适配到所有的系统

## 具体实现分段翻页
### 1，利用TextView自身特性
当文字内容超出TextView的时候，在页面上展示的内容会被自然的断开。但如果你设置了padding，那么有可能最末一句只能显示一半部分，因此可以对padding进行优化。
通过对每行的高度及TextView的实际内容高度，可以计算出合适的padding。

```android
private int getPaddingBottom(TextView textView) {
    int bottom = 0;
    DisplayMetrics display = mContext.getResources().getDisplayMetrics();
    int height = display.heightPixels;
    int textHeight = height - getStatusBarHeight() - textView.getPaddingBottom() - textView.getPaddingTop();
    double lineFloat = textHeight * 1.0 / textView.getLineHeight();
    int line = (int) lineFloat;
    double lineRemainder = lineFloat - line;
    if (lineRemainder < 0.9) {
        lineRemainder = lineRemainder + 0.1;
    }
    bottom = textView.getPaddingBottom() + (int) (lineRemainder * textView.getLineHeight());
    return bottom;
}
```

### 2，实现翻页功能
利用我们最上面提到的getOffsetForPosition函数，我们传入TextView最末坐标点，获取得到一页内最后一个字符的偏移量，通过该偏移量对TextView内的文字内容进行截取，然后将截取后的文字内容重新填充，就能得到下一页的内容展示。

### 3，实现回翻功能
记录每次后翻的偏移量，回翻的时候，将上一页偏移量加回来即可。
  下图为示意图
<img src="/images/2015-01-19-shi-xian-dui-textviewde-wen-zi-nei-rong-fen-duan-zhan-shi-1.jpg">

## 进一步优化
文字翻页功能比较常见的是应用于小说阅读器等中，可以先将小说内容缓冲部分到内存中，再进行分页展示。

源码已经放在github上面了，地址：[https://github.com/leostc/LHTextView](https://github.com/leostc/LHTextView)