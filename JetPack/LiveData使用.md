## LiveData使用

liveData有两种主要的观察者类型：``Observer``和``ObserverForever``。

**Observer**

``Observer``是一个普通的观察者，它会在数据发生变更时被调用。``LiveData``会自动管理``Observer``的注册和注销，确保在数据变更时只通知活跃的观察者（即处于活跃状态的UI组件，如Activity或Fragment）

**ObserverForever**

``ObserverForever``是一个特殊的观察者，它在注册后一直接收到数据变更的通知，即使观察者的生命周期状态发生变化。这意味着即使UI组件（如Activity或Fragment）不再活跃（例如被暂停或停止），``ObserverForever``仍然会接收到通知。


``LiveData``会根据观察者的生命周期状态来决定是否发送数据变更通知。当``Fragment``处于以下状态时，``LiveData``不会向观察者发送数据变更通知：

1. onStop()：当Fragment进入onStop()状态时，表示它已经不可见，此时LiveData不会发送数据变更通知。
2. onDestroyView()：在Fragment的视图被销毁后，LiveData也不会发送数据变更通知。
3. onDestroy()：当Fragment实例被销毁后，LiveData同样不会发送数据变更通知。
