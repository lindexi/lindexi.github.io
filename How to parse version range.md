# How to parse version range

Now we are making a solution that has to get the package reference. But the version of package reference is a range and the default version parser need input a version but not a version range.

This post will tell you how to parse the version range string to reference version.

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->

<!-- 标签：C# ，dotnetcore -->

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

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br />This <span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" rel="dct:type">work</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="https://lindexi.github.io" property="cc:attributionName" rel="cc:attributionURL">https://lindexi.github.io</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.