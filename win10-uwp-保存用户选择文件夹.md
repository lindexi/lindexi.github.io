
文件夹找不到报的异常`FileNotFoundException`

FutureAccessList

如果我们每次把临时处理的文件保存，都要让用户选择一次，用户会不会觉得uwp垃圾？如果我们每次打开应用，都从某个文件读取，而这个文件不在应用目录和已知的目录，那么每次都需要用户选择，用户会不会觉得uwp垃圾？

其实垃圾wr给了一个类，我们可以用这个类获得用户选择过的文件、文件夹。我们可以保存用户选择过的文件和文件夹。

这个类在`Windows.Storage.AccessCache`，可以使用最近使用或一个比较大的存储，最近使用自有25个，我这里就不说。FutureAccessList可以存储1k个文件或文件夹，但垃圾wr给了我们这个东西，找不到他就给1k而不是无限。

要使用这个类，其实我们就只用三个函数，其中我们需要知道，使用FutureAccessList拿出一个我们保存的文件或文件夹需要一个token。

我们先从让用户选择选择文件夹，选择文件夹需要FolderPicker。

不知道FolderPicker需要FileTypeFilter，这个设计。我们需要给他一个值，开头是"."

```
            FolderPicker pick=new FolderPicker();
            pick.FileTypeFilter.Add(".png");
```

然后让用户选择，这里是异步，我们可以让用户选择，然后我们做我们的。

```

            IAsyncOperation<StorageFolder> folderTask= pick.PickSingleFolderAsync();
            //做我们的
            StorageFolder folder = await folderTask;
```

我们可以省一点，让用户选择我们不做什么

```

            var folder =await pick.PickSingleFolderAsync();

```

判断用户选择，如果有选择folder ！=null

判断选择我们需要把它放进FutureAccessList，放进去可以拿到token

```

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

```

            string folderStr = "account";
            StorageFolder folder = ApplicationData.Current.LocalFolder;
```

创建一个文件夹account，因为如果是第一次我们就没有文件夹，如果不是我们就有

```

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

```


```