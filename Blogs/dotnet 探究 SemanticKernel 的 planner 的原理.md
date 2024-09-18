---
title: dotnet 探究 SemanticKernel 的 planner 的原理
description: 在使用 SemanticKernel 时，我着迷于 SemanticKernel 强大的 plan 能力，通过 plan 功能可以让 AI 自动调度拼装多个模块实现复杂的功能。我特别好奇 SemanticKernel 里的 planner 的原理，好奇底层具体是如何实现的。好在 SemanticKernel 是完全开源的，通过阅读源代码，我理解了 SemanticKernel 的工作机制，接下来我将和大家分享我所了解到的原理
tags: dotnet
category: 
---

<!-- CreateTime:2023/11/5 15:16:14 -->
<!-- 发布 -->
<!-- 博客 -->

从最底层的非玄学逻辑来说，可以认为 SemanticKernel 的底层通过 GPT 等 AI 层的输入和输出仅仅只有文本而已，而 Planner 需要执行编排调度多个功能任务从而实现功能。最方便理解的就是预先告诉 AI 层，当前有哪些功能或能力，接下来让 AI 决定这些功能和能力应该如何调度从而满足需求

换句话说就是作为工程师的人类提供了各种各样的功能能力，作为提出需求的用户人类给需求描述，接下来作为 AI 将根据用户输入的需求描述，配合工程师提供的各种功能能力完成用户的需求

比如说实现使用某个语言的作诗需求，用户的需求描述大概就是作一首什么样的诗，然后翻译为什么语言。这时候工程师提供的是一个作诗函数或插件，以及一个翻译的函数或插件。然后由 AI 层进行编排调度，先调用作诗函数进行作诗，接着将作诗结果作为翻译函数的翻译进行翻译，最后将翻译结果返回给到用户

以上这个需求在有 SemanticKernel 的辅助下，将会非常简单实现

接下来咱来尝试在不使用 SemanticKernel 提供的 Plan 工具的前提下，完成类似的功能。通过自己编写代码的方式代替 SemanticKernel 提供的 Plan 的功能，从而了解 SemanticKernel 的实现细节

大概的原理实现步骤如下图

<!-- ![](image/dotnet 探究 SemanticKernel 的 planner 的原理/dotnet 探究 SemanticKernel 的 planner 的原理0.png) -->

