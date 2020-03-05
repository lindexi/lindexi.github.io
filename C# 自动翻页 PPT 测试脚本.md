# C# 自动翻页 PPT 测试脚本

本文告诉大家一个可以使用的 C# 脚本，可以用来自动打开 PPT 文件，然后不断执行翻页。每次翻页都截图。翻页之后自动关闭 PPT 再次打开

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


最近发现给 Office 做的插件，会在一定翻页次数的时候，就 gg 了，所以我就写了这样的脚本，小伙伴可以拿去用

编译下面的代码，然后将几个需要测试的 PPTX 文件放在编译出来的程序相同文件夹，双击运行这个程序就可以进行测试

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var guid = Guid.NewGuid().ToString("N");
            for (int i = 1; i <= 10000; i++)
            {
                Process p = new Process();
                //设置要启动的应用程序
                p.StartInfo.FileName = "cmd.exe";
                //是否使用操作系统shell启动
                p.StartInfo.UseShellExecute = false;
                // 接受来自调用程序的输入信息
                p.StartInfo.RedirectStandardInput = true;
                //输出信息
                p.StartInfo.RedirectStandardOutput = true;
                // 输出错误
                p.StartInfo.RedirectStandardError = true;
                //不显示程序窗口
                p.StartInfo.CreateNoWindow = true;

                Thread.Sleep(2000);

                var directory = new DirectoryInfo($"第{i}次 {guid}");
                directory.Create();
                string pptFile = GetPPT();
                Console.WriteLine("启动文件： " + pptFile);

                Console.WriteLine(@"第 " + i + " 次启动：");
                p.Start();
                //向cmd窗口发送输入信息
                Thread.Sleep(2000);
                p.StandardInput.WriteLine("\"" + pptFile + "\"");
                p.StandardInput.AutoFlush = true;
                Console.WriteLine(@"开始启动客户端");

                Thread.Sleep(10000);

                //触发F5按键
                keybd_event(116, 0, 0, 0);

                // 翻多少页
                for (int j = 1; j < 10; j++)
                {
                    Thread.Sleep(2000);
                    keybd_event(vbKeyDown, 0, 0, 0);
                    Console.WriteLine(@"开始翻页");

                    // 翻一页截图
                    // 通过Graphics的CopyFromScreen方法把全屏图片的拷贝到我们定义好的一个和屏幕大小相同的空白图片中，
                    // 拷贝完成之后，CatchBmp就是全屏图片的拷贝了，然后指定为截图窗体背景图片就好了。
                    // 新建一个和屏幕大小相同的图片
                    Bitmap CatchBmp = new Bitmap(Screen.AllScreens[0].Bounds.Width, Screen.AllScreens[0].Bounds.Height);

                    // 创建一个画板，让我们可以在画板上画图
                    // 这个画板也就是和屏幕大小一样大的图片
                    // 我们可以通过Graphics这个类在这个空白图片上画图
                    Graphics g = Graphics.FromImage(CatchBmp);

                    // 把屏幕图片拷贝到我们创建的空白图片 CatchBmp中
                    g.CopyFromScreen(new Point(0, 0), new Point(0, 0),
                        new Size(Screen.AllScreens[0].Bounds.Width, Screen.AllScreens[0].Bounds.Height));

                    var file = new FileInfo(Path.Combine(directory.FullName, $"{j}.png"));

                    var fileStream = new FileStream(file.FullName, FileMode.Create, FileAccess.Write);

                    using (fileStream)
                    {
                        CatchBmp.Save(fileStream, ImageFormat.Png);
                        CatchBmp.Dispose();
                    }

                    Console.WriteLine("保存截图" + file.FullName);
                }

                Thread.Sleep(2000);

                keybd_event(18, 0, 0, 0);
                keybd_event(115, 0, 0, 0);
                keybd_event(18, 0, 2, 0);
                keybd_event(115, 0, 2, 0);

                Thread.Sleep(2000);

                Console.WriteLine("干掉进程");

                foreach (var temp in Process.GetProcesses())
                {
                    try
                    {
                        if (temp.ProcessName.IndexOf("power", StringComparison.InvariantCultureIgnoreCase) >= 0)
                        {
                            temp.Kill();
                        }
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine(e);
                    }
                }

                p.Kill();
            }
        }

        private static string GetPPT()
        {
            var directory = new DirectoryInfo(AppDomain.CurrentDomain.BaseDirectory);
            var list = directory.GetFiles("*.pptx").ToList();

            return list[_random.Next(list.Count)].FullName;
        }

        private static readonly Random _random = new Random();

        [DllImport("user32.dll")]
        public static extern void keybd_event(byte bVk, byte bScan, int dwFlags, int dwExtraInfo);

        public const byte vbKeyDown = 0x28; // DOWN ARROW 键
    }
```

用这个脚本运行几天然后我就去看截图，就可以发现在翻到一定的页面，我的插件就 gg 了。在我修复之后再次运行这个脚本，发现没有 gg 于是就可以和微软说我修复了我的插件，再次上传

如何写 Office 插件，推荐陈希章的[Office 365 开发概览系列](https://www.cnblogs.com/chenxizhang/category/967796.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
