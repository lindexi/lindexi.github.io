# VisualStudio 扩展开发

本文主要：如何开发一个 visual Studio 扩展，其实扩展也叫插件。

那么就是如何开发一个 vs插件。

本文也记录了我调试 VisualStudio 半个月过程遇到的坑。

<!--more-->
<!-- CreateTime:2019/5/28 19:51:49 -->

<!-- 标签：VisualStudio -->
<div id="toc"></div>

我写这博客时候，是我在开发一个插件：[编码规范工具](https://marketplace.visualstudio.com/items?itemName=lindexigd.vs-extension-18109)。记录的是我从不知道到发布插件，如果遇到了开发中的问题，欢迎交流。

## 安装 Visual Studio SDK

首先需要安装 Visual Studio SDK ，安装不需要其它的工具就可以，直接使用vs安装包。

我的是 Visual Studio 2015 ，所以我到这个页面：https://msdn.microsoft.com/en-us/library/bb166441 看教程。

垃圾wr（我说的就是微软），找个东西好难

首先是需要安装 SDK ，如果一开始没有安装的话，那么在控制面板，找到 vS 右击，修改 VS ，然后选择工具安装。

![](http://image.acmx.xyz/dac58ce0-c90f-489f-9ac0-83aadcf143db201711291648.jpg)

选择修改，然后选最后一个工具

![](http://image.acmx.xyz/dac58ce0-c90f-489f-9ac0-83aadcf143db201711291724.jpg)

我的是中文，可能翻译不一样，不过相信这一点压力对大家没有什么。



然后就来说下如何做插件，主要教程是看： http://dotneteers.net/blogs/divedeeper/archive/2008/01/06/LearnVSXNowPart3.aspx ，除了国外的还有国内的大神翻译：http://www.cnblogs.com/default/archive/2010/02/27/1674832.html 这里是一系列。

那么我将会来说下使用一个简单的方法去做一个 Command ，功能是可以判断 VS 工程的所有文件编码。

首先是新建一个插件项目。打开 vs ，新建一个 VSIXProject 

![](http://image.acmx.xyz/dac58ce0-c90f-489f-9ac0-83aadcf143db201711291848.jpg)

新建之后居然发现有一个  index.html 我开始还以为是 写html 来着，还好不是，这个 index.html 只是卖萌的而已。

## 添加菜单

那么新建完 VSIXProject 我们就开始编写按钮，虽然说是按钮，其实是菜单，在这里，全部的按钮都是和菜单一样。

那么我们直接新建 Command ，注意他的位置是在哪。

![](http://image.acmx.xyz/dac58ce0-c90f-489f-9ac0-83aadcf143db201711292118.jpg)

新建出来可以看到多了好多文件，其中 .vsct 是核心，如果想知道关于他更多，请去中文博客：http://www.cnblogs.com/default/archive/2010/06/28/1766451.html

我先放出做出了的菜单。

![](http://image.acmx.xyz/dac58ce0-c90f-489f-9ac0-83aadcf143db201711292847.jpg)

首先打开 `*.vsct` 在 Symbols 添加 id ，我们添加 EncodingNormalizerMenu ，EncodingNormalizerId2，他们的值随意给。关于这个 GUID 或者其它的，其实我也不懂。


```xml
    <GuidSymbol name="guidEncodingNormalizerPackageCmdSet" value="{0640f5ce-e6bc-43ba-b45e-497d70819a20}">
        <IDSymbol name="MyMenuGroup" value="0x1020" />
        <IDSymbol name="EncodingNormalizerId" value="0x0100" />
      
        <!--添加 EncodingNormalizerMenu ，EncodingNormalizerId2-->
        <IDSymbol name="EncodingNormalizerMenu" value="0x1021" />
        <IDSymbol name="EncodingNormalizerId2" value="0x0101" />

    </GuidSymbol>
```

然后是创建菜单 在`<Command>`下面使用`<Menus>`


```xml
    <Menus>
      <Menu guid="guidEncodingNormalizerPackageCmdSet" id="EncodingNormalizerMenu"
      type="Menu" priority="0x100">
        <!--注意这个id 和 type-->
        <Parent guid="guidSHLMainMenu" id="IDG_VS_MM_BUILDDEBUGRUN" />
        <!--这里的id是说他在哪-->
        <Strings>
          <!--按钮显示的字-->
          <ButtonText>规范编码</ButtonText>
          <!--命令名-->
          <CommandName>EncodingNormalizer</CommandName>
        </Strings>
      </Menu>
    </Menus>
  
```

然后修改 Groups ，修改 Parent 的 guid 为 Menu 的，修改 id 为 Menu 的。不一样就不在一个菜单。


```xml
      <Groups>
      <Group guid="guidEncodingNormalizerPackageCmdSet" id="MyMenuGroup" priority="0x0600">
        <Parent guid="guidEncodingNormalizerPackageCmdSet" id="EncodingNormalizerMenu"/>
      </Group>
    </Groups>
```

然后添加按钮，注意按钮需要 id 、priority、CommandName 和之前默认的不一样。

添加了按钮，我们需要添加事件，在 .cs 构造 
我们使用 CommandID 绑定，我们需要知道 按钮在哪个组，我们直接使用 CommandSet 还需要知道 按钮的 id ，第一个按钮是 0x0100 就是CommandId，第二个是 0x0101 ，于是就写 `new CommandID(CommandSet, 0x0101)` 使用第二个按钮

那么使用了按钮，我们需要关联事件使用，我的 MenuItemCallback 事件作为按钮点击使用函数。


```csharp
                  menuItem = new MenuCommand(this.MenuItemCallback, menuCommandID);
                commandService.AddCommand(menuItem);
```

代码全部在[全球同性交友平台](https://github.com/iip-easi/EncodingNormalior)，上面写的代码在 EncodingNormalizerVsx/EncodingNormalizer.cs

接着就是需要加上文件编码检查，在我之前写的 C# 判断文件编码 博客有说道如何检测文件编码。

关于 EncodingNormalizer 为何需要写私有构造以及他的代码为何是需要使用下面的结构，因为这时ms自己加的，我就不去改他了。默认创建的文件就是：

```csharp
       public static void Initialize(Package package)
        {
            Instance = new EncodingNormalizer(package);
        }

        public static EncodingNormalizer Instance { get; private set; }

              private EncodingNormalizer(Package package)
        {
            if (package == null)
            {
                throw new ArgumentNullException("package");
            }

            this.package = package;

            var commandService = ServiceProvider.GetService(typeof(IMenuCommandService)) as OleMenuCommandService;
            if (commandService != null)
            {
               //这里写按钮
                  menuItem = new MenuCommand(this.MenuItemCallback, menuCommandID);
                commandService.AddCommand(menuItem);
            }
        }

```

文件的其他都是后面加的，所以可以看到，需要添加的按钮写的在构造的哪里，而且打开 xxPagexx 的文件可以发现，这个创建就在 Initialize 使用，也就是下面的代码调用

```csharp
    public sealed class EncodingNormalizerPackage : Package
    {
        /// <summary>
        ///     EncodingNormalizerPackage GUID string.
        /// </summary>
        public const string PackageGuidString = "ffc5dabf-5ded-4433-8225-73b47e154210";

        #region Package Members

        /// <summary>
        ///     Initialization of the package; this method is called right after the package is sited, so this is the place
        ///     where you can put all the initialization code that rely on services provided by VisualStudio.
        /// </summary>
        protected override void Initialize()
        {
            EncodingNormalizer.Initialize(this);
            base.Initialize();
        }
    }
```

所以在开始传入自己，因为传入之后可以通过 ServiceProvider.GetService 获得按钮添加的位置，之后就在他这里添加按钮的对应事件，添加时需要使用和按钮一样的id，这里建议写一个变量。

## 增加选项

我们需要保存一些设置，那么如何自定义配置的界面，把配置页面放在工具->选项，可以参见 http://www.cnblogs.com/winkingzhang 提供的方法，我使用了他的方法，很简单。还有垃圾wr的方法 https://github.com/Microsoft/VSSDK-Extensibility-Samples/blob/646de671c1a65ca49e9fce397baefe217e9123e8/Options_Page/Readme.md


首先打开 `*Package.cs` ，不过我们需要新建一个类 DefinitionPage ，不需要在这个类写什么。


```csharp
    public class DefinitionPage
    {
        
    }
```

然后添加 ProvideProfile ，需要写类型、分类名称、页面名称、资源ID。关于 ProvideProfile 可以去：https://msdn.microsoft.com/zh-cn/library/microsoft.visualstudio.shell.provideprofileattribute.aspx


```csharp
  [ProvideProfile(typeof(DefinitionPage), "PowerExtension", "DefinitionPage",101,104,true)]
  [ProvideOptionPage(typeof(DefinitionPage), "PowerExtension", "DefinitionPage", 101, 104, true)]
   
```
PowerExtension 就是显示在选项的一级菜单，DefinitionPage就是二级菜单。

我这里把 PowerExtension 改为 EncodingNormalizer

我们选项不需要复杂的，只需要使用默认的，于是我们添加属性 `CriterionEncoding`。

添加时候，需要让 DefinitionPage 使用 DialogPage

```csharp
    [ClassInterface(ClassInterfaceType.AutoDual)]
    [Guid(GuidStrings.GuidDefinitionPage)]
    public class DefinitionPage : DialogPage
```

GuidStrings.GuidDefinitionPage) 是我自己定义的GUID放在一起的类，实际没有用

需要让属性知道设置的标题，点击显示的 Description ，于是我就写下面代码


```csharp
        /// <summary>
        /// 规范格式
        /// </summary>
        [Category("规定编码")]
        [Description("规定的编码，如果是ascii 那么无论是选择utf8或GBK都判断为对")]
        public Encoding CriterionEncoding { set; get; }
```

![](http://image.acmx.xyz/dac58ce0-c90f-489f-9ac0-83aadcf143db201711211410.jpg)

如果需要复杂窗体，新建一个 用户控件，注意是 WinForm 控件，然后 override Window。

假如我有一个控件 View.DefinitionPage ，那么我可以使用


```csharp
        protected override IWin32Window Window
        {
            get
            {
                if (_definitionPage == null)
                {
                    _definitionPage = new View.DefinitionPage();
                }
                _definitionPage.Owner = this;
                _definitionPage.InitializeLifetimeService();
                return _definitionPage;
            }
        }
        private View.DefinitionPage _definitionPage;
```

那么如何读取 vs扩展配置 ？

vsx的配置保存使用上面的方法，就自动保存配置注册表，于是如何去读，就是问题。

读取 vs插件 配置的方法是在 `*Package.cs` 写静态的 `Ensure*Package`


```csharp
        public static EncodingNormalizerPackage EnsureEncodingNormalizerPackage()
        {
            IVsShell shell = Package.GetGlobalService(typeof(SVsShell)) as IVsShell;
            if (shell != null)
            {
                IVsPackage package;
                Guid guid = new Guid(EncodingNormalizerPackage.PackageGuidString);
                if (ErrorHandler.Succeeded(shell.IsPackageLoaded(ref guid, out package)))
                {
                    return package as EncodingNormalizerPackage;
                }
                if (ErrorHandler.Succeeded(shell.LoadPackage(ref guid, out package)))
                {
                    return package as EncodingNormalizerPackage;
                }
            }
            return null;
        }

```

EncodingNormalizerPackage 是我的类，大家替换他为你的类，然后拿到了EncodingNormalizerPackage还需要拿到他的页面属性

DefinitionPage 就是我上面定义的选项

```csharp
        public static DefinitionPage DefinitionPage()
        {
            EncodingNormalizerPackage package = EnsureEncodingNormalizerPackage();
            return package?.GetDialogPage(typeof(DefinitionPage)) as DefinitionPage;
        }
```

那么在上面定义的 CriterionEncoding 也可以拿到，这样就可以读配置了。

修改 CriterionEncoding 关闭 vs，可以看到下次打开值还在，他是写在注册表。

我发现写注册表对于List的和一些类型都不好，于是我用了写文件，写文件可以写在用户文档。

`System.Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments)` 获取用户文档，在里面新建文件夹，然后就可以写入读取。


如果需要做方法去和 Resharper 的注册等差不多的页面，那么可以使用 Window 弹出。

首先创建一个 UserControl 然后写一个复杂的界面，假如我写了一个叫 ConformPage 的控件，那么我如何去显示他。可以使用下面代码。

```csharp
            System.Windows.Window window = new System.Windows.Window();
            ConformPage conformPage = new ConformPage();
            window.Content = conformPage;
            window.Title = "编码规范工具";
            window.Show();
```



从零做一个扩展，可以参见：http://www.cnblogs.com/stg609/p/3711443.html

## 传到商店

做完了功能，我们需要发布扩展，我们要把按钮加上好看的 ico 。

首先，下载一张图。

把图片改为 32bits 16x16。修改图片可以使用 http://www.easyicon.net/ 在线转换。

图片放到工程，我放在 Resource ，我给他名称 code_711px_16_easyicon.net.png

打开 `*.vsct` ，在 Symbols 添加 一个GUID。这需要给他名称和ID


```xml
      <GuidSymbol name="EncodingNormalizerIdICO" value="{B1E160E6-CC24-4FD5-857F-69F45DCEE27B}"
                >
      <IDSymbol name="bmpPic1" value="1" />
    </GuidSymbol>
```

然后在 Bitmaps 添加 Bitmap ，他的 guid 就是我们上面 EncodingNormalizerIdICO ，href 是图片所在文件夹，usedList 写 IDSymbol 。

```xml
        <Bitmap guid="EncodingNormalizerIdICO" href="Resources\code_711px_16_easyicon.net.png" usedList="bmpPic1"/>

```

然后在我们需要的按钮 Icon 的 guid 写 guid，在 id 写 IDSymbol 。

这样就好啦。

如果发现有自己做的和我讲的不同，那么一定是有一点我没说道，去我的github看我做的。

图片可以把多个图片放在一起，于是按照一个图片 16x16 读取，这就是多个`IDSymbol`做的。

## 获取工程所有项目

我需要获取用户工程的所有项目，我开始使用`dte.Solution.Projects`但是放在文件夹的项目获取不到，所以使用堆栈提供的方法。

这个方法写在[C＃ 解析 sln 文件](http://lindexi.oschina.io/lindexi/post/C-%E8%A7%A3%E6%9E%90-sln-%E6%96%87%E4%BB%B6/) 可是 vs 说找到不 Microsoft.Build.dll 所以这个方法还是不可以的。那么如何从 dte 获取所有项目？我找到一个大神博客：http://www.wwwlicious.com/2011/03/29/envdte-getting-all-projects-html/

开始判断是不是文件夹，如果是的话，递归函数获取文件夹所有项目。

我以为文件夹不是 `Project` 但是后来发现，工程的文件夹也是 `Project` 文件夹可以通过`ProjectKinds.vsProjectKindSolutionFolder`判断。

那么如何获得 文件夹所有文件夹和项目，其实 `Project` 有 ProjectItems 可以获取。

于是可以使用这个方法


```csharp
        private static List<Project> GetSolutionFolderProjects(Project solutionFolder)
        {
            List<Project> list = new List<Project>();
            for (var i = 1; i <= solutionFolder.ProjectItems.Count; i++)
            {
                var subProject = solutionFolder.ProjectItems.Item(i).SubProject;
                if (subProject == null)
                {
                    continue;
                }

                // If this is another solution folder, do a recursive call, otherwise add
                if (subProject.Kind == ProjectKinds.vsProjectKindSolutionFolder)
                {
                    list.AddRange(GetSolutionFolderProjects(subProject));
                }
                else
                {
                    list.Add(subProject);
                }
            }

            return list;
        }
```

一开始判断是不是文件夹，如果是就使用 GetSolutionFolderProjects 得到所有的项目，这样就可以获得工程所有项目。


```csharp
  foreach (var temp in dte.Solution.Projects)
  {
    if (temp is Project)
    {
        if (((Project) temp).Kind == ProjectKinds.vsProjectKindSolutionFolder)
        {
           project.AddRange( GetSolutionFolderProjects((Project) temp));
        }
        else
        {
           project.Add((Project)temp);
        }
    }

  }
```

<script src="https://gist.github.com/lindexi/3105bd0f0c5225bec4aa476f84dd29db.js"></script>

代码：https://gist.github.com/lindexi/3105bd0f0c5225bec4aa476f84dd29db

## 升级 2017

如果有之前扩展需要升级，参见https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-migrate-extensibility-projects-to-visual-studio-2017

1. 安装 vs2017 需要添加扩展

 关于vs2017 可以到我网盘下载，参见：http://lindexi.oschina.io/lindexi/post/C-7.0/

1. 打开 Nuget 升级，把所有提示升级的都升级。

 ![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017323202746.jpg)

1. 打开 source.extension.vsixmanifest

 选 InstallationTarget 包括各版本

 ![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017323202935.jpg)

 ![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017323202922.jpg)

1. 打开属性，修改路径

 启动外部程序`C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\devenv.exe`

 ![](http://image.acmx.xyz/AwCCAwMAItoFAMV%2BBQA28wYAAQAEAK4%2BAQBmQwIAaOgJAOjZ%2F2017323203341.jpg)

1. 启动项目

1. 把插件共享 https://visualstudiogallery.msdn.microsoft.com/site/upload/view

参见：http://blog.csdn.net/liuruxin/article/details/17955363

https://github.com/Microsoft/VSSDK-Extensibility-Samples

https://msdn.microsoft.com/zh-cn/library/cc138589

https://msdn.microsoft.com/zh-cn/library/dn705845

https://msdn.microsoft.com/zh-cn/library/mt683786

代码：https://github.com/lindexi/EncodingNormalior

如果开发中遇到问题，欢迎联系 [lindexi_gd@163.com](mailto:lindexi_gd@163.com)

现在我还有另一个插件：[图片注释](http://download.csdn.net/detail/lindexi_gd/9833388) 这个插件不是我写的，我是修改的，所以没有发布，如果需要就在这里下。

推荐[VS插件开发笔记 - 禾木 - CSDN博客](https://blog.csdn.net/lj22377/article/details/84641077 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。