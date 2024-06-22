# dotnet 6 破坏性改动 仅引用程序集输出路径变更

在 dotnet 5 开始，可以设置 ProduceReferenceAssembly 为 true 让项目构建时输出仅引用程序集。仅引用程序集是仅导出项目的公开成员定义，而不包含具体的实现的代码逻辑。只用来被其他项目引用，体积很小，但不用来作为最终发布文件

<!--more-->
<!-- CreateTime:2024/06/22 09:37:47 -->
<!-- 发布 -->
<!-- 博客 -->

在此前的如下博客里面已经告诉大家如何创建仅引用程序集：

- [msbuild 使用 ProduceOnlyReferenceAssembly 创建作为引用的仅公开成员程序集](https://blog.lindexi.com/post/msbuild-%E4%BD%BF%E7%94%A8-ProduceOnlyReferenceAssembly-%E5%88%9B%E5%BB%BA%E4%BD%9C%E4%B8%BA%E5%BC%95%E7%94%A8%E7%9A%84%E4%BB%85%E5%85%AC%E5%BC%80%E6%88%90%E5%91%98%E7%A8%8B%E5%BA%8F%E9%9B%86.html )
- [dotnet 使用 Refasmer 从现有的 DLL 里面导出公开的成员组装出新的仅作为引用用途的程序集](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-Refasmer-%E4%BB%8E%E7%8E%B0%E6%9C%89%E7%9A%84-DLL-%E9%87%8C%E9%9D%A2%E5%AF%BC%E5%87%BA%E5%85%AC%E5%BC%80%E7%9A%84%E6%88%90%E5%91%98%E7%BB%84%E8%A3%85%E5%87%BA%E6%96%B0%E7%9A%84%E4%BB%85%E4%BD%9C%E4%B8%BA%E5%BC%95%E7%94%A8%E7%94%A8%E9%80%94%E7%9A%84%E7%A8%8B%E5%BA%8F%E9%9B%86.html )

从 dotnet 6 开始，不再使用 ProduceOnlyReferenceAssembly 属性，且也无需配置 ProduceReferenceAssembly 属性。默认情况下都将生成仅引用程序集

原本的仅引用程序集是放在输出路径的 ref 文件夹里面。从 dotnet 6 开始，将默认生成仅引用程序集且放在 `$(IntermediateOutputPath)\refint` 文件夹里面，即 `obj\refint` 文件夹里

在 csproj 等代码里面可以使用 `$(TargetRefPath)` 获取仅引用程序集路径，无需自己拼接 `obj\refint` 文件夹

如此更改原因是仅引用程序集大部分情况下都是作为开发侧使用的，不参与实际产品发布，因此如作为最终可输出的文件则不合适。于是 dotnet 6 就将其放入到 obj 文件夹里

详细请参阅 [Breaking change: Write reference assemblies to IntermediateOutputPath - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/compatibility/sdk/6.0/write-reference-assemblies-to-obj )