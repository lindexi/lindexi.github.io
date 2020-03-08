# C＃命令行解析工具

我将告诉大家两个方法去获取C#输入的命令行参数。

<!--more-->
<!-- CreateTime:2020/2/2 9:06:56 -->


<div id="toc"></div>

第一个方法是不需要安装任何的库，这个方法是性能很高，但是封装不好。第二个方法是使用 CommandLineParser 库，这个库提高很好的封装但是性能会比较差。

第一个方法：

[林选臣](http://www.cnblogs.com/linxuanchen/p/c-sharp-command-line-argument-parser.html)大神写的，他的方法很简单。

首先复制两个类到项目

```csharp
      public class CommandLineArgumentParser
    {
        private readonly List<CommandLineArgument> _arguments;

        public CommandLineArgumentParser(string[] args)
        {
            _arguments = new List<CommandLineArgument>();

            for (var i = 0; i < args.Length; i++)
            {
                _arguments.Add(new CommandLineArgument(_arguments, i, args[i]));
            }
        }

        public static CommandLineArgumentParser Parse(string[] args)
        {
            return new CommandLineArgumentParser(args);
        }

        public CommandLineArgument Get(string argumentName)
        {
            return _arguments.FirstOrDefault(p => p == argumentName);
        }

        public IEnumerable<CommandLineArgument> GetEnumerator()
        {
            foreach (var temp in _arguments)
            {
                yield return temp;
            }
        }

        public bool Has(string argumentName)
        {
            return _arguments.Any(temp => temp == argumentName);
        }
    }

    public class CommandLineArgument
    {
        private readonly List<CommandLineArgument> _arguments;

        private readonly string _argumentText;

        private readonly int _index;

        internal CommandLineArgument(List<CommandLineArgument> args, int index, string argument)
        {
            _arguments = args;
            _index = index;
            _argumentText = argument;
        }

        public CommandLineArgument Next
        {
            get
            {
                if (_index < _arguments.Count - 1)
                {
                    return _arguments[_index + 1];
                }

                return null;
            }
        }

        public CommandLineArgument Previous
        {
            get
            {
                if (_index > 0)
                {
                    return _arguments[_index - 1];
                }

                return null;
            }
        }

        public CommandLineArgument Take()
        {
            return Next;
        }

        public IEnumerable<CommandLineArgument> Take(int count)
        {
            var list = new List<CommandLineArgument>();
            var parent = this;
            for (var i = 0; i < count; i++)
            {
                var next = parent.Next;
                if (next == null)
                    break;

                list.Add(next);

                parent = next;
            }

            return list;
        }

        public static implicit operator string(CommandLineArgument argument)
        {
            return argument?._argumentText;
        }

        public override string ToString()
        {
            return _argumentText;
        }
    }


```

复制完成就可以使用，在主函数可以使用下面代码进行转换，下面代码的 `args` 就是传入的参数字符串数组。


```csharp
    var arguments = CommandLineArgumentParser.Parse(args);
```

如果需要获得参数的信息，可以使用下面代码，也就是使用 Get 方法传入一个指定的值，通过这个值就可以拿到这个值的参数

```csharp
var f = arguments.Get("--lindexi").Take();
```

如命令输入`--lindexi doubi`，上面代码就可以拿到`doubi`，虽然使用这个库的写法的封装不是很好，但是性能很好。下面告诉大家使用另一个方法，十分容易写，但是性能比较差。

第二个方法需要使用 Nuget 安装 CommandLineParser 库，可以在控制台输入下面代码安装


```csharp
  Install-Package CommandLineParser
```

然后写一个工具类，代码是在[申龙斌大神博客](http://www.cnblogs.com/speeding/archive/2012/08/07/2626066.html) 抄的


```csharp
   class Options
   {

        // 短参数名称，长参数名称，是否是可选参数，默认值，帮助文本等

        // 第一个参数-d 如果使用比较高的 .net 那么第一个参数可能是 char 而不是字符串

        [Option("d", "dir", Required = true, HelpText = "PGN Directory to read.")]
        public string PgnDir { get; set; }

        // 新的版本使用的是 char 来作为第一个字符，也就是需要修改 "d" 为 'd' 字符
        // [Option('d', "dir", Required = true, HelpText = "PGN Directory to read.")]
        // public string PgnDir { get; set; }

        // 第二个参数-s 也可以使用 --step 

        [Option("s", "step", DefaultValue = 30, HelpText = "The maximum steps in PGN game to process.")]
        public int MaxStep { get; set; }

        [HelpOption]
        public string GetUsage()
        {

            // 应该可以根据前面的参数设置自动生成使用说明的，这里没有使用

            var usage = new StringBuilder();

            usage.AppendLine("OpeningBook 1.0");

            usage.AppendLine("-d PgnDir [-s MaxSteps=30]");

            return usage.ToString();
        }
}
```

主程序Main里使用可以使用下面代码，这里的 args 数组就是主函数传入函数。下面的代码是老版本的写法

```csharp

// 老版本的写法

var options = new Options();

if (Parser.Default.ParseArguments(args, options))
{

     string pgnDir = options.PgnDir;

     int maxStep = options.MaxStep;

     // 参数取出来了，可以随便使用了

     // 本例中参数比较简单，稍微有点大材小用了

}
else  
{
//转化失败

      Console.WriteLine(options.GetUsage());

}
```

上面的代码是老的版本，现在更新 2.2.0 版本是需要修改代码

```csharp
    class Options
    {
        // 注意 'd' 用的是字符
        // 短参数名称，长参数名称，是否是可选参数，默认值，帮助文本等

        [Option('d', "dir", Required = true, HelpText = "PGN Directory to read.")]
        public string PgnDir { get; set; }

        // 第二个参数-s

        [Option('s', "step", HelpText = "The maximum steps in PGN game to process.")]
        public int MaxStep { get; set; }
    }
```

原来的 DefaultValue 修改为 Default 可以传入任何类型

在 Main 函数需要修改代码

```csharp
            CommandLine.Parser.Default.ParseArguments<Options>(args)
                .WithParsed(options =>
                {
                    var step = options.MaxStep;
                })
                .WithNotParsed(errorList =>
                {
                    
                });
```

在 WithParsed 就是解析成功的委托，在 WithNotParsed 就是解析失败的

如何使用参见：http://www.cnblogs.com/speeding/archive/2012/08/07/2626066.html

[Quickstart · gsscoder/commandline Wiki](https://github.com/gsscoder/commandline/wiki/Quickstart)

参见：http://www.cnblogs.com/linxuanchen/p/c-sharp-command-line-argument-parser.html

http://www.cnblogs.com/speeding/archive/2012/08/07/2626066.html

[安利一款命令行参数](https://walterlv.github.io/post/microsoft-extensions-commandlineutils.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  