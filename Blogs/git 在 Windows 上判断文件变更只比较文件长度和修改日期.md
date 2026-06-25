---
title: git 在 Windows 上判断文件变更只比较文件长度和修改日期
description: 本文将介绍 git 在 Windows 上的文件变更检测机制，通过一个 WPF 小工具直观演示 git 如何依赖文件长度和最后修改日期判断文件状态，以及如何利用这一特性让 git "看不到" 文件内容的改变。
tags: git
category: 
---

<!-- 发布 -->
<!-- 博客 -->

本文内容由人类主导 AI 辅助编写

## 背景

在使用 git 管理代码时，大家都知道 `git status` 可以查看哪些文件发生了变更，`git diff` 可以看到具体的变更内容。但 git 底层是如何判断一个文件"是否被修改过"的呢？

在 Windows 上，git 判断文件是否变更的方式非常朴素：它比较的是文件的**长度（size）**和**最后修改日期（last write time）**。也就是说，git 并不会实时对文件内容做哈希校验。只要一个文件的长度和最后修改时间与 git 索引中记录的一致，git 就会认为这个文件没有被修改过。

这就带来了一个有趣的现象：如果修改了文件内容但保持文件长度不变，并且在修改后将文件的最后修改时间还原为修改前的值，那么 git 就会完全"无视"这次修改。

为了直观地演示这一行为，我写了一个简单的 WPF 拖拽工具。将文件拖入窗口，勾选"内容随机化（不改长度，还原修改时间）"，就能生成一个内容已变但 git 检测不到的文件。

## 工具界面

工具的主窗口非常简单，包含三个复选框和一个用于展示结果的列表：

- **创建时间**：将文件的创建时间修改为当前时间
- **最后修改时间**：将文件的最后修改时间修改为当前时间
- **内容随机化（不改长度，还原修改时间）**：随机修改文件中间部分字节，保持文件长度不变，并在修改后将最后修改时间还原

文件通过拖拽的方式进入工具，处理结果会显示在列表中。

## 核心逻辑

### 保存原始修改时间

在处理每个拖入的文件时，第一步是保存文件当前的最后修改时间。这个值在后续内容随机化之后需要用来"还原现场"：

```csharp
var originalLastWriteTime = File.GetLastWriteTime(file);
```

为什么要提前保存？因为一旦开始写入文件，文件的最后修改时间就会被操作系统自动更新。如果不提前记下来，后面就无法还原了。

### 随机化文件内容但保持长度不变

这是整个工具最核心的方法，也是演示 git 行为的关键所在：

```csharp
private static void RandomizeFileContent(string filePath)
{
    var info = new FileInfo(filePath);
    if (info.Length == 0)
        return;

    // 选择要随机修改的字节范围：文件中间区域的一部分
    var length = info.Length;
    var startOffset = length / 4;
    var regionLength = Math.Max(1, length / 8);

    var random = new Random();
    var buffer = new byte[regionLength];
    random.NextBytes(buffer);

    using var stream = new FileStream(filePath, FileMode.Open, FileAccess.Write, FileShare.None);
    stream.Position = startOffset;
    stream.Write(buffer, 0, buffer.Length);
}
```

这段代码做了以下几件事：

首先，通过 `FileInfo` 获取文件的当前长度。如果文件是空的，直接返回——空文件没有任何字节可以修改。

接着，计算要修改的字节范围。这里选择的是文件中间偏前的一段区域：从文件四分之一处开始，修改八分之一长度的字节。举个例子，如果一个文件长 800 字节，那么从第 200 字节开始，修改 100 个字节。为什么不修改文件头部或尾部？因为修改中间区域对大多数文件格式来说影响最小，同时也能确保文件长度完全不变。

然后，用 `Random.NextBytes` 生成随机字节填充缓冲区。这些随机字节就是将要写入文件的新内容。

