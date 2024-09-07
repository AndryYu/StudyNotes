## adb常用指令
1. 打印logcat崩溃日志  ``adb logcat -b crash``
2. adb查询关键日志打印  ``adb shell logcat | grep "xxx"``
3. 查看屏幕分辨率 ``adb shell wm size``
4. 查看屏幕像素密度（dpi） ``adb shell wm density``
5. 查看android系统版本 ``adb shell getprop ro.build.version.release``
6. 清除应用缓存 ``adb shell pm clear <包名> ``
7. 截屏
    ```
        截屏到电脑 
        adb exec-out screencap -p > sc.png
        adb版本可以先保存到车机
        adb shell screencap -p /sdcard/sc.png
    ```
8. 清空日志 ```adb logcat -c```
9. 查看应用版本号 
    ```
    adb shell dumpsys package <package_name> | findstr version
    
    adb shell dumpsys package <package_name> | grep version
    ```
10. 向获取焦点的输入框发送文本
    ```
    adb shell input text "hello World"
    ```