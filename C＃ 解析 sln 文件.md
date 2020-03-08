# C＃ 解析 sln 文件

我的项目，[编码工具](https://marketplace.visualstudio.com/items?itemName=lindexigd.vs-extension-18109) 需要检测打开一个工程，获取所有项目。

但是发现原来的方法，如果存在文件夹，把项目放在文件夹中，那么是无法获得项目，于是我就找了一个方法去获得sln文件的所有项目。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

原先使用的方法`dte.Solution.Projects`但是放在文件夹的项目获取不到，所以使用堆栈提供的方法。

首先添加引用  `Microsoft.Build` 注意版本

![这里写图片描述](http://img.blog.csdn.net/20170209103948364?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGluZGV4aV9nZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后把我三个类放到项目，其实放两个就好了，具体参见我的[github](https://gist.github.com/lindexi/b36feb816fe9e586ffbbdf58397b25da)


```csharp
    public class Solution
    {
        //internal class SolutionParser
        //Name: Microsoft.Build.Construction.SolutionParser
        //Assembly: Microsoft.Build, Version=4.0.0.0

        static readonly Type s_SolutionParser;
        static readonly PropertyInfo s_SolutionParser_solutionReader;
        static readonly MethodInfo s_SolutionParser_parseSolution;
        static readonly PropertyInfo s_SolutionParser_projects;

        static Solution()
        {
            //s_SolutionParser_projects.GetValue()
            s_SolutionParser = Type.GetType("Microsoft.Build.Construction.SolutionParser, Microsoft.Build, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a", false, false);
            if (s_SolutionParser != null)
            {
                s_SolutionParser_solutionReader = s_SolutionParser.GetProperty("SolutionReader", BindingFlags.NonPublic | BindingFlags.Instance);
                s_SolutionParser_projects = s_SolutionParser.GetProperty("Projects", BindingFlags.NonPublic | BindingFlags.Instance);
                s_SolutionParser_parseSolution = s_SolutionParser.GetMethod("ParseSolution", BindingFlags.NonPublic | BindingFlags.Instance);
            }
        }

        public List<SolutionProject> Projects { get; private set; }
        public List<SolutionConfiguration> Configurations { get; private set; }

        public Solution(string solutionFileName)
        {
            if (s_SolutionParser == null)
            {
                throw new InvalidOperationException("Can not find type 'Microsoft.Build.Construction.SolutionParser' are you missing a assembly reference to 'Microsoft.Build.dll'?");
            }
            var solutionParser = s_SolutionParser.GetConstructors(BindingFlags.Instance | BindingFlags.NonPublic).First().Invoke(null);
            using (var streamReader = new StreamReader(solutionFileName))
            {
                s_SolutionParser_solutionReader.SetValue(solutionParser, streamReader, null);
                s_SolutionParser_parseSolution.Invoke(solutionParser, null);
            }
            var projects = new List<SolutionProject>();
            var array = (Array)s_SolutionParser_projects.GetValue(solutionParser, null);
            for (int i = 0; i < array.Length; i++)
            {
                projects.Add(new SolutionProject(array.GetValue(i)));
            }
            this.Projects = projects;
            GetProjectFullName(solutionFileName);
            //Object cfgArray = //s_SolutionParser_configurations.GetValue
            //    s_SolutionParser_projects.GetValue(solutionParser, null);
            //PropertyInfo[] pInfos = null;
            //pInfos = cfgArray.GetType().GetProperties();
            //int count = (int)pInfos[1].GetValue(cfgArray, null);

            //var configs = new List<SolutionConfiguration>();
            //for (int i = 0; i < count; i++)
            //{
            //    configs.Add(new SolutionConfiguration(pInfos[2].GetValue(cfgArray, new object[] { i })));
            //}

            //this.Configurations = configs;
        }

        private void GetProjectFullName(string solutionFileName)
        {
            DirectoryInfo solution = (new FileInfo(solutionFileName)).Directory;
            foreach (var temp in Projects.Where
                //(temp=>temp.RelativePath.EndsWith("csproj"))
                (temp => !temp.RelativePath.Equals(temp.ProjectName))
            )
            {
                GetProjectFullName(solution, temp);
            }
        }

        private void GetProjectFullName(DirectoryInfo solution, SolutionProject project)
        {
            //Uri newUri =new Uri(,UriKind./*Absolute*/);
            //if(project.RelativePath)

            project.FullName = System.IO.Path.Combine(solution.FullName, project.RelativePath);
        }
    }

    [DebuggerDisplay("{ProjectName}, {RelativePath}, {ProjectGuid}")]
    public class SolutionProject
    {
        static readonly Type s_ProjectInSolution;
        static readonly PropertyInfo s_ProjectInSolution_ProjectName;
        static readonly PropertyInfo s_ProjectInSolution_RelativePath;
        static readonly PropertyInfo s_ProjectInSolution_ProjectGuid;
        static readonly PropertyInfo s_ProjectInSolution_ProjectType;

        static SolutionProject()
        {
            s_ProjectInSolution = Type.GetType("Microsoft.Build.Construction.ProjectInSolution, Microsoft.Build, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a", false, false);
            if (s_ProjectInSolution != null)
            {
                s_ProjectInSolution_ProjectName = s_ProjectInSolution.GetProperty("ProjectName", BindingFlags.NonPublic | BindingFlags.Instance);
                s_ProjectInSolution_RelativePath = s_ProjectInSolution.GetProperty("RelativePath", BindingFlags.NonPublic | BindingFlags.Instance);
                s_ProjectInSolution_ProjectGuid = s_ProjectInSolution.GetProperty("ProjectGuid", BindingFlags.NonPublic | BindingFlags.Instance);
                s_ProjectInSolution_ProjectType = s_ProjectInSolution.GetProperty("ProjectType", BindingFlags.NonPublic | BindingFlags.Instance);
            }
        }

        public string ProjectName { get; private set; }
        public string RelativePath { get; private set; }
        public string ProjectGuid { get; private set; }
        public string ProjectType { get; private set; }
        public string FullName { set; get; }

        public SolutionProject(object solutionProject)
        {
            this.ProjectName = s_ProjectInSolution_ProjectName.GetValue(solutionProject, null) as string;
            this.RelativePath = s_ProjectInSolution_RelativePath.GetValue(solutionProject, null) as string;
            this.ProjectGuid = s_ProjectInSolution_ProjectGuid.GetValue(solutionProject, null) as string;
            this.ProjectType = s_ProjectInSolution_ProjectType.GetValue(solutionProject, null).ToString();
        }
    }

    public class SolutionConfiguration
    {
        static readonly Type s_ConfigInSolution;
        static readonly PropertyInfo configInSolution_configurationname;
        static readonly PropertyInfo configInSolution_fullName;
        static readonly PropertyInfo configInSolution_platformName;

        static SolutionConfiguration()
        {
            s_ConfigInSolution = Type.GetType("Microsoft.Build.Construction.ConfigurationInSolution, Microsoft.Build, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a", false, false);
            if (s_ConfigInSolution != null)
            {
                configInSolution_configurationname = s_ConfigInSolution.GetProperty("ConfigurationName", BindingFlags.NonPublic | BindingFlags.Instance);
                configInSolution_fullName = s_ConfigInSolution.GetProperty("FullName", BindingFlags.NonPublic | BindingFlags.Instance);
                configInSolution_platformName = s_ConfigInSolution.GetProperty("PlatformName", BindingFlags.NonPublic | BindingFlags.Instance);
            }
        }

        public string configurationName { get; private set; }
        public string fullName { get; private set; }
        public string platformName { get; private set; }


        public SolutionConfiguration(object solutionConfiguration)
        {
            this.configurationName = configInSolution_configurationname.GetValue(solutionConfiguration, null) as string;
            this.fullName = configInSolution_fullName.GetValue(solutionConfiguration, null) as string;
            this.platformName = configInSolution_platformName.GetValue(solutionConfiguration, null) as string;
        }
    }
```

注意要引用


```csharp
     using System;
     using System.Collections.Generic;
     using System.Diagnostics;
     using System.IO;
     using System.Linq;
     using System.Reflection;

```

稍微说下上面代码，主要用的是反射。

用反射获得解析 sln 的 `s_SolutionParser_parseSolution` 他可以获得所有项目。

但是获得的项目路径是相对的，于是使用[C＃ 相对路径转绝对路径](http://lindexi.oschina.io/lindexi/post/C-%E7%9B%B8%E5%AF%B9%E8%B7%AF%E5%BE%84%E8%BD%AC%E7%BB%9D%E5%AF%B9%E8%B7%AF%E5%BE%84/)，可以转换项目路径。

## 使用

输入工程文件名就好，输入工程名，会自动获得所有项目。


```csharp
  Solution solution = new Solution(工程文件路径);
```

获得工程文件的所有项目


```csharp
  foreach (var temp in solution.Projects)
  {


  }
```



<script src="https://gist.github.com/lindexi/b36feb816fe9e586ffbbdf58397b25da.js"></script>

代码：https://gist.github.com/lindexi/b36feb816fe9e586ffbbdf58397b25da

参见：https://msdn.microsoft.com/en-us/library/microsoft.build.buildengine.project.propertygroups%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396

https://msdn.microsoft.com/en-us/library/microsoft.build.buildengine.project%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396

http://stackoverflow.com/questions/707107/parsing-visual-studio-solution-files

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。