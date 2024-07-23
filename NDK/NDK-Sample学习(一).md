##NDK-Sample学习(一)

###一、Hello-jni

####`c_str()`

​	c_str()函数返回一个指向正规C字符串的指针，内容与string字符串相同

​	C++中c_str()主要用法就是为了与C语言兼容，在C语言中没有string类型，故必须通过string的对象的成员函数c_str()把string对象转换	成C中的字符串样式。

​	c_str() 以 char* 形式传回string内含字符串。



####`	jstring NewStringUTF(const char* bytes)`

​	参数是char*    

* `cpp代码，一般是使用env ->NewStringUTF(char *)`

* `c代码，一般是使用指针 (*env)->NewStringUTF(env, char *)`



##### 	扩展

> ```	extern “C"``` - 这是C++的语法，用来告诉编译器使用C的方式来编译这部分代码，因为JNI函数是用C语言语法编写的

> ```	JNIEXPORT``` - 这个宏是由JNI提供的，用来标记这个函数是要被JNI调用的

> ```jstring``` - 这是一个JNI类型，代表一个Java的字符串

> ```	JNICALL``` - 这个宏也是由JNI提供，用来标记这个函数使用JNI调用约定进行编译的

> ```	JNIEXPORT```和```JNICALL```都是JNI用来确保Java和C/C++函数之间正确互调的标记。





###二、Hello-JniCallback

#### 使用android  log

> ​	1.引入```#include<android/log>```

> ​	2.定义tag 

 	```#define LOG_TAG "simple"```或```static const char *LOG_TAG = "hello-jniCallback"```

> ​	3.宏定义调用方法

​	```#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__)```

> ​	ANDORD_LOG_INFO是log的等级

* ANDORD_LOG_INFO          信息

* ANDROID_LOG_WARN       警告
* ANDROID_LOG_ERROR      错误



####`JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved)`

JNI编程中定义的一个原生方法，用于在本地库（通常是.so或.dll文件）被Java虚拟机（JVM）加载时执行必要的初始化操作。

* **`JavaVM *vm`**：指向Java虚拟机实例的指针。开发者可以通过这个指针获取到JNIEnv（JNI环境）实例，进而调用其他JNI函数。保存这个指针对于在本地代码中后续调用JNI函数非常关键
* **`void *reserved`**: 保留参数，目前在JNI规范中并未使用，通常传入`NULL`。

函数的主要职责可能包括但不限于：

- **初始化**: 执行任何必要的库级别初始化工作。
- **注册Native方法**: 使用`RegisterNatives`函数手动注册本地图像中的Native方法，使其能被Java代码调用。
- **版本兼容性检查**: 确认JVM的版本是否符合本地库的要求，如果不匹配，可以选择不加载本地库或采取其他兼容措施。
- **返回JNI版本**: 成功完成初始化后，应返回所支持的JNI版本号，如`JNI_VERSION_1_6`、`JNI_VERSION_1_8`等。这告知JVM本地代码所兼容的JNI接口版本。



####JVM交互方法

在JNI编程中，`JavaVM`接口提供了几个关键方法，用于管理Java虚拟机和本地线程之间的交互，其中最重要的三个是`GetEnv`,`AttachCurrentThread`和`DetachCurrentThread`.

1. **`JavaVM::GetEnv`**

   `GetEnv` 方法用于获取当前线程的JNI环境指针 (`JNIEnv*`)。这个方法允许本地代码访问Java虚拟机的功能。其原型如下：

   ```
   jint GetEnv(JNIEnv** env, jint version);
   ```

   - **参数**:
     - `env`: 输出参数，用于接收JNI环境指针。
     - `version`: 期望的JNI版本号，如 `JNI_VERSION_1_6`。
   - **返回值**:
     - `JNI_OK` 表示成功获取到JNI环境。
     - 其他非零值表示错误，比如线程尚未附着到JVM

2. **`JavaVM::AttachCurrentThread`**

   `AttachCurrentThread` 方法用于将当前的本地线程附着到Java虚拟机，使得这个线程可以调用JNI方法。这是在非Java线程中访问JNI功能所必需的步骤。其原型如下：

   ```
   jint AttachCurrentThread(JNIEnv** penv, void** args);
   ```

   - **参数**:
     - `penv`: 输出参数，用于接收JNI环境指针。
     - `args`: 可选参数，通常传入 `NULL`，但在某些情况下可以传递给VM以提供附加信息。
   - **返回值**: 同样是JNI错误码，`JNI_OK` 表示成功

