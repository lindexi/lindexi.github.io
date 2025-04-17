
# dotnet 将控制台 Console.WriteLine 内容输出到文件

很多伙伴喜欢使用 Console.WriteLine 打日志，也许是打起来顺手。打完了之后，又想着，要是能够输出到本机文件那就更好了。既然很多伙伴都有这个想法，那 dotnet 自然就是有方便的方法让咱来实现此需求。只需要调用 Console.SetOut 方法，即可将控制台的输出重定向到一个 TextWriter 里面，只要此 TextWriter 最终输出到本地文件里，即可实现将控制台的内容输出到文件。本文将来告诉大家这个实现方法

<!--more-->


<!-- CreateTime:2022/11/28 8:02:47 -->


<!-- 发布 -->
<!-- 博客 -->

先演示一下，通过 Console.SetOut 将控制台的内容输出到文件。阅读 Console.SetOut 方法定义，可以看到这个方法需要传入一个 TextWriter 参数。在 dotnet 里面，有很多个继承 TextWriter 的默认实现，例如 StreamWriter 和 StringWriter 等，通过 StreamWriter 可以将传入的字符串内容，写入到一个 Stream 里面，例如一个 FileStream 里面，就能最终写入到文件里面。这是非常自由，而且非常可定制的设计方法，因为 Stream 可以是文件，也可以是网络的请求，也可以是其他的输出源，也可以随意加上过滤加上加密加上压缩，特别方便自由定制

先创建一个空 WPF 项目作为演示项目，本文的演示代码可以在本文末尾获取。在 App 的构造函数里面，也就是整个应用的入口，配置好控制台的输出。先设置一个文件，让这个文件用来作为日志的输出

```csharp
        // 好孩子可不要在这里写相对路径哦
        var logFile = "log.txt";
```

对于日志等文件来说，不怎么合适使用相对路径，因为相对路径是比较不可控的。可以问问自己，你知道他运行起来，相对于谁的路径么？是当前 Exe 所在路径？不是，是工作路径。那工作路径又是哪个文件夹呢？这又有趣起来了，还请自行了解

推荐先转换一下，设置为绝对路径，这样要是写入出错了，还可以通过此绝对路径，看看写入到哪

```csharp
        // 将相对路径转换为绝对路径，这样要是写错地方了，在这里可以快速调试到
        logFile = Path.GetFullPath(logFile);
```

接下来通过 StreamWriter 辅助将控制台输出内容写入到此文件里

```csharp
        var streamWriter = new StreamWriter(logFile)
        {
            AutoFlush = true
        };
```

以上代码的 StreamWriter 设置了 AutoFlush 属性，如此即可不需要每次写入完成，手动调用 Flush 方法才能将内容写入到磁盘文件里面。设置之后，可以自动写入到磁盘里

接着设置控制台的输出重定向

```csharp
        Console.SetOut(streamWriter);
```

试试在界面里面加一个按钮，让按钮调用 Console.WriteLine 方法，看看日志文件是否能被写入内容

以下是 XAML 代码，用来创建一个按钮

```xml
    <Grid>
        <Button Margin="10,10,10,10" HorizontalAlignment="Left" VerticalAlignment="Top" Click="Button_OnClick">Click</Button>
    </Grid>
```

在 MainWindow.xaml.cs 后台代码里面，按钮点击随便写控制台

```csharp
    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        Console.WriteLine("Test");
    }
```

运行程序，不断点击按钮，可以看到日志文件写入了内容

这就完成了？还没有呢。刚才创建的 StreamWriter 需要记得在应用退出的时候释放哦。为什么需要在应用退出的时候释放？难道是怕内存泄露？其实完全和内存泄露没有关系，想想，进程都退出了，还有啥可以被泄露的，整个进程的内存都会被系统回收（这句话没全对）的哦。其实只是为了解决 StreamWriter 没有将最后的一些日志刷入到磁盘里面而已，上面代码设置了是自动写入，但是毕竟是自动的，谁知道他是不是刚好在退出的时候，还没有完全写入，手动在应用退出时释放，即可让缓存写入磁盘缓存

