# win10 uwp 非ui线程访问 ui 

大家都知道，不可以在 其他线程访问 UI 线程，访问 UI 线程包括给 依赖属性设置值、读取依赖属性、调用方法（如果方法里面修改了依赖属性）等。一旦访问UI线程，那么就会报错，为了解决这个问题，需要使用本文的方法，让后台线程访问 UI 线程。
<!--more-->
<!-- csdn -->

本文提供三个方法可以让其他线程访问 UI 线程

第一个方法是比较不推荐使用的，可能出现 [win10 uwp Window.Current.Dispatcher中Current为null](http://lindexi.oschina.io/lindexi//post/win10-uwp-Window.Current.Dispatcher%E4%B8%ADCurrent%E4%B8%BAnull/)

```csharp

```

为何不设置为 High ，参见
[CoreDispatcherPriority](https://docs.microsoft.com/en-us/uwp/api/Windows.UI.Core.CoreDispatcherPriority)


https://stackoverflow.com/questions/7401538/simple-example-of-dispatcherhelper

https://stackoverflow.com/questions/38149767/uwp-update-ui-from-task