3. **`JavaVM::DetachCurrentThread`**

   `DetachCurrentThread` 方法用于从Java虚拟机中分离当前线程，表明该线程不再需要执行JNI操作。这是对 `AttachCurrentThread` 的对应清理操作，确保资源正确释放。其原型如下：

   ```
   jint DetachCurrentThread();
   ```

   - **返回值**: 类似，返回JNI错误码，`JNI_OK` 表示成功分离。

**使用场景**

- 当你在非Java线程（即，由C/C++直接创建的线程）中需要调用Java方法或访问Java对象时，你需要先使用 `AttachCurrentThread` 获取JNI环境。

- 在完成JNI操作后，为了防止内存泄漏和保持线程管理的一致性，应当适时调用 `DetachCurrentThread` 来分离线程。

- `GetEnv` 则多用于检查当前线程是否已经附着到JVM，或在已知线程已附着的情况下直接获取JNIEnv指针。

  

####`pthread_mutex_t`

POSIX线程库中的一个类型，代表互斥锁（Mutex），用于保护共享资源免受并发访问冲突的同步工具。

**基本使用流程**

1. **初始化**：在使用互斥锁之前，需要先对其进行初始化。初始化可以通过静态或动态方式进行。

   - **静态初始化**：使用 `PTHREAD_MUTEX_INITIALIZER` 宏进行静态初始化。

     ```c
     pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
     ```

   - **动态初始化**：使用 `pthread_mutex_init` 函数进行动态初始化。

     ```c
     pthread_mutex_t mutex;
     int result = pthread_mutex_init(&mutex, NULL);
     if (result != 0) {
         // 处理错误
     }
     ```

2. **加锁**：当线程需要访问受保护的资源时，需要先调用 `pthread_mutex_lock` 函数获取锁。

   ```
   pthread_mutex_lock(&mutex);
   ```

3. **访问资源**：在锁被成功获取后，线程可以安全地访问共享资源。

4. **解锁**：完成资源访问后，必须调用 `pthread_mutex_unlock` 释放锁，以便其他等待的线程可以继续执行。

   ```
   pthread_mutex_unlock(&mutex);
   ```

5. **销毁锁**（动态初始化时）：如果在程序结束前需要手动销毁互斥锁，可以使用 `pthread_mutex_destroy`。

   ```
   int result = pthread_mutex_destroy(&mutex);
   if (result != 0) {
       // 处理错误
   }
   ```



#### pthread的基本使用

```
pthread_t threadInfo_;
pthread_attr_t threadAttr_;

pthread_attr_init(&threadAttr_);
pthread_attr_setdetachstate(&threadAttr_, PTHREAD_CREATE_DETACHED);

pthread_mutex_init(&g_ctx.lock, NULL);
int result = pthread_create(&threadInfo_, &threadAttr_, UpdateTicks, &g_ctx);
pthread_attr_destroy(&threadAttr_);
```

使用pthread需要引入头文件`include<pthread.h>`下面是代码的详细解析：

1. **线程标识符和属性初始化**:

   - `pthread_t threadInfo_;`: 定义了一个`pthread_t`类型的变量`threadInfo_`，用于存储新创建的线程标识符。
   - `pthread_attr_t threadAttr_;`: 定义了一个`pthread_attr_t`类型的变量`threadAttr_`，用于存储线程属性。

2. **线程属性设置**:

   ```
   pthread_attr_init(&threadAttr_);
   pthread_attr_setdetachstate(&threadAttr_, PTHREAD_CREATE_DETACHED);
   ```

   - `pthread_attr_init(&threadAttr_)`初始化线程属性对象。虽然在某些情况下可以省略此步骤直接设置属性，但初始化是个好习惯，确保所有属性处于已知状态。
   - `pthread_attr_setdetachstate(&threadAttr_, PTHREAD_CREATE_DETACHED)`设置线程的分离状态为`PTHREAD_CREATE_DETACHED`。这意味着当线程运行结束时，操作系统会自动释放该线程占用的资源，无需调用`pthread_join`函数等待线程结束。如果设置为`PTHREAD_CREATE_JOINABLE`，则主线程需要通过`pthread_join`等待线程结束并回收资源。

