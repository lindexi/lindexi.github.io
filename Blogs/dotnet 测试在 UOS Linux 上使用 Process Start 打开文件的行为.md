本文记录我在 UOS Linux 系统上使用 Process.Start 打开文件的行为

<!--more-->


<!-- CreateTime:2024/1/23 19:01:05 -->

<!-- 发布 -->
<!-- 博客 -->

## 使用 UseShellExecute 打开文本文件

我放入了名为 Test.txt 的文件，然后使用下面代码尝试打开文件。实际测试可以正常打开

```csharp
using System.Diagnostics;

var filePath = "../Test.txt";

Console.WriteLine($"文件存在 {File.Exists(filePath)}");

Process.Start(new ProcessStartInfo(filePath)
{
    UseShellExecute = true
});
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/bdc7f6d136079f726aa04b10859149c36c91a940/ChoqonerekiFojijihel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bdc7f6d136079f726aa04b10859149c36c91a940/ChoqonerekiFojijihel) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bdc7f6d136079f726aa04b10859149c36c91a940
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bdc7f6d136079f726aa04b10859149c36c91a940
```

获取代码之后，进入 ChoqonerekiFojijihel 文件夹

## 使用 UseShellExecute 打开文件夹

只是将传入参数从文件换成文件夹，可以正常

```csharp
using System.Diagnostics;

var filePath = "../../Documents";

Console.WriteLine($"文件夹存在 {Directory.Exists(filePath)}");

Process.Start(new ProcessStartInfo(filePath)
{
    UseShellExecute = true
});
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/11ac184046099dd6e8e558794abf314b1649d869/ChoqonerekiFojijihel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/11ac184046099dd6e8e558794abf314b1649d869/ChoqonerekiFojijihel) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 11ac184046099dd6e8e558794abf314b1649d869
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 11ac184046099dd6e8e558794abf314b1649d869
```

获取代码之后，进入 ChoqonerekiFojijihel 文件夹

## 使用 UseShellExecute 打开 URL 到浏览器

测试代码如下

```csharp
using System.Diagnostics;

Process.Start(new ProcessStartInfo("http://www.baidu.com")
{
    UseShellExecute = true
});
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/77daf740bd87ff4f086a247d853bad93c6d872c1/ChoqonerekiFojijihel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/77daf740bd87ff4f086a247d853bad93c6d872c1/ChoqonerekiFojijihel) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 77daf740bd87ff4f086a247d853bad93c6d872c1
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 77daf740bd87ff4f086a247d853bad93c6d872c1
```

获取代码之后，进入 ChoqonerekiFojijihel 文件夹

## 使用 xdg-open 打开文件夹

平替 Windows 下的使用 explorer 打开文件夹的方法，代码如下

```csharp
using System.Diagnostics;

var filePath = "../../Documents";

Console.WriteLine($"文件夹存在 {Directory.Exists(filePath)}");

Process.Start(new ProcessStartInfo("xdg-open", new []{ filePath })
{
    UseShellExecute = false
});
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/75b936382dc5e4eb0a06b6460df3f529b46efd86/ChoqonerekiFojijihel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/75b936382dc5e4eb0a06b6460df3f529b46efd86/ChoqonerekiFojijihel) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 75b936382dc5e4eb0a06b6460df3f529b46efd86
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 75b936382dc5e4eb0a06b6460df3f529b46efd86
```

获取代码之后，进入 ChoqonerekiFojijihel 文件夹

## 使用 xdg-open 打开文件

以下继续使用文本文件作为例子，代码如下

```csharp
using System.Diagnostics;

var filePath = "../Test.txt";

Console.WriteLine($"文件夹存在 {Directory.Exists(filePath)}");

Process.Start(new ProcessStartInfo("xdg-open", new []{ filePath })
{
    UseShellExecute = false
});
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/cc11838fe0b21a6ab4d4d5142fee15c33ab5fd96/ChoqonerekiFojijihel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/cc11838fe0b21a6ab4d4d5142fee15c33ab5fd96/ChoqonerekiFojijihel) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cc11838fe0b21a6ab4d4d5142fee15c33ab5fd96
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cc11838fe0b21a6ab4d4d5142fee15c33ab5fd96
```

获取代码之后，进入 ChoqonerekiFojijihel 文件夹

## 使用 xdg-open 打开 URL 地址

测试代码如下

```csharp
using System.Diagnostics;

Process.Start(new ProcessStartInfo("xdg-open", new[] { "http://blog.lindexi.com" })
{
    UseShellExecute = false
});
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/6d4a87d1734682bb1fb1e2202413c4758e42f44d/ChoqonerekiFojijihel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6d4a87d1734682bb1fb1e2202413c4758e42f44d/ChoqonerekiFojijihel) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6d4a87d1734682bb1fb1e2202413c4758e42f44d
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6d4a87d1734682bb1fb1e2202413c4758e42f44d
```

获取代码之后，进入 ChoqonerekiFojijihel 文件夹

## 更多阅读

其他关于测试在 Linux 的行为，请参阅

- [dotnet 测试在 Linux 系统上的 Environment.GetFolderPath 行为](https://blog.lindexi.com/post/dotnet-%E6%B5%8B%E8%AF%95%E5%9C%A8-Linux-%E7%B3%BB%E7%BB%9F%E4%B8%8A%E7%9A%84-Environment.GetFolderPath-%E8%A1%8C%E4%B8%BA.html )

更多 dotnet 在 Linux 的开发博客请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

更多关于 xdg-open 请参阅 [Ubuntu Manpage: xdg-open - opens a file or URL in the user's preferred application](https://manpages.ubuntu.com/manpages/focal/en/man1/xdg-open.1.html )
