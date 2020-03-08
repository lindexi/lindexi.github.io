# Roslyn 让编译时候 Message 内容默认输出

默认无论是在 VisualStudio 还是在 dotnet 命令行都会忽略项目文件或安装库里面的消息输出，而吕水小伙伴给了一个馊主意将所有需要输出给用户的消息换为警告，因为默认是会输出警告，于是消息就可以输出了。然后某个小伙伴就过来打我，因为他一编译整个项目原本是没有警告的，现在有很多警告。于是我就在找是否有方法可以做到让消息的内容默认输出

<!--more-->
<!-- CreateTime:2019/7/1 14:16:59 -->

<!-- csdn -->

<!-- 标签：Roslyn,MSBuild,编译器 -->

在编译的时候需要期望输出所有的消息，可以添加输出的日志等级，详细请看[How to output the target message in dotnet build command line](https://blog.lindexi.com/post/how-to-output-the-target-message-in-dotnet-build-command-line )

但是我的小伙伴是在 VisualStudio 编译的，他不期望输出的内容太多，而我又期望给他一点输出，特别是注明他现在使用的这个库是基于 GLWTPL 协议做的

```csharp
GLWTPL 协议

When I wrote this, only God and I understood what I was doing.
Now, only God knows.

当我写下这段代码的时候，只有上帝和我知道这是什么
现在只有上帝知道

也就是现在代码除了上帝没有知道是做什么
```

如果我写成 Message 默认小伙伴是不会看到，但如果我写成 Warning 他又会打我，好在有消息有一个属性，请看代码

```csharp
  	<Message Importance="High" Text="林德熙是逗比" />
```

添加属性 Importance 指定这个消息是否重要，如果指定为重要，那么消息默认就会输出

请试试将下面代码放在你的 csproj 文件中，然后点击编译

```csharp
  <Target Name="Lindexi" BeforeTargets="Build">
    <Message Importance="High" Text="林德熙是逗比" />
  </Target>
```

上面代码如果写在了 NuGet 库，可以在小伙伴安装我的库的时候编译就输出消息内容

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