完全的 App 类的代码如下

```csharp
public partial class App : Application
{
    public App()
    {
        // 好孩子可不要在这里写相对路径哦
        var logFile = "log.txt";
        // 将相对路径转换为绝对路径，这样要是写错地方了，在这里可以快速调试到
        logFile = Path.GetFullPath(logFile);

        var streamWriter = new StreamWriter(logFile)
        {
            AutoFlush = true
        };
        Console.SetOut(streamWriter);

        LogWriter = streamWriter;
    }

    protected override void OnExit(ExitEventArgs e)
    {
        base.OnExit(e);

        LogWriter.Dispose();
    }

    private StreamWriter LogWriter { get; }
}
```

那如果对写入到文件的日志，有格式的要求，例如加上时间，那可以自己定义一个类型，继承 StreamWriter 方法，重写 WriteLine 等方法，进行过滤，如下面代码

```csharp
class LogWriter : StreamWriter
{
    public LogWriter(string path) : base(path)
    {
    }

    public LogWriter(Stream stream) : base(stream)
    {

    }

    public override void WriteLine(string? value)
    {
        // 可以在这里对输出的字符串进行处理，例如加上时间
        var message = "[" + DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss,fff") + "] " + value;
        base.WriteLine(message);
    }
}
```

重写方法，对输入进行处理，处理完成再调用基类的方法。如此即可过滤日志输出内容

稍微修改 App 的构造函数代码，将 StreamWriter 换成 LogWriter 类型

```csharp
        var logWriter = new LogWriter(logFile)
        {
            AutoFlush = true,
        };

        Console.SetOut(logWriter);
```

以上的方法将会在每次打开应用时，清空原有的日志文件。如果只是期望追加的话，可以使用下面的代码

```csharp
        var fileStream = new FileStream(logFile, FileMode.Append, FileAccess.Write, FileShare.Read);
        logWriter = new LogWriter(fileStream)
        {
            AutoFlush = true,
        };
```

设置使用追加的方式，这样每次写入日志，就是在原有的日志文件上追加内容。而且设置了 FileShare 允许其他应用读取，这样一边写，一边就可以使用记事本等打开

写入到文件完成了，伙伴们也许还有其他疑惑。那我的 WPF 应用程序，如果我不想输出到文件，我就想打开控制台看输出，但是默认创建的项目是没有显示控制台的，可以如何显示控制台？这其实需要聊到 Windows 的 PE 文件格式，在 PE 文件格式里面，控制台应用和类似 WPF 这样的 GUI 应用是有不同的 PE 头控制的，也就是说读取一个 Exe 的文件的文件头信息，就决定了控制台是否显示出来。对应到项目里面，可以通过编辑 OutputType 属性进行修改

- Exe： 控制台
- WinExe：类似 WPF 这样的 GUI 应用

如果项目采用 SDK 分割的 csproj 项目文件，那编辑 csproj 文件，将 OutputType 修改为 Exe 即可。如果不想编辑 csproj 文件，可以右击项目属性，在常规输出类型设置为控制台应用程序即可

修改之后，重新构建，运行即可看到控制台了

伙伴们也许又有一个问题，在设置输出到文件之后，原本可以在控制台看到的输出就看不到了。这是因为输出到控制台的内容被重定向到文件里面了

解决方法其实非常简单，只需要让输出的内容分为两路，一路输出到文件，一路依然输出到控制台即可。这就是 C# dotnet 里面设计的魅力，可以非常方便自己组合和过滤和修改

先将原本的 Console.Out 获取到，这就是原本输出到控制台界面的输出源。接着将此输出源传入到 LogWriter 类型里面，让 LogWriter 类型在写入到基类之后也写入到原本的输出源里面，修改之后的代码如下

