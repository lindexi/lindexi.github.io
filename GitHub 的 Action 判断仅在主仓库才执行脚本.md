# GitHub 的 Action 判断仅在主仓库才执行脚本

我有一个 GitHub 项目，这个项目配置了仅需要在源仓库才能执行的 Action 如推送 NuGet 等发布动作。如何在 Action 里面设置让 Fork 的仓库不执行 Action 的步骤

<!--more-->
<!-- CreateTime:5/25/2020 2:29:40 PM -->



想要设置 Action 不在 Fork 的仓库执行，只需要添加 if 判断，判断当前执行的仓库和设置的相同，即可执行，否则不执行

在 GitHub 的 Action 的判断使用 `if` 方法，条件可以是判断 `repository` 或 `repository_owner` 的内容

如使用下面代码判断，让构建步骤仅在 `lindexi/doubi` 仓库执行

```
    - name: Build with dotnet
      run: dotnet build --configuration Release
      if: github.repository == 'lindexi/doubi'
```

上面的 `lindexi/doubi` 的意思就是 `用户名或组织名/项目名` 这就会跟随用户的 Fork 仓库修改值。也就是小伙伴 frendguo 他 fork 了我的仓库，那么在他仓库里面执行的 action 拿到的 github.repository 的值是 `frendguo/doubi` 也就是判断逻辑不通过

判断不仅可以放在步骤里面，还可以写在 job 里面，让整个 job 都需要通过某个条件

```yaml
jobs:
  build:
    runs-on: windows-latest
    if: github.repository == 'lindexi/doubi'
```

上面代码设置了只有在 lindexi 的 doubi 仓库才进行 build 这个 job 内容

当然上面代码写起来是针对仓库，不利于相同的一个组织内的复制粘贴代码。也就是我将代码粘贴到相同的组织里面的另一个仓库，可以看到另一个仓库是跑不起来的，因为仓库名不相同，此时可以尝试使用 `repository_owner` 这个值替换，请看代码

```
    if: github.repository_owner == 'lindexi'
```

上面代码判断的是只要拥有者是 lindexi 就执行。这里的 repository_owner 的值可以是用户也可以是组织，具体和仓库所在的拥有者相关

如果自己只是 fork 对方的仓库，改不动对方的代码，可以自己禁用 Action 请看 [GitHub 的 Action 如何禁用](https://blog.lindexi.com/post/GitHub-%E7%9A%84-Action-%E5%A6%82%E4%BD%95%E7%A6%81%E7%94%A8.html )

更多请看 [GitHub 操作的工作流程语法 - GitHub 帮助](https://help.github.com/cn/actions/reference/workflow-syntax-for-github-actions )

[Context and expression syntax for GitHub Actions - GitHub Help](https://help.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