![](http://cdn.lindexi.site/lindexi%2F2023115172604099.jpg)

先按照 [dotnet SemanticKernel 入门 将技能导入框架](https://blog.lindexi.com/post/dotnet-SemanticKernel-%E5%85%A5%E9%97%A8-%E5%B0%86%E6%8A%80%E8%83%BD%E5%AF%BC%E5%85%A5%E6%A1%86%E6%9E%B6.html ) 博客提供的方法，向 SemanticKernel 框架里面导入两个 SemanticFunction 函数，分别是作诗和翻译

```csharp
kernel.RegisterSemanticFunction("WriterPlugin", "ShortPoem", new PromptTemplateConfig()
{
    Description = "Turn a scenario into a short and entertaining poem.",
}, new PromptTemplate(
    @"Generate a short funny poem or limerick to explain the given event. Be creative and be funny. Let your imagination run wild.
Event:｛｛$input｝｝
", new PromptTemplateConfig()
    {
        Input = new PromptTemplateConfig.InputConfig()
        {
            Parameters = new List<PromptTemplateConfig.InputParameter>()
            {
                new PromptTemplateConfig.InputParameter()
                {
                    Name = "input",
                    Description = "The scenario to turn into a poem.",
                }
            }
        }
    }, kernel));

kernel.CreateSemanticFunction(@"Translate the input below into ｛｛$language｝｝

MAKE SURE YOU ONLY USE ｛｛$language｝｝.

｛｛$input｝｝

Translation:
", new PromptTemplateConfig()
{
    Input = new PromptTemplateConfig.InputConfig()
    {
        Parameters = new List<PromptTemplateConfig.InputParameter>()
        {
            new PromptTemplateConfig.InputParameter()
            {
                Name = "input",
            },
            new PromptTemplateConfig.InputParameter()
            {
                Name = "language",
                Description = "The language which will translate to",
            }
        }
    },
    Description = "Translate the input into a language of your choice",
}, functionName: "Translate", pluginName: "WriterPlugin");
```

以上的 SemanticFunction 的炼丹内容来源于 SemanticKernel 官方仓库的例子

通过以上代码即可注册 `WriterPlugin.ShortPoem` 以及 `WriterPlugin.Translate` 两个函数。大家可以看到在注册这两个函数的过程中，还很详细写出了这两个函数的功能描述，以及他的各个参数和参数的描述。这些描述内容就是专门用来给 AI 层阅读的，方便让 AI 层理解这些函数的功能，从而让 AI 层知道如何调用这些函数

原本我是先使用中文编写以上的 SemanticFunction 实现内容的，然而我的炼丹水平不过关，写不出一个好的例子，于是就使用官方的例子好了。以上函数里面的英文描述不是本文的重点，大家要是看不懂就请跳过，只需要知道预先准备了这两个函数就可以

完成准备工作之后，接下来咱将开始编写 Plan 的核心逻辑。核心实现其实也是一个类似 SemanticFunction 的功能，请了百万炼丹师编写了提示词内容，用来告诉 AI 层需要创建一个 XML 结构，这个 XML 结构里面就包含了如何进行调度的逻辑，以及各项参数应该传入什么值。由于我请不起百万炼丹师，于是只好白嫖微软的百万炼丹师的提示词

```csharp
var semanticFunction = kernel.CreateSemanticFunction(
    @"Create an XML plan step by step, to satisfy the goal given, with the available functions.

[AVAILABLE FUNCTIONS]

｛｛$available_functions｝｝

[END AVAILABLE FUNCTIONS]

To create a plan, follow these steps:
0. The plan should be as short as possible.
1. From a <goal> create a <plan> as a series of <functions>.
2. A plan has 'INPUT' available in context variables by default.
3. Before using any function in a plan, check that it is present in the [AVAILABLE FUNCTIONS] list. If it is not, do not use it.
4. Only use functions that are required for the given goal.
5. Append an ""END"" XML comment at the end of the plan after the final closing </plan> tag.
6. Always output valid XML that can be parsed by an XML parser.
7. If a plan cannot be created with the [AVAILABLE FUNCTIONS], return <plan />.

All plans take the form of:
<plan>
    <!-- ... reason for taking step ... -->
    <function.{FullyQualifiedFunctionName} ... />
    <!-- ... reason for taking step ... -->
    <function.{FullyQualifiedFunctionName} ... />
    <!-- ... reason for taking step ... -->
    <function.{FullyQualifiedFunctionName} ... />
    (... etc ...)
</plan>
<!-- END -->

To call a function, follow these steps:
1. A function has one or more named parameters and a single 'output' which are all strings. Parameter values should be xml escaped.
2. To save an 'output' from a <function>, to pass into a future <function>, use <function.{FullyQualifiedFunctionName} ... setContextVariable=""<UNIQUE_VARIABLE_KEY>""/>
3. To save an 'output' from a <function>, to return as part of a plan result, use <function.{FullyQualifiedFunctionName} ... appendToResult=""RESULT__<UNIQUE_RESULT_KEY>""/>
4. Use a '$' to reference a context variable in a parameter, e.g. when `INPUT='world'` the parameter 'Hello $INPUT' will evaluate to `Hello world`.
5. Functions do not have access to the context variables of other functions. Do not attempt to use context variables as arrays or objects. Instead, use available functions to extract specific elements or properties from context variables.

DO NOT DO THIS, THE PARAMETER VALUE IS NOT XML ESCAPED:
<function.Name4 input=""$SOME_PREVIOUS_OUTPUT"" parameter_name=""some value with a <!-- 'comment' in it-->""/>

DO NOT DO THIS, THE PARAMETER VALUE IS ATTEMPTING TO USE A CONTEXT VARIABLE AS AN ARRAY/OBJECT:
<function.CallFunction input=""$OTHER_OUTPUT[1]""/>

Here is a valid example of how to call a function ""_Function_.Name"" with a single input and save its output:
<function._Function_.Name input=""this is my input"" setContextVariable=""SOME_KEY""/>

Here is a valid example of how to call a function ""FunctionName2"" with a single input and return its output as part of the plan result:
<function.FunctionName2 input=""Hello $INPUT"" appendToResult=""RESULT__FINAL_ANSWER""/>

Here is a valid example of how to call a function ""Name3"" with multiple inputs:
<function.Name3 input=""$SOME_PREVIOUS_OUTPUT"" parameter_name=""some value with a &lt;!-- &apos;comment&apos; in it--&gt;""/>

Begin!

<goal>｛｛$input｝｝</goal>
");
```

以上的提示词内容也就是先插入名为 `available_functions` 的内容，将在后面被替换为当前可用的函数列表。接着就是告诉 AI 层如何制定计划，输出的 XML 格式应该是怎样的，还给他提供了一个例子，如下面代码

```
<plan>
    <!-- ... reason for taking step ... -->
    <function.{FullyQualifiedFunctionName} ... />
    <!-- ... reason for taking step ... -->
    <function.{FullyQualifiedFunctionName} ... />
    <!-- ... reason for taking step ... -->
    <function.{FullyQualifiedFunctionName} ... />
    (... etc ...)
</plan>
```

以及告诉 AI 层应该写什么以及不应该输出什么。以上的提示词内容看起来是经过了微软官方精心的设计的，我随便写的几个提示词都达不到以上的效果

由于我担心博客引擎因为两个 `{` 挂掉，于是我就将 `{` 换成全角的 `｛` 符号，实际使用中还是使用标准的 `{` 字符

完成了核心逻辑提示词的编写，创建了一个智能函数，接下来咱尝试调用这个智能函数实现功能

在开始之前，先注入可被使用的函数列表，如以下代码，通过 GetFunctionsManualAsync 方法即可导出当前注册到 SemanticKernel 里的各个函数，无论是 SemanticFunction 还是 NativeFunction 本机函数

```csharp
var relevantFunctionsManual = await kernel.Functions.GetFunctionsManualAsync(new SequentialPlannerConfig());
```

以上的 GetFunctionsManualAsync 方法将会返回注册进入的各个函数，以及函数的描述和函数的输入参数和参数描述，大概内容如下面代码

```
WriterPlugin.ShortPoem:
  description: Turn a scenario into a short and entertaining poem.
  inputs:
    - input: The scenario to turn into a poem.

WriterPlugin.Translate:
  description: Translate the input into a language of your choice
  inputs:
    - input: 
    - language: The language which will translate to
```

通过以上的输出内容，相信大家也就能理解为什么在定义 SemanticKernel 的函数时，需要编写函数的描述的原因了，不仅仅这些描述可以给人类阅读使用，同时也可以给机器阅读

将以上的输出代码放入到 `available_functions` 变量里面，从而让 AI 层了解到当前有哪些可以被使用的函数

```csharp
ContextVariables vars = new(goal)
{
    ["available_functions"] = relevantFunctionsManual
};
```

以上代码的 goal 变量是用户的输入需求，在这里也就是帮忙写一首诗，然后翻译为中文的需求，定义的代码如下

```csharp
var goal = "Write a poem about John Doe, then translate it into Chinese.";
```

或者这里可以直接输入中文的需求

```csharp
var goal = "帮忙写一首关于水哥的诗, 然后翻译为中文";
```

输入需求之后开始跑一下百万炼丹师的智能函数

```csharp
ContextVariables vars = new(goal)
{
    ["available_functions"] = relevantFunctionsManual
};

var planResult = await kernel.RunAsync(semanticFunction, vars);
string? planResultString = planResult.GetValue<string>()?.Trim();
```

以上拿到的 `planResultString` 就是 AI 层输出的计划调度 XML 配置结果了，大概内容如下

```xml
<plan>
    <!-- First, we create a short poem about "水哥" -->
    <function.WriterPlugin.ShortPoem input="水哥" setContextVariable="POEM"/>
    <!-- Then, we translate the poem into Chinese -->
    <function.WriterPlugin.Translate input="$POEM" language="Chinese" appendToResult="RESULT__FINAL_ANSWER"/>
</plan>
```

接下来咱需要编写一些 C# 代码，根据以上输出的 XML 调度任务转换为一个个的 Plan 任务，进行更细节的调度执行

```csharp
var xmlString = planResultString;
XmlDocument xmlDoc = new();
xmlDoc.LoadXml("<xml>" + xmlString + "</xml>");
XmlNodeList solution = xmlDoc.GetElementsByTagName("plan");
```

将逻辑转为 XML 之后，接下来就是有手就行，根据 XML 里面提到的函数以及参数进行调度和配置。对 XML 的解析毫无难度，相信大家一看需求就知道如何编写代码，而解析完成之后的具体执行，这时候就换成了在 SemanticKernel 里面如何执行函数的问题，相信这也是大家所熟悉的

为了更加方便了解咱这个实现的效果，以下代码我继续使用了 SemanticKernel 的 Plan 类型，方便快速导入实现

```csharp
XmlNodeList solution = xmlDoc.GetElementsByTagName("plan");

var plan = new Plan(goal);

foreach (XmlNode solutionNode in solution)
{
    foreach (XmlNode childNode in solutionNode.ChildNodes)
    {
        if (childNode.Name == "#text" || childNode.Name == "#comment")
        {
            // Do not add text or comments as steps.
            // TODO - this could be a way to get Reasoning for a plan step.
            continue;
        }

        if (childNode.Name.StartsWith("function.", StringComparison.OrdinalIgnoreCase))
        {
            var pluginFunctionName = childNode.Name.Split(new string[] { "function." }, StringSplitOptions.None)?[1] ?? string.Empty;
            SplitPluginFunctionName(pluginFunctionName, out var pluginName, out var functionName);

            if (!string.IsNullOrEmpty(functionName))
            {
                var function = kernel.Functions.GetFunction(pluginName,functionName);
                if (function != null)
                {
                    var planStep = new Plan(function);

                    var functionVariables = new ContextVariables();
                    var functionOutputs = new List<string>();
                    var functionResults = new List<string>();

                    var view = function.Describe();
                    foreach (var p in view.Parameters)
                    {
                        functionVariables.Set(p.Name, p.DefaultValue);
                    }

                    if (childNode.Attributes is not null)
                    {
                        foreach (XmlAttribute attr in childNode.Attributes)
                        {
                            if (attr.Name.Equals("setContextVariable", StringComparison.OrdinalIgnoreCase))
                            {
                                functionOutputs.Add(attr.InnerText);
                            }
                            else if (attr.Name.Equals("appendToResult", StringComparison.OrdinalIgnoreCase))
                            {
                                functionOutputs.Add(attr.InnerText);
                                functionResults.Add(attr.InnerText);
                            }
                            else
                            {
                                functionVariables.Set(attr.Name, attr.InnerText);
                            }
                        }
                    }

                    planStep.Outputs = functionOutputs;
                    planStep.Parameters = functionVariables;
                    foreach (var result in functionResults)
                    {
                        plan.Outputs.Add(result);
                    }

                    foreach (var result in functionResults)
                    {
                        plan.Outputs.Add(result);
                    }

                    plan.AddSteps(planStep);
                }
            }
        }
    }
}

Console.WriteLine(await kernel.RunAsync(plan));

static void SplitPluginFunctionName(string pluginFunctionName, out string pluginName, out string functionName)
{
    var pluginFunctionNameParts = pluginFunctionName.Split('.');
    pluginName = pluginFunctionNameParts?.Length > 1 ? pluginFunctionNameParts[0] : string.Empty;
    functionName = pluginFunctionNameParts?.Length > 1 ? pluginFunctionNameParts[1] : pluginFunctionName;
}
```

由于 SemanticKernel 的 Plan 的数据结构上是允许 Plan 里面套 Plan 的，于是就直接和 XML 的结构对应起来，注册各个函数掉算过程进去

最后依然使用的 SemanticKernel 的执行 Plan 的方法完成所有的功能，在 SemanticKernel 里面执行 Plan 就是按照步骤逐个递归 Plan 执行，执行的最底层依然都是 SemanticKernel 的函数

编写代码到这里，相信大家也就看出来 SemanticKernel 的 planner 的原理就是由百万炼丹师写出提示词内容，将用户输入的需求，先转换为 XML 格式的计划调度，接着编写 C# 代码解析 XML 内容，从 XML 转换为 Plan 类型，接着根据 Plan 对象逐个步骤调用，从而完成用户的需求

以上代码运行的输出结果大概如下，欢迎大家换成其他人的名字去试试输出结果

```
在一个说普通话的土地上，
住着一个名叫水哥的人，他是个狂热的粉丝，
对于清澈的水，
他会笑，他会欢呼，
整天嬉水，就像只有水人才能做的那样。

他会跳进湖里，发出大声的吼叫，
在河里游泳，从这岸到那岸，
在海里，他会欢蹦乱跳，
在雨中，他会跳舞，
哦，水哥热爱水，这点可以肯定！

他会在水坑里洗澡，如此快乐，
或者从小溪里喝水，如此平静，
有溅水声和溅水声，
还有一点火锅汤，
水哥，这个水人，生活得如此快乐！
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/f4448f4507145f1695b7ef81045ae030fc8f1a20/SemanticKernelSamples/Example12_Planner) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f4448f4507145f1695b7ef81045ae030fc8f1a20/SemanticKernelSamples/Example12_Planner) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f4448f4507145f1695b7ef81045ae030fc8f1a20
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f4448f4507145f1695b7ef81045ae030fc8f1a20
```

获取代码之后，进入 SemanticKernelSamples\Example12_Planner 文件夹
