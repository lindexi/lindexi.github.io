# dotnet 警惕设置 StreamReader 的 BaseStream 的 Position 属性

使用 StreamReader 的过程中，如果设置了 StreamReader 的 BaseStream 的 Position 属性，由于 StreamReader 内部缓存的影响，可能会在缓存消耗完成之前，依然是读取缓存的值，导致行为不符合预期

<!--more-->
<!-- CreateTime:2025/04/19 07:27:51 -->

<!-- 发布 -->
<!-- 博客 -->

如以下代码设置了 StreamReader 的 BaseStream 的 Position 属性，此时 StreamReader 在读取过程中，是没有感知到 BaseStream 的 Position 变更了，依然会继续使用 StreamReader 内部维护的缓存，直到缓存用完了，才会从 BaseStream 读取，这可能会导致读取行为不符合预期

```csharp
streamReader.BaseStream.Seek(0, SeekOrigin.Begin);
```

为了说明此问题，我构建了一个测试项目，让大家猜猜这个测试项目的运行行为会是什么样

先创建一份测试的文件，这个文件就包含了很多行，第 0 行填充 0 的字符，第 1 行填充 1 的字符……

```csharp
using System.Text;

// 创建测试的文件
var textFile = Path.Join(AppContext.BaseDirectory, "Text.txt");
WriteTestContent();

void WriteTestContent()
{
    var stringBuilder = new StringBuilder();
    for (int i = 0; i < 10; i++)
    {
        stringBuilder.AppendLine(string.Join("", Enumerable.Repeat(i.ToString(), 200)));
    }

    var testText = stringBuilder.ToString();
    File.WriteAllText(textFile, testText);
}
```

输出的 Text.txt 文件的内容大概如下

```
0000000...
1111111...
2222222...
...
9999999...
```

现在尝试使用 StreamReader 进行一行行读取，代码如下

```csharp
// 构建测试代码
using var fileStream = File.OpenRead(textFile);
using var streamReader = new StreamReader(fileStream);

// 读取两行
var line1 = streamReader.ReadLine(); // 0000000...
var line2 = streamReader.ReadLine(); // 1111111...
```

自然，前面读取的两行很符合预期，就是读取到一行 0 和一行 1 的值

接下来试试修改 BaseStream 的 Position 属性，代码如下

```csharp
var line1 = streamReader.ReadLine(); // 0000000...
var line2 = streamReader.ReadLine(); // 1111111...

streamReader.BaseStream.Seek(0, SeekOrigin.Begin);
```

设置完成之后，继续调用 ReadLine 方法，再读取几行

```csharp
var line1 = streamReader.ReadLine(); // 0000000...
var line2 = streamReader.ReadLine(); // 1111111...

streamReader.BaseStream.Seek(0, SeekOrigin.Begin);

var line3 = streamReader.ReadLine();
var line4 = streamReader.ReadLine();
var line5 = streamReader.ReadLine();
var line6 = streamReader.ReadLine();
var line7 = streamReader.ReadLine();
```

现在让大家猜猜，读取到的各个行分别是什么内容

现在公布答案，前面几行是需要 1111111... 读取下去，即 lin3 读取到 `2222222...` 的数据，就好像 `streamReader.BaseStream.Seek(0, SeekOrigin.Begin);` 毫无影响一样，但读取到 line6 的就是就离谱了，前面一半是 555555 但后面一半就是读取 0 值了，再继续 line7 就读取到了 `1111111` 的值

```csharp
streamReader.BaseStream.Seek(0, SeekOrigin.Begin);

// 此时继续读取两行，能够继续读取下去，但是读取到某个时刻，将会从头开始读取
var line3 = streamReader.ReadLine(); // 2222222...
var line4 = streamReader.ReadLine(); // 3333333...
var line5 = streamReader.ReadLine(); // 4444444...
var line6 = streamReader.ReadLine(); // 5555555...0000 ?!
var line7 = streamReader.ReadLine(); // 1111111 ???
```

这是为什么呢？这是因为 StreamReader 带一个内部的缓存，设置 BaseStream 到 0 位置后，StreamReader 还会继续使用自己内部的缓存，直到缓存用完，再从 BaseStream 读取。这就是为什么 line3 还是 2222222...，而 line6 读取到 5555555...0000000... 的原因。在读取 line3 的时候，内部的缓存还有值，于是继续使用内部的缓存，直到读取到 line6 一半时，才读取完了内部的缓存，需要从 BaseStream 读取，由于 BaseStream 被设置到开始位置，于是读取到 0000000... 的内容了

解决方法：

在设置 BaseStream 的 Position 属性后（或前一句），调用 StreamReader 的 DiscardBufferedData 方法，清空 StreamReader 的缓存

```csharp
streamReader.BaseStream.Seek(0, SeekOrigin.Begin);
streamReader.DiscardBufferedData();
```

此时继续读取就能读取到正确的符合预期的值

```csharp
streamReader.BaseStream.Seek(0, SeekOrigin.Begin);
streamReader.DiscardBufferedData();

var line3 = streamReader.ReadLine(); // 0000000...
var line4 = streamReader.ReadLine(); // 1111111...
var line5 = streamReader.ReadLine(); // 2222222...
```

官方的 StreamReader.DiscardBufferedData 文档： <https://learn.microsoft.com/en-us/dotnet/api/system.io.streamreader.discardbuffereddata?view=net-9.0>

只有当 BaseStream 可能发生变化，即 StreamReader 需要重新从 BaseStream 读取数据，缓存不在有效时，才能调用 DiscardBufferedData 方法

由于 DiscardBufferedData 本质就是丢掉 StreamReader 内部的缓存数据，这将会导致 StreamReader 重新从 BaseStream 读取数据，重复读取可能影响性能，或因为 BaseStream 行为变更导致重复读取拿不到不符合预期的数据。换句话说就是只有明确知道是 BaseStream 修改了 Position 需要清空 StreamReader 内部的缓存数据时，才能调用 DiscardBufferedData 方法，日常正常不应该调用 DiscardBufferedData 方法

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b6cbf5be402e98d225f01295d92112ab7fed514a/Workbench/JernelqeceljeNachijeeciwaykai) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/b6cbf5be402e98d225f01295d92112ab7fed514a/Workbench/JernelqeceljeNachijeeciwaykai) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b6cbf5be402e98d225f01295d92112ab7fed514a
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b6cbf5be402e98d225f01295d92112ab7fed514a
```

获取代码之后，进入 Workbench/JernelqeceljeNachijeeciwaykai 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

相关问题：[dotnet 已知问题 警惕 StreamReader 的 EndOfStream 卡住线程](https://blog.lindexi.com/post/dotnet-%E5%B7%B2%E7%9F%A5%E9%97%AE%E9%A2%98-%E8%AD%A6%E6%83%95-StreamReader-%E7%9A%84-EndOfStream-%E5%8D%A1%E4%BD%8F%E7%BA%BF%E7%A8%8B.html )
<!-- [dotnet 已知问题 警惕 StreamReader 的 EndOfStream 卡住线程 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18397603 ) -->