3. **互斥锁初始化**:

   ```
   pthread_mutex_init(&g_ctx.lock, NULL);
   ```

   这行代码初始化了一个互斥锁（mutex），`g_ctx.lock`假设是全局或某个结构体中的一个成员，用于保护共享资源免受并发访问的影响。`NULL`作为第二个参数，表示使用默认的锁属性。

4. **线程创建**:

   ```
   int result = pthread_create(&threadInfo_, &threadAttr_, UpdateTicks, &g_ctx);
   ```

   - ```
     pthread_create
     ```

     函数创建一个新的线程，其参数分别是：

     - `&threadInfo_`: 用于接收新线程的标识符。
     - `&threadAttr_`: 传递线程属性指针，这里使用了之前设置的属性，包括分离状态。
     - `UpdateTicks`: 线程的起始函数，当线程启动时会执行这个函数。
     - `&g_ctx`: 传递给线程函数的参数，这里是`g_ctx`结构体的地址，可以在`UpdateTicks`函数中访问和修改这个结构体的内容。

5. **线程属性清理**:

   ```
   pthread_attr_destroy(&threadAttr_);
   ```

   线程创建完成后，调用`pthread_attr_destroy`释放线程属性对象所占用的资源。这一步是良好的清理工作，特别是当线程属性是在运行时动态设置时尤为重要。



#### JNI调用Java方法

**1. 获取JNIEnv指针**

在JNI函数中，JNIEnv指针是必不可少的，它提供了调用Java方法、访问Java对象和数组等操作的接口。如果你在一个JNI函数（如`JNI_OnLoad`, `Java_YourClass_MethodName`）内部，JNIEnv通常作为参数提供。如果在其他本地线程中，你可能需要先通过`AttachCurrentThread`或`GetEnv`获取JNIEnv。

**2. 获取Java类和方法ID**

在调用Java方法之前，需要获取该方法的类以及方法的ID。这可以通过`JNIEnv::FindClass`和`JNIEnv::GetMethodID`或`JNIEnv::GetStaticMethodID`完成。

```
jclass clazz = (*env)->FindClass(env, "your/package/ClassName");
if (clazz == NULL) {
    // 错误处理
}

jmethodID methodId;
if (isStatic) {
    methodId = (*env)->GetStaticMethodID(env, clazz, "methodName", "(Ljava/lang/String;)V"); // 示例为静态方法签名
} else {
    methodId = (*env)->GetMethodID(env, clazz, "methodName", "(Ljava/lang/String;)V"); // 示例为非静态方法签名
}
if (methodId == NULL) {
    // 错误处理
}
```

**3. 调用Java方法**

有了方法ID后，就可以调用Java方法了。调用方式根据方法的类型（静态或非静态）和返回类型有所不同。下面是一个调用非静态方法的例子，该方法接受一个字符串参数并返回一个整数：

```
jstring jstrArg = (*env)->NewStringUTF(env, "Hello from JNI");
jint result = (*env)->CallIntMethod(env, jobject_instance, methodId, jstrArg); // jobject_instance是调用非静态方法时需要的对象实例
if ((*env)->ExceptionCheck(env)) {
    // 异常处理
}
// 使用result...
```

如果是静态方法，则不需要传递对象实例：

```
jint result = (*env)->CallStaticIntMethod(env, clazz, methodId, jstrArg);
```

**4. 清理**

记得在完成操作后，如果创建了局部引用，要适当释放它们，以避免内存泄露。对于上述示例中的`jstrArg`，如果它是局部引用，一般情况下JNI会自动管理其生命周期，但在复杂场景下可能需要手动调用`(*env)->DeleteLocalRef(env, jstrArg);`。

**拓展**

通过jobject获取jclass

```
jclass objectClass = (*env)->GetObjectClass(env, javaObject);
```



#### JNI反射获取java对象

**1. 导入必要的头文件**

确保你的C/C++代码中包含了必要的JNI头文件，通常是`jni.h`。

**2. 获取JNIEnv指针**

在本地方法中，JNIEnv指针会作为第一个参数传递。如果不在本地方法中，则需要通过其他方式获取，如`AttachCurrentThread`。

**3. 查找类**

使用`FindClass`方法根据全限定类名获取`jclass`引用。

```
jclass clazz = (*env)->FindClass(env, "java/lang/Class");
if (clazz == NULL) {
    // 处理错误，如找不到类
}
```

**4. 获取`Class`类的`forName`方法ID**

