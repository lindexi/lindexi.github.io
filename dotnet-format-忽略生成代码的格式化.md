
# dotnet format 忽略生成代码的格式化

我给团队引入了自动格式化代码机器人，这个机器人有点傻，会将生成的代码也进行格式化，每次都会我的代码生成工具打架。为了让这两个机器人和好，我探索了让 dotnet format 忽略对生成代码进行自动格式化的方法

<!--more-->


<!-- 发布 -->

实现的方法很简单，只需要指定某个生成代码文件，或者存放代码文件的文件夹作为 `generated_code` 生成代码即可

在 `.editorconfig` 文件里，可以指定当前文件夹内的包括子文件夹内的文件的格式化规则，通过在 `.editorconfig` 文件里，设置某些文件或文件夹是被 `generated_code` 即可让 dotnet format 在进行格式化的时候，进行忽略

例如指定某个 MainPage.g.i.cs 文件作为生成文件，代码如下

```yml
# Remove the line below if you want to inherit .editorconfig settings from higher directories
root = true
# C# files
[MainPage.g.i.cs]
## All files should be considered generated code.
generated_code = true
```

或者是对文件夹设置此文件夹存放的是生成代码，在文件夹里面存放 `.editorconfig` 文件，内容如下

```yml
# Remove the line below if you want to inherit .editorconfig settings from higher directories
root = true
# C# files
[*.cs]
## All files should be considered generated code.
generated_code = true
```

更多请看 [Analyzer configuration - Visual Studio (Windows) Microsoft Docs](https://docs.microsoft.com/en-us/visualstudio/code-quality/use-roslyn-analyzers?view=vs-2022&WT.mc_id=WD-MVP-5003260 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。