# win10 uwp 从StorageFile获取文件大小

本文主要：获取文件大小
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

        private async Task<ulong> FileSize(Windows.Storage.StorageFile file)
        {
            var size = await file.GetBasicPropertiesAsync();
            return size.Size;
        }//32ddd4227a66713e1329214424c4be9b
        
在群里看到有大神问我就写出，虽然少，在没看到他们说之前没想到，九幽开发者：53078485

参见：http://stackoverflow.com/questions/14168439/how-to-get-file-size-in-winrt

## 获取用户最近使用文件

一般我们有一个文件夹或文件不在我们应用目录，需要用户Pick获得权限，那么我们会让用户每次都Pick，这样是不行的。
我们有什么方法让UWP 记住用户选择文件或文件夹，或UWP不让用户每次选择文件

其实有两个方法

- MostRecentlyUsedList

- FutureAccessList 

第一个很简单，用户最近使用文件或文件夹，这个只能保存25，我就在这里坑，他会自动删除，找了[https://msdn.microsoft.com/zh-cn/windows/uwp/files/how-to-track-recently-used-files-and-folders](https://msdn.microsoft.com/zh-cn/windows/uwp/files/how-to-track-recently-used-files-and-folders)，其实我们可以使用FutureAccessList ，这个可以使用1k个，但是为什么只有1k，好少，垃圾wr，要就给无限

参见：http://lindexi.oschina.io/lindexi/post/win10-uwp-%E4%BF%9D%E5%AD%98%E7%94%A8%E6%88%B7%E9%80%89%E6%8B%A9%E6%96%87%E4%BB%B6%E5%A4%B9/


        
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。        

