
# dotnet OpenXML 从文档生成创建文档的代码的库

本文和大家介绍 Serialize.OpenXml.CodeGen 这个支持从某个文档生成用于创建出这个文档的 C# 或 VB 代码的库。作用就是可以让小伙伴在拿到一份模版文件之后，可以通过 Serialize.OpenXml.CodeGen 生成能创建出这份文档的 C# 或 VB 的代码，用于在这份代码上面更改功能，做到创建定制 Docx 或 PPTX 或 Xlsx 文档的功能

<!--more-->


<!-- 发布  -->

这是一个完全开源的库，代码放在 [https://github.com/rmboggs/Serialize.OpenXml.CodeGen](https://github.com/rmboggs/Serialize.OpenXml.CodeGen) 欢迎小伙伴访问

这个库的功能就是从 Office 文档里面，如 Excel 文档里面读取文档内容，生成 CodeCompileUnit 对象，通过 CodeCompileUnit 对象可以序列化为 C# 或 VB 代码

如从一个 xlsx 文件创建对应的 C# 代码，可以使用下面代码

假定在 C 盘有一个 Temp 文件夹，这个文件夹里面有一个 Sample1.xlsx 文件，在调用下面代码之后，将会创建了 Sample1.cs 文件

```csharp
       static void Main(string[] args)
        {
            var sourceFile = new FileInfo(@"C:\Temp\Sample1.xlsx");
            var targetFile = new FileInfo(@"C:\Temp\Sample1.cs");

            using (var source = sourceFile.Open(FileMode.Open, FileAccess.Read, FileShare.Read))
            {
                using (var xlsx = SpreadsheetDocument.Open(source, false))
                {
                    if (xlsx != null)
                    {
                        var codeString = new StringBuilder();
                        var cs = new CSharpCodeProvider();

                        // This will build the CodeCompileUnit object containing all of
                        // the commands that would create the source code to rebuild Sample1.xlsx
                        var code = xlsx.GenerateSourceCode();

                        // This will convert the CodeCompileUnit into C# source code
                        using (var sw = new StringWriter(codeString))
                        {
                            cs.GenerateCodeFromCompileUnit(code, sw, Cgo);
                        }

                        // Save the source code to the target file
                        using (var target = targetFile.Open(FileMode.Create, FileAccess.ReadWrite))
                        {
                            using (var tw = new StreamWriter(target))
                            {
                                tw.Write(codeString.ToString().Trim());
                            }
                        }
                    }
                }
            }

            Console.ReadKey();
        }
```

上面代码的核心是 `var code = xlsx.GenerateSourceCode();` 可以将 Word 和 PPT 和 Excel 文档转换 CodeCompileUnit 对象，而 CodeCompileUnit 对象可以序列化为 C# 或 VB 代码

上面代码将创建 Sample1.cs 代码，这个代码可以通过 CreatePackage 方法向一个 Stream 里面写入 Sample1.xlsx 文档内容，而写入的方法是通过代码的形式，因此可以通过修改 Sample1.cs 代码定制写入的内容

例如我给的 Sample1.xlsx 只是一个模版，里面有很多内容可以替换，此时就可以修改 Sample1.cs 的代码，将可以替换的逻辑替换为自己的逻辑



本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/c615c9b6421e459f0409f01409f4b22689aa7d85/HofahereheebeaHenikowhuhinem)欢迎小伙伴访问






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。