为了通过反射获取其他类的对象，我们通常需要调用`Class.forName`方法。首先，获取`Class`类的`forName`方法的ID。

```
jmethodID forNameMethod = (*env)->GetStaticMethodID(env, clazz, "forName", "(Ljava/lang/String;)Ljava/lang/Class;");
if (forNameMethod == NULL) {
    // 处理错误，找不到方法
}
```

**5. 调用`Class.forName`方法**

使用之前获取的方法ID调用`forName`方法，传入目标类的全限定名作为参数，以获取目标类的`Class`对象。

```
jstring classNameStr = (*env)->NewStringUTF(env, "your/full/qualified/ClassName");
jobject targetClassObj = (*env)->CallStaticObjectMethod(env, clazz, forNameMethod, classNameStr);
if ((*env)->ExceptionCheck(env)) {
    // 处理异常
}
```

**6. 获取构造函数或方法**

假设我们要通过反射创建目标类的实例，我们需要找到合适的构造函数。

```
jmethodID constructorId = (*env)->GetMethodID(env, targetClassObj, "<init>", "()V"); // 无参构造函数示例
if (constructorId == NULL) {
    // 处理错误，找不到构造函数
}
```

**7. 创建对象实例**

使用找到的构造函数创建目标类的新实例。

```
jobject newObj = (*env)->NewObject(env, targetClassObj, constructorId);
if (newObj == NULL) {
    // 处理错误，如构造失败
}
```

**8. 调用对象的方法或访问字段**

现在，`newObj`就是目标类的一个实例，你可以通过类似的方式获取方法ID并调用方法，或访问字段等。



####`NewGlobalRef`

在JNI编程中，当你需要从本地代码（如C或C++）长时间地访问Java对象时，就需要使用全局引用。这是因为默认情况下，JNI函数传递给本地代码的Java对象引用（称为局部引用local reference）仅在当前JNI函数的调用期间有效，超出这个范围后，局部引用就会自动被垃圾回收，可能导致未定义行为或程序崩溃。

**功能描述**

`NewGlobalRef`函数的原型如下：

```
jobject NewGlobalRef(JNIEnv *env, jobject obj);
```

- **参数**:
  - `env`: 一个指向JNIEnv的指针，用于访问JNI函数。
  - `obj`: 一个本地引用，指向你想要创建全局引用的Java对象。
- **返回值**:
  - 返回一个指向相同Java对象的全局引用。如果`obj`为`NULL`，则返回`NULL`。

**为什么使用**

- **生命周期管理**：全局引用的生命周期与创建它的本地线程相同，直到显式调用`DeleteGlobalRef`或`DeleteWeakGlobalRef`释放，否则它不会被垃圾回收。这使得全局引用非常适合在不同JNI调用之间持久化对象引用。
- **跨线程访问**：全局引用可以在多个线程间共享，而局部引用通常限制在创建它的线程中使用。因此，如果你需要在多个线程中访问同一个Java对象，使用全局引用是一个解决方案。
- **长期存储**：如果你的本地代码需要长时间存储对Java对象的引用，例如，将其保存在本地数据结构中供后续使用，使用全局引用是必要的。

**示例**

```
// 假设localObj是一个有效的局部引用
jobject globalObj = (*env)->NewGlobalRef(env, localObj);
// 现在globalObj可以跨JNI调用或线程安全地使用
// ...
// 使用完毕后，记得释放全局引用
(*env)->DeleteGlobalRef(env, globalObj);
```



#### JNI三种引用

**Global引用、Local引用以及weak引用**

* **Local引用**

  JNI中使用`jobject`,`jclass`以及`jstring`等来标志一个Java对象。局部引用可以直接使用`NewLocalRef`创建，虽然局部引用可以在跳出作用域后被回收，但还是希望在不使用的时候调用`DeleteLocalRef`来手动回收掉。

* **Global引用**

  多个地方需要使用的时候就会创建一个全局的引用（`NewGlobalRef`方法创建），显示调用`DeleteGlobalRef`的时候才会失效，不然会一直存在内存中。

* **weak引用**

  弱引用可以使用`NewWeakGlobalRef`的方式申明，区别在于弱引用在内存不足或者紧张的时候会自动回收掉，可能会出现短暂的内存泄露，但不会出现内存溢出的情况。建议不需要使用的时候，手动调用DeleteWeakGlobalRef释放引用。

  
