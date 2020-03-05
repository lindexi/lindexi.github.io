# 解决 Win10 UWP 无法使用 ss 连接

一旦使用了 ss， 那么很多应用就无法连接网络。

本文提供一个方法可以简单使用ss提供的代理。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


多谢 wtwsgs 提供方法：http://blog.csdn.net/wtwsgs/article/details/51333912

代码很少，可以自己复制放在vs运行


```csharp
     class Program
    {
        static void Main(string[] args)
        {
            //%USERPROFILE%\AppData\Local\Packages
            string str = "%USERPROFILE%\\AppData\\Local\\Packages";
            str = Environment.ExpandEnvironmentVariables(str);
            DirectoryInfo dir = new DirectoryInfo(str);
            foreach (var temp in dir.GetDirectories())
            {
                str = "CheckNetIsolation.exe LoopbackExempt -a -n=\"" + temp.Name + "\"";
                Control(str);
            }
        }

        private static string Control(string str)
        {
            Process p = new Process();
            p.StartInfo.FileName = "cmd.exe";
            p.StartInfo.UseShellExecute = false; 
            p.StartInfo.RedirectStandardInput = true; 
            p.StartInfo.RedirectStandardOutput = true; 
            p.StartInfo.RedirectStandardError = true; 
            p.StartInfo.CreateNoWindow = true; 
            p.StartInfo.StandardOutputEncoding = Encoding.UTF8;
            p.Start(); 

            p.StandardInput.WriteLine(str + "&exit");

            p.StandardInput.AutoFlush = true;

            string output = p.StandardOutput.ReadToEnd();
            output += p.StandardError.ReadToEnd();
          

            p.WaitForExit(); 
            p.Close();

            return output + "\r\n";
        }
    }
```

如果不想在 vs 编译，可以下载编译的：[下载](http://download.csdn.net/detail/lindexi_gd/9823838)

这个应用
需要使用管理员打开

参见：
https://github.com/shadowsocks/shadowsocks-windows/issues/932

http://krblog.krrrrr.xyz/md/post/105

http://www.win10.cm/?p=1226

本文同时放在九幽[解决 Win10 UWP 无法使用 ss 连接 | Win10.CM](http://www.win10.cm/?p=1226)

如果发现软件无法编译，可以使用下面方法 

VS中新建C#控制台应用，在.cs文件中以上面的代码替换空缺的program类，如果有报错则使用“快速操作”添加缺失的命名空间，然后build就可以了。为了不显示控制台可以在解决方案上右键在属性中把输出选项更改为 Windows 应用程序

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
