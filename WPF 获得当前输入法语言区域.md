# WPF 获得当前输入法语言区域

本文告诉大家如何获得 WPF 输入法的语言区域

<!--more-->
<!-- CreateTime:2019/6/23 11:51:21 -->


需要使用 user32 的方法，很简单，请看下面

```csharp
       [DllImport("user32.dll")] static extern IntPtr GetForegroundWindow();
        [DllImport("user32.dll")] static extern uint GetWindowThreadProcessId(IntPtr hwnd, IntPtr proccess);
        [DllImport("user32.dll")] static extern IntPtr GetKeyboardLayout(uint thread);

        public CultureInfo GetCurrentKeyboardLayout()
        {
            try
            {
                IntPtr foregroundWindow = GetForegroundWindow();
                uint foregroundProcess = GetWindowThreadProcessId(foregroundWindow, IntPtr.Zero);
                int keyboardLayout = GetKeyboardLayout(foregroundProcess).ToInt32() & 0xFFFF;
                return new CultureInfo(keyboardLayout);
            }
            catch (Exception)
            {
                return new CultureInfo(1033); // Assume English if something went wrong.
            }
        }
```

因为没有提供语言区域变化的事件，所以需要自己写一个循环来获取

在界面添加一个 TextBlock 请看下面

```csharp
    <Grid>
        <TextBlock x:Name="CeareamearTreseretal" HorizontalAlignment="Center" VerticalAlignment="Center"></TextBlock>
    </Grid>
```

这样可以在后台代码给一个值

```csharp
        private async void HairjurNalllairmo()
        {
            while (true)
            {
                await Task.Delay(100);
                CeareamearTreseretal.Text = GetCurrentKeyboardLayout().DisplayName;
            }
        }
```

这时修改语言区域就可以看到变化

![](http://image.acmx.xyz/lindexi%2F2018101211845978)

参见 [C#: Get current keyboard layout\input language](https://yal.cc/csharp-get-current-keyboard-layout/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
