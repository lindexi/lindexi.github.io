# C＃命令行解析工具

我将告诉大家两个方法去获取C#输入的命令行参数。



<!--more-->

第一个方法：

[林选臣](http://www.cnblogs.com/linxuanchen/p/c-sharp-command-line-argument-parser.html)大神写的，他的方法很简单。

第二个方法需要使用 Nuget


```csharp
  Install-Package CommandLineParser
```

然后写一个工具类，代码是在[申龙斌大神博客](http://www.cnblogs.com/speeding/archive/2012/08/07/2626066.html) 抄的


```csharp
   class Options

   {

        // 短参数名称，长参数名称，是否是可选参数，默认值，帮助文本等

        // 第一个参数-d

        [Option("d", "dir", Required = true, HelpText = "PGN Directory to read.")]

        public string PgnDir { get; set; }

        // 第二个参数-s

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

 主程序Main里使用

```csharp
 

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



如何使用参见：http://www.cnblogs.com/speeding/archive/2012/08/07/2626066.html



参见：http://www.cnblogs.com/linxuanchen/p/c-sharp-command-line-argument-parser.html

http://www.cnblogs.com/speeding/archive/2012/08/07/2626066.html