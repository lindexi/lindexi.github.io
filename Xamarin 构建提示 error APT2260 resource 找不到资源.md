# Xamarin 构建提示 error APT2260 resource 找不到资源

其实这是 VisualStudio 逗比的问题，尝试关闭 VisualStudio 然后干掉 Bin 和 Obj 文件夹，然后先开启安卓模拟器，然后重新构建就可以了。如果一次重新构建失败，那么再次右击重新生成就可以了

<!--more-->
<!-- CreateTime:2020/2/23 16:43:08 -->

<!-- 发布 -->

如果在新建一个 Xamarin 应用时，或安装了一个 NuGet 库之后，发现在构建时提示下面代码

```
2>C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Xamarin\Android\Xamarin.Android.Common.Debugging.targets(420,2): warning : 发生一个或多个错误。
2>C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Xamarin\Android\Xamarin.Android.Aapt2.targets(155,3): error APT2260: resource style/Theme.AppCompat.Light.Dialog (aka com.companyname.fecawjearwhalljearwugeweenere:style/Theme.AppCompat.Light.Dialog) not found.
2>G:\lindexi_gd\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere.Android\Resources\values\styles.xml(4): error APT2260: style attribute 'attr/colorAccent (aka com.companyname.fecawjearwhalljearwugeweenere:attr/colorAccent)' not found.
2>C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Xamarin\Android\Xamarin.Android.Aapt2.targets(155,3): error APT2260: resource style/Theme.AppCompat.Light.DarkActionBar (aka com.companyname.fecawjearwhalljearwugeweenere:style/Theme.AppCompat.Light.DarkActionBar) not found.
2>G:\lindexi_gd\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere.Android\Resources\values\styles.xml(2): error APT2260: style attribute 'attr/windowNoTitle (aka com.companyname.fecawjearwhalljearwugeweenere:attr/windowNoTitle)' not found.
2>G:\lindexi_gd\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere.Android\Resources\values\styles.xml(2): error APT2260: style attribute 'attr/windowActionBar (aka com.companyname.fecawjearwhalljearwugeweenere:attr/windowActionBar)' not found.
2>G:\lindexi_gd\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere.Android\Resources\values\styles.xml(2): error APT2260: style attribute 'attr/colorPrimary (aka com.companyname.fecawjearwhalljearwugeweenere:attr/colorPrimary)' not found.
2>G:\lindexi_gd\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere.Android\Resources\values\styles.xml(2): error APT2260: style attribute 'attr/colorPrimaryDark (aka com.companyname.fecawjearwhalljearwugeweenere:attr/colorPrimaryDark)' not found.
2>G:\lindexi_gd\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere.Android\Resources\values\styles.xml(3): error APT2260: style attribute 'attr/colorAccent (aka com.companyname.fecawjearwhalljearwugeweenere:attr/colorAccent)' not found.
2>G:\lindexi_gd\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere\FecawjearwhalljearWugeweenere.Android\Resources\values\styles.xml(4): error APT2260: style attribute 'attr/windowActionModeOverlay (aka com.companyname.fecawjearwhalljearwugeweenere:attr/windowActionModeOverlay)' not found.
2>C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Xamarin\Android\Xamarin.Android.Aapt2.targets(155,3): error APT2062: failed linking references.
```

那么基本都是 VisualStudio 的逗比问题，可以通过还原 NuGet 库和删除 Bin 和 Obj 文件夹或清理项目解决。注意清理项目时不会完全删除 Obj 文件哦，我推荐先手动删除 Bin 和 Obj 文件夹，如果删除失败，那么先将项目代码复制到另一个文件夹就可以了

通过 Git 管理的代码，可以通过下面的代码快速清理 Bin 和 Obj 文件夹，这个方法的缺点是也许会将一些代码也清理掉，除非是熟悉 Git 的小伙伴，不然请不要模仿

```
git clean -xdf
```

此外，如果是 NuGet 没有还原成功，因为网络的原因，可以尝试国内的源，请看 [我收集的各种公有 NuGet 源 - walterlv](https://blog.walterlv.com/post/public-nuget-sources.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
