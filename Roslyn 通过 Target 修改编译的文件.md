# Roslyn 通过 Target 修改编译的文件

本文告诉大家如何使用 Target 进行修改编译时的文件

<!--more-->
<!-- CreateTime:2018/8/29 9:10:46 -->

<!-- cdsn -->
<!-- 标签：Roslyn,MSBuild,编译器 -->

本文也是带着一个任务来开始。任务就是本渣是一个腹黑的开发者，想要在开发的过程替换一个文件，让开发者在编译出来的文件和他调用的文件是两个不同的文件。

所以下面让我告诉大家整个任务的过程，先创建一个项目

## 创建项目

这里推荐创建一个 dotnet core 控制台项目，因为创建这个项目就是使用 VisualStudio 2017 新的格式

如果创建的是其他的项目，请看我的博客转换为 VisualStudio 2017 新的格式


## 创建替换的文件

现在来创建两个不同的文件，其中一个文件是让开发者可以看到的文件，第二个文件是用来替换的实际的文件。

创建的文件是

 - Foo.cs

 - Foo 替换.cs

这里的两个文件需要隐藏`Foo 替换.cs`文件，隐藏的方法请看下面

打开项目文件，通过右击项目编辑

```csharp
  <ItemGroup>
    <Compile Remove="Foo 替换.cs" Visible="false"></Compile>
  </ItemGroup>
```

上面的代码的意思就是移除 `Compile` 的 `Foo 替换.cs` 文件，并且设置不可见

实际设置了移除就可以不设置不可见

## 类的属性

为了让大家可以看到两个类的不相同，所以我在两个类添加了不同的代码

```csharp
//Foo.cs
    class Foo
    {
        public string Name { get; set; } = "德熙是逗比";
    }
```

```csharp
//Foo 替换.cs
    class Foo
    {
        public string Name { get; set; } = "欢迎访问我博客 http://blog.csdn.net/lindexi_gd 里面有大量 UWP、WPF、dotnetcore 相关";
    }
```

这时在主函数调用一下

```csharp
        static void Main(string[] args)
        {
            var foo = new Foo();
            Console.WriteLine(foo.Name);

            while (true)
            {
                Console.Read();
            }
        }
```

## 编译时替换

如果运行了上面的代码，大家也知道会输出什么，但是腹黑的本渣就在编译的时候替换文件

先创建一个 Target 在项目文件，创建一个 Target 需要告诉 Target 的命名和触发的时间

```csharp
  <Target Name="DrumearDatroLanecereso" BeforeTargets="BeforeBuild">
   
  </Target>
```

这里的触发时间就是在编译之前，注意`BeforeBuild`是需要用户编译才会运行

现在知道了一个在用户编译之前的 Target 就可以在里面将两个文件替换

```csharp
  <Target Name="DrumearDatroLanecereso" BeforeTargets="BeforeBuild">
    <ItemGroup>
      <Compile Remove="Foo.cs" Visible="false"></Compile>
      <Compile Include="Foo 替换.cs" Visible="false"></Compile>
    </ItemGroup>
  </Target>
```

上面代码是移除 `Foo.cs` 引用另一个替换的文件，所以这时运行一下就会发现输出的是

```
欢迎访问我博客 http://blog.csdn.net/lindexi_gd 里面有大量 UWP、WPF、dotnetcore 相关
```

通过这个方法就可以在编译时替换文件，也就是给开发者看到很漂亮的代码，以为就是很简单的写法，然后在编译的时候就将很好看的代码替换为很乱的代码。

这个方法可以用来做 xaml 的资源引用方式，用来做混淆。还有哪些使用方法就需要大家在开发时去想。

<!-- TIM图片20180824091722.jpg -->
![](https://i.loli.net/2018/08/24/5b7f5cd130375.jpg)

更多关于 Roslyn 请看 [手把手教你写 Roslyn 修改编译](https://lindexi.oschina.io/lindexi/post/roslyn.html ) 

参见：[专栏：Roslyn 入门 - CSDN博客](https://blog.csdn.net/column/details/23159.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
