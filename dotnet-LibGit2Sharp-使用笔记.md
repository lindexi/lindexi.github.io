
# dotnet LibGit2Sharp 使用笔记

本文记录我对 LibGit2Sharp 库的使用笔记

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

LibGit2Sharp 库开源地址： <https://github.com/libgit2/libgit2sharp>

本文使用的版本是： 0.31.0

按照 dotnet 的惯例，使用之前先用 NuGet 安装，安装之后的 csproj 文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="LibGit2Sharp" Version="0.31.0" />
  </ItemGroup>

</Project>
```

我将使用我的博客代码仓库作为测试仓库，其本地路径地址是 `C:\lindexi\Code\lindexi\.git\` 还请大家更换为自己的文件夹

## 仓库初始化

```csharp
var folder = @"C:\lindexi\Code\lindexi\.git\";

var repository = new Repository(folder);
```

仓库初始化过程中需要传入的是 `.git` 文件夹路径

## 判断给定路径是否被忽略

比如判断 `bin\obj\Foo.exe` 文件是否应该在此 git 忽略列表内，可使用如下代码

```csharp
bool isPathIgnored = repository.Ignore.IsPathIgnored("bin/obj/Foo.exe");
```

以上代码的细节是路径斜杠需要使用 `/` 斜杠

## 获取 commit 的变更差异 Patch 内容

获取某个 commit 更改的文件，以及这些文件的更改内容，生成 Patch 文本的方法如下

```csharp
var queryableCommitLog = repository.Commits;
Commit commit = queryableCommitLog.First();

var patch = repository.Diff.Compare<Patch>(commit.Parents.First().Tree, commit.Tree);

foreach (PatchEntryChanges patchEntryChanges in patch)
{
    var path = patchEntryChanges.Path;
    
    string patchText = patchEntryChanges.Patch;
}
```

以上逻辑就是执行当前的 commit 和上一个 commit 的对比差异，通过差异的 Patch 获取当前的变更差异内容

以上的 `patchText` 可能的输出内容大概如下

```
    diff --git a/Workbench/ChearjinohecelKafemlairreena/Program.cs b/Workbench/ChearjinohecelKafemlairreena/Program.cs
    index 63d0e46..6a80004 100644
    --- a/Workbench/ChearjinohecelKafemlairreena/Program.cs
    +++ b/Workbench/ChearjinohecelKafemlairreena/Program.cs
    @@ -14,11 +14,18 @@ bool isPathIgnored = repository.Ignore.IsPathIgnored("bin/obj/Foo.exe");
     var queryableCommitLog = repository.Commits;
     Commit commit = queryableCommitLog.First();

    -var remoteCollection = repository.Network.Remotes;
    +ObjectId commitId = commit.Id;
    +GitObject gitObject = repository.Lookup(commitId);
    +
    +TreeChanges treeChanges = repository.Diff.Compare<TreeChanges>(commit.Parents.First().Tree,commit.Tree);
    +foreach (TreeEntryChanges treeEntryChanges in treeChanges)
    +{
    +    var path = treeEntryChanges.Path;
    +    var changeKind = treeEntryChanges.Status;
    +}

     HistoryDivergence historyDivergence = repository.ObjectDatabase.CalculateHistoryDivergence(queryableCommitLog.Skip(100).First(), commit);
     var historyDivergenceCommonAncestor = historyDivergence.CommonAncestor;

    -
     GC.KeepAlive(repository);
     Console.WriteLine("Hello, World!");
```

以上为一个个文件的获取 Patch 内容。获取整个 commit 的 Patch 内容可直接使用 Content 属性，如以下代码所示

```csharp
var patch = repository.Diff.Compare<Patch>(commit.Parents.First().Tree, commit.Tree);
var patchContent = patch.Content;
```

以上的 `patchContent` 就是整个 commit 的 Patch 内容，包含所有影响的文件和影响的内容

如果一个 commit 属于一个 Merge 的 commit 则可能 Parents 包含多项，这一点还需要大家根据自己的业务进行设置

## 查看历史差异

查看两个 commit 之间的历史差异，如哪个在前哪个在后，以及中间过程差了多少个 commit 数量，代码如下

```csharp
var queryableCommitLog = repository.Commits;

HistoryDivergence historyDivergence = repository.ObjectDatabase.CalculateHistoryDivergence(queryableCommitLog.Skip(100).First(), commit);
var historyDivergenceCommonAncestor = historyDivergence.CommonAncestor;
```

以上代码我特意选用了 `Skip(100)` 跳过当前 100 个 commit 去查找其差异作为示例

## 查看指定文件的 Git 历史

如以下代码，查看 Workbench/ChearjinohecelKafemlairreena/Program.cs 文件的变更历史

```csharp
var queryableCommitLog = repository.Commits;

var logEntries = queryableCommitLog.QueryBy("Workbench/ChearjinohecelKafemlairreena/Program.cs");
foreach (LogEntry logEntry in logEntries)
{
}
```

这里的细节是斜杠需要使用 `/` 斜杠

## 代码

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/60c472a79bfbc32b5b18071d5dec2f05046ff4db/Workbench/ChearjinohecelKafemlairreena) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/60c472a79bfbc32b5b18071d5dec2f05046ff4db/Workbench/ChearjinohecelKafemlairreena) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 60c472a79bfbc32b5b18071d5dec2f05046ff4db
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 60c472a79bfbc32b5b18071d5dec2f05046ff4db
```

获取代码之后，进入 Workbench/ChearjinohecelKafemlairreena 文件夹，即可获取到源代码

大家可以尝试拉取项目代码跑跑看。 此问题已经和 WPF 官方报告，详细请看 <https://github.com/dotnet/wpf/issues/10093>

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。