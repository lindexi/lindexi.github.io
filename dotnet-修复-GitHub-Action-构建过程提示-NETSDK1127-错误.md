
# dotnet 修复 GitHub Action 构建过程提示 NETSDK1127 错误

本文告诉大家，如何修复 GitHub Action 构建过程提示 error NETSDK1127: The targeting pack Microsoft.WindowsDesktop.App.WindowsForms is not installed. Please restore and try again. 错误

<!--more-->


<!-- 博客 -->
<!-- 发布 -->

在进行 GitHub Action 构建时，如果自己的项目是一个旧项目，采用旧的 .NET SDK 版本，将可以由于 GitHub Action 使用新的构建系统，缺乏旧的 SDK 导致构建失败

失败的错误信息输出例子如下

```
error NETSDK1127: The targeting pack Microsoft.WindowsDesktop.App.WindowsForms is not installed. Please restore and try again. [D:\a\X\X\NewLife.Core\NewLife.Core.csproj::TargetFramework=net6.0-windows]
```

简单的修复方法是给 GitHub 的 Action 的构建 yml 里加上 `Setup dotNET` 步骤，其 yml 添加的代码如下

```yml
    - name: Setup dotNET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: |
          6.0.x
          7.0.x
          8.x
```

以上的 dotnet-version 请罗列出你所需的 SDK 版本，当然了，罗列太多的话，安装时间也不短

以下是一份完全的构建 yml 文件的例子代码

```yml
name: BuildAndTest

on: [push]

jobs:
  build:

    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v1
    - name: Setup dotNET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: |
          6.0.x
          7.0.x
          8.x

    - name: Build
      run: dotnet build --configuration Release

    - name: Test
      run: dotnet test --configuration Release
```

具体的修复构建的 PR 请参阅 [尝试修复 CI 构建 by lindexi · Pull Request #135 · NewLifeX/X](https://github.com/NewLifeX/X/pull/135 )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。