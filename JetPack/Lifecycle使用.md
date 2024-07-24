## Lifecycle使用

``Lifecycle``是一个类，用于存储有关组件（如activity或fragment）的生命周期状态的信息，并允许其他对象观测此状态。``LifecycleRegistry``是Lifecycle的实现类

``Lifecycle``使用两种主要枚举跟踪其关联组件的生命周期状态：
* 活动（Events）
从框架和``Lifecycle``类分派的生命周期事件，这些事件映射到activity和fragment中的回调事件
* 状态 (States)
 ``Lifecycle``对象所跟踪的组件的当前状态

    ![alt text](.\pic\image.png)


#### LifecycleOwner
``LifecycleOwner`` 是只包含一个方法的接口，指明类具有 Lifecycle。它包含一个方法（即 ``getLifecycle()``），该方法必须由类实现