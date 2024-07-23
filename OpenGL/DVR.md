## DVR

###一、EGL

​		OpenGl是一套跨平台的接口，它与各个平台本地窗口系统之间的交互，是借助于一个中间控制层，这个中间控制层就是EGL。 EGL也有自己的一套标准API，由各个平台的系统来完成其具体实现。

​		EGL是OpenGL和本地窗口体系进行联系的桥梁，负责管理OpenGL的运行状态、渲染图像到本地窗口或缓冲区等功能

![image-20240723075255737](.\pic\image-20240723075255737.png)

连接实现：

1. 创建绘制内容的目标EGLDisplay(一个封装系统屏幕的数据类型)
2. 初始化显示设备：初始化与 EGLDisplay 之间的连接：eglInitialize()
3. 进行配置选项，使用eglChooseConfig()方法
4. 创建OpenGl的上下文环境 EGLContext 实例：eglCreateContext() eglCreateContext中的第三个参数可以传入一个EGLContext类型的变量，该变量的意义是可以与正在创建的上下文环境共享OpenGl资源，包括纹理ID,FrameBuffer以及其他Buffer资源。
5. 通过EGL库提供eglCreateWindowSurface可以创建一个实际可以显示的surface
6. OpenGl指令必须要在其上下文环境中才能执行，通过 eglMakeCurrent()方法来绑定该线程的显示设备及上下文。
7. 调用了eglSwapBuffers指令，把前台的FrameBuffers和后台的FrameBuffer进行交换
8. 销毁资源。必须在当前线程中进行，不然会报错滴。首先我们销毁显示设备（EGLSurface）,然后销毁上下文（EGLContext）,停止并释放线程，最后终止与EGLDisplay之间的链接



### 二、OpenGL

![image-20240723075906763](.\pic\image-20240723075906763.png)



### 三、离屏渲染

GPU屏幕渲染有以下两种方式：

* On-Screen Rendering：当前屏幕渲染，指的是GPU的渲染是在当前用于显示的屏幕缓冲区中进行
* Off-Screen Rendering：离屏渲染，指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区（FBO）进行渲染。

![image-20240723080418503](.\pic\image-20240723080418503.png)

![image-20240723080532727](D:\Workspace\GitHub\StudyNotes\OpenGL\pic\image-20240723080532727.png)



### 四、SurfaceFlinger

​		SurfaceFlinger 是一个独立的Service， 它接收所有Window的Surface作为输入，根据ZOrder， 透明度，大小，位置等参数，计算出每个Surface在最终合成图像中的位置，然后交由HWComposer或OpenGL生成最终的显示Buffer, 然后显示到特定的显示设备上。

   	SurfaceFlinger 主要有以下几个职责：分配图形缓冲区、合成图形缓冲区、管理 VSYNC 事件

![image-20240723080658112](.\pic\image-20240723080658112.png)



​	在 VSYNC 事件中，屏幕开始显示帧 N，而 SurfaceFlinger 开始为帧 N+1 合成，应用处理等待的输入并绘制生成帧 N+2。

![image-20240723080808544](.\pic\image-20240723080808544.png)

HWC描述上述信息的流程是这样的：

1. SurfaceFlinger向HWC提供所有Layer的完整列表，让HWC根据其硬件能力，决定如何处理这些Layer。
2. HWC会为每个Layer标注合成方式，是通过GPU还是通过HWC合成。
3. SurfaceFlinger负责先把所有注明GPU合成的Layer合成到一个输出Buffer，然后把这个输出Buffer和其他Layer（注明HWC合成的Layer）一起交给HWC，让HWC完成剩余Layer的合成和显示。



### 五、Camera Provider

**Camera Framework**：以jar包形式运行在app进程中，实现了Camera Api v2标准接口，通过Camera AIDL跨进程接口将请求发送至Camera Service

**Camera Service**：独立进程。通过HIDL跨进程接口将请求再次下发到Camera Provider

**amera Provider**：独立进程。内部加载了Camera HAL Module，该Module由OEM/ODM实现，遵循谷歌制定的标准Camera HAL3接口

**Camera Driver**: LINUX为视频采集设备制定了标准的V4L2接口，并在内核中实现了其基础框架V4L2 Core，视频采集设备驱动厂商按照V4L2 Core的要求开发自己的驱动程序

![image-20240723081049136](.\pic\image-20240723081049136.png)

































