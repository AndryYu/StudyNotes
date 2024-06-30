# Gradle 管理依赖

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