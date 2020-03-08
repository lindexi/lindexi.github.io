# win10 uwp 如何打包Nuget给其他人

本文告诉大家，如果自己有做一些好用的库，如何使用 Nuget 打包之后上传，分享给大家。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->

<div id="toc"></div>

首先需要知道一些 Nuget 打包需要知道的，请看 [win10 uwp 上传Nuget 让别人用我们的库](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E4%B8%8A%E4%BC%A0Nuget-%E8%AE%A9%E5%88%AB%E4%BA%BA%E7%94%A8%E6%88%91%E4%BB%AC%E7%9A%84%E5%BA%93/ )

但是 UWP 的包和上面说的有一些不同，需要对打包做一些修改。

可以到 csdn 下载 Nuget 的程序或者到[https://www.nuget.org/downloads](https://www.nuget.org/downloads)下载

## 创建空白的spec

使用 Nuget 命令在空白的文件夹进行创建空白的包，使用命令`nuget spec`

假如下载的 Nuget 放在 `E:\` ，空白文件是 "E:\1" 那么使用的命令就是进入空白文件夹，然后需要写 Nuget 的路径才可以使用。按win+R输入 cmd 打开命令行，然后输入下面代码

```csharp
E: 进入E盘
cd 1 进入1文件夹
然后把 Nuget 拉进命令行
E:\nuget.exe spec
```

这时可以看到命令行输出 成功创建

```csharp
E:\1>E:\nuget.exe spec
已成功创建“Package.nuspec”。
```

可以看到现在存在 `Package.nuspec`文件，打开他可以看到下面的东西

```csharp
<?xml version="1.0"?>
<package >
  <metadata>
    <id>Package</id>
    <version>1.0.0</version>
    <authors>lindexi</authors>
    <owners>lindexi</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Package description</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2017</copyright>
    <tags>Tag1 Tag2</tags>
    <dependencies>
      <dependency id="SampleDependency" version="1.0" />
    </dependencies>
  </metadata>
</package>
```

如果你已经看过我上面的博客，那么就知道这些东西是可以如何写，但是 UWP 有一些不同，我现在没有使用上面博客的方法可以成功上传，于是就需要做一些修改。

## 对空白spec进行修改

首先是版本，现在的版本和id什么都需要自己写，也就是上面的内容都需要自己全部写。如果需要在 description 使用换行，直接回车就好。如果自己的库需要依赖，那么请修改 dependencies ，依赖的版本参见

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017924153320.jpg)

## 创建简单的库

上面写的叫 metadata ，写完之后可以创建一个新的 UWP 库，我在这创建一个叫 NrzlmhRzvy 的库

在里面创建一个类

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017924153624.jpg)

## 批量创建不同平台 dll 可以给不同的需要

右击解决方法批处理

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201792415375.jpg)

可以看到有很多的方法，点全选

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017924153752.jpg)

点击重新生成

可以看到生成了很多文件

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017924153858.jpg)

## 打包

接下来就是创建 Nuget ，首先需要把空白的包放到库的文件夹，这里创建的库是`E:\1\NrzlmhRzvy\`所以把`Package.nuspec`放在`E:\1\NrzlmhRzvy`，现在使用 SublimeText打开这个spec，对他做一些修改

```csharp

<?xml version="1.0"?>
<package >
  <metadata>
    这里不写
  </metadata>
  <files>
     这里写文件
  </files>

</package>

```

添加文件就是写文件的放在哪，在使用nuget会按照放在的位置，在不同的平台使用库，如果写错了，使用这个库的程序就无法使用，这里需要添加的文件有不同平台的，请看下面的代码

```csharp
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.dll" target="runtimes\win10-arm\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-arm\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.pri" target="runtimes\win10-arm\lib\uap10.0"/>

        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.dll" target="runtimes\win10-x64\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-x64\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.pri" target="runtimes\win10-x64\lib\uap10.0"/>

        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.dll" target="runtimes\win10-x86\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-x86\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.pri" target="runtimes\win10-x86\lib\uap10.0"/>
```

还需要添加入口，现在的代码如果使用，就会出现 `提供了编译时引用程序集，但没有与 win10-arm 兼容的运行时程序集` 所以需要添加 ref 请看下面

```csharp
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.dll" target="ref\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.pdb" target="ref\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.pri" target="ref\uap10.0"/>
```

于是现在的 spec 就是下面的代码

```csharp
<?xml version="1.0"?>
<package >
   <metadata>
    <id>Package</id>
    <version>1.0.0</version>
    <authors>lindexi</authors>
    <owners>lindexi</owners>
    <licenseUrl>http://LICENSE_URL_HERE_OR_DELETE_THIS_LINE</licenseUrl>
    <projectUrl>http://PROJECT_URL_HERE_OR_DELETE_THIS_LINE</projectUrl>
    <iconUrl>http://ICON_URL_HERE_OR_DELETE_THIS_LINE</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Package description</description>
    <releaseNotes>Summary of changes made in this release of the package.</releaseNotes>
    <copyright>Copyright 2017</copyright>
    <tags>Tag1 Tag2</tags>
    <!-- <dependencies>
      <dependency id="SampleDependency" version="1.0" />
    </dependencies> -->
  </metadata>
  <files>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.dll" target="runtimes\win10-arm\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-arm\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.pri" target="runtimes\win10-arm\lib\uap10.0"/>

        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.dll" target="runtimes\win10-x64\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-x64\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.pri" target="runtimes\win10-x64\lib\uap10.0"/>

        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.dll" target="runtimes\win10-x86\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-x86\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.pri" target="runtimes\win10-x86\lib\uap10.0"/>

          <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.dll" target="ref\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.pdb" target="ref\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.pri" target="ref\uap10.0"/>
        
     </files>
</package>
```

然后尝试使用本地的库，新建另一个项目，打开Nuget命令行，输入下面的代码

```csharp
install-package Package -Source E:\1\NrzlmhRzvy
```

或者点击选项打开 Nuget 管理，输入本地地址

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017924204915.jpg)

这样就可以添加打包的库，安装之后需要重新编译才可以使用

如果发现安装还没发现这个程序的类，那么重新编译可能就可以使用。如果发现安装提示不兼容，找不到库，就需要升级库的版本，然后重新生成。

但是这样可以看到，虽然库可以使用，但是没有注释，因为生成没有注释，如果需要注释，那么需要在库右击属性，生成，输出 xml ，实际上输出一份就好了，自己把他复制到各个平台。

或者在生成那里点击输出 xml文档，选择所有的平台，然后修改包的内容，添加输出的xml，这样就可以使用注释

下面就是整个spec的内容

```csharp
<?xml version="1.0"?>
<package >
   <metadata>
    <id>lindexi</id>
    <version>1.0.2</version>
    <authors>lindexi</authors>
    <owners>lindexi</owners>
    <licenseUrl>http://lindexi.oschina.io</licenseUrl>
    <projectUrl>http://lindexi.oschina.io</projectUrl>
    <iconUrl>http://lindexi.oschina.io</iconUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description> description</description>
    <releaseNotes></releaseNotes>
    <copyright>Copyright 2017</copyright>
    <tags>Tag1 </tags>
  </metadata>
  <files>
          <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.dll" target="runtimes\win10-arm\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-arm\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.XML" target="runtimes\win10-arm\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\ARM\Debug\NrzlmhRzvy.pri" target="runtimes\win10-arm\lib\uap10.0"/>

        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.dll" target="runtimes\win10-x64\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-x64\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.XML" target="runtimes\win10-x64\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x64\Debug\NrzlmhRzvy.pri" target="runtimes\win10-x64\lib\uap10.0"/>

        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.dll" target="runtimes\win10-x86\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.pdb" target="runtimes\win10-x86\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.XML" target="runtimes\win10-x86\lib\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\x86\Debug\NrzlmhRzvy.pri" target="runtimes\win10-x86\lib\uap10.0"/>

          <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.dll" target="ref\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.pdb" target="ref\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.pri" target="ref\uap10.0"/>
        <file src=".\NrzlmhRzvy\bin\Debug\NrzlmhRzvy.XML" target="ref\uap10.0"/>

     </files>
</package>
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。