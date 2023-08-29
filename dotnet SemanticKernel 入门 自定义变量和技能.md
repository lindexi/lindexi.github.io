# dotnet SemanticKernel 入门 自定义变量和技能

本文将告诉大家如何在 SemanticKernel 框架内定义自定义的变量和如何开发自定义的技能

<!--more-->
<!-- CreateTime:2023/8/28 8:37:23 -->

<!-- 发布 -->
<!-- 博客 -->

本文属于 SemanticKernel 入门系列博客，更多博客内容请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

自定义变量是一个非常有用的技能，自定义变量可以让炼丹师和程序员进行并行工作。由炼丹师对 AI 模型进行训练，从而找到对某项问题比较好的解决方案，炼丹师此时不需要关注具体所解决的问题，可以大量使用占位符代替用户的实际输入。而程序员则可以用到炼丹师的成果进而替换占位符为具体的用户输入，从而实现功能。下面举一个具体的例子

比如说对 GPT 进行 prompt 炼丹，最后炼出一个分类器，这个分类器可以根据用户的输入内容进行分类。但是在炼丹的过程炼丹师是不会也不可能拿到所有用户可能的输入的，用户可能的输入是在炼丹师完成炼丹工作之后，程序员开发人机交互应用最后才能拿到用户的实际输入内容。这个时候自定义变量的功能相信大家就能知道了，通过自定义变量的功能，让炼丹师可以方便插入占位符，从而程序员进行对接

更进一步，自定义变量还可以更加方便技能的导入，由于许多技能都可以输入不止一个的输入内容，在有自定义变量的辅助之下，即可完成更加复杂的管道逻辑。比如说在经过某个技能之后，可以进行多变量输入和多变量输出，于是就可以传递更多丰富的信息给到后续的步骤。比如说 GPT 的分类功能，可以在分类之后对不同类别输出不同的输入要求，从而满足对接后续的技能。举一个具体的例子

比如说训练 GPT 可以输出用户的输入内容，将用户的输入类型分类为 1. 总结文本内容 2. 根据文字生成图片 等，通过对接自定义变量功能时。可以让炼丹师不需要关注特定的输入，而是统一采用 input 变量当成用户的输入，然后在完成之后，注入用户输入类型作为 type 变量。如果用户是总结文本内容的需求，则再添加 text 变量。如果是根据文字生成图片，则注入 request 变量和 size 尺寸变量。于是程序员可以编程开发，给 SemanticKernel 框架先设置 input 变量，这个 input 变量就是用户输入的文本内容。接着执行 GPT 的智能逻辑，再读取自定义变量 type 拿到用户期望执行的类型，分别调用不同的技能函数。比如说总结文本技能就需要用到 text 变量作为输入，而根据文字生成图片则需要 request 变量和 size 尺寸变量，这些都可以在一个顺序之中完成

看到这里，大家是不是想要试试看 SemanticKernel 框架赋予 AI 的强大能力了？放心，本文现在还不涉及到任何 AI 相关的逻辑，依然还在探索 SemanticKernel 框架的过程

先和大家介绍一下如何在 SemanticKernel 框架里面注入自定义变量。在 SemanticKernel 框架里面默认将所有的输入当成了 input 变量，也就是后续介绍到的 SemanticKernel 的 SK 函数里面，如果只有一个默认参数，那么这个参数将会被同时赋给 input 变量

在 SemanticKernel 注入自定义变量的方法可以是先新建 ContextVariables 对象，通过变量上下文对象进行类似字典的方式赋值。默认给到 ContextVariables 构造函数的就是后续会被当成 input 的变量，如以下代码。下面代码将演示在没有任何 AI 参与的情况下，输出今天的日期

```csharp
var variables = new ContextVariables("今天是: ");
variables.Set("day", DateTime.Now.ToString(CultureInfo.CurrentCulture));
```

如以上的代码就定义了 input 是 `"今天是: "` 而 day 变量就是通过 Set 方法设置为 `DateTime.Now.ToString(CultureInfo.CurrentCulture)` 的返回值

也就是说如果有逻辑能够将 input 和 day 拼接在一起，就能够完成一句话。当然，更多的时候变量是用来提供给到 AI 使用的。现在咱还不想使用魔法，先看看如果是纯写传统代码的情况下，如何完成这个功能

在聊到自定义变量的时候，就肯定会聊到自定义技能功能了。在之前的博客里面，大家也看到了调用框架自带的技能的方法，接下来我将和大家介绍如何自定义技能

自定义技能是 SemanticKernel 框架所强大的地方，通过自定义技能即可将 AI 和传统编程联系在一起，下面让咱编写一个技能，这个技能的用途是将日期追加到输入字符串里面

自定义技能的做法是创建一个方法且标记 SKFunction 特性，可选的加上描述信息。这个描述信息现在也只是给程序员看的，据说后面微软将准备出一个 GUI 设计器，这时候对技能的描述就可以更加方便给非编程的工程师进行开发 AI 功能

```csharp
class StaticTextSkill
{
    [SKFunction, Description("追加 day 变量到字符串")]
    public static string AppendDay
    (
        [Description("准备被追加的文本")] string input,
        [Description("追加到文本后面的字符串")]
        string day
    )
        => input + day;
}
```

以上的 AppendDay 方法里面的参数将会被 SemanticKernel 框架使用反射的方式进行注入，注入的就是参数名对应的变量。简单来说就是从 ContextVariables 里面尝试通过参数名获取到参数。如果没能从 ContextVariables 里读取到和参数名对应的变量，则会记录一个错误信息，例如当技能函数里面丢失了一个名为 xx 的变量时的输出如下

```
Missing value for parameter 'xx'
```

在一个类型里面可以定义许多个技能函数，此时就可以通过导出技能类然后使用技能类里面的多个技能方法，如下面代码定义一个非静态的技能方法在 StaticTextSkill 类型里面

```csharp
class StaticTextSkill
{
    [SKFunction, Description("将所有的文本字符串修改为大写")]
    public string Uppercase([Description("准备修改为大写的文本")] string input) =>
        input.ToUpperInvariant();

    [SKFunction, Description("追加 day 变量到字符串")]
    public static string AppendDay
    (
        [Description("准备被追加的文本")] string input,
        [Description("追加到文本后面的字符串")]
        string day
    )
        => input + day;
}
```

接下来需要做的就是将 ContextVariables 放入到 SemanticKernel 框架，通过管道方式调用 StaticTextSkill 技能，如以下代码

```csharp
IKernel kernel = new KernelBuilder().Build();
var text = kernel.ImportSkill(new StaticTextSkill(), "text");

var variables = new ContextVariables("今天是: ");
variables.Set("day", DateTime.Now.ToString(CultureInfo.CurrentCulture));

SKContext result = await kernel.RunAsync(variables,
    text["AppendDay"],
    text["Uppercase"]);

Console.WriteLine(result);
```

执行以上代码，即可看到输出了今天的时间

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/84c1e073be77bee177607596b5e03cabb0c0a719/SemanticKernelSamples/Example03_Variables) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/84c1e073be77bee177607596b5e03cabb0c0a719/SemanticKernelSamples/Example03_Variables) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 84c1e073be77bee177607596b5e03cabb0c0a719
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 84c1e073be77bee177607596b5e03cabb0c0a719
```

获取代码之后，进入 SemanticKernelSamples\Example03_Variables 文件夹