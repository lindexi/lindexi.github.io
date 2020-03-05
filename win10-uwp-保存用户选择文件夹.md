# win10 uwp 保存用户选择文件夹

如果我们每次把临时处理的文件保存，都要让用户选择一次，用户会不会觉得uwp垃圾？如果我们每次打开应用，都从某个文件读取，而这个文件不在应用目录和已知的目录，那么每次都需要用户选择，用户会不会觉得uwp垃圾？

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

其实垃圾wr给了一个类，我们可以用这个类获得用户选择过的文件、文件夹。我们可以保存用户选择过的文件和文件夹。

这个类在`Windows.Storage.AccessCache`，可以使用最近使用或一个比较大的存储，最近使用StorageItemMostRecentlyUsedList 只有存储25个，我这里就不说。如果需要存多点，我们可以使用 FutureAccessList ，使用`StorageApplicationPermissions.FutureAccessList` 可以存储1k个文件或文件夹，但垃圾wr给了我们这个东西，就不知道为何他就给1k而不是无限。

其实说是 1k ，和无限也差不多，因为他算的不只是文件，如果是文件夹，也算一个。假如我们有1000000000个文件，我们把它放在一个文件夹，把文件夹放 FutureAccessList ，算一个，不是 1000000000 个 。这个问题在博客上次没有说明白，让花神认为可以放的文件就 1k 个，其实可以放的文件比较多，就把文件放在文件夹，存文件夹。

要使用这个类，其实我们就只用三个函数，其中我们需要知道，使用 FutureAccessList 拿出一个我们保存的文件或文件夹时需要一个 token 。它是如何来的，在我们添加一个文件或文件夹，就会返回的，于是我们要把它保存。

我们先从让用户选择选择文件夹，选择文件夹需要 FolderPicker 。

不知道 FolderPicker 需要 FileTypeFilter，这个设计有个r用。他还必须要给，于是我们需要给他一个值，开头是"." 后面还需要一些e文，当然中文也可以。

```csharp
            FolderPicker pick=new FolderPicker();
            pick.FileTypeFilter.Add(".png"); 我不想说他这个有什么用，垃圾wr
```

然后让用户选择，这里是异步，我们可以让用户选择，然后我们在后台继续做我们的。

```csharp

            IAsyncOperation<StorageFolder> folderTask= pick.PickSingleFolderAsync();
            //做我们的 
            StorageFolder folder = await folderTask;
```

我们可以写省一点，让用户选择文件夹，而我们不做什么

```csharp

            var folder = await pick.PickSingleFolderAsync();

```

判断用户选择，如果有选择文件夹，可以看到 folder ！=null

判断选择之后，我们需要把它放进 FutureAccessList，放进去可以拿到token

```csharp

            if (folder != null)
            {
                Folder = folder;
                Address = folder.Path;
                Token = StorageApplicationPermissions.FutureAccessList.Add(folder);
            }
```

我们想拿出文件或文件夹，需要token，我们需要把token放到我们本地文件的位置。

我放在 `account/account.json` 文件里

先拿到本地文件夹

```csharp

            string folderStr = "account";
            StorageFolder folder = ApplicationData.Current.LocalFolder;
```

创建一个文件夹 account ，因为如果是第一次我们就没有文件夹，如果不是我们就有

```csharp

            try
            {
                folder = await folder.GetFolderAsync(folderStr);
            }
            catch (FileNotFoundException)  我们使用的是File没找到，不爽自己做一个Folder没找到
            {
                folder = await folder.CreateFolderAsync(folderStr);
            }
```

创建文件可以覆盖存在的Account.json

```csharp

            StorageFile file = await folder.CreateFileAsync(
                folderStr+".json", CreationCollisionOption.ReplaceExisting);
```

因为我们就保存一次，如果在保存中，用户关机，那么就和微软的诺基亚差不多了。一个好的做法是创建 temp 文件，写入 temp 文件，然后做完所有再把原来的改名字为以前做的，然后把temp改名字为account.json，注意保存原先改名字的文件，并把原先改名字的之前的文件删除。

如果我们在保存失败，那么我们的原先文件不会影响，如果我们保存文件在改名字出错了，可以通过原先恢复，这样才好。

我们使用json保存，json可以使用第三方，我有写过 win10 uwp json 保存相关的。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=eb992e37cd0bd5c07ae125648f6328bb)

```csharp

  var json = JsonSerializer.Create();
```
 
我们需要用 StringBuilder 来把json序列存放，如果直接用

```csharp

            json.Serialize(new JsonTextWriter(
                new StreamWriter(await file.OpenStreamForWriteAsync())), Account);
```

就会出现下次使用 UnauthorizedException，UWP UnauthorizedException 创建文件在 LocalFolder 。这个一般就是我们使用json错误，我提供简单方法，可以让保存不出现这个异常
。

我写的方法应该可以是可以把 token 保存的

```csharp

            StringBuilder str = new StringBuilder();
            StringWriter stream=new StringWriter(str);
            json.Serialize(new JsonTextWriter(
               stream ), Account);
            using (StorageStreamTransaction transaction = await file.OpenTransactedWriteAsync())
            {
                using (DataWriter dataWriter = new DataWriter(transaction.Stream))
                {
                    dataWriter.WriteString(str.ToString());
                    transaction.Stream.Size = await dataWriter.StoreAsync();
                    await transaction.CommitAsync();
                }
            }
```

我们应用开始我们就可以读取 account.json

读取就可以用 json 的 Deserialize

```csharp

                    StorageFile file = await folder.GetFileAsync(folderStr + ".json");
                    var json = JsonSerializer.Create();
                    Account = json.Deserialize<Account>(new JsonTextReader(
                        new StreamReader(await file.OpenStreamForReadAsync())));
          
```



我们从 FutureAccessList 拿文件就可以用 StorageApplicationPermissions.FutureAccessList.GetFileAsync 

```csharp

             file = await StorageApplicationPermissions.FutureAccessList.GetFileAsync(Account.Token);

```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。



