# dotnet SemanticKernel 入门 开篇

本文将开坑告诉大家什么是 SemanticKernel 以及如何使用框架

<!--more-->
<!-- CreateTime:2023/8/28 8:37:23 -->

<!-- 发布 -->
<!-- 博客 -->

众所周知 GPT 是一个大语言模型，能够参与的输入和输出是文本内容。而想要让 GPT 完成各项功能，则需要对接现有的编程世界。为了能够更好的复用这几十年的编程世界积累的知识和搭建的基础设施，微软推出 SemanticKernel 框架，通过 SemanticKernel 框架可以让传统的编程语言和 GPT 等 AI 更好的协作，赋予 AI 强大的能力

举个例子来说，当你和 GPT 说，请关灯的时候。此时你期望的也许不是 GPT 长篇大论的帮你关灯，而是更多的期望是 GPT 真的识别到你的意图，通过和你沟通的上下文，帮你将灯给关掉。然而只靠 GPT 本身，则是力不从心的，因为 GPT 本身没有关灯的能力。有关灯能力的是传统 IOT 能力。而通过 SemanticKernel 框架，则可以非常方便在打通 GPT 和关灯 IOT 编程之间的连接。只需要在 SemanticKernel 框架里面加入一个关灯技能，然后告诉 AI 有这个技能，这样 AI 就可以使用这个技能实现关灯的能力

我的博客的也将从原生技能开始，再到 SemanticKernel 调用 GPT 的对接，再到 AI 与技能的调用逻辑的顺序编写

我的博客里面将尽量采用微软官方提供例子，以及配上可执行的代码，方便大家阅读

我创建了 SemanticKernel 群: 623349574 欢迎大家加入交流

## 应用例子

- [使用Skonsole自动生成Git提交信息 - 宵伯特 - 博客园](https://www.cnblogs.com/xbotter/p/skonsole_intro.html)

- [SemanticKernel 将 LLM 自然语言函数和传统编程融合到一起](https://blog.lindexi.com/post/SemanticKernel-%E5%B0%86-LLM-%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E5%87%BD%E6%95%B0%E5%92%8C%E4%BC%A0%E7%BB%9F%E7%BC%96%E7%A8%8B%E8%9E%8D%E5%90%88%E5%88%B0%E4%B8%80%E8%B5%B7.html )

## 收藏的博客

以下是我收藏的网上大佬们写的博客：

- [Semantic Kernel 入门系列 ❤️‍🔥 - 宵伯特 - 博客园](https://www.cnblogs.com/xbotter/p/semantic_kernel_introduction.html )

- [Semantic Kernel 知多少 开启面向AI编程新篇章 - 哔哩哔哩](https://www.bilibili.com/read/cv22899046/ )