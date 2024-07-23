# 版本管理和APT

###一、Gradle 管理依赖

#### 1.ext

Gradle 提供了ext关键字去定义自身所需要的扩展属性，通过ext关键字对项目中的依赖进行全局配置

1. 直接在project的build.gradle中编写ext

   ```
   一、项目根目录 build.gradle
   
   ext {
       //版本号
       versions = [
               compileSdkVersion         : 31,
               buildToolsVersion         : "30.0.2",
               rxjavaVersion             : "2.2.16",
               rxAndroidVersion          : "2.1.1",
       ]
   
       //依赖库
       libraries =[
               rxjava                   : "io.reactivex.rxjava2:rxjava:$versions.rxjavaVersion",
               rxAndroid                : "io.reactivex.rxjava2:rxandroid:$versions.rxAndroidVersion",
       ]
   }
   
   
   二、module 中build.gradle
   android {'
       compileSdk versions.compileSdkVersion
   
       ...
   }
   
   dependencies {
       implementation libraries.rxjava
       implementation libraries.rxAndroid
   }
   ```

   

2. 在单独的gradle脚本中编写ext

```
一、单独的 config.gradle

ext {
    //版本号
    versions = [
            compileSdkVersion         : 31,
            buildToolsVersion         : "30.0.2",
            rxjavaVersion             : "2.2.16",
            rxAndroidVersion          : "2.1.1",
    ]

    //依赖库
    libraries =[
            rxjava                   : "io.reactivex.rxjava2:rxjava:$versions.rxjavaVersion",
            rxAndroid                : "io.reactivex.rxjava2:rxandroid:$versions.rxAndroidVersion",
    ]
}

二、项目根目录 build.gradle
apply from: '/config.gradle'

三、module中使用一样
```

* 优点：统一管理依赖，版本升级只需要修改ext属性

* 缺点：在module使用中，不支持代码提醒和点击跳转

#### 2. kotlin + buildSrc

不推荐



####3.Version Catalogs 管理 （gradle高版本）

gradle 目录下 libs.versions.toml

toml文件4个主要部分组成

1. [versions] 依赖项版本号
2. [libraries] 依赖库别名
3. [bundles] 依赖包
4. [plugins] 声明插件



### 二、APT(Annotation Processing Tool)

只在编译的时候使用，不会打包到apk

* **android-apt**

  在Java时代，代码编译期间我们可以通过注解的方式去生成代码。最开始的时候我们使用android-apt(个人开发者开发的)，只支持javac的方式。

  需要引入插件

  ```groovy
     classpath'com.neenbedankt.gradle.plugins:android-apt:1.8'
  ```

* **annotation Processor**

  谷歌在Android Gradle 插件 2.2 出了个APT代替android-apt，就是**annotationProcessor**；可以直接使用不需要引入插件同时支持Javac和Jack的方式编译，其他功能基本与android-apt相同。

  ###### APT的流程上分为

  代码扫描找到注解 -> 根据注解处理操作 -> 生成Java代码

* **kapt**

  顾名思义就是在使用Kotlin进行开发时，使用kapt，annotationProcessor只支持Java
  使用的时候，需要添加插件

  ```groovy
    apply plugin: 'kotlin-kapt'
  ```

  ###### kapt流程上分为

  kotlin源码 -> 生成Java Stubs -> APT(上面那个扫描生成) -> 生成源码

  然后当你的项目用了ButterKnife greendao ARouter  Glide Hilt Dagger等框架，然后又用上了kapt，会发现编译速度变得极其缓慢，究其原因就是在生成Java Stubs 的时候耗费了大量时间

* **ksp**

  就是谷歌新出的为了解决kapt缓慢的方案

  ###### KSP流程上分为

  kotlin源码 -> KSP -> 生成源码
  少了一个步骤毫无疑问会快了官方说 “速度提高多达2倍”