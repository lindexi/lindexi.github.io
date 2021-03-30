
# dotnet 读 WPF 源代码笔记 默认的 Main 函数是在哪创建的

在使用默认的 WPF 项目开发的时候，咱是不需要自己编写 Main 函数的，在 WPF 中的 Main 函数是存放在 App.g.cs 里面，看起来这个 Main 函数是生成的函数，本文将介绍在 WPF 框架中是如何创建这个入口函数

<!--more-->



<!-- 发布 -->
<!-- 标签：WPF，WPF源代码 -->

阅读本文你将了解 WPF 框架中，默认在 App.g.cs 生成入口 Main 函数的详细过程。阅读本文之前，你需要了解一些编译过程的知识以及代码生成的知识

默认的 Application 继承类命名是 App.xaml 如果在你的项目中，依然使用默认的命名，那么在 .NET 5 的 SDK 下，将会自动加上以下默认的代码

```xml
    <ApplicationDefinition Include="App.xaml">
      <Generator>MSBuild:Compile</Generator>
      <SubType>Designer</SubType>
    </ApplicationDefinition>
```

上面代码是将 App.xaml 识别作为 ApplicationDefinition 的特殊内容，这个内容将被作为创建入口函数的出发点文件。也就是 App.g.cs 文件里面存放入口函数就由此决定

在 `src\Microsoft.DotNet.Wpf\src\PresentationBuildTasks\MS\Internal\MarkupCompiler\MarkupCompiler.cs` 文件里面的 GenerateAppEntryPoint 函数，如此函数命名所示，这就是创建应用入口点的方法，大概逻辑如下

```csharp
        private void GenerateAppEntryPoint()
        {
            if (ApplicationFile.Length > 0)
            {

                // [STAThread]
                // public static void Main ()
                //
                CodeMemberMethod cmmMain = GenerateEntryPointMethod();

                if (cmmMain != null)
                {
                    //   SplashScreen splashScreen = new SplashScreen("SplashScreen.png");
                    //
                    CodeVariableReferenceExpression cvreSplashScreen = null;
                    if (!string.IsNullOrEmpty(_splashImage) && !HostInBrowser)
                    {
                        cvreSplashScreen = GenerateSplashScreenInstance(cmmMain);
                    }

                    //   MyApplication app = new MyApplication();
                    //
                    CodeVariableReferenceExpression cvreApp = GenerateAppInstance(cmmMain);

                    if (_ccRoot.InitializeComponentFn != null)
                    {
                        //   app.InitializeComponent();
                        //
                        CodeMethodInvokeExpression cmieIT = new CodeMethodInvokeExpression();
                        cmieIT.Method = new CodeMethodReferenceExpression(cvreApp, INITIALIZE_COMPONENT);
                        cmmMain.Statements.Add(new CodeExpressionStatement(cmieIT));
                    }

                    if (!HostInBrowser)
                    {
                        //   app.Run();
                        //
                        CodeMethodReferenceExpression cmreRun = new CodeMethodReferenceExpression(cvreApp, "Run");
                        CodeMethodInvokeExpression cmieRun = new CodeMethodInvokeExpression();
                        cmieRun.Method = cmreRun;

                        CodeStatement csRun = new CodeExpressionStatement(cmieRun);
                        cmmMain.Statements.Add(csRun);
                    }

                    _ccRoot.CodeClass.Members.Add(cmmMain);
                }
            }
        }
```

在 WPF 中不是拼接字符串的方式完成代码生成的，而是需要用上代码生成逻辑进行生成。在上面代码中有各个注释来告诉大家生成代码的作用，阅读方便

