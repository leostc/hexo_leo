---
layout: post
title: "利用Android Accessibility(辅助功能)实现屏幕取词"
date: 2014-12-14 21:12:16 +0800
comments: true
categories: Android
---
Accessibility 辅助功能，旨在辅助生理上有缺陷的人们一个功能，帮助他们更好的使用Android手机。
本文将使用Accessibility中的文字转语音功能来实现在非root手机上获取屏幕上的文字 如何创建一个Accessibility Service

## 如何创建一个Accessibility Service
首先你必须在Manifest中声明Service和Service的属性

```android
<manifest 
...
    <application
        ...
        <service android:name=".MyAccessibilityService"
                 android:label="@string/accessibility_service_label"
                 android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>
            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/serviceconfig" />
        </service>
    </application>
</manifest>
```

@xml/accessibility_service_config的定义:

```android
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
                       android:accessibilityEventTypes="typeAllMask"
                       android:accessibilityFeedbackType="feedbackAllMask"
                       android:accessibilityFlags="flagDefault"
                       android:canRetrieveWindowContent="true"
                       android:description="@string/accessibility_service_description"
                       android:notificationTimeout="0"
/>
```

然后在MyAccessibilityService中继承AccessibilityService
通过onAccessibilityEvent方法就能检测到设备上，所有界面切换时，界面上的文字信息就能监测到了
代码已经放在github上面了，地址：[https://github.com/leostc/AccessibilityDemo](https://github.com/leostc/AccessibilityDemo)