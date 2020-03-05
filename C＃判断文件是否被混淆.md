# C＃判断文件是否被混淆

可以使用混淆工具对一个DLL 和 exe 进行混淆。

但是如何知道一个文件是否已经混淆了。

在发布之前，需要知道是不是有文件忘了混淆。


<!--more-->
<!-- CreateTime:2019/9/2 12:57:37 -->


<div id="toc"></div>

要判断文件是否混淆，必须知道常用的混淆手法。

混淆就是因为编写的 C# 代码转换 IL ，可以很容易被反编译，从而知道了源代码，不利于保护软件，不利于防止破解。

所以可以通过混淆来让反编译困难。

但是经过混淆，只可以让好多反编译新手无法破解，对于大神，还是没有作用。

但是本文不讨论这个，还是来说下，常用的混淆。

 - 混淆变量名

 - 混淆流程

常见的也是混淆变量名，这也是本文的检测方法，对于混淆流程，暂时还没有方法。

简单方法去获得文件是否混淆变量名是反射。

可以通过加载文件，使用[动态加载 DLL](http://lindexi.oschina.io/lindexi/post/C-%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD%E5%8D%B8%E8%BD%BD-DLL/)，然后使用反射判断文件变量名是否存在不可读字符，如果存在，那么文件被混淆。

代码：


```csharp
    class ApplicationProxy : MarshalByRefObject
    {
        private static readonly string[] ConfuseNameCharacteristics =
        {
            "\u0001",
            "\u0002",
            "\u0003",
            "\u0004",
            "\u0005",
            "\u0006",
            "\u0007",
            "\u0008",
            "\u0009",
            "\u0010",
            "\u0011",
            "\u0012",
            "\u0013",
            "\u0014",
            "\u0015",
            "\u0016",
            "\u0017",
            "\u0018",
            "\u0019"
        };

        /// <summary>
        ///     判断一个文件是否混淆
        /// </summary>
        /// <param name="file">文件是exe dll</param>
        /// <returns></returns>
        public ConfuseType CheckFileConfuse(FileInfo file)
        {
            try
            {
                var assembly = Assembly.LoadFile(file.FullName);
                var types = assembly.GetTypes();
                if (
                    types.Any(
                        type => ConfuseNameCharacteristics.Any(ch => type.FullName.Contains(ch) || PeeConfuseType(type))))
                    return ConfuseType.Confused;
                return ConfuseType.NotConfused;
            }
            catch (ReflectionTypeLoadException)
            {
                return ConfuseType.NotSupported;
            }
            catch (Exception e)
            {
                if (e is BadImageFormatException || e is FileLoadException)
                    return ConfuseType.NotSupported;
                throw;
            }
        }

        /// <summary>
        ///     判断type的方法是否有混淆
        /// </summary>
        /// <param name="type"></param>
        private bool PeeConfuseType(Type type)
        {
            return ConfuseNameCharacteristics.Any(temp => type.GetFields().Any(field => field.Name.Contains(temp))) ||
                   ConfuseNameCharacteristics.Any(
                       temp => type.GetProperties().Any(property => property.Name.Contains(temp))) ||
                   ConfuseNameCharacteristics.Any(temp => type.GetMethods().Any(method => method.Name.Contains(temp)));
        }
    }

```
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 