
# WPF 从键盘事件 KeyEventArgs 里获取 Scan Code 的方法

本文将告诉大家如何在 WPF 里面，从键盘事件 KeyEventArgs 参数里获取到 Scan Code 键盘按键的设备独立标识符的方法

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

概念：

以下来自 bing 的答案

键盘的 **Scan Code** 是按键的设备独立标识符，对应于按键在硬件上的实际标识。每个按键都有一个唯一的扫描码，用于表示该按键。当用户按下一个键时，键盘会生成两个扫描码：**通码（Make Code）**和**断码（Break Code）**。通码表示按键被按下，而断码表示按键被释放。这些扫描码由键盘设备驱动解释并映射为**虚拟键码（Virtual Key Code）**，这是系统定义的设备独立值，用于标识键盘的按键²³.

虚拟键码与扫描码之间的区别在于，虚拟键码是系统定义的值，而扫描码是硬件上的实际标识符。虚拟键码通常用于处理键盘输入的应用程序，而扫描码更接近键盘的底层硬件表示。 。

总结一下：
- **Scan Code**：键盘按键的设备独立标识符，由硬件生成。
- **Virtual Key Code**：系统定义的设备独立值，用于标识键盘的按键，由键盘设备驱动解释扫描码并映射而来。

方法1： 推荐的方法，通过 Win32 函数获取，代码实现如下

```csharp
          KeyDown += MainWindow_KeyDown;

    private void MainWindow_KeyDown(object sender, KeyEventArgs e)
    {
        var key = e.Key;
        var virtualKey = KeyInterop.VirtualKeyFromKey(key);

        // MAPVK_VK_TO_VSC 0
        var scanCode = MapVirtualKeyW((uint) virtualKey, 0 /*MAPVK_VK_TO_VSC*/);
    }

    [DllImport("User32.dll")]
    private static extern uint MapVirtualKeyW(uint code, uint mapType);
```

方法2： 通过反射方式获取，不推荐，且 API 可能被变更

```csharp
    private void MainWindow_KeyDown(object sender, KeyEventArgs e)
    {
        var scanCodeFromWpf = typeof(KeyEventArgs).GetProperty("ScanCode", BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance)!.GetValue(e);
    }
```

这两个方法获取到的值是相同的，如使用下面代码，判断相等成立

```csharp
        Debug.Assert(scanCode == (int) scanCodeFromWpf!);
```

但如 MapVirtualKeyW 函数所述，确实存在一些情况下，获取不到相同的结果


本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/1806c3b45f5f10242da97a2fc1e3fa433fca783d/LaykechererolelQemyukilee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1806c3b45f5f10242da97a2fc1e3fa433fca783d/LaykechererolelQemyukilee) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1806c3b45f5f10242da97a2fc1e3fa433fca783d
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1806c3b45f5f10242da97a2fc1e3fa433fca783d
```

获取代码之后，进入 LaykechererolelQemyukilee 文件夹，即可获取到源代码




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。