
# How to parse version range

Now we are making a solution that has to get the package reference. But the version of package reference is a range and the default version parser need input a version but not a version range.
This post will tell you how to parse the version range string to reference version.

<!--more-->


<!-- csdn -->
<!-- 标签：C# ，dotnetcore-->

The format for reference version is like this

```csharp
[2.1.0.293,3.0)
[1.1.0.34,2.0)
(1.1.0.34,2.0]
2.1

```

For parse the reference version string, we should make some property.


```csharp

    public class ReferenceVersion
    {
        public ReferenceVersion(Version version)
        {
            Version = version;
            MinVersion = version;
            MaxVersion = version;
            IsIncludeMaxVersion = true;
            IsIncludeMinVersion = true;
        }

        public ReferenceVersion(Version minVersion, Version maxVersion, bool isIncludeMinVersion,
            bool isIncludeMaxVersion)
        {
            Version = null;
            MinVersion = minVersion;
            MaxVersion = maxVersion;
            IsIncludeMinVersion = isIncludeMinVersion;
            IsIncludeMaxVersion = isIncludeMaxVersion;
        }

        public Version Version { get; }

        public Version MinVersion { get; }

        public Version MaxVersion { get; }

        public bool IsIncludeMinVersion { get; }

        public bool IsIncludeMaxVersion { get; }
}
```

I will use regex to get the string and parse the string to version.

```csharp
      public static ReferenceVersion Parser(string str)
        {
            if (_regex == null)
            {
                _regex = new Regex(@"([(|\[])([\d|.]*),([\d|.]*)([)|\]])", RegexOptions.Compiled);
            }

            var res = _regex.Match(str);

            if (res.Success)
            {
                var isIncludeMinVersion = res.Groups[1].Value;
                var minVersion = res.Groups[2].Value;
                var maxVersion = res.Groups[3].Value;
                var isIncludeMaxVersion = res.Groups[4].Value;

                return new ReferenceVersion
                (
                    string.IsNullOrEmpty(minVersion) ? null : Version.Parse(minVersion),
                    string.IsNullOrEmpty(maxVersion) ? null : Version.Parse(maxVersion),
                    isIncludeMinVersion.Equals("["),
                    isIncludeMaxVersion.Equals("]")
                );
            }

            return new ReferenceVersion(Version.Parse(str));
        }

        private static Regex _regex;
```

We can get the reference version in the solution file and know the solution reference package.

Full code:

```csharp
    /// <summary>
    ///     引用的版本
    /// 用来转换  [2.1.0.293,3.0)、 (1.1.0.3,2.0]、 5.2 的版本
    /// </summary>
    public class ReferenceVersion
    {
        /// <summary>
        ///     创建引用版本
        /// </summary>
        /// <param name="version">版本</param>
        public ReferenceVersion(Version version)
        {
            Version = version;
            MinVersion = version;
            MaxVersion = version;
            IsIncludeMaxVersion = true;
            IsIncludeMinVersion = true;
        }

        /// <summary>
        ///     创建引用版本
        /// </summary>
        /// <param name="minVersion">最低版本</param>
        /// <param name="maxVersion">最高版本</param>
        /// <param name="isIncludeMinVersion">是否包括最低版本</param>
        /// <param name="isIncludeMaxVersion">是否包括最高版本</param>
        public ReferenceVersion(Version minVersion, Version maxVersion, bool isIncludeMinVersion,
            bool isIncludeMaxVersion)
        {
            Version = null;
            MinVersion = minVersion;
            MaxVersion = maxVersion;
            IsIncludeMinVersion = isIncludeMinVersion;
            IsIncludeMaxVersion = isIncludeMaxVersion;
        }


        /// <summary>
        ///     版本
        /// </summary>
        public Version Version { get; }

        /// <summary>
        ///     最低版本
        /// </summary>
        public Version MinVersion { get; }

        /// <summary>
        ///     最高版本
        /// </summary>
        public Version MaxVersion { get; }

        /// <summary>
        ///     是否包括最低版本
        /// </summary>
        public bool IsIncludeMinVersion { get; }

        /// <summary>
        ///     是否包括最高版本
        /// </summary>
        public bool IsIncludeMaxVersion { get; }

        /// <summary>
        ///     转换版本
        /// </summary>
        /// <param name="str"></param>
        /// <returns></returns>
        public static ReferenceVersion Parser(string str)
        {
            if (_regex == null)
            {
                _regex = new Regex(@"([(|\[])([\d|.]*),([\d|.]*)([)|\]])", RegexOptions.Compiled);
            }

            var res = _regex.Match(str);

            if (res.Success)
            {
                var isIncludeMinVersion = res.Groups[1].Value;
                var minVersion = res.Groups[2].Value;
                var maxVersion = res.Groups[3].Value;
                var isIncludeMaxVersion = res.Groups[4].Value;

                return new ReferenceVersion
                (
                    string.IsNullOrEmpty(minVersion) ? null : Version.Parse(minVersion),
                    string.IsNullOrEmpty(maxVersion) ? null : Version.Parse(maxVersion),
                    isIncludeMinVersion.Equals("["),
                    isIncludeMaxVersion.Equals("]")
                );
            }

            return new ReferenceVersion(Version.Parse(str));
        }

        private static Regex _regex;
    }

```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。