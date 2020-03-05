# C# 代码占用的空间

是不是代码会占用空间，如果一个程序初始化需要 100M 的代码，那么在他初始化之后，这些代码就没有作用了，他会不会占空间？本文经过测试发现，代码也是会占空间。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


我写了2k个垃圾类代码，然后把他放在一个项目 BhgpsWnb，使用另一个项目去引用他。是不是觉得软件在运行的时候就需要很多的内存来放代码？

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171212144236.jpg)

引用垃圾程序的项目是 ReKlnma ，先只是在引用添加项目引用，然后在不使用 BhgpsWnb 这个项目的代码，我运行下面的代码

```csharp
        static void Main(string[] args)
        {
            Console.ReadKey();
        }
```

占用 7 M 内存，而如果运行了存在 2k 垃圾代码 BhgpsWnb 程序，就需要 8M 。使用方法是创建一个类，这个类就是垃圾代码里面的一个，这样就需要把dll放在内存。

```csharp
        static void Main(string[] args)
        {
            var ablkekbuuimc = new Ablkekbuuimc();
            ablkekbuuimc.Aaxfyerenjmfe(2);
            Console.ReadKey();
        }
```

如果觉得因为创建一个类需要的内存太大，那么我使用下面的代码，只是拿到一个类型，但是需要的内存是 8M 因为程序会把另一个程序加载

```csharp
        static void Main(string[] args)
        {
            Type t = typeof(Ablkekbuuimc);
            Console.ReadKey();
        }

```

下面来换个方式写，取消对垃圾程序的直接引用。使用 Load 方法去加载，可以看到垃圾程序 BhgpsWnb 有 8M ，一般的库可没有那么大。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017121214470.jpg)

```csharp
        static void Main(string[] args)
        {
            var file = new FileInfo("BhgpsWnb.exe");
            Assembly.LoadFile(file.FullName);
            Console.ReadKey();
        }
```

结果使用内存需要 8M 多，所以代码也是需要内存的，一旦加载了就不会从程序集卸载。

如果是加载程序集，那么加载程序集就需要很多的内存，即使卸载程序也没有用

```csharp
        static void Main(string[] args)
        {
            var app = Load();

            GC.Collect();
            GC.WaitForFullGCComplete();
            Console.ReadKey();

            AppDomain.Unload(app);

            Console.ReadKey();
            GC.Collect();
            GC.WaitForFullGCComplete();
        }

        private static AppDomain Load()
        {
            var file = new FileInfo("BhgpsWnb.exe");
            var otherAssemblyBytes = File.ReadAllBytes(file.FullName);
            var app = AppDomain.CreateDomain("BhgpsWnb");

            app.Load(otherAssemblyBytes);

            return app;
        }
```

上面的代码使用了 Load 就需要 20M的内存，在后面使用 Unload 之后实际上内存也没有减少，所以建议不要使用程序集加载方式，这个方式使用很多内存。

可以通过指定名称加载，可以看到下面的代码需要使用内存比较小，需要 9M ，但是 Unload 之后没有减少内存

```csharp
        static void Main(string[] args)
        {
            var app = Load();

            GC.Collect();
            GC.WaitForFullGCComplete();
            Console.ReadKey();

            AppDomain.Unload(app);
            GC.Collect();
            GC.WaitForFullGCComplete();
            Console.ReadKey();
        }

        private static AppDomain Load()
        {
            var file = new FileInfo("BhgpsWnb.exe");

            var app = AppDomain.CreateDomain("BhgpsWnb", null, file.DirectoryName, file.DirectoryName, false);

            app.Load("BhgpsWnb, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null");

            return app;
        }
```

所以在加载 dll ，千万不要使用把文件作为 byte 读出来，然后加载，这个方法需要很多的内存。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  