调用链关系上，通过 [Roslyn 如何了解某个项目在 msbuild 中所有用到的属性以及构建过程](https://blog.lindexi.com/post/Roslyn-%E5%A6%82%E4%BD%95%E4%BA%86%E8%A7%A3%E6%9F%90%E4%B8%AA%E9%A1%B9%E7%9B%AE%E5%9C%A8-msbuild-%E4%B8%AD%E6%89%80%E6%9C%89%E7%94%A8%E5%88%B0%E7%9A%84%E5%B1%9E%E6%80%A7%E4%BB%A5%E5%8F%8A%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B.html ) 的方法，可以看到在构建过程中，将会先使用 UsingTask 加载 MarkupCompilePass2 任务

```xml
  <UsingTask TaskName="Microsoft.Build.Tasks.Windows.MarkupCompilePass2" AssemblyFile="$(_PresentationBuildTasksAssembly)" xmlns="http://schemas.microsoft.com/developer/msbuild/2003" />
```

接着有一个专门的 Target 用来执行，如下面代码

```xml
  <Target Name="MarkupCompilePass2" Condition="Exists('$(IntermediateOutputPath)$(AssemblyName)_MarkupCompile.lref')" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <Message Text="(in) References: '@(ReferencePath);@(AssemblyForLocalTypeReference)'" Condition="'$(MSBuildTargetsVerbose)'=='true'" />
    <MarkupCompilePass2 AssemblyName="$(AssemblyName)" OutputType="$(OutputType)" Language="$(Language)" LocalizationDirectivesToLocFile="$(LocalizationDirectivesToLocFile)" RootNamespace="$(RootNamespace)" References="@(ReferencePath);@(AssemblyForLocalTypeReference)" KnownReferencePaths="$(MSBuildBinPath);$(TargetFrameworkDirectory);@(_TargetFrameworkSDKDirectoryItem);@(KnownReferencePaths)" AssembliesGeneratedDuringBuild="@(AssembliesGeneratedDuringBuild)" AlwaysCompileMarkupFilesInSeparateDomain="$(AlwaysCompileMarkupFilesInSeparateDomain)" XamlDebuggingInformation="$(XamlDebuggingInformation)" GeneratedBaml="" OutputPath="$(IntermediateOutputPath)" ContinueOnError="false">
      <!--
               Output Items for MarkupCompilePass2

               If MarkupCompilePass2 is only for SatelliteAssembly, Append all the generated baml files into SatelliteEmbeddedFiles, No FileClassifier is required.
               If MarupCompilePass2 is for Main Assembly as well, output the Baml files into GeneratedBaml, FileClassifier task will be invoked later.
          -->
      <Output ItemName="GeneratedBamlWithLocalType" TaskParameter="GeneratedBaml" Condition="'$(MSBuildTargetsVerbose)'=='true'" />
      <Output ItemName="GeneratedBaml" TaskParameter="GeneratedBaml" Condition="'$(_RequireMCPass2ForMainAssembly)' == 'true'" />
      <Output ItemName="SatelliteEmbeddedFiles" TaskParameter="GeneratedBaml" Condition="'$(_RequireMCPass2ForSatelliteAssemblyOnly)' == 'true'" />
      <!-- Put the generated files in item FileWrites so that they can be cleaned up appropriately in a next Rebuild -->
      <Output ItemName="FileWrites" TaskParameter="GeneratedBaml" />
    </MarkupCompilePass2>
    <Message Text="(out) After MarkupCompilePass2, SatelliteEmbeddedFiles: '@(SatelliteEmbeddedFiles)'" Condition="'$(MSBuildTargetsVerbose)'=='true'" />
    <Message Text="(out) GeneratedBamlWithLocalType: '@(GeneratedBamlWithLocalType)'" Condition="'$(MSBuildTargetsVerbose)'=='true'" />
  </Target>
```

因此 `GenerateAppEntryPoint` 函数的调用是放在 WPF 项目构建过程中执行，细节请看 [WPF 程序的编译过程 - walterlv](https://blog.walterlv.com/post/how-wpf-assemblies-are-compiled.html )

在 MarkupCompilePass2 里面，将会经过层层调用，使用 `GenerateAppEntryPoint` 函数创建出 App.g.cs 的入口函数。在 GenerateAppEntryPoint 函数包含如下步骤

- 使用 GenerateEntryPointMethod 创建空的 Main 函数本身
- 使用 GenerateSplashScreenInstance 创建 SplashScreen 的调用，如果开发者有设置的话才会调用
- 通过 GenerateAppInstance 创建 App 对象，在接下来逻辑调用 InitializeComponent 和 Run 方法

下面是详细的步骤

在 `GenerateAppEntryPoint` 函数使用 `GenerateEntryPointMethod` 函数即可创建 Main 函数本身，里面不包含任何的逻辑

```csharp
        private CodeMemberMethod GenerateEntryPointMethod()
        {
            CodeMemberMethod cmmMain = null;
            CodeDomProvider codeProvider = EnsureCodeProvider();

            // 如果支持创建入口函数。咱这里都是支持的，放心
            if (codeProvider.Supports(GeneratorSupport.EntryPointMethod))
            {
                //
                // [STAThread]
                // public static void Main ()
                //
                
                // 创建入口点函数定义
                cmmMain = new CodeEntryPointMethod();
                // 设置这是静态的公开的，其实 Main 函数也可以不是公开的
                cmmMain.Attributes = MemberAttributes.Public | MemberAttributes.Static;
                // 标记 STAThread 特性，加上这个特性可以用来解决 WPF 调用 OLE 以及 COM 的提示不兼容，当然 STA 是很复杂的，还请大家自行了解
                cmmMain.CustomAttributes.Add(new CodeAttributeDeclaration(typeof(STAThreadAttribute).FullName));
                // 加上 DebuggerNonUserCodeAttribute 标记，这样调试器默认就不会进入这个函数了。假装 WPF 很厉害，没有 Main 函数，当然这解决新手不小心修改 Main 函数
                AddDebuggerNonUserCodeAttribute(cmmMain);
                // 加上了 GeneratedCodeAttribute 标记，可以用来告诉库开发者这是由哪个版本生成的
                AddGeneratedCodeAttribute(cmmMain);
                GenerateXmlComments(cmmMain, "Application Entry Point.");
                // 设置没有返回
                cmmMain.ReturnType = new CodeTypeReference(typeof(void));
            }

            return cmmMain;
        }
```

调用以上代码，就可以生成如下的逻辑，可以看到这是一个空白的 Main 方法

```csharp
        [System.STAThreadAttribute()]
        [System.Diagnostics.DebuggerNonUserCodeAttribute()]
        [System.CodeDom.Compiler.GeneratedCodeAttribute("PresentationBuildTasks", "5.0.1.0")]
        public static void Main() 
```

在接下来的逻辑就是生成 SplashScreen 的代码了

```csharp
                    //   SplashScreen splashScreen = new SplashScreen("SplashScreen.png");
                    //
                    CodeVariableReferenceExpression cvreSplashScreen = null;
                    if (!string.IsNullOrEmpty(_splashImage) && !HostInBrowser)
                    {
                        cvreSplashScreen = GenerateSplashScreenInstance(cmmMain);
                    }
```

这里的判断核心就是用户有在 csproj 中设置某个图片作为 SplashScreen 图片，如下面代码

```xml
  <ItemGroup>
    <SplashScreen Include="SplashScreen.png" />
  </ItemGroup>
```

如果有做这个设置，那么 `_splashImage` 字段将存在值。这里稍微吐槽 WPF 的 MarkupCompiler\MarkupCompiler.cs 的诡异设计，在这个类里面有以下的定义

```csharp
        /// <summary>
        /// Splash screen image to be displayed before application init
        /// </summary>
        public string SplashImage
        {
            set { _splashImage = value; }
        }
```

这是一个只开放给外面设置的属性，只有在 CompilerWrapper.cs 类里面设置，而 CompilerWrapper 是通过一个只设置的属性进行设置

```csharp
        /// <summary>
        /// Splash screen image to be displayed before application init
        /// </summary>
        internal string SplashImage
        {
            set { _mc.SplashImage = value; }
        }
```

而 CompilerWrapper 的 SplashImage 属性仅在 MarkupCompilePass1.cs 的 DoMarkupCompilation 函数进行设置，核心获取的值是在 MarkupCompilePass1 的 SplashScreen 属性获取

```csharp
        public ITaskItem[] SplashScreen
        {
            get { return _splashScreen; }
            set { _splashScreen = value; }
        }
```

通过这个定义可以了解到这是在编译过程中获取的

回到入口函数的创建，在 GenerateSplashScreenInstance 函数里面将会在用户有设置 SplashScreen 时加上 SplashScreen 对象的创建逻辑，如下面代码

```csharp
        private CodeVariableReferenceExpression GenerateSplashScreenInstance(CodeMemberMethod cmmMain)
        {
            // SplashScreen splashScreen = new SplashScreen(Assembly.GetExecutingAssembly(), "splash.png");
            // 获取创建的逻辑，也就是 new SplashScreen 的逻辑
            CodeObjectCreateExpression coceApplicationSplashScreen = new CodeObjectCreateExpression(SPLASHCLASSNAME, new CodePrimitiveExpression(GetSplashResourceId()));
            // ApplicationSplashScreen splashScreen = ...
            // 定义局部变量，其实上面的注释没有写对，是 SplashScreen 而不是 ApplicationSplashScreen 类型
            CodeVariableDeclarationStatement cvdsAppSplash = new CodeVariableDeclarationStatement(SPLASHCLASSNAME, SPLASHVAR, coceApplicationSplashScreen);
            // 将这个逻辑放入到 Main 函数的代码
            cmmMain.Statements.Add(cvdsAppSplash);

            // splashScreen.Show(true);
            // 如注释，下面两句生成上面注释的代码
            CodeVariableReferenceExpression cvreAppSplash = new CodeVariableReferenceExpression(SPLASHVAR);
            CodeMethodInvokeExpression cmieShowSplashScreen = new CodeMethodInvokeExpression(cvreAppSplash, "Show", new CodePrimitiveExpression(true));
            // 放入到 Main 函数
            cmmMain.Statements.Add(cmieShowSplashScreen);

            return cvreAppSplash;
        }
```

上面代码的 SPLASHCLASSNAME 的含义是 SplashClassName 定义如下面代码

```csharp
        private const string SPLASHCLASSNAME = "SplashScreen";
```

而 SPLASHVAR 的含义是 `Splash var` 也就是 SplashScreen 对象的变量名，定义如下面代码

```csharp
private const string SPLASHVAR = "splashScreen";
```

接下来就是调用 `GenerateAppInstance` 函数以来创建 App 对象

```csharp
  //   MyApplication app = new MyApplication();
  //
  CodeVariableReferenceExpression cvreApp = GenerateAppInstance(cmmMain);
```

在 GenerateAppInstance 的逻辑如下

```csharp
        private CodeVariableReferenceExpression GenerateAppInstance(CodeMemberMethod cmmMain)
        {
        	// 获取当前的 App 类型是哪个，因为开发者可以定义任意的命名文件，放在某个命名空间
            string appClassName = _ccRoot.SubClass.Length > 0 ? _ccRoot.SubClass
                                               : GetFullClassName(_ccRoot.CodeNS.Name, _ccRoot.CodeClass.Name);

            //  MyNS.MyApplication app = new MyNS.MyApplication();
            //
            CodeObjectCreateExpression coce;
            CodeVariableReferenceExpression cvre = new CodeVariableReferenceExpression(APPVAR);
            CodeExpression[] ctorParams = {};

            // 加入 new MyNS.MyApplication(); 的代码，这里的 MyNS 是 My namespace 的缩写
            coce = new CodeObjectCreateExpression(appClassName, ctorParams);

            // 加上 MyNS.MyApplication app = .. 的逻辑，加起来就是创建 App 的代码
            CodeVariableDeclarationStatement cvds = new CodeVariableDeclarationStatement(appClassName, APPVAR, coce);

            // 放入到 Main 函数
            cmmMain.Statements.Add(cvds);

            return cvre;
        }
```

执行到上面的代码，大概在入口函数就有以下代码

```csharp
        [System.STAThreadAttribute()]
        [System.Diagnostics.DebuggerNonUserCodeAttribute()]
        [System.CodeDom.Compiler.GeneratedCodeAttribute("PresentationBuildTasks", "5.0.1.0")]
        public static void Main() 
        {
            SplashScreen splashScreen = new SplashScreen("SplashScreen.png");
            splashScreen.Show(true);
            App app = new App();
```

可以看到用代码生成的逻辑会比拼接字符串需要更多的代码，如果用拼接字符串就不需要几行代码。上面代码的 SplashScreen 创建逻辑是可选的

在创建 App 完成之后，将会尝试判断是否存在 InitializeComponent 函数，如果存在就调用一下

```csharp
                    if (_ccRoot.InitializeComponentFn != null)
                    {
                        //   app.InitializeComponent();
                        //
                        CodeMethodInvokeExpression cmieIT = new CodeMethodInvokeExpression();
                        cmieIT.Method = new CodeMethodReferenceExpression(cvreApp, INITIALIZE_COMPONENT);
                        cmmMain.Statements.Add(new CodeExpressionStatement(cmieIT));
                    }
```

上面代码的 `INITIALIZE_COMPONENT` 定义如下面代码

```csharp
        private const string            INITIALIZE_COMPONENT = "InitializeComponent";
```

最后还需要加上 Run 方法执行，如下面代码

```csharp
                    if (!HostInBrowser)
                    {
                        //   app.Run();
                        //
                        CodeMethodReferenceExpression cmreRun = new CodeMethodReferenceExpression(cvreApp, "Run");
                        CodeMethodInvokeExpression cmieRun = new CodeMethodInvokeExpression();
                        cmieRun.Method = cmreRun;

                        CodeStatement csRun = new CodeExpressionStatement(cmieRun);
                        cmmMain.Statements.Add(csRun);
                    }
```

这样就完成了 Main 函数的创建，创建完成需要将 Main 函数加入到类中

```csharp
                    _ccRoot.CodeClass.Members.Add(cmmMain);
```

这就是在 WPF 中创建入口函数的所有逻辑。如果大家不熟悉代码创建的编写方式，就假装 WPF 是通过拼接字符串的形式创建的就可以


当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建


更多请看 [WPF 程序的编译过程 - walterlv](https://blog.walterlv.com/post/how-wpf-assemblies-are-compiled.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。