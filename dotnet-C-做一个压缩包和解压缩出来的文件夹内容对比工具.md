
# dotnet C# 做一个压缩包和解压缩出来的文件夹内容对比工具

我需要判断一个由某个压缩包文件解压缩出来的文件夹里面的文件内容，是否和压缩包里面记录的相同，于是就写了这个工具

<!--more-->


<!-- CreateTime:2025/04/10 09:02:07 -->

<!-- 发布 -->
<!-- 博客 -->

本文记录的工具能够实现以下情况的对比：

- 文件夹里面丢失了某个在压缩包里面记录的文件
- 文件夹里面的文件和压缩包里面的文件的内容不相同
- 文件夹多出了压缩包里面没有的文件

我这里新建的是一个 .NET 9 的 WPF 应用，只有简单的界面，理论上核心代码可以应用在任何 UI 框架上，甚至控制台项目也可以

界面代码如下，非常简单，只有两个输入框，用来分别输入压缩包文件路径和解压缩的文件夹路径，和一个按钮

```xml
    <Grid>
        <Grid HorizontalAlignment="Center" VerticalAlignment="Center">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"></RowDefinition>
                <RowDefinition Height="Auto"></RowDefinition>
                <RowDefinition Height="Auto"></RowDefinition>
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"></ColumnDefinition>
                <ColumnDefinition MinWidth="200"></ColumnDefinition>
            </Grid.ColumnDefinitions>

            <TextBlock VerticalAlignment="Center" Text="压缩包路径："></TextBlock>
            <TextBox x:Name="ZipFilePathTextBox" Grid.Row="0" Grid.Column="1" VerticalAlignment="Center" Text="C:\lindexi\Work\Source.zip"></TextBox>

            <TextBlock Grid.Row="1" Grid.Column="0" Margin="0,10,0,0" VerticalAlignment="Center" Text="解压缩文件夹："></TextBlock>
            <TextBox x:Name="UnzipFolderPathTextBox" Grid.Row="1" Grid.Column="1" Margin="0,10,0,0" VerticalAlignment="Center" Text="C:\lindexi\Work\Unzip\"></TextBox>

            <Button Grid.Row="2" Grid.Column="1" Margin="10,10,0,10" HorizontalAlignment="Right" Content="对比" Click="Button_OnClick"></Button>
        </Grid>
    </Grid>
```

我编写了一个 ZipComparer 类型，用这个类型可以用来对比压缩包文件和解压缩文件夹里面的内容，按钮的 `Button_OnClick` 代码如下

```csharp
    private async void Button_OnClick(object sender, RoutedEventArgs e)
    {
        try
        {
            var zipCompareOptions = new ZipCompareOptions(ReturnFast: true, IgnoreExtra: false);

            var result = await ZipComparer.Compare(new FileInfo(ZipFilePathTextBox.Text), new DirectoryInfo(UnzipFolderPathTextBox.Text), zipCompareOptions);

            if (result.IsSuccess)
            {
                MessageBox.Show("文件一致");
            }
            else
            {
                var stringBuilder = new StringBuilder();
                stringBuilder.AppendLine("文件不一致");
                foreach (var item in result.DifferenceList)
                {
                    stringBuilder.AppendLine(item.RelativeFilePath);
                    stringBuilder.AppendLine(item.DifferenceType.ToString());
                }
                MessageBox.Show(stringBuilder.ToString());
            }
        }
        catch
        {
            // 忽略
        }
    }
```

核心代码是 ZipComparer 类，代码如下

