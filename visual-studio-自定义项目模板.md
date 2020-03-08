# Visual Studio 自定义项目模板

经常需要新建一个项目，项目有很多重复的东西，如然后新建View文件夹，ViewModel文件夹，Model文件夹，还有把我们的ViewModelBase放入ViewModel，如果还用框架，还需要加上好多。

还需要在每个文件夹加上声明，于是每次做的重复的就有很多。

<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

而我一般还有用九幽统计，需要修改好多东西，每新建一个项目都要做这个，这样我觉得不好，在网上看到了自定义模板，不过垃圾微软官方说的好差，看不懂，看了老周的，还是觉得不懂，我就自己来创建一个试试。

第一步需要打开我们目录：C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\ProjectTemplates\CSharp\Windows Root\Windows UAP 

可以看到里面有很多个文件夹，一般我们打开最后一个，我也不知道你看到我这篇，垃圾微软把它改为最大多少，所以，一个一个来，我现在打开1033，（1033是老周博客写的）我的其实还有更后的，这个如果都是数字，就选最大的。

---

11月更新：

1033是数字版本`LCID（Locale ID，区域性标识符）`，1033代表英语，2052代表简体中文

多谢UltimateLove大神

---

第二步是把文件夹复制到桌面或其他离回收站比较近的地方，然后压缩一份保存。因为怕自己弄坏了。
<!-- 我们先把文件夹复制到我们用户文档或者自己程序的项目位置，然后压缩一份保存，因为怕自己弄坏 -->

然后我们用 Visual Studio 打开文件，记住，要打开 .csproj 是要用 vs 的菜单 文件->打开。

我们先打开 BlankApplication 里的 BlankApplication.vstemplate

我们要修改是 项目包含文件，把需要包括的文件放进去， TemplateContent 就是我们项目包含的文件

