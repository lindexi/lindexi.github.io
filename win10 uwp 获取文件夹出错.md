# win10 uwp 获取文件夹出错

获取文件夹时出现


```csharp
    Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED))
```


<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


<div id="toc"></div>
<!-- csdn -->

代码


```csharp
    FolderPicker fp = new FolderPicker();
fp.SuggestedStartLocation = PickerLocationId.ComputerFolder;
fp.FileTypeFilter.Add("*");
var f = await fp.PickSingleFolderAsync();
```

上面代码有3个地方错误。

1. 需要知道的，在 FileTypeFilter 添加的是具体的类型，不能使用 "`*`" 

 即使设置为特殊类型，但是文件夹没有这类型的文件也是可以。

 这属性对结果好像不会有r用，垃圾ms



2. 设置了 `PickerLocationId.ComputerFolder` 

3. 代码写在 Load 函数。

 需要知道 上面的错误是 写在 Load 函数错误。

 修改，把上面代码写在其他函数调用，不在 Load 调用。

参见：http://stackoverflow.com/a/42969965/6116637

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792391647.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  