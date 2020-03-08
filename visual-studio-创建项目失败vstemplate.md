# Visual studio 创建项目失败vstemplate

Visual studio 创建项目失败 提示 the vstemplate file references the wizard class 'Microsoft.VisualStudio.WinRT.TemplateWizards.ApplicationInsights.Wizard' which does not exsist in the assembly 'Microsoft.VisualStudio.WinRT.TemplateWizards, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'.
<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

```csharp
vstemplate文件引用的向导类“Microsoft.VisualStudio.WinRT.TemplateWizards.ApplicationInsights.Wizard"在程序集”Microsoft.VisualStudio.WinRT.TemplateWizards,Version=14.0.0.0,Culture=neutral,PublicKeyToken=b03f5f7f11d50a3a"中不存在。


```

![这里写图片描述](http://img.blog.csdn.net/20160828152032748) 


解决方法简单，在我们VisualStudio安装，C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\ProjectTemplates\CSharp\Windows Root\Windows UAP

---
感谢anngg2008 
如果在CSharp文件夹没找到，请到`Common7\IDE\ProjectTemplatesCache\CSharp\Windows UAP*\BlankApplication\BlankApplication.vstemplate `

`Common7\IDE\ProjectTemplates\CSharp\Windows UAP*\BlankApplication\BlankApplication.vstemplate`

---

可以找到我们的文件夹，一般是1033,如果有比较高的文件夹，那么都选择，进入，打开BlankApplication，把BlankApplication.vstemplatet拖到VisualStudio


![这里写图片描述](http://img.blog.csdn.net/20160828152134623)

找到Microsoft.VisualStudio.WinRT.TemplateWizards, Version=14.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a，FullClassName：Microsoft.VisualStudio.WinRT.TemplateWizards.ApplicationInsights.Wizard删除

![这里写图片描述](http://img.blog.csdn.net/20160828152347823)

然后保存到桌面，从桌面复制，这样是没法直接保存在C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\ProjectTemplates\CSharp\Windows Root\Windows UAP\1033\BlankApplication

复制选择管理员，这个需要我们管理员复制才可以

复制我们就可以新建我们的项目

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。




