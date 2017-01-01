# win10 uwp 保存用户选择文件夹

如果我们每次把临时处理的文件保存，都要让用户选择一次，用户会不会觉得uwp垃圾？如果我们每次打开应用，都从某个文件读取，而这个文件不在应用目录和已知的目录，那么每次都需要用户选择，用户会不会觉得uwp垃圾？
<!--more-->

其实垃圾wr给了一个类，我们可以用这个类获得用户选择过的文件、文件夹。我们可以保存用户选择过的文件和文件夹。

这个类在`Windows.Storage.AccessCache`，可以使用最近使用或一个比较大的存储，最近使用自有25个，我这里就不说。FutureAccessList可以存储1k个文件或文件夹，但垃圾wr给了我们这个东西，找不到他就给1k而不是无限。

要使用这个类，其实我们就只用三个函数，其中我们需要知道，使用FutureAccessList拿出一个我们保存的文件或文件夹需要一个token。

我们先从让用户选择选择文件夹，选择文件夹需要FolderPicker。

不知道FolderPicker需要FileTypeFilter，这个设计。我们需要给他一个值，开头是"."

```csharp
            FolderPicker pick=new FolderPicker();
            pick.FileTypeFilter.Add(".png");
```

然后让用户选择，这里是异步，我们可以让用户选择，然后我们做我们的。

```csharp

            IAsyncOperation<StorageFolder> folderTask= pick.PickSingleFolderAsync();
            //做我们的
            StorageFolder folder = await folderTask;
```

我们可以省一点，让用户选择我们不做什么

```csharp

            var folder =await pick.PickSingleFolderAsync();

```

判断用户选择，如果有选择folder ！=null

判断选择我们需要把它放进FutureAccessList，放进去可以拿到token

```csharp

            if (folder != null)
            {
                Folder = folder;
                Address = folder.Path;
                Token=StorageApplicationPermissions.FutureAccessList.Add(folder);
            }
```

然后我们需要把token放到我们本地文件的位置。

我放在account/account.json

先拿到本地文件夹

```csharp

            string folderStr = "account";
            StorageFolder folder = ApplicationData.Current.LocalFolder;
```

创建一个文件夹account，因为如果是第一次我们就没有文件夹，如果不是我们就有

```csharp

            try
            {
                folder = await folder.GetFolderAsync(folderStr);
            }
            catch (FileNotFoundException)
            {
                folder = await folder.CreateFolderAsync(folderStr);
            }
```

创建文件

```csharp

            StorageFile file = await folder.CreateFileAsync(
                folderStr+".json", CreationCollisionOption.ReplaceExisting);
```

因为我们就保存一次，一个好的做法是创建temp，然后做完所有再把原来的改名字为以前做的，然后把temp改名字为account.json，注意保存原先改名字的文件，并把原先改名字的之前的文件删除。

如果我们在保存失败，那么我们的原先文件不会影响，如果我们保存文件在改名字出错了，可以通过原先恢复，这样才好。

我们使用json保存，使用我们第三方。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=eb992e37cd0bd5c07ae125648f6328bb)

```csharp

  var json = JsonSerializer.Create();
```
 
我们需要用StringBuilder来把json序列存放，如果直接用

```csharp

            json.Serialize(new JsonTextWriter(
                new StreamWriter(await file.OpenStreamForWriteAsync())), Account);
```

就会出现下次使用UnauthorizedException，UWP UnauthorizedException 创建文件在LocalFolder一般就是我们使用json错误

Account就是我们的数据。

我写的方法应该可以是可以把token保存的

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

我们应用开始我们就可以读取

读取就可以用json的

```csharp

                    StorageFile file = await folder.GetFileAsync(folderStr + ".json");
                    var json = JsonSerializer.Create();
                    Account = json.Deserialize<Account>(new JsonTextReader(
                        new StreamReader(await file.OpenStreamForReadAsync())));
          
```
folder要进去account

我们从FutureAccessList拿文件

```csharp

             file = await StorageApplicationPermissions.FutureAccessList.GetFileAsync(Account.Token);

```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。



