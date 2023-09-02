# dotnet SemanticKernel 入门 调用原生本机技能

本文将告诉大家如何在 SemanticKernel 里面调用原生本机技能，所谓原生本机技能就是使用 C# 代码编写的原生本地逻辑技能，这里的技能可讲的可不是游戏角色里面的技能哈，指的是实现某个功能的技能，这是构成 AI 强大能力的基础

<!--more-->
<!-- CreateTime:2023/8/28 8:37:23 -->

<!-- 发布 -->


本文属于 SemanticKernel 入门系列博客，更多博客内容请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html ) 或 [博客园的合集](https://www.cnblogs.com/lindexi/collections/6439)


众所周知 GPT 是一个大语言模型，能够参与的输入和输出是文本内容。而想要让 GPT 完成各项功能，则需要对接现有的编程世界。为了能够更好的复用这几十年的编程世界积累的知识和搭建的基础设施，微软推出 SemanticKernel 框架，通过 SemanticKernel 框架可以让传统的编程语言和 GPT 等 AI 更好的协作，赋予 AI 强大的能力

举个例子来说，当你和 GPT 说，请关灯的时候。此时你期望的也许不是 GPT 长篇大论的帮你关灯，而是更多的期望是 GPT 真的识别到你的意图，通过和你沟通的上下文，帮你将灯给关掉。然而只靠 GPT 本身，则是力不从心的，因为 GPT 本身没有关灯的能力。有关灯能力的是传统 IOT 能力。而通过 SemanticKernel 框架，则可以非常方便在打通 GPT 和关灯 IOT 编程之间的连接。只需要在 SemanticKernel 框架里面加入一个关灯技能，然后告诉 AI 有这个技能，这样 AI 就可以使用这个技能实现关灯的能力

在 SemanticKernel 框架里面定义的技能，都是一个基础的 C# 实现逻辑，比如说本文的例子就是 TextSkill 技能，这个 TextSkill 包含了基础的文本字符串处理功能，比如大小写转换，裁剪文本前后空白，获取字符串长度等等

熟悉 GPT 功能的伙伴大概也不陌生，对于 GPT 来说，所有的输出都是凭 "感觉" 的。而 Skill 技能则可以很好补足这个缺点，让一些确定性的基础逻辑交给传统编程语言编写，如此即可利用传统编程的输入输出稳定性保证输出的可靠性

本文将尝试创建一个项目，在这个项目里面使用 TextSkill 技能作为例子，只使用技能类而还不需要接触 SemanticKernel 框架

先新建一个 dotnet 7 的控制台项目，接着编辑 csproj 项目文件，修改为如下代码用来快速安装 Microsoft.SemanticKernel 库

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net7.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.SemanticKernel" Version="0.20.230821.4-preview" />
  </ItemGroup>

</Project>
```

也可以通过右击项目，点击管理 NuGet 程序包，搜 Microsoft.SemanticKernel 然后进行安装

值得一提的是，当前的 Microsoft.SemanticKernel 还是一个预览版本，有可能在正式发布的时候变更了 API 导致本文的代码在正式版本跑不起来

先按照传统的 C# 编程方法，创建出 TextSkill 对象，如下面代码

```csharp
using Microsoft.SemanticKernel.Skills.Core;

// 创建技能
var text = new TextSkill();
```

接着即可和之前的 C# 代码调用一样，调用 TextSkill 里面提供的方法，比如修改字符串大小写

```csharp
// 直接调用技能里的方法
var result = text.Uppercase("ciao");
```

试试输出结果，自然是很符合预期的所有字符都大写

```csharp
Console.WriteLine(result);
```

所有的代码如下

```csharp
using Microsoft.SemanticKernel.Skills.Core;

// 创建技能
var text = new TextSkill();

// 直接调用技能里的方法
var result = text.Uppercase("ciao");

Console.WriteLine(result);
```

可以看到在没有使用到 SemanticKernel 框架的前提下，这是丝毫没有魔法的。这样的设计可以让咱非常方便进行技能的调试，因为这时候的技能类型就和其他的类型没有什么不相同的。在 [下一篇](https://blog.lindexi.com/post/dotnet-SemanticKernel-%E5%85%A5%E9%97%A8-%E5%B0%86%E6%8A%80%E8%83%BD%E5%AF%BC%E5%85%A5%E6%A1%86%E6%9E%B6.html ) 将告诉大家如何将技能导入框架

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/eef729b3a5ff8017161fdae58a786812ad2a7df0/SemanticKernelSamples/Example01_NativeFunctions) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/eef729b3a5ff8017161fdae58a786812ad2a7df0/SemanticKernelSamples/Example01_NativeFunctions) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin eef729b3a5ff8017161fdae58a786812ad2a7df0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin eef729b3a5ff8017161fdae58a786812ad2a7df0
```

获取代码之后，进入 SemanticKernelSamples\Example01_NativeFunctions 文件夹