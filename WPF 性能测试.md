# WPF 性能测试

本文收藏我给 WPF 做的性能测试。在你开始认为 WPF 的性能存在问题的时候，不妨来这篇博客里找找看我做过的测试。我记录的测试都是比较纯净的测试项目，没有业务逻辑的干扰，写法也正常，可以更加真实反映 WPF 的性能，减少因为奇怪的业务逻辑以及逗比的写法的影响

<!--more-->
<!-- CreateTime:2023/3/9 9:14:41 -->

<!-- 发布 -->
<!-- 博客 -->

## 资源字典

### 大量 Geometry 资源对启动的影响

在资源字典里面存放了 5k 个 Geometry 对象，资源字典加入到 App.Xaml 的资源字典里面，意味着启动时就会加载这些资源。根据 WPF 对资源对象创建的定义，可以了解到，在 WPF 里面不会立刻创建资源对象，只有在资源对象首次被使用时才会被创建。也就是说加入到 App.Xaml 的资源字典的 5k 个 Geometry 对象将只会被记录到 App 的资源字典里面，但没有实际创建出来

实际测试性能大概是在我电脑上加载只需 50 毫秒左右

以上测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/1d3883ac7feba5dd7752e1edccd33f943c02f7f9/JojeryiheenelNearfinelwhea) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/1d3883ac7feba5dd7752e1edccd33f943c02f7f9/JojeryiheenelNearfinelwhea) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 JojeryiheenelNearfinelwhea 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 1d3883ac7feba5dd7752e1edccd33f943c02f7f9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 1d3883ac7feba5dd7752e1edccd33f943c02f7f9
```

获取代码之后，进入 JojeryiheenelNearfinelwhea 文件夹


## 一千个半透明矩形做动画

[WPF 动画性能测试应用 一千个半透明矩形做动画](https://blog.lindexi.com/post/WPF-%E5%8A%A8%E7%94%BB%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E5%BA%94%E7%94%A8-%E4%B8%80%E5%8D%83%E4%B8%AA%E5%8D%8A%E9%80%8F%E6%98%8E%E7%9F%A9%E5%BD%A2%E5%81%9A%E5%8A%A8%E7%94%BB.html )

## 画10万个矩形


测试代码放在[github](https://github.com/lindexi/lindexi_gd/tree/4983492acb47c040ecb80b7417f7cf364d1e3e19/NarlearcefearNuyikallair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/4983492acb47c040ecb80b7417f7cf364d1e3e19/NarlearcefearNuyikallair) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个名为 NarlearcefearNuyikallair 的空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 4983492acb47c040ecb80b7417f7cf364d1e3e19
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 4983492acb47c040ecb80b7417f7cf364d1e3e19
```

获取代码之后，进入 NarlearcefearNuyikallair 文件夹