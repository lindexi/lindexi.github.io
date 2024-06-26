# dotnet 解决 UNO 在 OpenKylin 麒麟系统运行找不到默认字体启动失败

本文记录 UNO 应用在 OpenKylin 麒麟系统运行找不到默认字体启动失败的解决方法

<!--more-->
<!-- CreateTime:2024/06/26 07:25:47 -->

<!-- 发布 -->
<!-- 博客 -->

本文方法适用于 5.2.161 版本的 UNO 应用，更高版本我没有经过充分测试

在 OpenKylin 系统启动 Avalonia 应用失败，在 FontDetailsCache 的 GetFontInternal 方法里面抛出空异常

核心原因是 UNO 尝试寻找默认的字体，然而找不到

解决方法是手动设置默认字体，编辑 `App.xaml.cs` 加上如下代码

```csharp
    public App()
    {
        this.InitializeComponent();
#if HAS_UNO
        // Fix run fail in Kylin system.
        // https://github.com/unoplatform/uno/issues/17287
        FeatureConfiguration.Font.DefaultTextFontFamily = "Noto Sans CJK SC";
#endif
    }
```

此问题已经报告给 UNO 官方，请看 : <https://github.com/unoplatform/uno/issues/17287>

相关博客： [dotnet 解决 Avalonia 在 OpenKylin 麒麟系统运行找不到默认字体启动失败](https://blog.lindexi.com/post/dotnet-%E8%A7%A3%E5%86%B3-Avalonia-%E5%9C%A8-OpenKylin-%E9%BA%92%E9%BA%9F%E7%B3%BB%E7%BB%9F%E8%BF%90%E8%A1%8C%E6%89%BE%E4%B8%8D%E5%88%B0%E9%BB%98%E8%AE%A4%E5%AD%97%E4%BD%93%E5%90%AF%E5%8A%A8%E5%A4%B1%E8%B4%A5.html )

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/bf72c0981965bb4aed7f082ef81eed2e4b6d6913/UnoDemo/DalekairwiJebonacaki) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bf72c0981965bb4aed7f082ef81eed2e4b6d6913/UnoDemo/DalekairwiJebonacaki) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bf72c0981965bb4aed7f082ef81eed2e4b6d6913
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bf72c0981965bb4aed7f082ef81eed2e4b6d6913
```

获取代码之后，进入 UnoDemo/DalekairwiJebonacaki 文件夹