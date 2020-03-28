# dotnet 将C#编译为wasm让前端html使用

其实 dotnet 是全栈的首选，原因是因为可以开发的方向太多，比如大本营PC端，以及后台。还有移动端，包括 IOS 和安卓端。现在还能用来写前端，本文就来告诉大家如何在前端使用现有的C#代码，通过 WebAssembly 使用 C# 的代码支持完全静态的网页，也就是不需要任何后台的存在。同时使用 C# 编写的 WebAssembly 可以省去 js 编译时间，同时使用二进制的本地指令，运行效率也有极大的提升。兼顾了开发的友好以及更高的性能

<!--more-->
<!-- CreateTime:2020/1/9 20:25:34 -->

<!-- 发布 -->

这需要搜 WebAssembly 就可以找到超级多的赞扬的文章，我这里也就不需要多说了。接下来告诉大家使用一个超级简单的代码入门

使用 WebAssmebly 的方式不会影响原有的任何业务，也就是我在已经写了几年的页面里面，可以直接加入 WebAssmembly 的特性，就像多添加一个 js 引用一样。不需要对现有的页面做任何的改动

此时在 C# 里面用的代码都是虚的，不再本文关注的范围内，所以通过 `dotnet new console -o YadernawcoLofeleabe` 创建一个控制台项目

在控制台项目添加一个类，这个类添加静态方法，这个静态方法就是让前端调用的入口方法，给这个字符串添加字符串参数，方便传入

```csharp
using System;

namespace YadernawcoLofeleabe
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }

    public class Example
    {
    	public static string Hello(string yourName)
    	{
    		return $"Hello {yourName}";
    	}
    }
} 
``` 

这里的代码不是重点，大概就是从 Hello 拿到输入，然后修改输入然后输出

接下来就是重点了，如何将 C# 代码编译为 WebAssmebly 了

这里的 C# 需要通过 mono 的辅助用于将 IL 转换为 WebAssembly 的代码，所以需要在[Mono](https://www.mono-project.com/download/stable/ )官网下载最新的 Mono 的 SDK 安装

