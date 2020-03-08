# win10 uwp 获取指定的文件 AQS

很多时候不需要获取整个文件夹的文件，是需要获取文件夹里指定的文件。

那么 UWP 如何对文件夹里的文件进行过滤，只拿出自己需要的文件？

本文：如何使用通配符或文件匹配方式在uwp获取文件夹中指定的文件

如果阅读中发现有任何句子不通的，请告诉我 lindexi_gd@163.com
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


假如需要文件 有前缀"latest_" ，简单的方法是拿出 文件夹所有的文件，使用简单的对比，但是这样需要获取文件夹所有文件，速度比较慢。


```csharp
    var previousInfo = (await rootFolder.GetFilesAsync()).Where(file => file.DisplayName.StartsWith("latest_")).FirstOrDefault();
```

一个好的方法是使用 [Advanced Query Syntax ](https://msdn.microsoft.com/zh-cn/library/windows/desktop/bb266512(v=vs.85).aspx)

第一步，新建QueryOptions


```csharp
    var queryOptions = new QueryOptions();
queryOptions.ApplicationSearchFilter = "System.FileName:latest_*";
```

上面的 ApplicationSearchFilter 就是根据 [Using Advanced Query Syntax Programmatically (Windows)](https://msdn.microsoft.com/zh-cn/library/windows/desktop/bb266512(v=vs.85).aspx) 写出对应判断

然后从当前的 文件夹 获取匹配


```csharp
    StorageFileQueryResult queryResult = folder.CreateFileQueryWithOptions(queryOptions);
```

最后就是从结果拿出文件


```csharp
    var files = await queryResult.GetFilesAsync(); 
```

参见：http://stackoverflow.com/a/43829407/6116637

## UWP 获取指定后缀的文件

修改上面的 ApplicationSearchFilter 为下面格式就可以获得后缀


```csharp
    System.FileExtension:=".后缀"
```

如获取 txt 可以使用


```csharp
    queryOptions.ApplicationSearchFilter = "System.FileExtension:=\".txt\"";
```

设置后缀除了上面的方法，可以使用简单的设置


```csharp
             var queryOptions = new QueryOptions();
            queryOptions.FileTypeFilter.Add(".txt");
```


除了后缀，还可以设置文件大小


```csharp
    System.Size: 

    System.Size: <1kb
```

如果有多个判断，使用 and 连接或 or 连接，但是关于他的语法文档很少。

如果需要的搜索是包括子文件夹的内容，请设置  queryOptions.FolderDepth


```csharp
    queryOptions.FolderDepth = FolderDepth.Deep
```


参见：[Advanced Query Syntax](https://msdn.microsoft.com/zh-cn/library/windows/apps/aa965711.aspx)

https://docs.microsoft.com/en-us/uwp/api/windows.storage.search.queryoptions

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 

