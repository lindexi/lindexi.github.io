# VisualStudio 合并代码文件

如果有相同的类，一般可以使用 partial 让他写在多个文件，那么如何把多个文件合并？请看 MainWindow.xaml 和 MainWindow.xaml.cs 其中 代码文件被折叠，那么如何做代码的折叠


<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


简单的方法，使用 SublimeText 打开 工程文件，当然 如果使用 VSC 也是可以，但是因为我用了 SublimeText 比较顺，所以就推荐使用。我收藏很多的工具，大家可以在我的博客看到。

工程文件就是创建一个工程自动生成的，如果创建一个空白的程序，那么就可以从创建的文件夹看到这个文件

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017722155815.jpg)

使用 SublimeText打开，可以看到他里面包含了很多文件

```csharp
    <Compile Include="MainWindow.xaml.cs">
      <DependentUpon>MainWindow.xaml</DependentUpon>
      <SubType>Code</SubType>
    </Compile>
```

那么这时如果创建一个类，请看下面的代码

```csharp
    public partial class Kapen
    {
        public int Leroy { get; set; }
    }

    public partial class Kapen
    {
        public Kapen()
        {
        }

        public string a { get; set; }
    }
```

把上面的代码放到两个文件，那么可以看到在解决方案存在两个文件，接下来就要把他们放在一起

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201772216156.jpg)

可以看到在刚才打开的工程文件，有这样的代码

```csharp
    <Compile Include="Kapen.cs" >
      
    </Compile>
    <Compile Include="Kapen1.cs" >
      
    </Compile>
```

需要把他在需要被折的文件添加折他的文件，请看代码

```csharp
    <Compile Include="Kapen.cs" >  这个文件就是折下面的文件
      
    </Compile>
    <Compile Include="Kapen1.cs" >
      <DependentUpon>Kapen.cs</DependentUpon> 添加折他的文件
      <SubType>Code</SubType> 
    </Compile>
```

方法的缺点

这个方法是有缺点的，可能有些小白就看到一个文件，不会展开，然后就不知道还别的文件。

这个方法不会让反射无法获取到属性，我在界面添加了一个 按钮，点击时就获得所有的类，可以获得写在两个文件的所有属性，所以这样写是可以的。

```csharp
           Assembly assembly = Assembly.GetExecutingAssembly();
            foreach (var temp in assembly.GetTypes().Where(temp => temp == typeof(Kapen)))
            {
                string str = temp.GetMembers().Select(s => s.Name).Aggregate("", (c, s) => c + s + " ");

            }
```

输出 s 就可以获得所有的属性，写在多个文件在编译 vs 合并，所以看起来是多个，实际还是一个文件。

如果想获得反射的知识，请看
[win10 uwp 反射](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E5%8F%8D%E5%B0%84/)

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20177221639.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