```csharp
class LogWriter : StreamWriter
{
    public LogWriter(string path, TextWriter textWriter) : base(path)
    {
        _textWriter = textWriter;
    }
 
    private readonly TextWriter _textWriter;

    public override void WriteLine(string? value)
    {
        // 可以在这里对输出的字符串进行处理，例如加上时间
        var message = "[" + DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss,fff") + "] " + value;
        base.WriteLine(message);

        _textWriter.WriteLine(message);
    }
}
```

修改 App 构造函数代码如下

```csharp
        var logWriter = new LogWriter(logFile, Console.Out)
        {
            AutoFlush = true,
        };
```

试试运行应用，可以看到输出到文件，也输出到控制台

也许伙伴们又有一个问题，那为什么有些应用，控制台的内容能够输出到 VisualStudio 的调试的输出窗口里面，有些又不行，可以如何配置才能输出到 VisualStudio 的输出窗口？其实 VisualStudio 的输出窗口的内容也是 VS 的黑科技，是通过重定向输出实现的，不管 Visual Studio 如何玩，咱都可以自己强行给他配置输出

在 VisualStudio 调试，将某个内容输出到 VisualStudio 的输出窗口，需要使用 Debugger.Log 方法，如上面的代码，再加一个输出到 Visual Studio 输出窗口

```csharp
class LogWriter : StreamWriter
{

    public LogWriter(string path, TextWriter textWriter) : base(path)
    {
        _textWriter = textWriter;
    }

    private readonly TextWriter _textWriter;

    public override void WriteLine(string? value)
    {
        // 可以在这里对输出的字符串进行处理，例如加上时间
        var message = "[" + DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss,fff") + "] " + value;
        base.WriteLine(message);

        _textWriter.WriteLine(message);
        Debugger.Log(0, null, message + "\r\n");
    }
}
```

尝试运行应用，可以看到 VS 的输出窗口也可以看到日志

这就将控制台给玩起来了，可以愉快使用控制台输出日志

这时，也许有伙伴会来告诉你，不要用控制台输出日志信息，因为会卡住。这是为什么呢？这其实只是因为控制台的输出源被卡住而已，控制台的输出源如果被卡住了，那自然调用控制台的输出方法也会被卡住，这是一个同步方法。卡住控制台的输出十分简单，只需要在 CDM 里面，进入选择模式，也就是在 Win10 系统上，鼠标点击一下控制台的内容，拖动选择一点字符，就可以卡住控制台了。这个在调试下也非常好用，可以方便让应用暂停起来

另外，如果当前的应用的标准输出被其他应用重定向了，而其他的应用没有足够快的输出处理好消息，也会导致当前应用使用控制台输出卡住，请看 [设置进程的 RedirectStandardOutput 重定向输出后，如果不将输出读出来，会卡死此进程 - walterlv](https://blog.walterlv.com/post/standard-output-must-be-read-if-you-redirect-standard-output.html)

但认真阅读完成本文的伙伴们会表示，这些都不会是问题。因为卡住的问题，其实只是因为默认的标准输出源被卡住，导致应用被卡住而已。自己设置了 SetOut 方法，那么所有的控制台输出都是自己处理的。如果自己不输出到标准的输出源，那自然就不会有此困扰了。如果真的还需要输出，但是也不想被卡住呢？详细大家也想到了方法了，那就是不要同步输出，自己玩一个异步即可，如果怕异步的输出被卡住导致有很多日志存放在内存，那自己写一个限制日志存放最大数量也是非常简单的事情

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/cbd72d1d76b97fab09ad395f4f8b780e3d5f2fc2/HaybibuqaJadereferejo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/cbd72d1d76b97fab09ad395f4f8b780e3d5f2fc2/HaybibuqaJadereferejo) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cbd72d1d76b97fab09ad395f4f8b780e3d5f2fc2
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cbd72d1d76b97fab09ad395f4f8b780e3d5f2fc2
```

获取代码之后，进入 HaybibuqaJadereferejo 文件夹

更多相关博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。