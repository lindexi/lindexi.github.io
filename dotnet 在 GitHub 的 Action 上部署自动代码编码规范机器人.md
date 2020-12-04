# dotnet 在 GitHub 的 Action 上部署自动代码编码规范机器人

我们的项目中会包含有很多文件，但是可能我们没有注意到的，我们的文件的编码不一定是 UTF-8 编码，这就可能让构建出来的应用程序在别人电脑运行时出现乱码，或者别人拉下来代码，却发现代码里面的中文都是乱码。为了解决文件编码的问题，咱需要一个编码规范工具，本文将告诉大家在 GitHub 上仓库，可以利用 GitHub 的 Action 部署自动代码文件编码规范的机器人，这个机器人可以自动协助咱规范文件的编码规范。可以设置为每次上传代码的时候，自动帮忙设置文件编码为 UTF-8 编码。或者在每次代码合并到主分支之后，机器人将会尝试修复文件的编码，如存在文件需要修复的，那机器人将会创建一个代码审查

<!--more-->
<!-- CreateTime:2020/10/19 8:22:54 -->



只想配置机器人，而不想看原理？没关系，给你一个快餐，复制 [dotnet-format.yml](https://github.com/dotnet-campus/AsyncWorkerCollection/blob/37960b9f99811a22130967122f5772866e3e3314/.github/workflows/dotnet-format.yml) 文件到你的 GitHub 仓库的 `.github\workflows` 文件夹里面即可。效果如何，你试试就知道了，或者在等待 GitHub 的 Action 过程中阅读一下本文

此方法不单适合于 dotnet 系的语言，也适合于其他任何使用文本记录代码的语言。因为本文使用的工具准确来说是针对于文本文件的编码的

本文的修复文件编码规范的工具是利用了在 GitHub 上完全开源的 [dotnet-campus/EncodingNormalior](https://github.com/dotnet-campus/EncodingNormalior) 项目的工具，这个项目链接是 [https://github.com/dotnet-campus/EncodingNormalior](https://github.com/dotnet-campus/EncodingNormalior) 欢迎大家参与开发

修复文件编码的方法是使用 dotnetCampus.EncodingNormalior 这个 dotnet tool 工具完成的，这个工具的使用方法如下

在使用之前先使用命令行安装或更新，请在命令行输入下面代码

```csharp
dotnet tool install -g dotnetCampus.EncodingNormalior
```

修复某个文件夹里面的所有文本文件的编码规范可以使用如下命令

```
EncodingNormalior -f E:\lindexi\EncodingNormalior --TryFix true
```

上面代码的 `E:\lindexi\EncodingNormalior` 为需要修复文本文件的编码规范的文件夹

这个 dotnetCampus.EncodingNormalior 工具的修复文本文件的编码规范的原理是先尝试判断出文本文件的编码，如此文件的编码不符合规范，如上面命令要求默认的规范的编码是 UTF-8 编码，那么此工具将会使用判断出来的文件编码去读取此文件，然后重新按照约定的规范编码将读取出来的文本重新写入到文件中

因为当前世界上不存在一个方法可以准确判断出不带BOM编码格式的文件的编码，因此这个工具也许会判断错文件的编码，如果判断错了编码，也许就会在修复之后让文件乱码

有了这个工具的支持，在 GitHub 上的 Action 做自动代码编码规范机器人就简单很多了，参考 [dotnet 基于 dotnet format 的 GitHub Action 自动代码格式化机器人](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-dotnet-format-%E7%9A%84-GitHub-Action-%E8%87%AA%E5%8A%A8%E4%BB%A3%E7%A0%81%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%9C%BA%E5%99%A8%E4%BA%BA.html) 的方法，可以在 GitHub 的 Action 中通过 dotnetCampus.EncodingNormalior 工具尝试修复文本文件的编码规范，如果有文件更改，那么提一个代码审查出来，或者直接推送代码到原分支

在 `.github\workflows` 文件夹里面创建的所有 `yml` 文件都会当成构建脚本，咱来创建一个叫 dotnet-format.yml 的文件。按照 GitHub 的 Action 的规定，每个构建脚本文件都应该给定一个名字，如下面代码

```yml
name: Code Encoding Check
```

然后设置构建脚本的触发时机，如下面代码设置了在推送了 master 分支时，触发构建脚本

```yml
on: 
  push:
    branches: 
      - master
```

其他触发时机等，还请大家去阅读官方文档

下一步是指定运行在什么设备上，如下面代码

```yml
jobs:
  dotnet-format:
    runs-on: windows-latest
```

接下来就是将代码拉下来了，可以通过如下代码将当前分支的最新代码拉下来

```yml
    steps:
      - name: Checkout repo
        uses: actions/checkout@v2
        with:
          ref: ${{ github.head_ref }}
```

本文的格式化方法是使用 dotnetCampus.EncodingNormalior 工具格式化的，在使用这个工具之前，需要先安装，请使用如下代码进行安装

```yml
      - name: Install dotnet-format
        run: dotnet tool install -g dotnetCampus.EncodingNormalior
```

被微软收购的 GitHub 默认构建环境肯定是包含了整个 dotnet 的，因此大家不需要额外再去安装。但如果大家用的是自己的环境，请使用如下代码安装 dotnet 环境

```yml
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 3.1.300
```

下一步是使用编码规范工具尝试修复代码文件夹里面的文本文件的编码

```yml
      - name: Fix encoding
        run: EncodingNormalior -f . --TryFix true
```

如果有某些特殊文件需要忽略文件的编码的，还请参阅 [dotnet-campus/EncodingNormalior](https://github.com/dotnet-campus/EncodingNormalior) 项目的命令行篇的更多用法

如果项目文件夹里面存在文件被更改，那么证明此文件的编码不符合约定的规范。通过 Git 将此更改签入，用于后续的步骤

```yml
      - name: Commit files
        # 下面将使用机器人的账号，你可以替换为你自己的账号
        run: |
          git config --local user.name "github-actions-dotnet-formatter[bot]"
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git commit -a -m 'Automated dotnet-format update'
        continue-on-error: true
```

上面代码中的邮箱地址和用户名都是使用 GitHub 机器人的账号，这里的代码可以自行替换

在上面代码没有判断是否存在文件变化，是因为在 Git 里面，如果没有文件更改，那么执行 git commit 将会失败，也就是啥都不会做。利用这个特性就可以减少一个判断更改的步骤了

而最后一句话 `continue-on-error: true` 是因为 git commit 会失败也是预期的，这个步骤可以忽略失败。加上这个命令就可以在这个步骤失败的时候，不会影响其他步骤

在这里可以分为两个不同的方向做，一个方向是将这个更改创建一个代码审查，另一个方向是推送到当前的分支上面去

我比较推荐的做法是创建一个代码审查。尽管使用工具的行为都是预期的，但是根据我这边团队的理念，所有的代码更改都应该被审查。即使是机器人的更改，也是需要审查一下的。这个做法和 dotnet 组织的 WPF 以及 Xamarin 等仓库的做法是相同的

在 GitHub 的 Action 里面创建一个代码审查，可以使用下面代码

```yml
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
        with:
          title: '[Bot] Automated PR to fix formatting errors'
          body: |
            Automated PR to fix formatting errors
          committer: GitHub <noreply@github.com>
          author: github-actions[bot] <41898282+github-actions[bot]@users.noreply.github.com>
          # 以下是给定代码审查者，需要设置仓库有权限的开发者
          assignees: lindexi,walterlv
          reviewers: lindexi,walterlv
          # 对应的上传分支
          branch: t/bot/fix-codeformatting
```

上面代码中的给定代码审查者，需要设置仓库有权限的开发者。这里利用了创建代码审查的一个功能，如果两个分支的代码没有历史的差异，是不会创建代码审查的。也就是如果文件夹里面没有文件更改，此时 git commit 将啥都不会做，而创建代码审查时，因为上一步 git commit 没有更改历史，因此两个分支的历史依然相同，不会创建代码审查

利用这个功能，就可以减少判断文件更改的逻辑了。如果文件夹里面的文件都符合编码规范，那么将啥都不会做。而如果有文件不符合编码规范，此时将会自动修改文件编码，然后创建一个代码审查

如果小伙伴觉得不需要创建代码审查，因为整个团队就几个开发者在开发，可以将修改编码之后的文件推送到当前的分支上面去，那么请使用下面代码推送。如果文件夹里面的文件都符合编码规范，那么将啥都不会做，因为没有内容可以推送

```yml
      - name: Push changes
        uses: ad-m/github-push-action@v0.5.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.head_ref }}
```

但是这个方法也许会让开发者不开森，因为他下一次上传代码的时候需要先拉代码，也许因为格式化给他了额外的改动。另外的，如 Xamarin 仓库的注释，其实代码推送无法用在 fork 的仓库上，也就是说如果这个代码审查是另一个开发者在他 fork 的仓库里面发起的，此时的这个方法将会失效

我现在在 [dotnet-campus/AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection) 仓库就接入这个机器人，用起来还不错

我将 [自动代码格式化机器人](https://blog.lindexi.com/post/dotnet-%E5%9F%BA%E4%BA%8E-dotnet-format-%E7%9A%84-GitHub-Action-%E8%87%AA%E5%8A%A8%E4%BB%A3%E7%A0%81%E6%A0%BC%E5%BC%8F%E5%8C%96%E6%9C%BA%E5%99%A8%E4%BA%BA.html) 和本文的自动文件编码规范机器人合并一起接入，欢迎小伙伴到 [https://github.com/dotnet-campus/AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection) 项目参观效果

更多关于这个工具请看下面博客

- [VisualStudio 编码规范工具 2.6 修改当前文件编码](https://blog.lindexi.com/post/VisualStudio-%E7%BC%96%E7%A0%81%E8%A7%84%E8%8C%83%E5%B7%A5%E5%85%B7-2.6-%E4%BF%AE%E6%94%B9%E5%BD%93%E5%89%8D%E6%96%87%E4%BB%B6%E7%BC%96%E7%A0%81.html)

- [C＃ 判断文件编码](https://blog.lindexi.com/post/C-%E5%88%A4%E6%96%AD%E6%96%87%E4%BB%B6%E7%BC%96%E7%A0%81.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
