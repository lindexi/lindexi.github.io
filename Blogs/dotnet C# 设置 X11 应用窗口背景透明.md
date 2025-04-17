---
title: dotnet C# 设置 X11 应用窗口背景透明
description: 本文将告诉大家如何在 X11 里面设置窗口透明
tags: dotnet,X11,C#
category: 
---

<!-- CreateTime:2024/03/28 07:23:12 -->

<!-- 发布 -->
<!-- 博客 -->

不同于在 WPF 里面可以使用 AllowsTransparency 简单方便的设置透明，在 X11 里面设置窗口透明的方法比较绕。需要获取用于传入给到 XCreateWindow 的 Visual 指针，才能实现窗口透明

感谢 [walterlv](https://github.com/walterlv) 大佬提供此方法，我只是代为记录的工具人

以下是一个简单的示例代码，示例代码里面被我忽略掉一些 P/Invoke 调用封装代码，这些被忽略代码可以从本文末尾找到，可以从本文末尾找到整个示例代码的下载方式

先创建一个空的控制台应用，然后编辑 csproj 项目文件，替换为如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <InvariantGlobalization>true</InvariantGlobalization>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>

</Project>
```

以上的 csproj 项目文件代码里和空控制台核心不同在于使用 AllowUnsafeBlocks 开启不安全代码

打开 Program.cs 文件，开始编写 X11 透明窗口示例应用代码

按照 X11 的基础使用方法，先获取 Display 和 Screen 和 RootWindow 对象/指针，代码如下

```csharp
var display = XOpenDisplay(0);
var defaultScreen = XDefaultScreen(display);
var rootWindow = XRootWindow(display, defaultScreen);
```

接着使用 GetVisual 方法获取 visual 指针，代码如下

```csharp
var visual = GetVisual(display, defaultScreen);
```

以上代码的 GetVisual 方法的定义如下

```csharp
static unsafe nint GetVisual(nint deferredDisplay, int defaultScreen)
{
    var glx = new GlxInterface();
    nint fbconfig = 0;
    XVisualInfo* visual = null;
    int[] baseAttribs =
        [
            GLX_X_RENDERABLE, 1,
            GLX_RENDER_TYPE, GLX_RGBA_BIT,
            GLX_DRAWABLE_TYPE, GLX_WINDOW_BIT | GLX_PBUFFER_BIT,
            GLX_DOUBLEBUFFER, 1,
            GLX_RED_SIZE, 8,
            GLX_GREEN_SIZE, 8,
            GLX_BLUE_SIZE, 8,
            GLX_ALPHA_SIZE, 8,
            GLX_DEPTH_SIZE, 1,
            GLX_STENCIL_SIZE, 8,
        ];

    foreach (var attribs in new[] { baseAttribs })
    {
        var ptr = glx.ChooseFBConfig(deferredDisplay, defaultScreen,
            attribs, out var count);
        for (var c = 0; c < count; c++)
        {

            var v = glx.GetVisualFromFBConfig(deferredDisplay, ptr[c]);
            // We prefer 32 bit visuals
            if (fbconfig == default || v->depth == 32)
            {
                fbconfig = ptr[c];
                visual = v;
                if (v->depth == 32)
                {
                    break;
                }
            }
        }

        if (fbconfig != default)
        {
            break;
        }
    }

    return visual->visual;
}
```

获取 Visual 指针的方法就是本文的核心逻辑了，也是 X11 窗口透明的关键

以上的 GlxInterface 类型定义如下

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection.Emit;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;

namespace BlankX11App.X11;
internal unsafe class GlxInterface
{
    private const string libGL = "libGL.so.1";

    public GlxInterface()
    {
        Initialize(SafeGetProcAddress);
    }

    private void Initialize(Func<string, nint> getProcAddress)
    {
        nint addr;
        // Initializing ChooseFBConfig
        addr = 0;
        addr = getProcAddress("glXChooseFBConfig");
        if (addr == default) throw new EntryPointNotFoundException("_addr_ChooseFBConfig");
        _addr_ChooseFBConfig = (delegate* unmanaged[Stdcall]<nint, int, int*, out int, nint*>)addr;
        // Initializing GetVisualFromFBConfig
        addr = 0;
        addr = getProcAddress("glXGetVisualFromFBConfig");
        if (addr == default) throw new EntryPointNotFoundException("_addr_GetVisualFromFBConfig");
        _addr_GetVisualFromFBConfig = (delegate* unmanaged[Stdcall]<nint, nint, XVisualInfo*>)addr;
    }

    delegate* unmanaged[Stdcall]<nint, int, int*, out int, nint*> _addr_ChooseFBConfig;
    public nint* ChooseFBConfig(nint @dpy, int @screen, int[] @attrib_list, out int @nelements)
    {
        fixed (int* @__p_attrib_list = attrib_list)
        {
            return _addr_ChooseFBConfig(@dpy, @screen, @__p_attrib_list, out @nelements);
        }
    }

    delegate* unmanaged[Stdcall]<nint, nint, XVisualInfo*> _addr_GetVisualFromFBConfig;
    public XVisualInfo* GetVisualFromFBConfig(nint @dpy, nint @config)
    {
        return _addr_GetVisualFromFBConfig(@dpy, @config);
    }


    // Ignores egl functions.
    // On some Linux systems, glXGetProcAddress will return valid pointers for even EGL functions.
    // This makes Skia try to load some data from EGL,
    // which can then cause segmentation faults because they return garbage.
    public static nint SafeGetProcAddress(string proc)
    {
        if (proc.StartsWith("egl", StringComparison.InvariantCulture))
        {
            return nint.Zero;
        }

        return GlxGetProcAddress(proc);
    }

    [DllImport(libGL, EntryPoint = "glXGetProcAddress")]
    public static extern nint GlxGetProcAddress(string buffer);
}
```

获取到 visual 指针之后，即可传入到 XCreateWindow 里面，如以下代码

```csharp
var visual = GetVisual(display, defaultScreen);
var valueMask = SetWindowValuemask.BackPixmap
    | SetWindowValuemask.BackPixel
    | SetWindowValuemask.BorderPixel
    | SetWindowValuemask.BitGravity
    | SetWindowValuemask.WinGravity
    | SetWindowValuemask.BackingStore
    | SetWindowValuemask.ColorMap;
var attr = new XSetWindowAttributes
{
    backing_store = 1,
    bit_gravity = Gravity.NorthWestGravity,
    win_gravity = Gravity.NorthWestGravity,
    override_redirect = 0,  // 参数：_overrideRedirect
    colormap = XCreateColormap(display, rootWindow, visual, 0),
};

var handle = XCreateWindow(display, rootWindow, 100, 100, 320, 240, 0,
    32,
    (int)CreateWindowArgs.InputOutput,
    visual,
    (nuint)valueMask, ref attr);
```

另一个获取 visual 的方法是通过 XMatchVisualInfo 方法获取，如此获取更加简单，不需要借助 OpenGL 模块，代码如下

```csharp
XMatchVisualInfo(display, defaultScreen, 32, 4, out var info);
var visual = info.visual;
```

接下来就是使用 XMapWindow 显示和关联窗口，代码如下

```csharp
XMapWindow(display, handle);
XFlush(display);
while (XNextEvent(display, out var xEvent) == default)
{
}
```

如果运行以上代码，没有看到窗口透明，那可能就是桌面窗口合成管理器没有安装或没有安装正确。还请自行修复一下

比如安装 [compiz](https://en.wikipedia.org/wiki/Compiz) 窗口合成管理器，安装和运行的命令行如下

```
sudo apt-get install compiz
compiz
```

比如在 UOS 里，可在系统设置->个性化->通用里，开启窗口特效

<!-- ![](image/dotnet C# 设置 X11 应用窗口背景透明/dotnet C# 设置 X11 应用窗口背景透明0.png) -->
![](http://cdn.lindexi.site/lindexi%2F2024327114343340.jpg)

如果开启之后依然没有透明窗口背景效果，则请调查一下是否 UOS 里默认的 KWin 窗口合成管理器损坏或被替换为其他的窗口合成管理器，查看当前的窗口合成管理器可使用以下命令

```
sudo apt-get install inxi
inxi -Gxx | grep compositor
```

如能输出 `compositor: kwin_x11` 之类的，则证明依然使用的是 kwin 窗口合成管理器。如输出的字符串里面 compositor 包含的是其他字符串，则请自行了解一下对应的窗口合成管理器是否支持窗口透明或需要进行哪些配置。更多国产 UOS 系统开发相关，欢迎加入 810052083 群交流

完全的 Program.cs 文件的代码如下

```csharp
using System.Collections.Immutable;

using BlankX11App.X11;

using static BlankX11App.X11.XLib;
using static BlankX11App.X11.GlxConsts;
using System.Diagnostics;

var display = XOpenDisplay(0);
var defaultScreen = XDefaultScreen(display);
var rootWindow = XRootWindow(display, defaultScreen);
var visual = GetVisual(display, defaultScreen);
var valueMask = SetWindowValuemask.BackPixmap
    | SetWindowValuemask.BackPixel
    | SetWindowValuemask.BorderPixel
    | SetWindowValuemask.BitGravity
    | SetWindowValuemask.WinGravity
    | SetWindowValuemask.BackingStore
    | SetWindowValuemask.ColorMap;
var attr = new XSetWindowAttributes
{
    backing_store = 1,
    bit_gravity = Gravity.NorthWestGravity,
    win_gravity = Gravity.NorthWestGravity,
    override_redirect = 0,  // 参数：_overrideRedirect
    colormap = XCreateColormap(display, rootWindow, visual, 0),
};

var handle = XCreateWindow(display, rootWindow, 100, 100, 320, 240, 0,
    32,
    (int)CreateWindowArgs.InputOutput,
    visual,
    (nuint)valueMask, ref attr);
XMapWindow(display, handle);
XFlush(display);

while (XNextEvent(display, out var xEvent) == default)
{
}

XUnmapWindow(display, handle);
XDestroyWindow(display, handle);

static unsafe nint GetVisual(nint deferredDisplay, int defaultScreen)
{
    var glx = new GlxInterface();
    nint fbconfig = 0;
    XVisualInfo* visual = null;
    int[] baseAttribs =
        [
            GLX_X_RENDERABLE, 1,
            GLX_RENDER_TYPE, GLX_RGBA_BIT,
            GLX_DRAWABLE_TYPE, GLX_WINDOW_BIT | GLX_PBUFFER_BIT,
            GLX_DOUBLEBUFFER, 1,
            GLX_RED_SIZE, 8,
            GLX_GREEN_SIZE, 8,
            GLX_BLUE_SIZE, 8,
            GLX_ALPHA_SIZE, 8,
            GLX_DEPTH_SIZE, 1,
            GLX_STENCIL_SIZE, 8,
        ];

    foreach (var attribs in new[] { baseAttribs })
    {
        var ptr = glx.ChooseFBConfig(deferredDisplay, defaultScreen,
            attribs, out var count);
        for (var c = 0; c < count; c++)
        {

            var v = glx.GetVisualFromFBConfig(deferredDisplay, ptr[c]);
            // We prefer 32 bit visuals
            if (fbconfig == default || v->depth == 32)
            {
                fbconfig = ptr[c];
                visual = v;
                if (v->depth == 32)
                {
                    break;
                }
            }
        }

        if (fbconfig != default)
        {
            break;
        }
    }

    return visual->visual;
}
```

以上代码放在 <https://github.com/walterlv/Walterlv.BlankUnoApp> 仓库里，欢迎大家拉取代码运行。拉取代码之后，打开 BlankUnoApp.sln 文件，即可找到本文的 BlankX11App 示例项目
