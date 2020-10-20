# dotnet tool 文件编码规范命令行工具

在咱项目里面，大家是否有关注过文件的编码，一个文件是作为 Ascii 编码保存的，还是作为 GBK 编码保存的，还是 UTF8 编码保存的？不同的编码是否对应用有影响？其实是会有影响的，在 C# 里面的字符串常量等都会受到文件编码的影响。例如我的应用主输出是 UTF8 编码，此时我在二进制可执行文件里面保存的输出字符串的二进制是采用 GBK 编码的内容，在一些用户的设备上也许就会显示出乱码

本文来安利大家一个 dotnet 工具，这个工具可以用来协助大家找到项目里面的编码不规范文件

<!--more-->
<!-- CreateTime:2020/10/17 17:40:13 -->

<!-- 发布 -->

本文告诉大家的这个工具的源代码在 GitHub 完全开源，这个开源仓库是一个古老的仓库，核心功能是提供给 VisualStudio 插件，用于扫整个项目里面的所有文件，尝试找到所有编码不规范的文件。而本文只是用到这个仓库里面的 dotnet tool 工具

仓库请看 [dotnet-campus/EncodingNormalior: 规范化文件编码。Make the file's encoding standard.](https://github.com/dotnet-campus/EncodingNormalior )

在使用之前，请使用下面代码安装或更新工具

```
dotnet tool install -g dotnetCampus.EncodingNormalior
```

安装完成工具之后，可以采用如下命令使用工具

```
EncodingNormalior -f E:\lindexi\EncodingNormalior
```

上面命令的 `E:\lindexi\EncodingNormalior` 就是需要扫编码规范的文件夹，使用这个命令可以将这个文件夹里面的所有文本文件扫一次，如果有文件不符合规范的，将会输出出来

如果大家期望使用这个工具自动转换文件编码，此时可以添加 `--TryFix true` 参数，如下面命令

```
EncodingNormalior -f E:\lindexi\EncodingNormalior --TryFix true
```

需要说明的是现在没有一个算法能够准确了解一个没有带编码BOM的文件的编码，因此加上了自动转换文件编码参数之后，工具将会按照自己认为的编码去读取文件，然后再次写入，也许会让文件乱码

更多关于这个命令行的使用方法，还请大家到开源仓库 [https://github.com/dotnet-campus/EncodingNormalior](https://github.com/dotnet-campus/EncodingNormalior )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