```csharp

public static class ZipComparer
{
    public static async Task<ZipCompareResult> Compare(FileInfo zipFile, DirectoryInfo unzipFolder, ZipCompareOptions options)
    {
        HashSet<string/*RelativePath*/>? visitedFileSet = null;
        if (!options.IgnoreExtra)
        {
            visitedFileSet = [];
        }

        List<ZipCompareDifferenceFileInfo>? differenceList = null;

        await using var fileStream = zipFile.OpenRead();
        using var zipArchive = new System.IO.Compression.ZipArchive(fileStream, System.IO.Compression.ZipArchiveMode.Read, leaveOpen: true);
        foreach (var zipArchiveEntry in zipArchive.Entries)
        {
            var name = zipArchiveEntry.FullName;
            visitedFileSet?.Add(name);
            var filePath = Path.Join(unzipFolder.FullName, name);

            if (!File.Exists(filePath))
            {
                // 文件不存在
                AddDifference(new ZipCompareDifferenceFileInfo(name, ZipCompareDifferenceType.Miss));

                if (options.ReturnFast)
                {
                    break;
                }
                else
                {
                    continue;
                }
            }

            var fileInfo = new FileInfo(filePath);
            if (fileInfo.Length != zipArchiveEntry.Length)
            {
                // 文件大小不同
                AddDifference(new ZipCompareDifferenceFileInfo(name, ZipCompareDifferenceType.ContentLengthDifference));
                if (options.ReturnFast)
                {
                    break;
                }
                else
                {
                    continue;
                }
            }

            // 先不引入 Crc32 对比逻辑。要真正想用，需要引入 System.IO.Hashing 包
            //var crc32 = zipArchiveEntry.Crc32;
            //if (crc32 != 0 && Crc32.IsSupported)
            //{
            //    var fileCrc32 = await GetFileZipCrcAsync(fileInfo);
            //    if (crc32 == fileCrc32)
            //    {
            //        continue;
            //    }
            //    else
            //    {
            //        // CRC32 不同
            //        AddDifference(new ZipCompareDifferenceFileInfo(name, ZipCompareDifferenceType.ContentDifference));
            //        if (options.ReturnFast)
            //        {
            //            break;
            //        }
            //        else
            //        {
            //            continue;
            //        }
            //    }
            //}

            // 开始对比文件内容
            await using var zipStream = zipArchiveEntry.Open();
            await using var currentFileStream = fileInfo.OpenRead();
            var success = await CompareStream(zipStream, currentFileStream);
            if (!success)
            {
                // 文件内容不同
                AddDifference(new ZipCompareDifferenceFileInfo(name, ZipCompareDifferenceType.ContentDifference));
                if (options.ReturnFast)
                {
                    break;
                }
            }
        }

        var isDifferenceFastReturn = options.ReturnFast && differenceList is { Count: > 0 };

        if (!options.IgnoreExtra
            // 如果是快速返回，则不需要检查额外的文件，此时已经存在不相同的文件
            && !isDifferenceFastReturn)
        {
            // 如果不能忽略额外的文件，则需要检查解压缩文件夹中是否有额外的文件
            Debug.Assert(visitedFileSet != null);

            foreach (var file in unzipFolder.EnumerateFiles("*", new EnumerationOptions()
            {
                RecurseSubdirectories = true
            }))
            {
                var relativePath = Path.GetRelativePath(unzipFolder.FullName, file.FullName);
                if (!visitedFileSet.Contains(relativePath))
                {
                    // 额外的文件
                    AddDifference(new ZipCompareDifferenceFileInfo(relativePath, ZipCompareDifferenceType.Extra));
                    if (options.ReturnFast)
                    {
                        break;
                    }
                }
            }
        }

        return new ZipCompareResult
        {
            DifferenceList = differenceList,
        };

        void AddDifference(ZipCompareDifferenceFileInfo info)
        {
            differenceList ??= new List<ZipCompareDifferenceFileInfo>();
            differenceList.Add(info);
        }
    }

    //private static async ValueTask<uint> GetFileZipCrcAsync(FileInfo fileInfo)
    //{
    //    const int bufferLength = 4 * 1024;
    //    var buffer = ArrayPool<byte>.Shared.Rent(bufferLength);
    //    uint crc = 0;
    //    try
    //    {
    //        await using var fileStream = fileInfo.OpenRead();
    //        var memory = buffer.AsMemory(0, bufferLength);

    //        while (true)
    //        {
    //            var readLength = await fileStream.ReadAsync(memory);

    //            if (readLength == 0)
    //            {
    //                return crc;
    //            }

    //            for (int i = 0; i < readLength; i++)
    //            {
    //                crc = Crc32.ComputeCrc32(crc, memory.Span[i]);
    //            }
    //        }
    //    }
    //    finally
    //    {
    //        ArrayPool<byte>.Shared.Return(buffer);
    //    }
    //}

    private static async ValueTask<bool> CompareStream(Stream a, Stream b)
    {
        const int bufferLength = 4 * 1024;
        var bufferA = ArrayPool<byte>.Shared.Rent(bufferLength);
        var bufferB = ArrayPool<byte>.Shared.Rent(bufferLength);

        try
        {
            while (true)
            {
                var memoryA = bufferA.AsMemory(0, bufferLength);

                var readLength = await a.ReadAsync(memoryA);
                if (readLength == 0)
                {
                    // 读取完毕
                    return true;
                }

                var memoryB = bufferB.AsMemory(0, readLength);
                await b.ReadExactlyAsync(memoryB);

                if (!memoryA.Span.Slice(0, readLength).SequenceEqual(memoryB.Span))
                {
                    return false;
                }
            }
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(bufferA);
            ArrayPool<byte>.Shared.Return(bufferB);
        }
    }
}

/// <summary>
/// 比较的选项
/// </summary>
/// <param name="ReturnFast">找到第一个差异就立刻结束</param>
/// <param name="IgnoreExtra">忽略额外的文件。忽略解压缩文件夹存在，但压缩包不存在的文件</param>
public readonly record struct ZipCompareOptions(bool ReturnFast, bool IgnoreExtra);

public readonly record struct ZipCompareResult
{
    [MemberNotNullWhen(false, nameof(DifferenceList))]
    public bool IsSuccess => DifferenceList is null || DifferenceList.Count == 0;

    public IReadOnlyList<ZipCompareDifferenceFileInfo>? DifferenceList { get; internal init; }
}

public readonly record struct ZipCompareDifferenceFileInfo(string RelativeFilePath, ZipCompareDifferenceType DifferenceType);

public enum ZipCompareDifferenceType
{
    /// <summary>
    /// 文件不存在
    /// </summary>
    Miss,

    /// <summary>
    /// 文件大小不同
    /// </summary>
    ContentLengthDifference,

    /// <summary>
    /// 文件内容不同
    /// </summary>
    ContentDifference,

    /// <summary>
    /// 额外的文件，即解压缩文件夹存在，但压缩包不存在的文件
    /// </summary>
    Extra,
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/88e3c7fe8009b4ad239481404c2c4b5089f05953/WPFDemo/CardawnarheaCahichemga) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/88e3c7fe8009b4ad239481404c2c4b5089f05953/WPFDemo/CardawnarheaCahichemga) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 88e3c7fe8009b4ad239481404c2c4b5089f05953
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 88e3c7fe8009b4ad239481404c2c4b5089f05953
```

获取代码之后，进入 WPFDemo/CardawnarheaCahichemga 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。