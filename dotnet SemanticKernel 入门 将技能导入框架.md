# dotnet SemanticKernel 入门 将技能导入框架

在上一篇博客中和大家简单介绍了 SemanticKernel 里的技能概念，接下来咱准备将 技能 导入到 SemanticKernel 框架里面，进行一个管道式调用

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

本文属于 SemanticKernel 入门系列博客，更多博客内容请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

别着急，本篇博客还不涉及到任何的 GPT 相关的魔法，仅仅只是在 C# 层面上的框架使用而已

按照上一篇博客中的创建项目的例子，也就是安装 Microsoft.SemanticKernel 库之后，就可以开始本篇的代码

使用 SemanticKernel 框架的第一步就是通过 KernelBuilder 创建器创建出 IKernel 对象。可以在 KernelBuilder 创建器配置各项内容，比如日志或者是配置和 AzureAI 或 OpenAI 进行对接的逻辑，甚至配置与第三方大语言模型进行对接

本文这里将采用最简单的啥都不配置的方式，创建 IKernel 对象

```csharp
IKernel kernel = new KernelBuilder().Build();
```

将技能导入 SemanticKernel 框架的方法是调用 IKernel 的 ImportSkill 方法。之所以提供导入技能给到 SemanticKernel 框架是为了可以方便按需导入，避免一口气导入太多的技能，让 GPT 不知道调用哪个。同时也方便咱自己定义和扩展技能

```csharp
// 加载技能
var text = kernel.ImportSkill(new TextSkill());
```

加载技能之后，使用 SemanticKernel 框架提供的管道方式调用了，如以下代码，就是先对字符串进行裁剪再修改为大写

```csharp
SKContext result = await kernel.RunAsync("    i n f i n i t e     s p a c e     ",
    text["TrimStart"],
    text["TrimEnd"],
    text["Uppercase"]);

Console.WriteLine(result);
```

运行代码，可以看到输出如下

```
I N F I N I T E     S P A C E
```

也就是以上代码是按照 `text["TrimStart"]` `text["TrimEnd"]` `text["Uppercase"]` 顺序，分别调用 TextSkill 类型的 TrimStart 和 TrimEnd 和 Uppercase 方法，而且还是将上一个方法的返回值当成下一个方法的输入值

通过 SemanticKernel 框架的这个能力，可以非常方便的提供多个技能进行排列组合，从而完成更加复杂的任务

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/eca54565cd10135ae5acbf6242113ba4e25c2c7c/SemanticKernelSamples/Example02_Pipeline) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/eca54565cd10135ae5acbf6242113ba4e25c2c7c/SemanticKernelSamples/Example02_Pipeline) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin eca54565cd10135ae5acbf6242113ba4e25c2c7c
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin eca54565cd10135ae5acbf6242113ba4e25c2c7c
```

获取代码之后，进入 SemanticKernelSamples\Example02_Pipeline 文件夹