# VisualStudio 2019 新创建项目添加 git 仓库

在 VisualStudio 2017 在新建项目的时候给出创建 git 仓库的选项，但是在 VisualStudio 2019 去掉了新建项目的页面，默认新建的项目都是没有带仓库。本文告诉大家如何在 vs2019 里面添加版本管理仓库

<!--more-->
<!-- CreateTime:2019/7/25 15:08:15 -->

<!-- csdn -->

在 VisualStudio 2019 的一大改进就是更改新建项目的页面

<!-- ![](image/VisualStudio 2019 新创建项目添加 git 仓库/VisualStudio 2019 新创建项目添加 git 仓库1.png) -->

![](http://image.acmx.xyz/lindexi%2F201972515142244)

如上图，在 VisualStudio 2019 去掉了在新建项目的同时创建 git 仓库的功能

此时可以通过在新建完成项目之后，点击右下角的添加到源代码管理，然后选择 git 就可以新建代码仓库

<!-- ![](image/VisualStudio 2019 新创建项目添加 git 仓库/VisualStudio 2019 新创建项目添加 git 仓库0.png) -->

![](http://image.acmx.xyz/lindexi%2F20197251534592)

或者在文件里面点击添加到源代码管理的选项

<!-- ![](image/VisualStudio 2019 新创建项目添加 git 仓库/VisualStudio 2019 新创建项目添加 git 仓库2.png) -->

![](http://image.acmx.xyz/lindexi%2F201972515630516)

使用 VisualStudio 新建的 Git 仓库可以自动添加 `.gitignore` 和 `.gitattributes` 文件，在忽略文件里面包含了编译创建的文件，这样就不会上传编译创建的文件

详细请看 [Redesigning the New Project Dialog](https://devblogs.microsoft.com/visualstudio/redesigning-the-new-project-dialog/ )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
