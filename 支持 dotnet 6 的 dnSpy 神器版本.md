# 支持 dotnet 6 的 dnSpy 神器版本

官方的 dnSpy 在 2021 时，由于某些吃瓜的原因 wtfsck 将 dnSpy 给 Archived 掉，在大佬被哄好之前，预计是不再更新。最新官方版本对 dotnet 6 的支持较弱，对于很多 dotnet 6 应用都无法成功调试，附加调试上去将会让应用卡住。好在 dnSpy 是开源的，也刚好 lsj 大佬改得动，于是改了一个支持 dotnet 6 的版本

<!--more-->
<!-- CreateTime:2022/5/11 19:57:30 -->

<!-- 标签：dnSpy -->
<!-- 发布 -->


什么是 dnSpy 神器？请看 [神器如 dnSpy，无需源码也能修改 .NET 程序 - walterlv](https://blog.walterlv.com/post/edit-and-recompile-assembly-using-dnspy.html )

我现在使用最多的调试工具，除了 VisualStudio 之外，就是 dnSpy 工具了。使用 dnSpy 可以让我方便调试用户端的应用。在完成了将团队里面最大的项目升级到 dotnet 6 时，就在升级过程遇到了一些问题，想要调试，却发现没有合适的工具，详细请看 [dotnet 6 在 Win7 系统证书链错误导致 HttpWebRequest 内存泄露](https://blog.lindexi.com/post/dotnet-6-%E5%9C%A8-Win7-%E7%B3%BB%E7%BB%9F%E8%AF%81%E4%B9%A6%E9%93%BE%E9%94%99%E8%AF%AF%E5%AF%BC%E8%87%B4-HttpWebRequest-%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2.html )

为了让我减少加班，我请了 [lsj](https://blog.sdlsj.net) 帮忙改改 dnSpy 神器，让 dnSpy 可以调试 dotnet 6 的应用

这是支持 dotnet 6 版本的 dnSpy 神器下载地址，也是修改之后开源的地址： [https://github.com/kkwpsv/dnSpy/releases/tag/6.1.9](https://github.com/kkwpsv/dnSpy/releases/tag/6.1.9)

如果大家下载不了，可以发邮件让我用其他方式发给你

那改好了是不是就完成了？还有一个问题是为什么 dnSpy 对 dotnet 6 的支持较弱呢？有以下几个原因，对应的修复方法还请看 [lsj](https://blog.sdlsj.net) 的改动 [https://github.com/kkwpsv/dnSpy/commit/a217b257453147c5d9db45070f7555f6395329bf](https://github.com/kkwpsv/dnSpy/commit/a217b257453147c5d9db45070f7555f6395329bf)

如 [[DAC][DBI] ICorDebugModule::GetMetaDataInterface fails in net6.0 for "Anonymously Hosted DynamicMethods Assembly" in unit test project. · Issue #62977 · dotnet/runtime](https://github.com/dotnet/runtime/issues/62977 ) 所说的原因，由于 `"Anonymously Hosted DynamicMethods Assembly"` 没有定义 IMetaDataImport2 接口，因此在 [https://github.com/dnSpy/dnSpy/blob/2b6dcfaf602fb8ca6462b8b6237fdfc0c74ad994/Extensions/dnSpy.Debugger/dnSpy.Debugger.DotNet.CorDebug/Impl/ModuleCreator.cs#L94-L96](https://github.com/dnSpy/dnSpy/blob/2b6dcfaf602fb8ca6462b8b6237fdfc0c74ad994/Extensions/dnSpy.Debugger/dnSpy.Debugger.DotNet.CorDebug/Impl/ModuleCreator.cs#L94-L96) 将拿到空，如以下代码，将抛出 InvalidOperationException 错误

```csharp
            var comMetadata = dnModule.CorModule.GetMetaDataInterface<IMetaDataImport2>();
            if (comMetadata is null)
                throw new InvalidOperationException();
```

修复的方式就是不抛出异常，而是自己定义一个 继承 DmdLazyMetadataBytes 类型的 DmdLazyMetadataBytesNull 类型，进行返回，如以下代码

```csharp
            if (comMetadata is null)
                // "Anonymously Hosted DynamicMethods Assembly" not implement IMetaDataImport2, we just return DmdLazyMetadataBytesNull
                return () => new DmdLazyMetadataBytesNull();
```

同时在 DmdAppDomainImpl.cs 里面，返回 DmdNullMetadataReader 即可，如此也许会影响读取程序集的信息，但好过无法调试

这个 `"Anonymously Hosted DynamicMethods Assembly"` 没有定义 IMetaDataImport2 接口，也影响 [https://github.com/dnSpy/dnSpy/blob/2b6dcfaf602fb8ca6462b8b6237fdfc0c74ad994/Extensions/dnSpy.Debugger/dnSpy.Debugger.DotNet.CorDebug/dndbg/Engine/CorAssembly.cs#L56-L60](https://github.com/dnSpy/dnSpy/blob/2b6dcfaf602fb8ca6462b8b6237fdfc0c74ad994/Extensions/dnSpy.Debugger/dnSpy.Debugger.DotNet.CorDebug/dndbg/Engine/CorAssembly.cs#L56-L60) 的代码，如以下代码，拿到的 ManifestModule 是空值。好在这里只是在 dnSpy 应用的 Debug 模式才会炸掉

```csharp
                var module = ManifestModule;
                Debug2.Assert(module is not null);
```

修复的方法只是将 Assert 的代码干掉即可

经过以上更改，就可以让 dnSpy 支持 dotnet 6 的调试

嗯，现在我觉得 dnSpy 作者做的还是太对了，我似乎越来越觉得开源也是一个不对的事情，那就是 SB 太多了。本来今天是想着将 dnSpy 在堆栈网上广告一下，造福一下国外的小伙伴。可惜遇到了傻比，最后我将问题和回答都删除了。这个事让我更加理解了很多开源作者最后选择了闭源的做法，以及让我更加佩服维护社区的大佬们。有人问，原作者关闭了 dnSpy 的瓜是什么？其中一个瓜就是被傻子给气的，开源了不收钱了，反而被当成理所当然，还要这要那，不满足了还被骂。具体的瓜还请自己去找了

我认为所有的参与开源和分享知识的行为都理应受到表扬称赞，共同维护一个良好的开发者生态

十分感谢 [wtfsck](https://github.com/wtfsck) 大佬提供的 dnSpy 神器，和 [lsj](https://blog.sdlsj.net) 让 dnSpy 支持 dotnet 6 调试

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

