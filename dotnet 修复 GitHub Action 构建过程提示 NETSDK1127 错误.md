# dotnet 修复 GitHub Action 构建过程提示 NETSDK1127 错误

本文告诉大家，如何修复 GitHub Action 构建过程提示 error NETSDK1127: The targeting pack Microsoft.WindowsDesktop.App.WindowsForms is not installed. Please restore and try again. 错误

<!--more-->
<!-- CreateTime:2024/1/16 9:35:20 -->

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