![这里写图片描述](http://img.blog.csdn.net/20160902105740743)

可以看到需要包含项目 Application.csproj，这个值后面 写了`ReplaceParameters="true"`，这就是我们会把东西代换，代换的我在后面讲。

然后就是项目包含文件，如 App.xaml `<ProjectItem ReplaceParameters="true" TargetFileName="App.xaml">App.xaml</ProjectItem>`

上面代码意识：从本地找到`App.xaml`文件，放在新建项目的`App.xaml`。因为放入之前需要代换，于是加上`ReplaceParameters="true"`

<!-- 那我们会代什么，我们打开`App.xaml` -->

那么代换是把什么换为什么？先看看文档做了哪些改变

![这里写图片描述](http://img.blog.csdn.net/20160902110135270)

看到`$safeprojectname$`这就是我们代换的项目名，于是接下来讲下可以代换的有哪些

<table Responsive="true"><tr Responsive="true"><th><p>
                <span id="mt17" class="sentence" data-guid="03144cce1fcdacdbe993e5266c0bf3f3" data-source="Parameter" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">
  <sentenceText xmlns="http://www.w3.org/1999/xhtml">参数</sentenceText>
</sentenceText></span>
              </p></th><th><p>
                <span id="mt18" class="sentence" data-guid="67daf92c833c41c95db874e18fcb2786" data-source="Description" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">描述</sentenceText></span>
              </p></th></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt19" class="sentence" data-guid="c133fa31e454fa144f7d7d90e9ae1693" data-source="clrversion" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">clrversion</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt20" class="sentence" data-guid="99a29f3977417776732298c48e65dfe4" data-source="Current version of the common language runtime (CLR)." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">公共语言运行时 (CLR) 的当前版本。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt21" class="sentence" data-guid="49705cbf330537640981f0ab18accad6" data-source="GUID [1-10]" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">GUID [1-10]</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt22" class="sentence" data-guid="e0fa6a3b9ee8f16086de726310c4b135" data-source="A GUID used to replace the project GUID in a project file." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">用于替换项目文件中的项目 GUID 的 GUID。</sentenceText></span>  <span id="mt23" class="sentence" data-guid="496e67ee5856defb304ddf191ba41dba" data-source="You can specify up to 10 unique GUIDs (for example, &lt;span class=&quot;code&quot;&gt;guid1)&lt;/span&gt;." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">最多可以指定 10 个唯一的 GUID（例如，<span class="code" xmlns="http://www.w3.org/1999/xhtml">guid1)</span>）。</sentenceText></span>  </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt24" class="sentence" data-guid="3e3640c36bf50aec770a87493828e76d" data-source="itemname" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">itemname</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt25" class="sentence" data-guid="45f07a95e0e4d41abc8629973ec2626a" data-source="The name provided by the user in the &lt;strong&gt;Add New Item&lt;/strong&gt; dialog box." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">用户在<strong xmlns="http://www.w3.org/1999/xhtml">添加新项</strong>对话框中提供的名称。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt26" class="sentence" data-guid="62f2e1d94ea5a730c4dfda5ead0bde29" data-source="machinename" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">machinename</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt27" class="sentence" data-guid="07677bf070f0091903f60cd3dffbfc75" data-source="The current computer name (for example, Computer01)." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">当前的计算机名称（例如，Computer01）。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt28" class="sentence" data-guid="0c79925e09c937c71744ced5b822bb9c" data-source="projectname" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">projectname</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt29" class="sentence" data-guid="5dc7586ced55345b6c0596d20776a515" data-source="The name provided by the user in the &lt;strong&gt;New Project&lt;/strong&gt; dialog box." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">用户在<strong xmlns="http://www.w3.org/1999/xhtml">新建项目</strong>对话框中提供的名称。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt30" class="sentence" data-guid="b8cb8cda7a77173c8cd60820bac24c51" data-source="registeredorganization" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">registeredorganization</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt31" class="sentence" data-guid="32030027fe55b00c51bee4c32d919923" data-source="The registry key value from HKLM\Software\Microsoft\Windows NT\CurrentVersion\RegisteredOrganization." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">HKLM\Software\Microsoft\Windows NT\CurrentVersion\RegisteredOrganization 中的注册表项值。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt32" class="sentence" data-guid="5d813332b11ed6b05867ff78e5dc118e" data-source="rootnamespace" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">rootnamespace</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt33" class="sentence" data-guid="fb3308554d201bf0102437cc193e91cb" data-source="The root namespace of the current project." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">当前项目的根命名空间。</sentenceText></span>  <span id="mt34" class="sentence" data-guid="2e96c802b934ca14367b106bc8c23617" data-source="This parameter applies only to item templates." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">此参数仅适用于项目模板。</sentenceText></span>  </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt35" class="sentence" data-guid="1d776f1950944fd630b39229db934fef" data-source="safeitemname" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">safeitemname</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt36" class="sentence" data-guid="5c6bf9887241db9208137f7209ff1535" data-source="The name provided by the user in the &lt;strong&gt;Add New Item&lt;/strong&gt; dialog box, with all unsafe characters and spaces removed." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">用户在<strong xmlns="http://www.w3.org/1999/xhtml">“添加新项”</strong>对话框中提供的名称，名称中移除了所有不安全的字符和空格。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt37" class="sentence" data-guid="5eee0cd89154fbf6bd9e081b5a94ced6" data-source="safeprojectname" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">safeprojectname</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt38" class="sentence" data-guid="944b58f8edf11ae44c537b00d5b613e9" data-source="The name provided by the user in the &lt;strong&gt;New Project&lt;/strong&gt; dialog box, with all unsafe characters and spaces removed." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">用户在<strong xmlns="http://www.w3.org/1999/xhtml">“新建项目”</strong>对话框中提供的名称，名称中移除了所有不安全的字符和空格。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt39" class="sentence" data-guid="07cc694b9b3fc636710fa08b6922c42b" data-source="time" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">time</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt40" class="sentence" data-guid="5007398966b1b1418146c463e385a194" data-source="The current time in the format DD/MM/YYYY 00:00:00." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">以 DD/MM/YYYY 00:00:00 格式表示的当前时间。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt41" class="sentence" data-guid="20529023398a5d29e764f6e787169d00" data-source="SpecificSolutionName" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">SpecificSolutionName</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt42" class="sentence" data-guid="6aa4abce5410b2782dae55878d8f0e93" data-source="The name of the solution." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">解决方案的名称。</sentenceText></span>  <span id="mt43" class="sentence" data-guid="0f28d8c9eee8721e7a248bffe1acae05" data-source="When &quot;create solution directory&quot; is checked, &lt;strong&gt;SpecificSolutionName&lt;/strong&gt; has the solution name." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">当“创建解决方案的目录”被选中，<strong xmlns="http://www.w3.org/1999/xhtml">SpecificSolutionName</strong> 具有解决方案的名称。</sentenceText></span>  <span id="mt44" class="sentence" data-guid="8f19232114d8a1d29699331a4cbce1aa" data-source="When &quot;create solution directory&quot; is not checked, &lt;strong&gt;SpecificSolutionName&lt;/strong&gt; is blank." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">当“创建解决方案的目录”没有被选中，<strong xmlns="http://www.w3.org/1999/xhtml">SpecificSolutionName</strong>是空。</sentenceText></span>  </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt45" class="sentence" data-guid="cbbacbb40776f2eda9611f5d6a1aceb7" data-source="userdomain" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">userdomain</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt46" class="sentence" data-guid="bdd1425d9a1a601a0cef89e8998dc36d" data-source="The current user domain." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">当前的用户域。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt47" class="sentence" data-guid="14c4b06b824ec593239362517f538b29" data-source="username" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">username</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt48" class="sentence" data-guid="2104843e8c143db53f8ab885786712fd" data-source="The current user name." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">当前的用户名。</sentenceText></span>
              </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt49" class="sentence" data-guid="3e729841ba9695cb7b4b18a70d26d9cd" data-source="webnamespace" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">webnamespace</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt50" class="sentence" data-guid="dfedbc6a8db02a652582ef84fa3f8855" data-source="The name of the current Web site." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">当前网站的名称。</sentenceText></span>  <span id="mt51" class="sentence" data-guid="53ed8b23db00a4418f607bbc6e330322" data-source="This parameter is used in the Web form template to guarantee unique class names." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">在 Web 窗体模板中使用此参数以确保类名称是唯一的。</sentenceText></span>  <span id="mt52" class="sentence" data-guid="0e5c2b875b8ed8d2eaa94599237b8633" data-source="If the Web site is at the root directory of the Web server, this template parameter resolves to the root directory of the Web Server." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">如果网站位于 Web 服务器的根目录下，则此模板参数将解析为 Web 服务器的根目录。</sentenceText></span>  </p></td></tr><tr><td data-th="&#xA;                Parameter&#xA;              "><p>
                <strong>
                  <span id="mt53" class="sentence" data-guid="84cdc76cabf41bd7c961f6ab12f117d8" data-source="year" xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">year</sentenceText></span>
                </strong>
              </p></td><td data-th="&#xA;                描述&#xA;              "><p>
                <span id="mt54" class="sentence" data-guid="ff973168b9c603228da0de366494f6df" data-source="The current year in the format YYYY." xml:space="preserve"><sentenceText xmlns="http://www.w3.org/1999/xhtml">以 YYYY 格式表示的当前年份。</sentenceText></span>
              </p></td></tr></table>

我们在添加我们的文件，如果需要放在项目的文件夹，而不是根目录，那么需要写`<ProjectItem ReplaceParameters="true" TargetFileName="Properties\AssemblyInfo.cs">AssemblyInfo.cs</ProjectItem>`在TargetFileName写上路径。

写完我们打开 `Application.csproj` ，他和一般的 .csproj 一样， ItemGroup 是项目文件，我们需要写入我们的文件是编译还是内容，添加文件很多是编译 Compile 。假如添加 NotifyProperty ，可以这样写

`<Compile Include="ViewModel\NotifyProperty.cs" />`

如果我们需要文件夹，空白的文件夹，我们可以这样写

```xml
    <Folder Include="Model\" />
```

把需要添加的文件和文件夹写完，进行保存。

然后把文件夹复制到`C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\ProjectTemplates\CSharp\Windows Root\Windows UAP\2052\BlankApplication` 这个位置需要和你复制出的位置一样。也就是数字 2052 对应的你复制出来的路径。

复制进去需要管理员。

然后关闭vs，再打开vs，注意需要把vs关闭看下任务管理，杀vs进程。然后新进项目，这时可以看到新建的模板。

如果想新建模板，不是替换 BlankApplication ，可以修改`TemplateID`为我们的，不使用原来的。不把原来的 BlankApplication 文件夹覆盖。打开vs 新建就可以看到自己新建的。



新建项目如果没有找到刚才做的项目，那么可以打开`C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\ProjectTemplatesCache`对应刚才相同位置。把文件复制，然后关闭vs，重新打开就可以看到。

除了手动复制还可以使用命令行输入 `devenv /setup`

如果提示`'devenv' 不是内部或外部命令，也不是可运行的程序`，在命令行输入

    

```csharp
cd C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE

devenv /setup
```





如果出现`未能正确加载“Microsoft.VisualStudio.Editor.Implementation.EditorPackage”包`

先重启下，如果还是不好，那么使用
    

```csharp
devenv /resetuserdata

```

### 修改图片

我们可以修改显示的图标，显示在新建的图标，需要我们复制一个图标进去，然后`PreviewImage`作为我们复制的图标。

### 修改Template名称

我们可以看到如果我们新建一个模板，看到的名称和原来一样，那么如何自定义自己模板名称？其实我们可以修改`TemplateData`

我们把`Name`改为自己的名字，把`Description`改为自己的对模板的修改或者别的，如“这是一个模板”。
    

```xml
<Description  >这是一个模板</Description>

```

还有其他的，如果遇到不知道的，欢迎联系 lindexi_gd@163.com

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。




