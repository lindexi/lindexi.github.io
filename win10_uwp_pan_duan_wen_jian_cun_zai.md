# win10 uwp 判断文件存在

本文主要翻译http://stackoverflow.com/questions/37119464/uwp-check-if-file-exists/37152526#37152526

我们有多种方法可以判断文件是否存在，最简单的方法是异常
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

假如我们的文件叫 file，那么如何判断他是否存在，可以使用下面方法

```csharp
            string str = "file";
            try
            {
                StorageFolder folder = ApplicationData.Current.LocalFolder;
                StorageFile file = await StorageFile.GetFileFromPathAsync(folder.Path + "\\" + str);
            }
            catch (Exception e)
            {
                reminder = "文件不存在";
            }
```

发现已经有大神写了 http://www.cnblogs.com/bomo/p/4934447.html

我上面写的异常其实不对，因为exception会获得全部异常，如果文件存在，也是会报异常如果文件被写或者什么原因，看了大神的博客使用`FileNotFoundException`我们可以通过异常知道文件不存在。

但是异常效率太低

所以下面方法可以看到使用遍历

```csharp
public async Task<bool> isFilePresent(string fileName)
{ 
    bool fileExists = false;
    var allfiles = await ApplicationData.Current.LocalFolder.GetFilesAsync();
    foreach (var storageFile in allfiles)
    {
        if (storageFile.Name == fileName)
        {
            fileExists = true;
        }
    }
    return fileExists;
}
```

如果我有很多文件，那么这样也是不好

我们有比较好的方法

```csharp
            StorageFile file;
            StorageFolder folder = ApplicationData.Current.LocalFolder;
            file = await folder.TryGetItemAsync(str) as StorageFile;
            if (file == null)
            {
                reminder = "文件不存在";
            }
```

我们还可以使用下面方法

```csharp
            StorageFolder folder = ApplicationData.Current.LocalFolder;
            str = folder.Path + "\\" + str;
            FileInfo file = new FileInfo(str);
            if (!file.Exists)
            {
                reminder = "文件不存在";
            }
```

这方法可以不使用async，我建议使用的是try，虽然效率不知，但是一般使用async慢点也没事 因为我现在还没移动开发所以对效率还没有那么看

<a href="http://www.codeproject.com/script/Articles/BlogFeedList.aspx?amid=12520573" rel="tag">CodeProject</a>

https://blogs.msdn.microsoft.com/shashankyerramilli/2014/02/17/check-if-a-file-exists-in-windows-phone-8-and-winrt-without-exception/

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

