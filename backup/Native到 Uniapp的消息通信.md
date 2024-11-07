
# Native到 Uniapp的消息通信

主要思路：
1. FFI： 通过ABI接口把数据传输到 Uniapp
2. 直接通过发送消息的接口发送数据

其中1在安卓的接口中是确实支持的，在IOS中暂时没看到相关接口
Android: [DCUniMPJSCallback.invoke(data)](https://nativesupport.dcloud.net.cn/UniMPDocs/API/android-v2.html#dcunimpjscallback-invoke)

考虑到统一，可以考虑用方案2，通过发送event

> 宿主主动触发事件到正在运行的小程序。注意：需要已有小程序在运行才可成功

[Android](https://nativesupport.dcloud.net.cn/UniMPDocs/API/android-v2.html#iunimp-sendunimpevent)
```java
/// @param event 事件名称: String
/// @param data  数据: String
/// @return :bool
IUniMP.sendUniMPEvent(event, data)
```

[IOS](https://nativesupport.dcloud.net.cn/UniMPDocs/API/ios.html#sendunimpevent)
```objective-c
/// @param event 事件名称
/// @param data 数据：NSString 或 NSDictionary 类型
(void)sendUniMPEvent:(NSString *)event data:(id __nullable)data;
```

Uniapp
```js
//uni小程序JS代码 监听宿主触发给小程序的事件
uni.onNativeEventReceive(function(event, data){
    console.log("onNativeEventReceive----="+event);
});
```
最好的方案是在uniapp收到消息后，能把消息传递到新的线程中做处理，不阻塞监听消息的函数
但是不确定uniapp的jsruntime中是否支持多线程

