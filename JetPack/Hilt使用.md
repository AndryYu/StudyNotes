## Jetpack - Hilt使用

### 一、将依赖项注入 Android 类

在 Application 类中设置了 Hilt 且有了应用级组件后，Hilt 可以为带有 @AndroidEntryPoint 注解的其他 Android 类提供依赖项：

Hilt 目前支持以下 Android 类：

* Application（通过使用 @HiltAndroidApp）
* ViewModel（通过使用 @HiltViewModel）
* Activity
* Fragment
* View
* Service
* BroadcastReceiver

下面这几个都是用@AndroidEntryPoint进行注解。如果您使用 @AndroidEntryPoint 为某个 Android 类添加注解，则还必须为依赖于该类的 Android 类添加注解。例如，如果您为某个 fragment 添加注解，则还必须为使用该 fragment 的所有 activity 添加注解。

### 二、Hilt模块
Hilt 模块是一个带有`` @Module`` 注解的类。与 Dagger 模块不同的是，您必须使用 ``@InstallIn ``为 Hilt 模块添加注解，以告知 Hilt 每个模块将用在或安装在哪个 Android 类中。

**使用 @Binds 注入接口实例**

@Binds 注解会告知 Hilt 在需要提供接口的实例时要使用哪种实现。

带有注解的函数会向 Hilt 提供以下信息：
* 函数返回类型会告知 Hilt 该函数提供哪个接口的实例。
* 函数参数会告知 Hilt 要提供哪种实现。

```
@Module
@InstallIn(SingletonComponent.class)
public abstract class DataSourceModule {

    @Binds
    @Named("local")
    abstract DataSource provideLocalDataSource(LocalDataSource localDataSource);

    @Binds
    @Named("remote")
    abstract DataSource provideRemoteDataSource(RemoteDataSource remoteDataSource);
}
```
``@Named``注解，这样就可以在需要的时候明确地请求任何一个

**使用 @Provides 注入实例**

Hilt 模块内创建一个函数，并使用 @Provides 为该函数添加注解。

带有注解的函数会向 Hilt 提供以下信息：

* 函数返回类型会告知 Hilt 函数提供哪个类型的实例。
* 函数参数会告知 Hilt 相应类型的依赖项。
* 函数主体会告知 Hilt 如何提供相应类型的实例。每当需要提供该类型的实例时，Hilt 都会执行函数主体。

```
 @Provides
 fun providesWindow(activity: Activity): Window = activity.window
```

**Hilt 中的预定义限定符**

Hilt 提供了一些预定义的限定符。例如，由于您可能需要来自应用或 activity 的 Context 类，因此 Hilt 提供了 ``@ApplicationContext`` 和 ``@ActivityContext`` 限定符。

### 三、为 Android 类生成的组件

Hilt 提供了以下组件：

| Hilt 组件	| 注入器面向的对象 |
|-------|------|
|SingletonComponent	| Application|
|ActivityRetainedComponent	|不适用|
|ViewModelComponent	 | ViewModel |
|ActivityComponent	| Activity |
|FragmentComponent	| Fragment |
|ViewComponent	| View |
|ViewWithFragmentComponent	| 带有 @WithFragmentBindings 注解的 View| 
|ServiceComponent	| Service |



**作用域Scope**

作用域帮助你管理依赖项的生命周期，确保它们在适当的时间被创建和销毁，从而避免内存泄漏和生命周期相关的问题。以下是Hilt中常见的几种作用域：

* @Singleton:

    这是最常用的作用域，表示依赖项在整个应用的生命周期中只创建一次。@Singleton依赖项在整个应用范围内共享一个实例，这非常适合那些初始化开销较大或状态需要在多个组件间共享的服务或数据源。

* @ApplicationScoped:

    这个作用域与@Singleton类似，但它更明确地表明依赖项的生命周期与Application的生命周期绑定。在大多数情况下，@Singleton和@ApplicationScoped的效果相同，但在一些边缘情况下，明确使用@ApplicationScoped可以提高代码的可读性和意图表达。

* @ActivityScoped:

    当依赖项的生命周期应该与Activity的生命周期绑定时，使用此作用域。这意味着每当创建新的Activity实例时，相关的依赖项也将被创建；当Activity销毁时，依赖项也会被销毁。

* @FragmentScoped:

    类似于@ActivityScoped，但针对Fragment。这确保了依赖项与Fragment的生命周期同步，即在Fragment创建时创建依赖项，在Fragment销毁时销毁依赖项。

* @ViewModelScoped:

    这个作用域将依赖项的生命周期与ViewModel的生命周期绑定。由于ViewModel的设计目的是在配置变更时保持状态，因此@ViewModelScoped的依赖项也会在配置变更后继续存在，直到ViewModel被销毁。

* @ForSubComponent:

    这个作用域用于子组件（Subcomponent）。它允许你定义特定于子组件的依赖项，这些依赖项只在子组件的生命周期内有效。这可以让你更灵活地管理局部的依赖关系。



