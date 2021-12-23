
# 使用 Silk.NET 创建 OpenGL 空窗口项目例子

本文告诉大家如何使用 Silk.NET 创建 OpenGL 空窗口项目。在 dotnet 基金会下，开源维护 Silk.NET 仓库，此仓库提供了渲染相关的封装逻辑，包括 DX 和 OpenGL 等等的封装，利用此封装可以用来代替原有的 SharpDx 等库。这是一个全新写的项目，使用上了 dotnet 和 C# 很多新的特性，相对来说也很活跃，我准备开始入坑这个项目

<!--more-->


<!-- 发布 -->

本文的例子完全是从 [https://github.com/dotnet/Silk.NET](https://github.com/dotnet/Silk.NET) 里面抄的，这是官方的使用 OpenGL 的例子

当前是 2021.12.22 官方完成的应用高层封装的只有 OpenGL 一个，加上 Vulkan 版本。基础底层封装完成了 DirectX 系列（但还完成没有 D2D 部分）和 OpenAL OpenCL OpenGL OpenXR Vulkan SDL 等

本文的例子是采用高层封装的 OpenGL 创建空窗口。根据官方 OpenGL Tutorials 的 `Tutorial 1.1 - Hello Window` 的代码，只需一个 Program 类即可完成启动应用

在开始之前，先通过 NuGet 安装 Silk.NET 库，对于新项目格式，可以编辑 csproj 项目文件，修改为如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Silk.NET" Version="2.11.0" />
  </ItemGroup>

</Project>
```

在 Main 函数里面，使用进行创建窗口。创建窗口需要初始化创建所需参数，在 Silk.NET 提供了预设的选项，如下面代码

```csharp
            var options = WindowOptions.Default;
```

此预设的代码等同于如下配置

```csharp
            var options = new WindowOptions
            (
                true,
                new Vector2D<int>(50, 50),
                new Vector2D<int>(1280, 720),
                0.0,
                0.0,
                new GraphicsAPI
                (
                    ContextAPI.OpenGL,
                    ContextProfile.Core,
                    ContextFlags.ForwardCompatible,
                    new APIVersion(3, 3)
                ),
                "",
                WindowState.Normal,
                WindowBorder.Resizable,
                false,
                false,
                VideoMode.Default
            );
```

通过 GraphicsAPI 可以看到默认采用的是 OpenGL 作为渲染。在 ContextAPI 可选的参数，当前有三个，但是如下面代码注释，当前还没有完成 Vulkan 的高层封装

```csharp
    /// <summary>
    /// Represents which API the graphics context should use.
    /// </summary>
    public enum ContextAPI
    {
        /// <summary>
        /// Don't use any API. This is necessary for linking an external API, such as Vulkan, to the window.
        /// </summary>
        None = 0,

        /// <summary>
        /// Use Vulkan. Silk.NET doesn't support this yet.
        /// </summary>
        Vulkan,

        /// <summary>
        /// Use core OpenGL. This is standard for software intended for desktop computers.
        /// </summary>
        OpenGL,

        /// <summary>
        /// Use OpenGL ES. This is standard for software intended to be compatible with embedded systems, such as phones.
        /// </summary>
        OpenGLES
    }
```

获取到默认的创建选项之后，如果需要修改窗口创建参数，可以修改对应的属性。例如修改窗口尺寸等

```csharp
            options.Size = new Vector2D<int>(800, 600);
            options.Title = "LearnOpenGL with Silk.NET";
```

完成窗口创建参数配置，即可通过 Silk.NET.Windowing.Window 的 Create 方法创建窗口，如下面代码

```csharp
        private static IWindow _window;

        private static void Main(string[] args)
        {
            //Create a window.
            var options = WindowOptions.Default;
            options.Size = new Vector2D<int>(800, 600);
            options.Title = "LearnOpenGL with Silk.NET";

            _window = Window.Create(options);
        }
```

完成窗口创建之后，可以使用 Run 方法执行代码

```csharp
            //Run the window.
            _window.Run();
```

在执行 Run 方法时，将会和 WPF 一样开启消息循环。为了实现窗口里面的初始化逻辑，需要在 Run 之间加上事件

```csharp
            _window.Load += OnLoad;
            _window.Update += OnUpdate;
            _window.Render += OnRender;
```

修改之后的 Main 函数如下

```csharp
        private static void Main(string[] args)
        {
            //Create a window.
            var options = WindowOptions.Default;
            options.Size = new Vector2D<int>(800, 600);
            options.Title = "LearnOpenGL with Silk.NET";

            _window = Window.Create(options);

            //Assign events.
            _window.Load += OnLoad;
            _window.Update += OnUpdate;
            _window.Render += OnRender;

            //Run the window.
            _window.Run();
        }
```

在 OnLoad 里面，可以进行很多初始化逻辑，例如获取输入键盘内容，如下面代码

```csharp
        private static void OnLoad()
        {
            //Set-up input context. 
            IInputContext input = _window.CreateInput();
            for (int i = 0; i < input.Keyboards.Count; i++)
            {
                input.Keyboards[i].KeyDown += KeyDown;
            }
        }
```

默认拿到的键盘只有一个，在 KeyDown 函数，可以通过参数判断当前按键

```csharp
        private static void KeyDown(IKeyboard arg1, Key arg2, int arg3)
        {
            //Check to close the window on escape.
            if (arg2 == Key.Escape)
            {
                _window.Close();
            }
        }
```

而 OnUpdate 和 OnRender 方法分别是做具体的渲染准备和渲染的逻辑，这些就不是本文的内容了

以下是 Program 的代码

```csharp
using Silk.NET.Input;
using Silk.NET.Maths;
using Silk.NET.Windowing;

namespace WemkuhewhallYekaherehohurnije
{
    class Program
    {
        private static IWindow _window;

        private static void Main(string[] args)
        {
            //Create a window.
            var options = WindowOptions.Default;

            options = new WindowOptions
            (
                true,
                new Vector2D<int>(50, 50),
                new Vector2D<int>(1280, 720),
                0.0,
                0.0,
                new GraphicsAPI
                (
                    ContextAPI.OpenGL,
                    ContextProfile.Core,
                    ContextFlags.ForwardCompatible,
                    new APIVersion(3, 3)
                ),
                "",
                WindowState.Normal,
                WindowBorder.Resizable,
                false,
                false,
                VideoMode.Default
            );

            options.Size = new Vector2D<int>(800, 600);
            options.Title = "LearnOpenGL with Silk.NET";

            _window = Window.Create(options);

            //Assign events.
            _window.Load += OnLoad;
            _window.Update += OnUpdate;
            _window.Render += OnRender;

            //Run the window.
            _window.Run();
        }

        private static void OnLoad()
        {
            //Set-up input context. 
            IInputContext input = _window.CreateInput();
            for (int i = 0; i < input.Keyboards.Count; i++)
            {
                input.Keyboards[i].KeyDown += KeyDown;
            }
        }

        private static void OnRender(double obj)
        {
            //Here all rendering should be done.
        }

        private static void OnUpdate(double obj)
        {
            //Here all updates to the program should be done.
        }

        private static void KeyDown(IKeyboard arg1, Key arg2, int arg3)
        {
            //Check to close the window on escape.
            if (arg2 == Key.Escape)
            {
                _window.Close();
            }
        }
    }
}
```

按下 F5 运行代码，即可看到创建了空窗口





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。