最后，通过 `FileStream` 以写入模式打开文件，将流指针定位到计算好的偏移位置，写入随机字节后关闭流。整个过程中文件长度没有任何变化，只是中间某段字节被替换成了随机值。

### 还原修改时间以骗过 git

在调用 `RandomizeFileContent` 之后，文件的最后修改时间已经被操作系统更新了。如果就这样放着，git 当然会检测到文件发生了变化。所以需要把修改时间还原回去：

```csharp
if (randomizeContent)
    RandomizeFileContent(file);

if (updateCreation)
    File.SetCreationTime(file, now);

if (updateLastWrite)
    File.SetLastWriteTime(file, now);
else if (randomizeContent)
    File.SetLastWriteTime(file, originalLastWriteTime);
```

这段逻辑的顺序设计得很仔细。先执行内容随机化，然后根据用户勾选的选项决定时间如何处理：

如果用户勾选了"最后修改时间"，那就把修改时间设置为当前时间 `now`。这种情况下 git 会正常检测到文件变更——因为修改时间变了。

但如果用户只勾选了"内容随机化"而没有勾选"最后修改时间"，那么就会走 `else if` 分支，将文件的最后修改时间还原为 `originalLastWriteTime`，也就是修改前的原始时间。此时文件内容虽然已经变了，但文件长度不变、最后修改时间也不变——git 的 `status` 和 `diff` 就无法检测到这次修改。

创建时间的处理则独立进行，如果勾选了就设置为当前时间。创建时间并不影响 git 的文件变更判断。

### 拖拽接收文件

文件通过 Windows 拖拽机制进入工具：

```csharp
private void Window_Drop(object sender, DragEventArgs e)
{
    if (!e.Data.GetDataPresent(DataFormats.FileDrop))
        return;

    var files = (string[])e.Data.GetData(DataFormats.FileDrop);
    if (files is null)
        return;
    // ... 处理每个文件
}
```

`DataFormats.FileDrop` 是 Windows 拖拽操作中传递文件路径的标准格式。`GetData` 返回的是一个字符串数组，每个元素都是一个完整的文件路径。

`DragEnter` 和 `DragOver` 事件中设置了 `e.Effects = DragDropEffects.Copy`，让鼠标在拖拽经过窗口时显示复制光标，给用户明确的视觉反馈。

## 实验验证

用这个工具可以做一个简单的实验：

1. 在一个 git 仓库中创建一个文本文件，提交一次 commit
2. 运行 `git status`，确认工作区是干净的
3. 将文件拖入工具，只勾选"内容随机化（不改长度，还原修改时间）"
4. 再次运行 `git status`

你会发现 git 报告工作区仍然是干净的，没有任何变更。但实际上用记事本打开文件就能看到，文件中间的一些字符已经变成了乱码——内容确实被修改了，只是 git 没有发现。

这个实验说明了 git 在 Windows 上判断文件变更的机制：它依赖的是文件系统中记录的文件长度和最后修改日期，而不是对文件内容做实时哈希比对。

## 总结

git 在 Windows 上通过文件长度和最后修改日期来判断文件是否发生了变更。这并不是一个安全漏洞，而是一种性能上的权衡——对每个文件实时计算哈希会消耗大量 IO 和 CPU 资源。在日常开发中，这一机制足够可靠，因为正常情况下修改文件内容几乎总会导致文件长度变化或修改时间更新。了解这一底层行为，有助于更好地理解 git 的工作方式，也能避免在某些特殊场景下产生困惑。

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6ca2dfc0c4394722e274c92b54e0f65ef4842e3a/WPFDemo/FawduhachalereNawfurgaywaifar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6ca2dfc0c4394722e274c92b54e0f65ef4842e3a/WPFDemo/FawduhachalereNawfurgaywaifar) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6ca2dfc0c4394722e274c92b54e0f65ef4842e3a
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6ca2dfc0c4394722e274c92b54e0f65ef4842e3a
```

获取代码之后，进入 WPFDemo/FawduhachalereNawfurgaywaifar 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
