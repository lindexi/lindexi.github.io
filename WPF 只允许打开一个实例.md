# WPF 只允许打开一个实例

我们有时候只希望我们的程序只打开一个实例，也就是我们的软件只有一次被打开。

那么我们可以通过一个办法知道，在这个软件打开前是不是打开过一个，还没关闭。也就是是否存在另一个程序在运行。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


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

