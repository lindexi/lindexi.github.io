
# WPF 只允许打开一个实例

我们有时候只希望我们的程序只打开一个实例，也就是我们的软件只有一次被打开。
那么我们可以通过一个办法知道，在这个软件打开前是不是打开过一个，还没关闭。也就是是否存在另一个程序在运行。

<!--more-->



<div id="toc"></div>

下面是一个简单方法

```csharp

                    // 确保不存在程序的其他实例
                    singleInstanceWatcher = new Semaphore(
                        0, // Initial count.
                        1, // Maximum count.
                        Assembly.GetExecutingAssembly().GetName().Name, out createdNew);
                    if (createdNew)
                    {
                        //之前没有运行过
                    }
                    else
                    {
                    	//重复运行
                        MessageBox.Show("请不要重复运行(ノ｀Д)ノ");
                        Environment.Exit(-2);
                    }

```


另一个方法

```csharp
            string mutexName = Properties.Resources.ProgramTitle + "Mutex";
            singleInstanceWatcher = new Mutex(false, mutexName, out createdNew);
            if (!createdNew)
            {
                MessageBox.Show("程序已经运行!", "错误", MessageBoxButton.OK, MessageBoxImage.Error);
                Environment.Exit(-1);
            }

```

		

```csharp
bool createdNew;
System.Threading.Mutex instance = new System.Threading.Mutex(true, "MutexName", out createdNew);
if (createdNew)
{
    Application.Run(new LoginForm());
    instance.ReleaseMutex();
}
else
{
    Application.Exit();
}

```


http://www.cnblogs.com/z_lb/archive/2012/09/16/2687487.html





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。