---
layout: post
title:  "Android GradleScript配置"
date:   2016-06-06
categories: [android]
---

## 编译时动态设置 Android resValue

语法：`resValue "string", "AppName", "app1"` 编译后会在res/string.xml中增加一个名为AppName，值为app1的资源。
位置：buildTypes、productFlavors、defaultConfig等

## 在编译时动态设置 Android BuildConfig

语法：`buildConfigField "String", "ENDPOINT", "\"http://example.com\""`编译后会在BuildConfig中新增一个ENDPOINT字段。
位置：buildTypes、productFlavors、defaultConfig等

**通常可以做一些全局性的配置，比如一般我们在测试环境的时候会有一些日志需要输出，正式环境的时候为保证性能就需要关掉日志输出，这时就可以在buildTypes中配置**

## 动态设置Android Manifest

在AndroidManifest中定义一个变量，在build.gradle中动态的替换掉，十分方便，语法也十分简单。对比上面的功能，我们需要动态替换友盟的appkey，需要在AndroidManifest中定义一个变量

```xml
<meta-data
         android:name="UMENG_APPKEY"
         android:value="${umeng_app_key}"/>
```

接着，我们在build.gradle文件中根据不同的环境，生成不同appkey的apk。

```xml
buildTypes {
        debug {
         manifestPlaceholders = [umeng_app_key: "你替代的内容"]
        }
        release {
     　　manifestPlaceholders = [umeng_app_key: "你替代的内容"]
        }
        develop {
    　　 manifestPlaceholders = [umeng_app_key: "你替代的内容"]
        }
    }
```

**注意：这里的“你替代的内容”，不能为特殊关键词，比如：TRUE，否则在Java代码中获取不到meta-data中的值。**

## 读取properties文件

Android开发中我们会经常配置一些跨模块的全局变量：版本号、一些key值等。这时可以将这些变量定义在properties里面，比如：

```
appVersionCode = 515
appVersionName = 5.1.5

## TODO: Supply YouTube API key and sender ID for your project
youtube_api_key = UNDEFINED
```

使用

```
defaultConfig {
        versionCode = Integer.valueOf("" + appVersionCode).intValue()
        versionName "${appVersionName}"
        buildConfigField("String", "YOUTUBE_API_KEY", "\"${youtube_api_key}\"")
        ...
    }
```

## 参考

- [使用 gradle 在编译时动态设置 Android resValue / BuildConfig / Manifes中<meta-data>变量的值](http://blog.csdn.net/xx326664162/article/details/49247815)