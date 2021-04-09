
# 用于辅助做二分调试的构建每个 commit 的工具

在日常开发调试中，有一个超级调试方法，二分调试。二分调试可以用来辅助调试在某个版本是好的，但现在是坏的问题。或者说用来辅助定位某个问题是啥时候写出来的。二分调试的做法就是尝试每个版本的代码构建出来的应用，看这个版本的应用是否符合有坑，当然为了提升效率，就采用了二分算法，不需要每个 commit 版本的代码都构建。本文提供了一个工具用来辅助构建每个 commit 版本的代码，将构建输出的应用保存到自定义的某个文件夹，用来在进行二分调试的时候，不需要重新构建一次

<!--more-->


<!-- 发布 -->

二分调试的做法，其实就是尝试每个版本的代码构建出来的应用，只是如果真的每个版本都尝试一下，那么效率不够高，于是借鉴了排序里面的二分思想，选择了有限个数的版本代码进行测试就可以了解到大概在哪个版本的代码出现问题或修复了问题

二分的思想的简单例子是，假定知道了在 A 版本是没有问题的，但是在 当前版本 存在了坑。而 A 版本的代码距离当前版本有 100 个版本。按照二分的思想，是先取 100 个版本的中间版本，也就是第 50 个版本，试试这个版本是否没有问题。假定这个版本有问题，那么再取这个 50 个版本的中间，也就是第 25 个版本进行测试。如果此时 第 25 个版本没问题，那么取第 25 到第 50 个版本的中间，也就是第 37 个版本进行尝试

大家可以看出来，即使是中间距离有 1000 个版本，也只需要很少的次数的尝试就能大概找到某个版本

在进行二分调试的优势在于，可以不了解业务，不了解代码，也可以不用思考，只需要不断进行测试就可以了

但是二分调试存在一个问题是，如果项目的构建速度不够快，那么有大量的时间都在等待构建上。本文提供了一个工具，可以预先帮你将每个版本 commit 代码都构建出来，让你将构建出来的输出应用存放在你自己的某个文件夹里面。这样你想要进行二分调试的时候，就不需要重新进行构建而可以使用之前构建好的软件，这样可以提升调试的效率

这个工具在 GitHub 上完全开源，请看 [https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/)

在使用这个工具之前，你需要先准备好代码文件夹，以及存放构建输出的文件夹和工作文件夹。使用这个工具的效果如下，例如代码文件夹是 `C:\Code` 文件夹，而存放构建输出内容的文件夹是 `C:\Storage` 文件夹。那么此工具将会遍历 `C:\Code` 代码文件夹的 Git 的 Commit 每个版本，对每个版本调用 `msbuild` 构建，然后将构建输出内容，按照 commit 号作为文件夹名，将构建输出放入到 commit 号对应的文件夹，存放到 `C:\Storage` 文件夹。假定构建的输出是 `lindexi.exe` 文件，那么运行之后的 `C:\Storage` 文件夹里面的内容大概如下

```
C:\Storage\00c10ad0421526344d899af2a5cc4db3fac3c74c\lindexi.exe
C:\Storage\4a3d052832c386ec8cb9f2be5b8d487df6860e2c\lindexi.exe
C:\Storage\08742ede9ff78dbfe41b45cc44ad36b67c182976\lindexi.exe
```

在进行二分调试的时候，就可以根据 git 的 commit 号，找到对应的构建输出文件夹，然后运行构建输出文件夹里面的输出文件，进行测试。这样就可以减少编译的步骤，提高效率

这个辅助二分的自动构建工具是一个 dotnet tool 在使用之前需要使用下面命令安装

```
dotnet tool update -g dotnetCampus.CopyAfterCompileTool
```

进入上文所说的准备好的 工作文件夹 里面，在 工作文件夹 里面核心是用来存放自动构建的配置文件，以及启动自动构建的脚本文件。在工作文件夹里面创建一个 `Build.coin` 纯文本文件，这个文件是 coin 格式文件，详细请看 [dotnet-campus/dotnetCampus.Configurations: COIN，Configuration\n。高性能应用程序配置文件和此配置文件的解析库](https://github.com/dotnet-campus/dotnetCampus.Configurations )

这个 `Build.coin` 文件是一个配置文件，里面将包含代码仓库的路径，以及构建输出内容的存放文件夹

配置文件里面包含的项如下：

```
> 代码文件夹
CodeDirectory
C:\Code\
> 应用输出文件夹
OutputDirectory
C:\Code\bin\release
> 保存构建完成的文件夹
TargetDirectory
C:\Storage
>
OriginBranch
origin/master
> 使用 LastCommit 表示上次程序运行构建到的 commit 号，这是可选的
LastCommit
9d19fa49b644b382ed06bd5ba7fefe7f4ec7eb19
>
```

要求传入的是代码文件夹和应用输出文件夹，其中应用输出文件夹就是在构建代码之后的输出文件夹是哪个文件夹，这个文件夹的内容将会在构建完成之后被移动到 `保存构建完成的文件夹` 里面

传入的 OriginBranch 将会被用来构建的时候，从 LastCommit 构建到 OriginBranch 里面，从 LastCommit 到 OriginBranch 中间的每个 commit 都会被构建

配置完成之后，再新建一个 `Build.cmd` 文件放在工作文件夹，里面内容如下

```
cd %~dp0
dotnet tool update -g dotnetCampus.CopyAfterCompileTool
CopyAfterCompile
```

先执行 `cd %~dp0` 切换到 `Build.cmd` 文件所在的工作文件夹，这样就方便在计划任务等里面调用这个 Build.cmd 文件启动

然后先更新一下工具，接着执行工具

在 CopyAfterCompile 工具启动的时候，将会先读取 `Build.coin` 文件的配置，获取代码所在的文件夹，以及当前期望构建到的分支是哪个。接着执行 `git fetch --all --tags` 命令，拉取 git 仓库。接着使用 git log 命令，获取 commit 号列表。然后遍历所有的 commit 号列表，通过 `git checkout xxxxxx -f` 切换到对应的 commit 号上，执行 `git clean -xdf` 清理代码空间，清理完成之后通过 `msbuild` 命令进行构建，执行的构建命令是 `msbuild /p:Configuration=Release -restore` 命令

在构建完成之后，将构建输出文件夹的内容，也就是配置文件里面的 OutputDirectory 对应文件夹内容，移动到保存构建完成的文件夹内，放在 保存构建完成的文件夹 的对应 commit 号里面

如果在使用过程中有任何的建议，欢迎到 GitHub 上交流 [https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。