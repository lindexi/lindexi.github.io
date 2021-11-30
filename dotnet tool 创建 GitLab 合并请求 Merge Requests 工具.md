# dotnet tool 创建 GitLab 合并请求 Merge Requests 工具

本文来告诉大家如何使用 dotnetCampus.GitLabMergeRequestCreator 工具，命令行创建 GitLab 合并请求 Merge Requests 的方法

<!--more-->
<!-- CreateTime:2021/11/29 8:52:59 -->

<!-- 草稿 -->

这是在 GitHub 上完全开源的工具，请看 [https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK)

此工具是基于 dotnet tool 制作发布的，安装部署方法特别简单，只需在命令行输入以下代码即可

```
dotnet tool update -g dotnetCampus.GitLabMergeRequestCreator
```

命令行参数创建 合并请求 Merge Requests 的例子如下

```
CreateGitLabMergeRequest -GitLab https://gitlab.sdlsj.net -Token $Token -TargetBranch dev -SourceBranch release -ProjectId $CI_PROJECT_ID -Title "Merge release to dev"
```

命令行参数命令的含义如下：

- 

以上代码的 `$Token` 是存放在 GitLab 的私密信息，设置方法是在 CI/CD Settings 里面的 Variables 添加变量，详细请看 [GitLab CI/CD variables](https://docs.gitlab.com/ee/ci/variables/ )

生成 Token 方法如下：



在 GitLab 的配置需要放入到 `.gitlab-ci.yml` 文件，如以下代码

```yml
 - "dotnet tool update -g dotnetCampus.GitLabMergeRequestCreator"
 - 'CreateGitLabMergeRequest -GitLab https://gitlab.sdlsj.net -Token $Token -TargetBranch dev -SourceBranch release -ProjectId $CI_PROJECT_ID -Title "Merge release to dev"'
```

可以使用此工具实现自动合并 Release 分支到 Dev 分支的功能，如以下代码

```yml
stages:
  - build

ReleaseToDev:
  # 自动从 Release 分支合并到 Dev 分支的机器人
  # 行为是有代码合入到 Release 时，将创建从 Release 到 Dev 的 MergeRequest 请求
  stage: build
  script:
    - "chcp 65001" # 切换编码，解决乱码问题
    - "dotnet tool update -g dotnetCampus.GitLabMergeRequestCreator"
    - 'CreateGitLabMergeRequest -GitLab https://gitlab.sdlsj.net -Token $Token -TargetBranch dev -SourceBranch release -ProjectId $CI_PROJECT_ID -Title "Merge release to dev"'
  except:
    - dev
```

我所在的团队约定了，产品项目进入发布过程，将切换到 Release 分支，封锁功能

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
