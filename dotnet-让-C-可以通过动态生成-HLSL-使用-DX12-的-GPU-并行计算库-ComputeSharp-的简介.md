
# dotnet 让 C# 可以通过动态生成 HLSL 使用 DX12 的 GPU 并行计算库 ComputeSharp 的简介

本文来安利大家一个超强的库，这个库可以让你的 C# 代码利用上 GPU 显卡的性能，进行一些并行计算。这个库是基于 DirectX12GameEngine 的 ComputeSharp 库。在这个库里面将会动态生成 HLSL 代码，使用着色器的方式在 GPU 上跑起来

<!--more-->


<!-- 发布 -->

这个 ComputeSharp 库在 GitHub 完全开源，请看 [https://github.com/Sergio0694/ComputeSharp](https://github.com/Sergio0694/ComputeSharp)

在开始之前，按照 dotnet 的基本玩法，先通过 NuGet 安装库，然后使用，咱先来创建一个空白的项目。这个 ComputeSharp 库能支持的 .NET Standard 2.1 和以上的版本



通过 NuGet 安装 ComputeSharp 库，在 NuGet 包命令行输入下面代码

```csharp
Install-Package ComputeSharp
```

或者在 csproj 文件上添加下面代码

```xml
  <ItemGroup>
    <PackageReference Include="ComputeSharp" Version="1.4.1" />
  </ItemGroup>
```

下面使用一个并行给一个 float 数组设置顺序值的例子告诉大家如何使用这个库

先定义 Shader 的实现，请看下面代码

```csharp
    public readonly struct MyShader : IComputeShader
    {
        // 这是特意的命名，请不要更改
        public readonly ReadWriteBuffer<float> buffer;

        public MyShader(ReadWriteBuffer<float> buffer)
        {
            this.buffer = buffer;
        }

        public void Execute(ThreadIds ids)
        {
            buffer[ids.X] = ids.X;
        }
    }
```

这段代码将会被用来生成 HLSL 代码，因此一些属性的命名是不能更改的

然后从 GPU 中申请一段内存

```csharp
            // Allocate a writeable buffer on the GPU, with the contents of the array
            ReadWriteBuffer<float> buffer = Gpu.Default.AllocateReadWriteBuffer<float>(1000);
```

接着在 GPU 上运行

```csharp
            // Run the shader
            Gpu.Default.For(1000, new MyShader(buffer));
```

运行完成之后，可以使用下面代码拿到值

```csharp
            // Get the data back
            float[] array = buffer.GetData();
```

使用下面代码可以输出数组里面的值

```csharp
            Console.WriteLine(string.Join(",", array.Select(temp => temp.ToString())));
```

此时可以看到这个数组里面的值都按照顺序被设置了对应的值

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/90d5f5dd/QeyirakarkuWherfoqaybal)欢迎小伙伴访问

还有更多高级的用法，还请看 [https://github.com/Sergio0694/ComputeSharp](https://github.com/Sergio0694/ComputeSharp)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。