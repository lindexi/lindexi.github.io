# win10 uwp 打开文件管理器选择文件

本文：让文件管理器选择文件，不是从文件管理器获得文件。

假如已经获得一些文件，那么如何从文件管理器选择这些文件？

使用方法很简单。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


从网上拿图来说

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017427204051.jpg)

打开文件夹自动选择所有文件

首先需要获得文件夹，因为这个呆磨，实际不会这样做


```csharp
           FolderPicker p = new FolderPicker();
        p.FileTypeFilter.Add(".txt");
        StorageFolder folder = await p.PickSingleFolderAsync();
```

我要用户从文件管理打开，选择所有文件

需要使用 Folder​Launcher​Options ，他可以使用 ItemsToSelect 让管理器选择文件

当然文件夹也是可以

但是 ItemsToSelect 是只读，不可以在构造使用，于是写一个变量

 
```csharp
     var t = new FolderLauncherOptions();
```
 
获得文件夹的所有文件，然后打开

 
```csharp
     await Launcher.LaunchFolderAsync(folder, t);
```

参见：https://codedocu.com/Details?d=1542&a=9&f=181&l=1&v=d&t=UWP:-How-to-Open-the-file-Explorer-from-a-Windows-app

https://docs.microsoft.com/en-us/uwp/api/Windows.System.Launcher#Windows_System_Launcher_LaunchFolderAsync_Windows_Storage_IStorageFolder_Windows_System_FolderLauncherOptions_

本文同时发在九幽[win10 uwp 打开文件管理器选择文件 | Win10.CM](http://www.win10.cm/?p=1222)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
