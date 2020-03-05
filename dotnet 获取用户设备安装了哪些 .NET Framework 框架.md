# dotnet 获取用户设备安装了哪些 .NET Framework 框架

从注册表可以拿到当前用户安装的 .NET Framework 版本，本文告诉大家如何解析这些信息

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在注册表的当前设备的 `SOFTWARE\Microsoft\NET Framework Setup\NDP\` 可以拿到在设备安装的 .NET Framework 版本

大概从注册表拿到的数据就是这样

```csharp
v2.0.50727=.NET Framework 2.0 SP2; v3.0=.NET Framework 3.0 SP2; v3.5=.NET Framework 3.5 SP1; v4 Client=.NET Framework 4.5 Client Profile; v4 Full=.NET Framework 4.5; 
```

本文主要告诉大家如何从上面的字符串解析出用户安装了哪些 .NET Framework 版本

各个版本的 .NET Framework 依赖的系统请看[.NET Framework Versions and Dependencies](https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#targeting-and-running-net-framework-apps-for-version-45-and-later?wt.mc_id=MVP )

```csharp
    public class UserNetFramework
    {
        /// <summary>
        /// .NET Framework 2.0 SP2
        /// </summary>
        public bool InstalledNETFramework20SP2 => _list.Contains(NETFramework20SP2);

        private const string NETFramework20SP2 = ".NET Framework 2.0 SP2";

        /// <summary>
        /// .NET Framework 3.0 SP2
        /// </summary>
        public bool InstalledNETFramework30SP2 => _list.Contains(NETFramework30SP2);

        private const string NETFramework30SP2 = ".NET Framework 3.0 SP2";

        /// <summary>
        /// .NET Framework 3.5 SP1
        /// </summary>
        public bool InstalledNETFramework35SP1 => _list.Contains(NETFramework35SP1);

        private const string NETFramework35SP1 = ".NET Framework 3.5 SP1";

        /// <summary>
        /// .NET Framework 4 Client Profile
        /// </summary>
        public bool InstalledNETFramework4CP => _list.Contains(NETFramework4CP);

        private const string NETFramework4CP = ".NET Framework 4 Client Profile";


        /// <summary>
        /// .NET Framework 4 Full
        /// </summary>
        public bool InstalledNETFramework4F => _list.Contains(NETFramework4F);

        private const string NETFramework4F = ".NET Framework 4 Full";

        /// <summary>
        /// .NET Framework 4.5 Client Profile
        /// </summary>
        public bool InstalledNETFramework45CP => _list.Contains(NETFramework45CP);

        private const string NETFramework45CP = ".NET Framework 4.5 Client Profile";

        /// <summary>
        /// .NET Framework 4.5
        /// </summary>
        public bool InstalledNETFramework45 => _list.Contains(NETFramework45);

        private const string NETFramework45 = ".NET Framework 4.5";

        /// <summary>
        /// .NET Framework 4.5.1
        /// </summary>
        public bool InstalledNETFramework451 => _list.Contains(NETFramework451);

        private const string NETFramework451 = ".NET Framework 4.5.1";

        /// <summary>
        /// .NET Framework 4.5.2
        /// </summary>
        public bool InstalledNETFramework452 => _list.Contains(NETFramework452);

        private const string NETFramework452 = ".NET Framework 4.5.2";

        /// <summary>
        /// .NET Framework 4.6
        /// </summary>
        public bool InstalledNETFramework46 => _list.Contains(NETFramework46);

        private const string NETFramework46 = ".NET Framework 4.6";

        /// <summary>
        /// .NET Framework 4.6.1
        /// </summary>
        public bool InstalledNETFramework461 => _list.Contains(NETFramework461);

        private const string NETFramework461 = ".NET Framework 4.6.1";

        /// <summary>
        /// .NET Framework 4.7 Client Profile
        /// </summary>
        public bool InstalledNETFramework47CP => _list.Contains(NETFramework47CP);

        private const string NETFramework47CP = ".NET Framework 4.7 Client Profile";

        /// <summary>
        /// .NET Framework 4.7
        /// </summary>
        public bool InstalledNETFramework47 => _list.Contains(NETFramework47);

        private const string NETFramework47 = ".NET Framework 4.7";

        private readonly HashSet<string> _list = new HashSet<string>();

        public static UserNetFramework Parser(string str)
        {
            var userNetFramework = new UserNetFramework();

            foreach (var temp in new[]
            {
                NETFramework47CP, 
                NETFramework47, 
                NETFramework461, 
                NETFramework46, 
                NETFramework452, 
                NETFramework451,
                NETFramework45CP,
                NETFramework45, 
                NETFramework4CP,
                NETFramework4F,
                NETFramework35SP1, 
                NETFramework30SP2,
                NETFramework20SP2,
            })
            {
                if (str.Contains(temp))
                {
                    str = str.Replace(temp, "");

                    userNetFramework._list.Add(temp);
                }
            }

            return userNetFramework;
        }
    }

```

[.NET Framework Versions and Dependencies](https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#targeting-and-running-net-framework-apps-for-version-45-and-later?wt.mc_id=MVP )


[.NET Framework 4.x 程序到底运行在哪个 CLR 版本之上 - walterlv](https://walterlv.gitee.io/dotnet/2017/09/22/dotnet-version.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
