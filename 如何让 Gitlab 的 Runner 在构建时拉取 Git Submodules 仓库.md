# 如何让 Gitlab 的 Runner 在构建时拉取 Git Submodules 仓库

默认的 GitLab 的 Runner 在构建时不会去拉取 Git Submodules 仓库，将会提示 Skipping Git submodules setup 跳过初始化 Git Submodule 仓库

<!--more-->
<!-- 发布 -->

如[官方文档](https://docs.gitlab.com/ee/ci/git_submodules.html) 的描述，只需要加上以下代码在 `.gitlab-ci.yml` 文件即可

```yml
variables:
  GIT_SUBMODULE_STRATEGY: recursive # 拉取 Submodule 内容
```

加入的逻辑和 stages 是同级，如下面例子

```yml
stages:
  - build
  - test
  - publish
# 上面代码定义了打包步骤，定义编译需要两个 job 分别是编译测试和发布，注意不同的 job 是在完全空白的项目，不会用到上一个job编译的文件

variables:
  GIT_SUBMODULE_STRATEGY: recursive # 拉取 Submodule 内容
```

设置之后可以在 GitLab 的 Runner 构建时看到如下输出

```csharp
Updating/initializing submodules recursively
```

也就是说将会自动拉取 submodules 内容

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
