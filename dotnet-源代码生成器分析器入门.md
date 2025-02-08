
# dotnet 源代码生成器分析器入门

本文将带领大家入门 dotnet 的 SourceGenerator 源代码生成器技术，期待大家阅读完本文能够看懂理解和编写源代码生成器和分析器

<!--more-->


<!-- CreateTime:2025/02/08 08:47:10 -->

<!-- 草稿 -->

恭喜你看到了本文，进入到 C# dotnet 的深水区。如果你还是在浅水玩耍的小鲜肉，推荐你点击右上方的关闭按钮，避免受到过于深入的知识的污染

在开始之前期望大家已经了解基础的 dotnet C# 基础知识，了解基础的概念和项目组织结构

- 项目搭建


- 编写单元测试的方法
- 日常调试的方法 IsRoslynComponent

- 如何打包 NuGet 包

- 使用视觉辅助了解语法

- 演练 使用 Interceptor 的技术
- 演练 将构建时间写入源代码
- 演练 禁用API调用 分析器

- 源代码生成技术实现中文编程语言 生成的源代码保存到本地文件

- 常用的方法
- 获取配置
- 获取文件的实际本地路径 compilation.Options.SourceReferenceResolver 0b5550ce0e2736df6f2aac01f1f65ca37103fbdf Workbench\KonanohallreGonurliyage




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。