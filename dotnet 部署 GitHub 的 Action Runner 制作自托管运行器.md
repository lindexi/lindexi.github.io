# dotnet 部署 GitHub 的 Action Runner 制作自托管运行器

本文告诉大家如何在自己的 CI 服务器上部署一个私有的 GitHub Action Runner 用来执行 GitHub 上的仓库的构建

<!--more-->
<!-- CreateTime:2020/12/15 10:22:10 -->

<!-- 发布 -->

## 安装

为了作为一个 dotnet 的 GitHub Action Runner 的服务器，首先需要在自己的 CI 服务器上安装足够的负载。我下载了 VS 安装了所有能装的功能

而 GitHub 的 Action Runner 运行器需要从 GitHub 仓库拉下来代码，此时就需要本地有全局配置了 Git 工具，在 [https://git-scm.com/](https://git-scm.com/) 载安装最新版本的 Git 工具

## 部署

根据官方文档 [添加自托管的运行器 - GitHub Docs](https://docs.github.com/cn/free-pro-team@latest/actions/hosting-your-own-runners/adding-self-hosted-runners ) 进行部署

以下是我的部署代码

```ps
# Create a folder under the drive root
$ mkdir actions-runner; cd actions-runner
# Download the latest runner package
$ Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.274.2/actions-runner-win-x64-2.274.2.zip -OutFile actions-runner-win-x64-2.274.2.zip
# Extract the installer
$ Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD/actions-runner-win-x64-2.274.2.zip", "$PWD")
```

上面代码是放在 PowerShell 上一条条执行的，在 PowerShell 里面 `#` 代表这一行是注释。而 `$` 表示这是一行命令，因此咱只需要在 PowerShell 里面输入 `$` 后面的内容

其实上面代码只是从 [https://github.com/actions/runner/releases/download/v2.274.2/actions-runner-win-x64-2.274.2.zip](https://github.com/actions/runner/releases/download/v2.274.2/actions-runner-win-x64-2.274.2.zip) 下载运行器，下载完成之后解压缩到文件夹

而使用 `Invoke-WebRequest` 的下载速度不够快，此时我和大家安利的下载工具 dotnetCampus.FileDownloader 工具。这是一个纯 dotnet 开发的 dotnet tool 工具，在 GitHub 上完全开源，请看 [https://github.com/dotnet-campus/dotnetCampus.FileDownloader](https://github.com/dotnet-campus/dotnetCampus.FileDownloader)

安装此下载工具可以使用下面命令

```csharp
dotnet tool install -g dotnetCampus.FileDownloader.Tool
```

安装完成之后，可以使用下面命令下载，这个下载器提供了多线程下载

```
downloadfile -u https://github.com/actions/runner/releases/download/v2.274.2/actions-runner-win-x64-2.274.2.zip -o actions-runner-win-x64-2.274.2.zip 
```

下载完成之后，解压缩到文件夹，调用 config.cmd 进行配置

## 配置

在你的 GitHub 的 Action 配置界面里面，可以看到配置的命令内容，不同的开发者的配置内容不同

```
./config.cmd --url https://github.com/dotnet-campus --token AD2PSJSDOSETWXBS3M7GEVK73ATKS
```

上面代码请不要抄，因此你的配置一定和我的参数不相同

配置基本上一路下一步按回车就可以

为了在服务器上有足够的权限运行脚本，还需要使用管理员权限打开 PowerShell 输入下面代码进行配置

```
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope LocalMachine
```

以上命令能解决构建的时候提示 `The 
file C:\dotnet-campus\actions-runner\_work\_temp\ea5a132c-3f36-45cc-afc6-1bced18a0cd1.ps1 is not digitally signed. You 
cannot run this script on the current system.` 的错误

如下面代码

```
Run dotnet build --configuration Release
. : File C:\dotnet-campus\actions-runner\_work\_temp\ea5a132c-3f36-45cc-afc6-1bced18a0cd1.ps1 cannot be loaded. The 
file C:\dotnet-campus\actions-runner\_work\_temp\ea5a132c-3f36-45cc-afc6-1bced18a0cd1.ps1 is not digitally signed. You 
cannot run this script on the current system. For more information about running scripts and setting execution policy, 
see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:3
+ . 'C:\dotnet-campus\actions-runner\_work\_temp\ea5a132c-3f36-45cc-afc ...
+   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
Error: Process completed with exit code 1.
```

但是上面的 PowerShell 是不安全的，在输入之前，还请大家先阅读官方文档 [about_Execution_Policies - PowerShell](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.1&WT.mc_id=DX-MVP-5003606 )

## 运行

下一步调用 run.cmd 运行

```
./run.cmd
```

在咱自己的项目里面，可以用上刚才自己搭建的服务器，在 GitHub 的 Action 上，需要通过在 Yaml 配置文件上设置在哪个服务器上运行

在 [dotnet 部署 github 的 Action 进行持续集成](https://blog.lindexi.com/post/dotnet-%E9%83%A8%E7%BD%B2-github-%E7%9A%84-Action-%E8%BF%9B%E8%A1%8C%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90.html ) 可以了解是如何写 YAML 配置文件的，而咱需要在这个配置文件里面修改使用自己部署的运行器

```yaml
jobs:
  build:
    # 将 windows-latest 换为 self-hosted 就可以了
    # runs-on: windows-latest
    runs-on: [self-hosted]

    steps:
    - uses: actions/checkout@v1
 
    - name: Build with dotnet
      run: dotnet build --configuration Release

    - name: Test
      run: dotnet test --configuration Release
```

修改 runs-on 的内容就可以了

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
