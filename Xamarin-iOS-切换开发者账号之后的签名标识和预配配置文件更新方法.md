
# Xamarin iOS 切换开发者账号之后的签名标识和预配配置文件更新方法

其实这一篇和 Xamarin 开发几乎没有关系，这是苹果开发的一个坑。在原有开发者账号下的设备上，切换为另一个开发者账号，此时的部署也许会找错 预配配置 文件和证书。苹果：渣渣开发者，这是特性

<!--more-->



<!-- 发布 -->

在 XCode 的选项账户里面的 Download Manual Profiles 按钮里面，下载的证书文件放在 `~/Library/MobileDevice/Provisioning\ Profiles` 文件夹里面，如下图

<!-- ![](image/Xamarin iOS 切换开发者账号之后的签名标识和预配配置文件更新方法/Xamarin iOS 切换开发者账号之后的签名标识和预配配置文件更新方法0.png) -->

![](http://image.acmx.xyz/lindexi%2F20201026119519989.jpg)

我推荐在苹果上切换开发者的时候，先删除原有的证书文件，删除方法是在命令行输入下面代码

```
~/Library/MobileDevice/Provisioning\ Profiles
rm *
```

<!-- ![](image/Xamarin iOS 切换开发者账号之后的签名标识和预配配置文件更新方法/Xamarin iOS 切换开发者账号之后的签名标识和预配配置文件更新方法1.png) -->

![](http://image.acmx.xyz/lindexi%2F202010261110507273.jpg)

删除完成之后，重新在 XCode 的选项账户里面点击 Download Manual Profiles 按钮

回到 VS 的 Xamarin 项目里面，右击选项，点击 iOS 捆绑包签名，重新选择签名标识，如下图，我建议不选择自动

<!-- ![](image/Xamarin iOS 切换开发者账号之后的签名标识和预配配置文件更新方法/Xamarin iOS 切换开发者账号之后的签名标识和预配配置文件更新方法2.png) -->

![](http://image.acmx.xyz/lindexi%2F202010261112186489.jpg)

参考文档：

[删除Xcode中多余的证书provisioning profile](https://blog.csdn.net/xyxjn/article/details/38081177)

[Xcode Provisioning Profiles Location](https://stackoverflow.com/q/45625347/6116637)

[使用 Xamarin 在 iOS 真机上部署应用进行调试](https://blog.walterlv.com/post/deploy-and-debug-ios-app-using-xamarin.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。