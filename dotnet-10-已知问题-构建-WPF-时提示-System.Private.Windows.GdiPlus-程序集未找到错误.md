
# dotnet 10 已知问题 构建 WPF 时提示 System.Private.Windows.GdiPlus 程序集未找到错误

本文记录 dotnet 10 的已知问题，将会导致 WPF 项目构建的时候给出错误

<!--more-->


<!-- CreateTime:2025/11/15 07:29:18 -->

<!-- 发布 -->
<!-- 博客 -->

此问题开始是在 <https://github.com/dotnet/wpf/issues/11246> 被报告的

此问题已修复，详细请看 <https://github.com/dotnet/dotnet/pull/3120>

本文写于 2025年11月15日 此时为 .NET 10 首个正式版本发布，此问题为首个正式版本带来的。此问题在首个正式版本发布之后才修复，需要等待下个版本 SDK 发布才能解决此问题

详细错误信息如下

```
error CS0012: The type 'IImage' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.Private.Windows.GdiPlus, Version=10.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089'
```

如果等不及 SDK 更新，可使用以下补丁方法：

```xml
    <!-- Workaround for https://github.com/dotnet/wpf/issues/11246 -->
    <Target Name="FixWpfReferences" AfterTargets="ResolveTargetingPackAssets" Condition="'$(UseWPF)' == 'true'">
        <ItemGroup>
            <SystemPrivateWindowsCoreRef Include="@(Reference)" Condition="'%(Filename)' == 'System.Private.Windows.Core'" />
            <ReferencePath Include="@(SystemPrivateWindowsCoreRef->'%(RootDir)%(Directory)System.Private.Windows.GdiPlus.dll')">
                <AssemblyName>System.Private.Windows.GdiPlus</AssemblyName>
            </ReferencePath>
        </ItemGroup>
    </Target>
```

等待下个 .NET 10 正式版本发布之后，以上补丁代码需要删掉




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。