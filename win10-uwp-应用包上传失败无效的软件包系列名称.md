
# win10 uwp 应用包上传失败无效的软件包系列名称

在 2021 的时候，微软为了解决开发者不小心将应用商店证书上传到 GitHub 而不再提供应用商店证书文件下载到代码文件夹里面，因此日常开发中将会使用自己本地创建的证书。但是在实际准备发布的时候，如果没有切换证书，那么将会使用本地创建的证书进行打包，而在上传应用商店步骤失败

<!--more-->


<!-- CreateTime:2021/3/21 19:40:43 -->

<!-- 发布 -->

在上传应用商店的时候，可以看到如下的提示

```
BitStamp_2.7.3.0_x86_x64_arm_bundle.appxupload36.0 MB

无效的软件包系列名称: 43179.1161685EE70AE_s32203668fxst (应为: 43179.1161685EE70AE_ajj8jc175maf4)

无效的软件包发布者名称: CN=lindexi (应为: CN=227D1644-D24B-430C-AFA3-3FD86CE65409)
```

原因是我的本地测试证书里面生成的信息里面的发布者等和应用商店要求的不符合，因此提示 无效的软件包系列名称 和 无效的软件包发布者名称 上传失败

解决方法是在发布之前，先删除本地创建的证书，然后右击项目，选择发布，点击将应用程序与应用商店关联。接着选择你的应用，点击下一步

双击 Package.appxmanifest 进入打包页面，可以看到显示了具体的发布者，点击选择证书，此时可以显示出来证书指纹。如果此时没有显示证书，那么请点击从存储选择

如果依然失败，那么请清理文件夹，如使用 Git 的 `git clean -xdf` 命令清理掉所有没有被 Git 管理的文件。当然，在清理之前，如果你对 Git 不熟悉，我更建议你拷贝代码重新创建项目







<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。