[点击下载](https://www.mono-project.com/download/stable/)

默认的 Mono 将会安装到 `c:\Program Files\Mono\bin\` 文件夹，如果是下载 x86 的就会安装到 `c:\Program Files(x86)\Mono\bin\` 文件夹

然后下载 mono 在 wasm 的运行时，请 [点击下载](https://jenkins.mono-project.com/job/test-mono-mainline-wasm/label=ubuntu-1804-amd64/lastSuccessfulBuild/Azure/processDownloadRequest/4448/ubuntu-1804-amd64/sdks/wasm/mono-wasm-345550fb99a.zip) 将下载的 zip 文件夹解压缩到本地的文件夹，同时记住这个文件夹，如我将 zip 文件夹解压缩到 `f:/lindexi/mono` 文件夹

此时准备环境工作就完成了，下一步就是命令行编译了。当然这些步骤都是最基础的步骤，也有封装好的命令，也就是 `dotnet wasm xx.csproj` 完成编译，不过这一步需要先安装工具（注意这个工具还没正式发布）

通过 csc 命令将 C# 代码编译为 IL 文件。打开 VisualStudio 开发者命令行工具，进入到刚才创建的 Program.cs 所在文件夹

```
csc /target:library -out:Example.dll /noconfig /nostdlib /r:f:/lindexi/mono/wasm-bcl/wasm/mscorlib.dll /r:f:/lindexi/mono/wasm-bcl/wasm/System.dll /r:f:/lindexi/mono/wasm-bcl/wasm/System.Core.dll /r:f:/lindexi/mono/wasm-bcl/wasm/Facades/netstandard.dll /r:f:/lindexi/mono/wasm-bcl/wasm/System.Net.Http.dll /r:f:/lindexi/mono/framework/WebAssembly.Bindings.dll /r:f:/lindexi/mono/framework/WebAssembly.Net.Http.dll Program.cs
```

注意将 `f:/lindexi/mono` 文件夹替换为你刚才解压缩的 mono 运行时所在的文件夹

上面的代码通过引用 mono 运行时的库，将 Program.cs 文件编译为 Example.dll 文件

```
当然这里的 Example.dll 文件现在还是 IL 文件，还需要通过 mono 再次编译为 wasm 文件。注意这里说的编译为 wasm 并不是真的将 IL 编译 wasm 文件，而是编译为运行在 wasm 的 .NET 运行时可解析的文件。

上面这句话已经过时，只是我逗比看文档理解不对，其实上面这一步编译的 IL 文件已经可以在 wasm 执行了。原因是在 wasm 会先运行一个 .NET 的运行时，由 .NET 运行时执行这个 IL 文件
```

单独一个 Example.dll 文件是不能直接运行的，如上面所说，需要添加一个.NET运行时。但是一个 .NET 运行时是超级大的，难道要用户每次打开网页都下载一个这么大的运行时？此时就需要用到 `packager.exe` 工具，通过这个工具，可以只添加引用的同时支持在 wasm 运行的库

```
"c:\Program Files\Mono\bin\mono" "f:/lindexi/mono/packager.exe" --copy=always --out=./publish Example.dll
```

注意上面的路径，如果安装的是 x86 的 mono 那么需要修改路径为 `c:\Program Files(x86)\Mono\bin\mono` 此外需要将 `f:/lindexi/mono/packager.exe` 替换为你解压缩的 mono 运行时文件夹

执行上面命令如果看到下面输出，那么就是运行成功

```csharp
cp: Always - f:\temp\WpfApp1\YadernawcoLofeleabe\Example.dll -> ./publish\managed\Example.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\mscorlib.dll -> ./publish\managed\mscorlib.dll
cp: Always - f:\lindexi\mono\framework\WebAssembly.Bindings.dll -> ./publish\managed\WebAssembly.Bindings.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\Facades\netstandard.dll -> ./publish\managed\netstandard.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.dll -> ./publish\managed\System.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\Mono.Security.dll -> ./publish\managed\Mono.Security.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Xml.dll -> ./publish\managed\System.Xml.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Numerics.dll -> ./publish\managed\System.Numerics.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Core.dll -> ./publish\managed\System.Core.dll
cp: Always - f:\lindexi\mono\framework\WebAssembly.Net.WebSockets.dll -> ./publish\managed\WebAssembly.Net.WebSockets.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\Facades\System.Memory.dll -> ./publish\managed\System.Memory.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Data.dll -> ./publish\managed\System.Data.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Transactions.dll -> ./publish\managed\System.Transactions.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Data.DataSetExtensions.dll -> ./publish\managed\System.Data.DataSetExtensions.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\Facades\System.Drawing.Common.dll -> ./publish\managed\System.Drawing.Common.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.IO.Compression.dll -> ./publish\managed\System.IO.Compression.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.IO.Compression.FileSystem.dll -> ./publish\managed\System.IO.Compression.FileSystem.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.ComponentModel.Composition.dll -> ./publish\managed\System.ComponentModel.Composition.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Net.Http.dll -> ./publish\managed\System.Net.Http.dll
cp: Always - f:\lindexi\mono\framework\WebAssembly.Net.Http.dll -> ./publish\managed\WebAssembly.Net.Http.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Runtime.Serialization.dll -> ./publish\managed\System.Runtime.Serialization.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.ServiceModel.Internals.dll -> ./publish\managed\System.ServiceModel.Internals.dll
cp: Always - f:\lindexi\mono\wasm-bcl\wasm\System.Xml.Linq.dll -> ./publish\managed\System.Xml.Linq.dll
```

此时打开 Program.cs 所在的文件夹，可以看到文件夹包含了 publish 文件夹，这个文件夹里面的内容就是 wasm 使用的文件了，而刚才编译的 Example.dll 就放在 managed 文件夹里面

下一步就是如何在 html 中使用刚才编译出来的 Excample.dll 文件了，这部分感谢前端的小智的协助

需要在 html 中引用 publish 文件夹下的 `mono-config.js` 和 `runtime.js` 和 `dotnet.js` 文件夹

```html
	<script type="text/javascript" src="./mono-config.js"></script>
	<script type="text/javascript" src="./runtime.js"></script>
	<script async type="text/javascript" src="./dotnet.js"></script>	
```

接下来就是如何在 js 代码调用 C# 编译的 dll 了

通过 `Module.mono_bind_static_method` 可以将 js 的一个方法绑定到一个静态的方法里面

```
Module.mono_bind_static_method("[Example] YadernawcoLofeleabe.Example:Hello");
```

使用格式是 `Module.mono_bind_static_method("[dll文件名] 命名空间.类名:静态方法");` 如上面代码

尝试复制下面代码放在 html 里面

```html
        <script type="text/javascript">
            let that = this;
		var App = 
		{
			onClick: function () 
			{
				that.output.value = "Please wait";
				
				that.output.value = that.execute("Ali");
			},

			init: function () 
			{
				that.execute = Module.mono_bind_static_method("[Example] YadernawcoLofeleabe.Example:Hello");
                
				that.output = document.getElementById("output");
				that.button = document.getElementById("button");

				that.button.disabled = false;
			}
		};		

        </script>
```

如果你的 dll 命名和命名空间和我不相同，那么请自己修改

接下来就是添加简单的界面了 

```html
<!DOCTYPE doctype html>
<html lang="en">
    <head>
        <!-- Required meta tags -->
        <meta charset="utf-8">
            <meta content="width=device-width, initial-scale=1, shrink-to-fit=no" name="viewport">
                <!-- Bootstrap CSS -->
                <link crossorigin="anonymous" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.0/css/bootstrap.min.css" integrity="sha384-SI27wrMjH3ZZ89r4o+fGIJtnzkAnFs3E4qz9DIYioCQ5l9Rd/7UAa8DHcaL8jkWt" rel="stylesheet">
                    <title>
                        Hello, Mono WASM!
                    </title>
                </link>
            </meta>
        </meta>
    </head>
    <body>
        <div class="container">
            <h1>
                Hello, world!
            </h1>
            <form>
                <div class="form-group">
                    <label for="output">
                        Output from C#:
                    </label>
                    <textarea class="form-control" id="output" rows="10">
                    </textarea>
                </div>
                <div class="form-group">
                    <button class="btn btn-primary" id="button" onclick="App.onClick" type="button">
                        Run WASM, Run!
                    </button>
                </div>
            </form>
        </div>
        <script type="text/javascript">
            let that = this;
		var App = 
		{
			onClick: function () 
			{
				that.output.value = "Please wait";
				
				that.output.value = that.execute("Ali");
			},

			init: function () 
			{
				that.execute = Module.mono_bind_static_method("[Example] YadernawcoLofeleabe.Example:Hello");
				that.output = document.getElementById("output");
				that.button = document.getElementById("button");

				that.button.disabled = false;
			}
		};		

		document.getElementById("button").addEventListener("click", App.onClick);
		document.body.addEventListener("load", App.init);
        </script>
        <script src="./mono-config.js" type="text/javascript">
        </script>
        <script src="./runtime.js" type="text/javascript">
        </script>
        <script async="" src="./dotnet.js" type="text/javascript">
        </script>
    </body>
</html>
```

尝试开启一个静态的 HTTP 服务器，然后在浏览器访问这个 html 文件，注意将 dll 文件设置用户可下载，这样就完成了。例子可以访问[https://0x414c49.github.io/wasm-example/index.html](https://0x414c49.github.io/wasm-example/index.html) 这里有所有的文件

其实我在入门翻了车，多谢下面大佬的博客，本文大部分代码都是抄下面博客

[Run C# natively in the browser through the web assembly via mono-wasm](https://itnext.io/run-c-natively-in-the-browser-through-the-web-assembly-via-mono-wasm-60f3d55dd05a )

看到这里小伙伴想到了什么？没错，微软 Blazor 就是用这个原理，用 C# 写前端

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
