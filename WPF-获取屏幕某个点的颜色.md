
# WPF 获取屏幕某个点的颜色

我在做一个笔迹性能测试工具，想要在笔迹绘制到某个点的时候输出绘制的速度，通过判断屏幕颜色修改判断笔迹绘制到哪。此时需要在不截图屏幕获取屏幕某个点的颜色

<!--more-->


<!-- CreateTime:6/29/2020 3:02:27 PM -->

本文的方法可以在 WinForms 等使用

```csharp
  using System;
  using System.Drawing;
  using System.Runtime.InteropServices;
  sealed class Win32
  {
      [DllImport("user32.dll")]
      static extern IntPtr GetDC(IntPtr hwnd);

      [DllImport("user32.dll")]
      static extern Int32 ReleaseDC(IntPtr hwnd, IntPtr hdc);

      [DllImport("gdi32.dll")]
      static extern uint GetPixel(IntPtr hdc, int nXPos, int nYPos);

      static public System.Drawing.Color GetPixelColor(int x, int y)
      {
       IntPtr hdc = GetDC(IntPtr.Zero);
       uint pixel = GetPixel(hdc, x, y);
       ReleaseDC(IntPtr.Zero, hdc);
       Color color = Color.FromArgb((int)(pixel & 0x000000FF),
                    (int)(pixel & 0x0000FF00) >> 8,
                    (int)(pixel & 0x00FF0000) >> 16);
       return color;
      }
   }
```

感谢[Jeremy Thompson](https://stackoverflow.com/a/62630169/6116637)的方法





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。