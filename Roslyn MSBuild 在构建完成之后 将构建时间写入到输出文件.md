# Roslyn MSBuild 在构建完成之后 将构建时间写入到输出文件

我期望在每次构建完成之后，创建一个文件，在这个文件里面写入是什么时间构建的。这个需求实现非常简单，只需要使用 Target 在构建完成，使用 WriteLinesToFile 方法写入时间到输出文件即可

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

先写一个 Target 设置在 Build 之后执行

```xml
  <Target Name="WriteBuildInfoTarget" AfterTargets="Build">
  </Target>
```

在 Target 里面执行 WriteLinesToFile 将当前时间写入到文件。例如写入到输出文件夹的 BuildTime.txt 里面

```xml
  <Target Name="WriteBuildInfoTarget" AfterTargets="Build">
    <WriteLinesToFile File="$(OutputPath)\BuildTime.txt" Lines="$([System.DateTimeOffset]::get_Now().ToString())" Overwrite="true"/>
  </Target>
```

尝试构建一下项目，可以在输出文件夹找到 BuildTime.txt 文件，在这个文件里面可以看到当前的构建时间，大概内容如下

```
2022/10/28 16:12:53 +08:00
```

如果提示 error MSB4185: 类型“System.DateTimeOffset”上的函数“get_Now”无法作为 MSBuild 属性函数执行 那么可以将 DateTimeOffset 降级使用 DateTime 代替。或者更新 Visual Studio 到最新版本

更改之后的代码如下

```xml
  <Target Name="WriteBuildInfoTarget" AfterTargets="Build">
    <WriteLinesToFile File="$(OutputPath)\BuildTime.txt" Lines="$([System.DateTime]::get_Now().ToString())" Overwrite="true"/>
  </Target>
```

使用 DateTimeOffset 在这里会比 DateTime 更优，因为 DateTimeOffset 能写入时区

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/2d9b071c62ae6d47c0b88d4b5abccb1bf4b60778/YearkelbeneaqeahaicoChurinocechu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2d9b071c62ae6d47c0b88d4b5abccb1bf4b60778/YearkelbeneaqeahaicoChurinocechu) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2d9b071c62ae6d47c0b88d4b5abccb1bf4b60778
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2d9b071c62ae6d47c0b88d4b5abccb1bf4b60778
```

获取代码之后，进入 YearkelbeneaqeahaicoChurinocechu 文件夹

更多关于 WriteLinesToFile 的使用，请参阅 [Roslyn 使用 WriteLinesToFile 解决参数过长无法传入](https://lindexi.gitee.io/post/Roslyn-%E4%BD%BF%E7%94%A8-WriteLinesToFile-%E8%A7%A3%E5%86%B3%E5%8F%82%E6%95%B0%E8%BF%87%E9%95%BF%E6%97%A0%E6%B3%95%E4%BC%A0%E5%85%A5.html )

更多编译相关请看[手把手教你写 Roslyn 修改编译](https://blog.lindexi.com/post/roslyn.html )
