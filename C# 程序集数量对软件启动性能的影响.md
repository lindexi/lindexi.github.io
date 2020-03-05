# C# 程序集数量对软件启动性能的影响

本文通过很多的数据测试分析在一个项目引用很多个外部项目和将外部项目的类合并到一个项目之间的启动性能的不同。

通过分析知道了如果一个项目引用了很多项目，而且在启动过程会全部调用这些项目，这时的软件性能会比将这些项目的代码合并到一个项目的慢很多

本文的数据为 [预编译框架，开发高性能应用 - 课程 - 微软技术暨生态大会 2018 - walterlv](https://walterlv.com/post/dotnet-build-and-roslyn-course-in-tech-summit-2018.html ) 提供

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->



<!-- 标签：C#，性能测试 -->

最近在做一个编译器相关的项目，这个[项目](https://github.com/dotnet-campus/SourceYard )是将多个库作为源代码的 nuget 包，这样就可以在开发的时候是使用多个不同的项目，避免项目之间耦合。编译的时候将多个项目编译为 一个 dll 提高了软件启动性能。而且通过源代码包的引用方式可以极大避免了在不同的平台迁移的难度，只要是代码兼容的，甚至代码部分不兼容可以使用宏的方式在不同的平台使用不同的代码。

为了告诉大家这个项目的用处，于是本文就使用代码创建的方式创建了很多代码，通过对比这些代码的运行可以知道将类分在多个项目，和将类放在一个项目在运行过程的性能

我通过创建两个不同的工程，第一个工程是包含一个项目，这个项目里有 5000 个空类，在启动之后会创建这 5000 个类中的 1000 个类。

第二个工程包括了 1000 个项目，每个项目有 5 个空类，这里的空类和前面项目的空类是一样的创建方法。然后再添加一个启动项目，这个启动项目引用了前面的 1000 个项目，在启动之后会创建 1000 个项目中每个项目的一个类，也就是创建了 1000 个类，只是每个类都在不同的项目。

经过了很长时间的编译，我运行了一个项目5000个类的项目，初次运行时间是 54 ms ，接下来两次运行时间分别是 52 ms 和 53 ms 时间很短。

然后运行 1000 个项目，一个项目 5 个类的项目，冷启动时间是 15246 毫秒，之后的运行时间如下

第二次 580 ms

第三次 552 ms

第四次 546 ms

第五次 563 ms

第六次 568 ms

第七次 569 ms

下面表格是对比两个工程运行时间

<!-- ![](image/C# 程序集数量对软件启动性能的影响/C# 程序集数量对软件启动性能的影响0.png) -->

![](http://image.acmx.xyz/lindexi%2F2018101710821441)


|           工程            | 冷启动   | 第二次 | 第三次 |
| :-----------------------: | -------- | ------ | ------ |
|     一个项目5000个类      | 54 ms    | 52 ms  | 53 ms  |
| 1000个项目，一个项目5个类 | 15246 ms | 580 ms | 552 ms |

从上面表格可以看到，冷启动的性能差是 280 倍，此后的运行的性能差大概是 10 倍

然后我还测试了 1000 个项目，一个项目 1000 个类的运行时间，冷启动 22993 毫秒，热启动三次的数据是 885 毫秒，871 毫秒和 861 毫秒


测试项目的代码可以从 csdn 下载，如果没有积分请发邮件给我。如果觉得上面的数据很诡异，请自己运行一下编译一下

[一个项目5000 个类-CSDN下载](https://download.csdn.net/download/lindexi_gd/10725642 )

[在 1000 个项目，一个项目里有 5 个类-CSDN下载](https://download.csdn.net/download/lindexi_gd/10725691 )

创建测试项目的代码请看下面

创建一个项目这个项目里有 5000 个类，在启动之后调用这 5000 个类里的 1000 个

{% raw %}

```csharp
        private static void LownearkeajooSasouStegisti()
        {
            var saryawpirmiGerekipoNehiti = new DirectoryInfo("MayairJowya");

            saryawpirmiGerekipoNehiti.Create();

            var fawniSorhaHereni = new List<string>();

            var deleeTacarirouWulall = new WhairchooHerdo();

            for (int gupoudigorKihirkercou = 0; gupoudigorKihirkercou < 1000; gupoudigorKihirkercou++)
            {
                var teaJawtu = deleeTacarirouWulall.LemgeDowbovou();

                for (int mirxarJeredrairsear = 0; mirxarJeredrairsear < 5; mirxarJeredrairsear++)
                {
                    var cicirRarsonisallJearwelxe = deleeTacarirouWulall.LemgeDowbovou();

                    var facoSaijeesereniXaimow = $@"
using System;
using System.Collections.Generic;
using System.Text;

namespace {teaJawtu}
{{
    public class {cicirRarsonisallJearwelxe}
    {{
        public string Foo {{ get; set; }}
    }}
}}";
                    if (mirxarJeredrairsear == 0)
                    {
                        fawniSorhaHereni.Add(teaJawtu + "." + cicirRarsonisallJearwelxe);
                    }

                    File.WriteAllText(
                        Path.Combine(saryawpirmiGerekipoNehiti.FullName, cicirRarsonisallJearwelxe + ".cs"),
                        facoSaijeesereniXaimow);
                }
            }

            var jawjearPalfokallPuwuTearbourer = new StringBuilder();

            jawjearPalfokallPuwuTearbourer.Append(@"<Project Sdk=""Microsoft.NET.Sdk"">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
");


            jawjearPalfokallPuwuTearbourer.Append(@"  </ItemGroup>

</Project>");

            File.WriteAllText(Path.Combine(saryawpirmiGerekipoNehiti.FullName, "TirkalltremceFalgawCouwabupu.csproj"),
                jawjearPalfokallPuwuTearbourer.ToString());

            jawjearPalfokallPuwuTearbourer.Clear();

            var cepepiSowneKorrer = @"using System;
using System.Diagnostics;

namespace CouwharjeMerball
{
    class Program
    {
        static void Main(string[] args)
        {
            var dafuWhayroubaXouma = new Stopwatch();
            dafuWhayroubaXouma.Start();
            var kawgeDeesearsofas = new KawgeDeesearsofas();
            kawgeDeesearsofas.LurtrajaboPearbubirXinene();
            dafuWhayroubaXouma.Stop();
            Console.WriteLine(dafuWhayroubaXouma.ElapsedMilliseconds);
        }
    }
}
";
            File.WriteAllText(Path.Combine(saryawpirmiGerekipoNehiti.FullName, "Program.cs"), cepepiSowneKorrer);

            jawjearPalfokallPuwuTearbourer.Append(@"namespace CouwharjeMerball
{
    class KawgeDeesearsofas
    {
        public void LurtrajaboPearbubirXinene()
        {
");


            foreach (var ferosarTadir in fawniSorhaHereni)
            {
                jawjearPalfokallPuwuTearbourer.Append("            new " + ferosarTadir + "();");
                jawjearPalfokallPuwuTearbourer.Append("\r\n");
            }

            jawjearPalfokallPuwuTearbourer.Append(@"        }
    }
}");

            File.WriteAllText(Path.Combine(saryawpirmiGerekipoNehiti.FullName, "KawgeDeesearsofas.cs"),
                jawjearPalfokallPuwuTearbourer.ToString());
        }

```

{% endraw %}

创建 1000 个项目，每个项目有 5 个类，在启动项目引用这 1000 个项目，在启动之后创建 1000 个类，这 1000 个类每个类都在不同的项目里

{% raw %}

```csharp
        private static void KijeSabacher()
        {
            var jisqeCorenerairTurpalhee = new DirectoryInfo("StuLartearou");

            jisqeCorenerairTurpalhee.Create();

            var jairtallworBeakoo = new WhairchooHerdo();

            List<string> geeberecereHouroudo = new List<string>();

            List<string> xawsosapawTabejetai = new List<string>();

            for (int qeltasmisVigallSearniste = 0; qeltasmisVigallSearniste < 1000; qeltasmisVigallSearniste++)
            {
                string louwebirPemtrasrereYorta = "";

                var fismeerurniDawwall = jairtallworBeakoo.LemgeDowbovou();

                var nemirchouDamounu = jisqeCorenerairTurpalhee.CreateSubdirectory(fismeerurniDawwall);

                var beltuzoKoma = @"<Project Sdk=""Microsoft.NET.Sdk"">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>

</Project>
";
                xawsosapawTabejetai.Add(fismeerurniDawwall);

                File.WriteAllText(Path.Combine(nemirchouDamounu.FullName, fismeerurniDawwall + ".csproj"), beltuzoKoma);

                for (int roupairDufallne = 0; roupairDufallne < 5; roupairDufallne++)
                {
                    var whowjallKelpirhorWirweSemjaneldroo = jairtallworBeakoo.LemgeDowbovou();

                    if (roupairDufallne == 0)
                    {
                        louwebirPemtrasrereYorta = fismeerurniDawwall + "." + whowjallKelpirhorWirweSemjaneldroo;
                    }

                    var facoSaijeesereniXaimow = $@"
using System;
using System.Collections.Generic;
using System.Text;

namespace {fismeerurniDawwall}
{{
    public class {whowjallKelpirhorWirweSemjaneldroo}
    {{
        public string Foo {{ get; set; }}
    }}
}}";

                    File.WriteAllText(
                        Path.Combine(nemirchouDamounu.FullName, whowjallKelpirhorWirweSemjaneldroo + ".cs"),
                        facoSaijeesereniXaimow);
                }

                geeberecereHouroudo.Add(louwebirPemtrasrereYorta);
            }

            var jawjearPalfokallPuwuTearbourer = new StringBuilder();


            var dirceDadaipaHowbistairneeQabijel = "CouwharjeMerball";
            var suleLougirwhe = jisqeCorenerairTurpalhee.CreateSubdirectory(dirceDadaipaHowbistairneeQabijel);

            jawjearPalfokallPuwuTearbourer.Append(@"<Project Sdk=""Microsoft.NET.Sdk"">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
");

            foreach (var ciraZeajanipou in xawsosapawTabejetai)
            {
                jawjearPalfokallPuwuTearbourer.Append(
                    $@"    <ProjectReference Include=""..\{ciraZeajanipou}\{ciraZeajanipou}.csproj"" />");
                jawjearPalfokallPuwuTearbourer.Append("\r\n");
            }

            jawjearPalfokallPuwuTearbourer.Append(@"  </ItemGroup>

</Project>");

            File.WriteAllText(Path.Combine(suleLougirwhe.FullName, dirceDadaipaHowbistairneeQabijel + ".csproj"),
                jawjearPalfokallPuwuTearbourer.ToString());

            jawjearPalfokallPuwuTearbourer.Clear();

            var cepepiSowneKorrer = @"using System;
using System.Diagnostics;

namespace CouwharjeMerball
{
    class Program
    {
        static void Main(string[] args)
        {
            var dafuWhayroubaXouma = new Stopwatch();
            dafuWhayroubaXouma.Start();
            var kawgeDeesearsofas = new KawgeDeesearsofas();
            kawgeDeesearsofas.LurtrajaboPearbubirXinene();
            dafuWhayroubaXouma.Stop();
            Console.WriteLine(dafuWhayroubaXouma.ElapsedMilliseconds);
        }
    }
}
";
            File.WriteAllText(Path.Combine(suleLougirwhe.FullName, "Program.cs"), cepepiSowneKorrer);

            jawjearPalfokallPuwuTearbourer.Append(@"namespace CouwharjeMerball
{
    class KawgeDeesearsofas
    {
        public void LurtrajaboPearbubirXinene()
        {
");


            foreach (var ferosarTadir in geeberecereHouroudo)
            {
                jawjearPalfokallPuwuTearbourer.Append("            new " + ferosarTadir + "();");
                jawjearPalfokallPuwuTearbourer.Append("\r\n");
            }

            jawjearPalfokallPuwuTearbourer.Append(@"        }
    }
}");

            File.WriteAllText(Path.Combine(suleLougirwhe.FullName, "KawgeDeesearsofas.cs"),
                jawjearPalfokallPuwuTearbourer.ToString());
        }

```

{% endraw %}

{% raw %}
{% endraw %}

参见 

[C# 程序内的类数量对程序启动的影响](https://lindexi.oschina.io/lindexi/post/C-%E7%A8%8B%E5%BA%8F%E5%86%85%E7%9A%84%E7%B1%BB%E6%95%B0%E9%87%8F%E5%AF%B9%E7%A8%8B%E5%BA%8F%E5%90%AF%E5%8A%A8%E7%9A%84%E5%BD%B1%E5%93%8D.html )

[C# 直接创建多个类和使用反射创建类的性能](https://lindexi.oschina.io/lindexi/post/C-%E7%9B%B4%E6%8E%A5%E5%88%9B%E5%BB%BA%E5%A4%9A%E4%B8%AA%E7%B1%BB%E5%92%8C%E4%BD%BF%E7%94%A8%E5%8F%8D%E5%B0%84%E5%88%9B%E5%BB%BA%E7%B1%BB%E7%9A%84%E6%80%A7%E8%83%BD.html )

[C# 性能分析 反射 VS 配置文件 VS 预编译](https://lindexi.oschina.io/lindexi/post/C-%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90-%E5%8F%8D%E5%B0%84-VS-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-VS-%E9%A2%84%E7%BC%96%E8%AF%91.html )

[预编译框架，开发高性能应用 - 课程 - 微软技术暨生态大会 2018 - walterlv](https://walterlv.com/post/dotnet-build-and-roslyn-course-in-tech-summit-2018.html ) 

[dotnet-campus/SourceYard: Add a NuGet package only for dll reference? By using dotnetCampus.SourceYard, you can pack a NuGet package with source code. By installing the new source code package, all source codes behaviors just like it is in your project.](https://github.com/dotnet-campus/SourceYard )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
