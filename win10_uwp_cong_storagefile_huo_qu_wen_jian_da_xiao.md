# win10 uwp 从StorageFile获取文件大小

本文主要：获取文件大小

        private async Task<ulong> FileSize(Windows.Storage.StorageFile file)
        {
            var size = await file.GetBasicPropertiesAsync();
            return size.Size;
        }//32ddd4227a66713e1329214424c4be9b
        
在群里看到有大神问我就写出，虽然少，在没看到他们说之前没想到，九幽开发者：53078485

参见：http://stackoverflow.com/questions/14168439/how-to-get-file-size-in-winrt

##获取用户最近使用文件

一般我们有一个文件夹或文件不在我们应用目录，需要用户Pick获得权限，那么我们会让用户每次都Pick，这样是不行的。
我们有什么方法让UWP 记住用户选择文件或文件夹，或UWP不让用户每次选择文件

其实有两个方法

- MostRecentlyUsedList

- FutureAccessList 

第一个很简单，用户最近使用文件或文件夹，这个只能保存25，我就在这里坑，他会自动删除，找了[https://msdn.microsoft.com/zh-cn/windows/uwp/files/how-to-track-recently-used-files-and-folders](https://msdn.microsoft.com/zh-cn/windows/uwp/files/how-to-track-recently-used-files-and-folders)，其实我们可以使用FutureAccessList ，这个可以使用1k个，但是为什么只有1k，好少，垃圾wr，要就给无限


##win10 uwp 读取文本ASCII错误

我使用NotePad记事本保存文件，格式ASCII，用微软示例打开文件方式读取，出现错误

在多字节的目标代码页中，没有此 Unicode 字符可以映射到的字符

英文 No mapping for the Unicode character exists in the target multi-byte code page

这个问题看来很简单，不就是编码错误，可以我就弄了一晚上

我先换个说法，让大家容易搜索到

 - UWP 读ASCII错误

 - UWP read ASCII
 - UWP GBK
 - UWP 读取记事本

其实不知道垃圾wr怎么想，现在没法读ASCII，官方给的，直接错

用了nos大神的代码http://blog.csdn.net/nomasp/article/details/50310357，也是报错

用了我csdn博客置顶代码，就直接乱码 所有中文代为 "?" 

查了一下WPF使用默认可以读，也就是我们保存时GBK，查询到Encoding没有GBK，没有默认

于是我就在网上找，很久没找到，但是找到http://www.cnblogs.com/yffswyf/p/4826207.html，写到一半我就不想写，好难

在网上看到

参考：http://stackoverflow.com/questions/35296213/read-unicode-string-from-text-file-in-uwp-app/38299563#38299563

http://www.cnblogs.com/loyieking/p/5617508.html



        
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。        