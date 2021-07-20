# GitHub 的 Action 接入 Stryker.NET 进行自动化测试单元测试鲁棒性

假设有一个捣蛋的小伙伴加入了你的团队，这个捣蛋的小伙伴喜欢乱改代码，请问此时的单元测试能否拦住这些逗比行为？如果不能拦住逗比行为，是否代表着单元测试有所欠缺，或者有某些分支逻辑没有考虑到。本文将告诉大家的 Stryker.NET 就属于这样的一个捣蛋的小伙伴，这个工具将会在执行测试的时候乱改你的代码，看看你的单元测试是否能拦住这样的行为。如果在乱改代码之后，单元测试依然是通过的，那证明单元测试没有拦住此行为，说不定就需要改改单元测试了

<!--more-->
<!-- CreateTime:2021/7/18 19:29:53 -->


<!-- 发布 -->

大家都知道 GitHub 的 Action 可以非常方便将 dotnet tool 加入到工具链中，刚好 Stryker.NET 也是通过 dotnet tool 发布的，因此在 GitHub 的 Action 上接入十分简单

在 GitHub 的 Action 用上 Stryker.NET 就可以自动测试一下自己编写的单元测试的鲁棒性，看看单元测试是否能帮忙拦下一些不符合预期的行为变更。因为在开源项目中，单元测试很重要的一点在于，协助新加入的开发者了解自己编写的代码是否能在此开源项目中工作，可以认为新加入的开发者写的代码都是在乱改的情况下，单元测试能否帮忙拦下不符合预期的更改。如果不能拦下，那就是单元测试写的不够

我从张队长的博客看到了 [.NET测试用例写的好不好？让变种来测试一下](https://blog.csdn.net/sD7O95O/article/details/118833224 ) 这篇博客，了解到了 Stryker.NET 这个神奇的工具，于是在我的 [AsyncWorkerCollection: 高性能的多线程异步工具库](https://github.com/dotnet-campus/AsyncWorkerCollection ) 中接入。本文接下来也使用此项目作为例子来告诉大家如何在 GitHub 的 Action 接入

开始之前，先聊一下 Stryker.NET 的原理，其实做法很简单，就是对现有的项目代码进行瞎改，例如将判断相等修改为判断不相等，在修改之后，再次执行单元测试，看看单元测试能否通过。如果单元测试依然通过，那证明单元测试没有考虑到此更改的行为。例如原先一个业务是需要判断相等的，但是被修改为判断不相等，此时单元测试居然还能过，那就证明单元测试没有考虑到从判断相等被改为判断不相等的行为

能被 Stryker.NET 更改的内容有很多，可以从 [https://stryker-mutator.io/docs/stryker-net/Mutators](https://stryker-mutator.io/docs/stryker-net/Mutators) 找到完全的功能。例如将加法修改为减法，将大于判断修改为小于判断，将字符串修改为空字符串等等

在开始接入 GitHub 的 Action 之前，先在自己本地测试一下

使用 [AsyncWorkerCollection: 高性能的多线程异步工具库](https://github.com/dotnet-campus/AsyncWorkerCollection ) 作为例子，先进入单元测试所有的文件夹

```
cd test\AsyncWorkerCollection.Tests
```

按照惯例，使用 dotnet tool 的第一步就是安装工具，请使用如下代码进行安装

```
dotnet tool install -g dotnet-stryker
```

接着执行如下命令，让 Stryker.NET 自动测试

```
dotnet stryker -p="AsyncWorkerCollection.csproj" --log-file  -r "['html', 'progress']"
```

以上的核心命令就是 `-p="AsyncWorkerCollection.csproj"` 用来告诉 Stryker.NET 可以进行乱改代码的项目是哪个。执行上面代码之后，将会让 Stryker.NET 进行对 AsyncWorkerCollection.csproj 项目里面的代码乱改，在修改了代码之后，执行当前的单元测试，看看单元测试能否通过。如果单元测试不能通过了，那证明单元测试写的不错。大概的执行的输出如下

```
Killed:     8
Survived: 145
Timeout:    5
```

以上代码证明乱改的代码上，有 8 个乱改的代码被单元测试拦住，也就是被单元测试杀掉。而有 145 个乱改的代码能通过单元测试，证明单元测试其实和没有的差不多。剩下 5 个是在乱改之后单元测试超时了

接入到 GitHub 的 Action 也非常简单，只需要在 `.github\workflows` 文件夹里面再新建一个叫 stryker.yml 的文件即可。打开 stryker.yml 文件，添加自动测试的代码

```yaml
name: Stryker

on: 
  push:
    branches: 
      - master

jobs:
  Stryker:
    runs-on: windows-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v2
        with:
          ref: ${{ github.head_ref }}

      - name: Install Stryker
        run: dotnet tool install -g dotnet-stryker

      - name: Test
        run: |
          cd test\AsyncWorkerCollection.Tests
          dotnet stryker -p="AsyncWorkerCollection.csproj" --log-file  -r "['html', 'progress']"
```

步骤十分简单，首先是只有在推送到 master 的时候才执行

```yaml
on: 
  push:
    branches: 
      - master
```

接着是将代码拉下

```yaml
    steps:
      - name: Checkout repo
        uses: actions/checkout@v2
        with:
          ref: ${{ github.head_ref }}
```

然后安装工具和执行测试

```yaml
      - name: Install Stryker
        run: dotnet tool install -g dotnet-stryker

      - name: Test
        run: |
          cd test\AsyncWorkerCollection.Tests
          dotnet stryker -p="AsyncWorkerCollection.csproj" --log-file  -r "['html', 'progress']"
```

将此文件推送到 GitHub 上，合入 master 即可

详细更改请参考 [https://github.com/dotnet-campus/AsyncWorkerCollection/pull/60](https://github.com/dotnet-campus/AsyncWorkerCollection/pull/60)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
