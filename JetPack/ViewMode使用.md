## ViewMode 使用
ViewModel 类是一种业务逻辑或屏幕级状态容器。它用于将状态公开给界面，以及封装相关的业务逻辑。 它的主要优点是，它可以缓存状态，并可在配置更改后持久保留相应状态。这意味着在 activity 之间导航时或进行配置更改后（例如旋转屏幕时），界面将无需重新提取数据。

ViewMode类的主要优势：
* 允许持久保留界面状态
* 可以提供对业务逻辑的访问权限

实例化 ViewModel 时，您会向其传递实现 ``ViewModelStoreOwner``接口的对象。它可能是 Navigation 目的地、Navigation 图表、activity、fragment 或实现接口的任何其他类型。然后，ViewModel 的作用域将限定为 ``ViewModelStoreOwner`` 的 ``Lifecycle``。它会一直保留在内存中，直到其 ``ViewModelStoreOwner``永久消失。

有一系列类是 ``ViewModelStoreOwner`` 接口的直接或间接子类。直接子类为 ComponentActivity、Fragment 和 NavBackStackEntry。

当``ViewModelStoreOwner`` 在 ViewModel 的生命周期内销毁 ViewModel 时，ViewModel 会调用``onCleared ``方法。这样，您就可以清理遵循 ViewModel 生命周期的任何工作或依赖项。