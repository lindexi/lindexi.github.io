# Github 添加 Action 编译图标

在我的仓库里面，可以在首页添加图标显示当前是否编译通过

<!--more-->
<!-- CreateTime:2019/12/27 9:58:36 -->

<!-- 发布 -->

在 `README.md` 文件添加下面代码作为图标

```
![](https://github.com/组织或个人/仓库
/workflows/执行Action的Name注意转码/badge.svg)
```

如我在 dotnet-campus 的 dotnetcampus.DotNETBuildSDK 仓库里面的 `.NET Core` 编译任务，可以这样写

```
![](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK
/workflows/.NET%20Core/badge.svg)
```

注意这里的 `.NET Core` 是 Action 对应的 Name 同时需要转码才能用

下面是 Action 配置

```
name: .NET Core

on: [push]

jobs:
  build:

    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v1
 
    - name: Build with dotnet
      run: dotnet build --configuration Release
```

注意 Action 的 Name 不是文件名

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
