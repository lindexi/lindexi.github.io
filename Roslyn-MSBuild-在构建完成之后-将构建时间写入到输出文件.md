
# Roslyn MSBuild 在构建完成之后 将构建时间写入到输出文件

我期望在每次构建完成之后，创建一个文件，在这个文件里面写入是什么时间构建的。这个需求实现非常简单，只需要使用 Target 在构建完成，使用 WriteLinesToFile 方法写入时间到输出文件即可

<!--more-->


<!-- CreateTime:2022/10/28 16:16:38 -->

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
2022/10/29 16:12:53 +08:00
```

如果提示 error MSB4185: 类型“System.DateTimeOffset”上的函数“get_Now”无法作为 MSBuild 属性函数执行 那么可以将 DateTimeOffset 降级使用 DateTime 代替。或者更新 Visual Studio 到最新版本

更改之后的代码如下

```xml
  <Target Name="WriteBuildInfoTarget" AfterTargets="Build">
    <WriteLinesToFile File="$(OutputPath)\BuildTime.txt" Lines="$([System.DateTime]::get_Now().ToString())" Overwrite="true"/>
  </Target>
```

使用 DateTimeOffset 在这里会比 DateTime 更优，因为 DateTimeOffset 能写入时区

如果担心遇到语言文化的问题，也就在其他的国家里面的对日期的格式化和中国不同，导致行为的不同。可以强行给定格式，例如对应的这样写


```xml
  <Target Name="WriteBuildInfoTarget" AfterTargets="AfterBuild">
    <!--
        编译完成之后，自动生成 BuildTime.txt 文件到输出文件夹，用来给业务端判断构建时间，决定某些功能的执行。例如埋点模块，判断距离实际运行时间太过久远，就不上报信息，减少流量占用
        放在 Startup 程序集，可以减少在其他程序集构建导致增量构建失效
    -->
    <WriteLinesToFile File="$(OutputPath)\BuildTime.txt" Lines="$([System.DateTime]::get_Now().ToString('yyyy-MM-dd HH:mm:ss,fff'))" Overwrite="true"/>
  </Target>
```

以上输出的就是格式如 `2022-10-20 10:56:02,123` 的字符串

在业务代码里面，可以使用如下代码进行解析

```csharp
var buildTime = DateTime.ParseExact(text, "yyyy-MM-dd HH:mm:ss,fff", null);
```

采用此方法可以解决多语言文化的问题，也就是此应用放在其他国家的机器上运行，也可以符合预期使用

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




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。