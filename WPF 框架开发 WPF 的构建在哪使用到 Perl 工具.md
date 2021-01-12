# WPF 框架开发 WPF 的构建在哪使用到 Perl 工具

在构建 WPF 开源仓库的时候，需要先搭建 Perl 环境，此时大家是否想了解在 WPF 构建的哪里用到 Perl 工具

<!--more-->
<!-- 标签：WPF框架开发 -->
<!-- 发布 -->

在完全开源的 WPF 仓库里面，其实可以看到很多 Perl 的影子，大部分都是用来做构建的脚本，小部分是在做我也看不懂的逻辑

做构建脚本的包括了生成一些代码，如 GenerateAvTraceMessages 这个构建步骤，代码定义如下

```xml
<Project>
  <Target
    Name="GenerateAvTraceMessages"
    BeforeTargets="CoreCompile"
    Condition="'$(AvTraceMessages)'!=''"
    Inputs="$(AvTraceMessages)"
    Outputs="$(IntermediateOutputPath)AvTraceMessages.cs">
    <Error Condition="!Exists($(PerlCommand.Replace('&quot;','')))" Text="Perl is not found: PerlCommand=$(PerlCommand)" />
    <MakeDir Condition="!Exists('$(IntermediateOutputPath)')" Directories="$(IntermediateOutputPath)" />
    <Message Text="Generating $(IntermeidateOutputPath)AvTraceMessages.cs from $(AvTraceMessages)"/>
    <Exec
      Command="$(PerlCommand) $(MSBuildThisFileDirectory)AvTrace\GenTraceStrings.pl -o $(IntermediateOutputPath)AvTraceMessages.cs -i $(AvTraceMessages)"
      StandardErrorImportance="normal"
      Outputs="$(IntermediateOutputPath)AvTraceMessages.cs">
      <Output TaskParameter="Outputs" ItemName="Compile"/>
    </Exec>
  </Target>
</Project>
```

可以看到上面代码调用了 PerlCommand 来执行 GenTraceStrings.pl 文件

除此之外，在 PresentationFramework.csproj 文件里面也有使用 Perl 的代码

```xml
 <Exec Command="$(PerlCommand) template.pl %(CollectionTemplate.Pattern) %(CollectionTemplate.Template) &gt; $(IntermediateOutputPath)\%(CollectionTemplate.Identity)" Outputs="$(IntermediateOutputPath)\%(CollectionTemplate.Identity)" StandardOutputImportance="normal" />
```

在 PresentationUI.csproj 生成主题的代码也是用到 Perl 工具

```xml
  <Target Name="AfterBuild" Inputs="@(Page)" Outputs="$(OutputPath)sdk\PUI.%(Page.Filename)%(Page.Extension)">
    <Exec Command="$(PerlCommand) ThemeGenerator.pl %(Page.Identity) @(Page -> '$(IntermediateOutputPath)PUI.%(Filename)%(Extension)')" Condition="'%(Page.GenerateTheme)' == 'true'" StandardOutputImportance="normal" />
    <ItemGroup>
      <DataFile Condition="'%(Page.GenerateTheme)' == 'true' and Exists('$(IntermediateOutputPath)PUI.%(Page.Filename)%(Page.Extension)')" Include="$(IntermediateOutputPath)PUI.%(Page.Filename)%(Page.Extension)">
        <SubFolder>sdk</SubFolder>
      </DataFile>
    </ItemGroup>
  </Target>
```

上面代码使用 Perl 通过 ThemeGenerator.pl 生成主题

在 WindowsBase.csproj 里面同样也是用来生成代码

```xml
  <Target Name="GenerateSources" BeforeTargets="CoreCompile" Inputs="MS\Internal\IO\Packaging\PackageXmlNamespaces.txt;MS\Internal\IO\Packaging\PackageXmlStrings.txt;" Outputs="$(IntermediateOutputPath)\PackageXmlStringTable.cs">
    <Error Condition="!Exists($(PerlCommand.Replace('&quot;','')))" Text="Perl is not found: PerlCommand=$(PerlCommand)" />
    <MakeDir Condition="!Exists('$(IntermediateOutputPath)')" Directories="$(IntermediateOutputPath)" />
    <Exec Command="$(PerlCommand) $(GenXmlStringTable) -o $(IntermediateOutputPath)\PackageXmlStringTable.cs -n MS\Internal\IO\Packaging\PackageXmlNamespaces.txt -x MS\Internal\IO\Packaging\PackageXmlStrings.txt -e MS.Internal.IO.Packaging.PackageXmlEnum -c MS.Internal.IO.Packaging.PackageXmlStringTable" StandardOutputImportance="normal" Outputs="$(IntermediateOutputPath)\PackageXmlStringTable.cs">
      <Output TaskParameter="Outputs" ItemName="Compile" />
    </Exec>
  </Target>
```

也就是说 Perl 在 WPF 仓库最大的作用就是用来辅助生成代码，通过代码模版生成代码。大概就是古代没有一个好用的代码模版生成器，才使用了Perl工具，但是官方也没有计划去更改这部分的逻辑

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
