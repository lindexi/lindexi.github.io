---
title: dotnet 测试在 Linux 系统上的 Environment.GetFolderPath 行为
description: 由于 Environment.GetFolderPath 可以传入的参数里面，有许多都是 Windows 系统特有的，在 Linux 上不存在的，也没有映射对应的文件夹。本文将在 WSL Debian 和 UOS 系统上测试 Environment.GetFolderPath 行为
tags: dotnet
category: 
---

<!-- CreateTime:2024/1/17 16:37:20 -->

<!-- 发布 -->
<!-- 博客 -->

测试使用 Environment.SpecialFolder 的各个枚举获取路径的代码如下

```csharp
            foreach (var name in Enum.GetNames<Environment.SpecialFolder>())
            {
                Console.WriteLine($"{name} = {Environment.GetFolderPath(Enum.Parse<Environment.SpecialFolder>(name))}");
            }
```

在 WSL Debian 的运行结果如下

```
Desktop =
Programs =
MyDocuments =
Personal =
Favorites =
Startup =
Recent =
SendTo =
StartMenu =
MyMusic =
MyVideos =
DesktopDirectory =
MyComputer =
NetworkShortcuts =
Fonts =
Templates =
CommonStartMenu =
CommonPrograms =
CommonStartup =
CommonDesktopDirectory =
ApplicationData = /home/user/.config
PrinterShortcuts =
LocalApplicationData = /home/user/.local/share
InternetCache =
Cookies =
History =
CommonApplicationData = /usr/share
Windows =
System =
ProgramFiles =
MyPictures =
UserProfile = /home/user
SystemX86 =
ProgramFilesX86 =
CommonProgramFiles =
CommonProgramFilesX86 =
CommonTemplates =
CommonDocuments =
CommonAdminTools =
AdminTools =
CommonMusic =
CommonPictures =
CommonVideos =
Resources =
LocalizedResources =
CommonOemLinks =
CDBurning =
```

在 UOS 系统的运行结果如下

```
Desktop = /home/lin/Desktop
Programs = 
MyDocuments = /home/lin/Documents
Personal = /home/lin/Documents
Favorites = 
Startup = 
Recent = 
SendTo = 
StartMenu = 
MyMusic = /home/lin/Music
MyVideos = /home/lin/Videos
DesktopDirectory = /home/lin/Desktop
MyComputer = 
NetworkShortcuts = 
Fonts = 
Templates = /home/lin/.Templates
CommonStartMenu = 
CommonPrograms = 
CommonStartup = 
CommonDesktopDirectory = 
ApplicationData = /home/lin/.config
PrinterShortcuts = 
LocalApplicationData = /home/lin/.local/share
InternetCache = 
Cookies = 
History = 
CommonApplicationData = /usr/share
Windows = 
System = 
ProgramFiles = 
MyPictures = /home/lin/Pictures
UserProfile = /home/lin
SystemX86 = 
ProgramFilesX86 = 
CommonProgramFiles = 
CommonProgramFilesX86 = 
CommonTemplates = 
CommonDocuments = 
CommonAdminTools = 
AdminTools = 
CommonMusic = 
CommonPictures = 
CommonVideos = 
Resources = 
LocalizedResources = 
CommonOemLinks = 
CDBurning = 
```

可以看到 UOS 上有更多的属性是存在值的，存在一些行为差异

另外，根据 [UOS 官方文档](https://doc.chinauos.com/content/M7kCi3QB_uwzIp6HyF5J) 的如下说明：


软件包不允许直接向$HOME目录直接写入文件，后期系统将会使用沙箱技术重新定向$HOME,任何依赖该特性的行为都可能失效。
应用使用如下环境变量指示的目录写入应用数据和配置：

```
    $XDG_DATA_HOME
    $XDG_CONFIG_HOME
    $XDG_CACHE_HOME
```

对于appid为org.deepin.browser的应用，其写入目录为：

```
    $XDG_DATA_HOME/org.deepin.browser
    $XDG_CONFIG_HOME/org.deepin.browser
    $XDG_CACHE_HOME/org.deepin.browser
```

我同时也测试了以上的 `XDG_DATA_HOME` 和 `XDG_CONFIG_HOME` 和 `XDG_CACHE_HOME` 环境变量的内容，在我的设备上的输出如下

```
XDG_DATA_HOME = /home/lin/.local/share
XDG_CONFIG_HOME = /home/lin/.config
XDG_CACHE_HOME = /home/lin/.cache
```

可以看到 `XDG_DATA_HOME` 和 `LocalApplicationData` 是对应的值。而 `XDG_CONFIG_HOME` 和 `ApplicationData` 是对应的值

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/61a7e77b8b86e17ccf2b5d1a9d0460d09cc95036/NurbeakairweWaharbaner) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/61a7e77b8b86e17ccf2b5d1a9d0460d09cc95036/NurbeakairweWaharbaner) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 61a7e77b8b86e17ccf2b5d1a9d0460d09cc95036
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 61a7e77b8b86e17ccf2b5d1a9d0460d09cc95036
```

获取代码之后，进入 NurbeakairweWaharbaner 文件夹

这里的 XDG 是 X Desktop Group 的缩写，更多 XDG 知识请参阅：

- [XDG基本目录规范 DeepinWiki](https://wiki.deepin.org/zh/03_%E6%8A%80%E6%9C%AF%E8%A7%84%E8%8C%83/02_XDG%E8%A7%84%E8%8C%83/XDG%E5%9F%BA%E6%9C%AC%E7%9B%AE%E5%BD%95%E8%A7%84%E8%8C%83 )
- [Linux 基本目录规范 ——XDG Winddoing's Notes](https://winddoing.github.io/post/ef694e1f.html#:~:text=%24XDG_DATA_HOME%20%E5%AE%9A%E4%B9%89%E4%BA%86%E5%BA%94%E5%AD%98%E5%82%A8%E7%94%A8%E6%88%B7%E7%89%B9%E5%AE%9A%E7%9A%84%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6%E7%9A%84%E5%9F%BA%E5%87%86%E7%9B%AE%E5%BD%95%E3%80%82,%E9%BB%98%E8%AE%A4%E5%80%BC%E6%98%AF%20%24HOME%2F.local%2Fshare%20%E3%80%82 )

在 dotnet 的 runtime 底层的 Environment.GetFolderPath 实现如下

```csharp
    public static partial class Environment
    {
        private static string GetFolderPathCore(SpecialFolder folder, SpecialFolderOption option)
        {
            // Get the path for the SpecialFolder
            string path = GetFolderPathCoreWithoutValidation(folder) ?? string.Empty;
            Debug.Assert(path != null);

            // If we didn't get one, or if we got one but we're not supposed to verify it,
            // or if we're supposed to verify it and it passes verification, return the path.
            if (path.Length == 0 ||
                option == SpecialFolderOption.DoNotVerify ||
                Interop.Sys.Access(path, Interop.Sys.AccessMode.R_OK) == 0)
            {
                return path;
            }

            // Failed verification.  If None, then we're supposed to return an empty string.
            // If Create, we're supposed to create it and then return the path.
            if (option == SpecialFolderOption.None)
            {
                return string.Empty;
            }

            Debug.Assert(option == SpecialFolderOption.Create);

            Directory.CreateDirectory(path);

            return path;
        }

        private static string? GetFolderPathCoreWithoutValidation(SpecialFolder folder)
        {
            // First handle any paths that involve only static paths, avoiding the overheads of getting user-local paths.
            // https://www.freedesktop.org/software/systemd/man/file-hierarchy.html
            switch (folder)
            {
                case SpecialFolder.CommonApplicationData: return "/usr/share";
                case SpecialFolder.CommonTemplates: return "/usr/share/templates";
#if TARGET_OSX
                case SpecialFolder.ProgramFiles: return "/Applications";
                case SpecialFolder.System: return "/System";
#endif
            }

            // All other paths are based on the XDG Base Directory Specification:
            // https://specifications.freedesktop.org/basedir-spec/latest/
            string? home = null;
            try
            {
                home = PersistedFiles.GetHomeDirectory();
            }
            catch (Exception exc)
            {
                Debug.Fail($"Unable to get home directory: {exc}");
            }

            // Fall back to '/' when we can't determine the home directory.
            // This location isn't writable by non-root users which provides some safeguard
            // that the application doesn't write data which is meant to be private.
            if (string.IsNullOrEmpty(home))
            {
                home = "/";
            }

            // TODO: Consider caching (or precomputing and caching) all subsequent results.
            // This would significantly improve performance for repeated access, at the expense
            // of not being responsive to changes in the underlying environment variables,
            // configuration files, etc.

            switch (folder)
            {
                case SpecialFolder.UserProfile:
                    return home;

                case SpecialFolder.Templates:
                    return ReadXdgDirectory(home, "XDG_TEMPLATES_DIR", "Templates");
                // TODO: Consider merging the OSX path with the rest of the Apple systems here:
                // https://github.com/dotnet/runtime/blob/main/src/libraries/System.Private.CoreLib/src/System/Environment.iOS.cs
#if TARGET_OSX
                case SpecialFolder.Desktop:
                case SpecialFolder.DesktopDirectory:
                    return Interop.Sys.SearchPath(NSSearchPathDirectory.NSDesktopDirectory);
                case SpecialFolder.ApplicationData:
                case SpecialFolder.LocalApplicationData:
                    return Interop.Sys.SearchPath(NSSearchPathDirectory.NSApplicationSupportDirectory);
                case SpecialFolder.MyDocuments: // same value as Personal
                    return Interop.Sys.SearchPath(NSSearchPathDirectory.NSDocumentDirectory);
                case SpecialFolder.MyMusic:
                    return Interop.Sys.SearchPath(NSSearchPathDirectory.NSMusicDirectory);
                case SpecialFolder.MyVideos:
                    return Interop.Sys.SearchPath(NSSearchPathDirectory.NSMoviesDirectory);
                case SpecialFolder.MyPictures:
                    return Interop.Sys.SearchPath(NSSearchPathDirectory.NSPicturesDirectory);
                case SpecialFolder.Fonts:
                    return Path.Combine(home, "Library", "Fonts");
                case SpecialFolder.Favorites:
                    return Path.Combine(home, "Library", "Favorites");
                case SpecialFolder.InternetCache:
                    return Interop.Sys.SearchPath(NSSearchPathDirectory.NSCachesDirectory);
#else
                case SpecialFolder.Desktop:
                case SpecialFolder.DesktopDirectory:
                    return ReadXdgDirectory(home, "XDG_DESKTOP_DIR", "Desktop");
                case SpecialFolder.ApplicationData:
                    return GetXdgConfig(home);
                case SpecialFolder.LocalApplicationData:
                    // "$XDG_DATA_HOME defines the base directory relative to which user specific data files should be stored."
                    // "If $XDG_DATA_HOME is either not set or empty, a default equal to $HOME/.local/share should be used."
                    string? data = GetEnvironmentVariable("XDG_DATA_HOME");
                    if (data is null || !data.StartsWith('/'))
                    {
                        data = Path.Combine(home, ".local", "share");
                    }
                    return data;
                case SpecialFolder.MyDocuments: // same value as Personal
                    return ReadXdgDirectory(home, "XDG_DOCUMENTS_DIR", "Documents");
                case SpecialFolder.MyMusic:
                    return ReadXdgDirectory(home, "XDG_MUSIC_DIR", "Music");
                case SpecialFolder.MyVideos:
                    return ReadXdgDirectory(home, "XDG_VIDEOS_DIR", "Videos");
                case SpecialFolder.MyPictures:
                    return ReadXdgDirectory(home, "XDG_PICTURES_DIR", "Pictures");
                case SpecialFolder.Fonts:
                    return Path.Combine(home, ".fonts");
#endif
            }

            // No known path for the SpecialFolder
            return string.Empty;
        }

        private static string GetXdgConfig(string home)
        {
            // "$XDG_CONFIG_HOME defines the base directory relative to which user specific configuration files should be stored."
            // "If $XDG_CONFIG_HOME is either not set or empty, a default equal to $HOME/.config should be used."
            string? config = GetEnvironmentVariable("XDG_CONFIG_HOME");
            if (config is null || !config.StartsWith('/'))
            {
                config = Path.Combine(home, ".config");
            }
            return config;
        }

        private static string ReadXdgDirectory(string homeDir, string key, string fallback)
        {
            Debug.Assert(!string.IsNullOrEmpty(homeDir), $"Expected non-empty homeDir");
            Debug.Assert(!string.IsNullOrEmpty(key), $"Expected non-empty key");
            Debug.Assert(!string.IsNullOrEmpty(fallback), $"Expected non-empty fallback");

            string? envPath = GetEnvironmentVariable(key);
            if (envPath is not null && envPath.StartsWith('/'))
            {
                return envPath;
            }

            // Use the user-dirs.dirs file to look up the right config.
            // Note that the docs also highlight a list of directories in which to look for this file:
            // "$XDG_CONFIG_DIRS defines the preference-ordered set of base directories to search for configuration files in addition
            //  to the $XDG_CONFIG_HOME base directory. The directories in $XDG_CONFIG_DIRS should be separated with a colon ':'. If
            //  $XDG_CONFIG_DIRS is either not set or empty, a value equal to / etc / xdg should be used."
            // For simplicity, we don't currently do that.  We can add it if/when necessary.

            string userDirsPath = Path.Combine(GetXdgConfig(homeDir), "user-dirs.dirs");
            if (Interop.Sys.Access(userDirsPath, Interop.Sys.AccessMode.R_OK) == 0)
            {
                try
                {
                    using (var reader = new StreamReader(userDirsPath))
                    {
                        string? line;
                        while ((line = reader.ReadLine()) != null)
                        {
                            // Example lines:
                            // XDG_DESKTOP_DIR="$HOME/Desktop"
                            // XDG_PICTURES_DIR = "/absolute/path"

                            // Skip past whitespace at beginning of line
                            int pos = 0;
                            SkipWhitespace(line, ref pos);
                            if (pos >= line.Length) continue;

                            // Skip past requested key name
                            if (string.CompareOrdinal(line, pos, key, 0, key.Length) != 0) continue;
                            pos += key.Length;

                            // Skip past whitespace and past '='
                            SkipWhitespace(line, ref pos);
                            if (pos >= line.Length - 4 || line[pos] != '=') continue; // 4 for ="" and at least one char between quotes
                            pos++; // skip past '='

                            // Skip past whitespace and past first quote
                            SkipWhitespace(line, ref pos);
                            if (pos >= line.Length - 3 || line[pos] != '"') continue; // 3 for "" and at least one char between quotes
                            pos++; // skip past opening '"'

                            // Skip past relative prefix if one exists
                            bool relativeToHome = false;
                            const string RelativeToHomePrefix = "$HOME/";
                            if (string.CompareOrdinal(line, pos, RelativeToHomePrefix, 0, RelativeToHomePrefix.Length) == 0)
                            {
                                relativeToHome = true;
                                pos += RelativeToHomePrefix.Length;
                            }
                            else if (line[pos] != '/') // if not relative to home, must be absolute path
                            {
                                continue;
                            }

                            // Find end of path
                            int endPos = line.IndexOf('"', pos);
                            if (endPos <= pos) continue;

                            // Got we need.  Now extract it.
                            string path = line.Substring(pos, endPos - pos);
                            return relativeToHome ?
                                Path.Combine(homeDir, path) :
                                path;
                        }
                    }
                }
                catch (Exception exc)
                {
                    // assembly not found, file not found, errors reading file, etc. Just eat everything.
                    Debug.Fail($"Failed reading {userDirsPath}: {exc}");
                }
            }

            return Path.Combine(homeDir, fallback);
        }

        private static void SkipWhitespace(string line, ref int pos)
        {
            while (pos < line.Length && char.IsWhiteSpace(line[pos])) pos++;
        }
    }
```

注： 在 dotnet 6.0.26 和 dotnet 7 版本，获取的 MyDocuments 的值将会和 UserProfile 相同，都是指向 `$HOME` 环境变量的路径，如以下代码

```csharp
                case SpecialFolder.UserProfile:
                case SpecialFolder.MyDocuments: // same value as Personal
                     return home;
```

更详细的代码如下

```csharp
        private static string GetFolderPathCoreWithoutValidation(SpecialFolder folder)
        {
            // First handle any paths that involve only static paths, avoiding the overheads of getting user-local paths.
            // https://www.freedesktop.org/software/systemd/man/file-hierarchy.html
            switch (folder)
            {
                case SpecialFolder.CommonApplicationData: return "/usr/share";
                case SpecialFolder.CommonTemplates: return "/usr/share/templates";
#if TARGET_OSX
                case SpecialFolder.ProgramFiles: return "/Applications";
                case SpecialFolder.System: return "/System";
#endif
            }

            // All other paths are based on the XDG Base Directory Specification:
            // https://specifications.freedesktop.org/basedir-spec/latest/
            string? home = null;
            try
            {
                home = PersistedFiles.GetHomeDirectory();
            }
            catch (Exception exc)
            {
                Debug.Fail($"Unable to get home directory: {exc}");
            }

            // Fall back to '/' when we can't determine the home directory.
            // This location isn't writable by non-root users which provides some safeguard
            // that the application doesn't write data which is meant to be private.
            if (string.IsNullOrEmpty(home))
            {
                home = "/";
            }

            // TODO: Consider caching (or precomputing and caching) all subsequent results.
            // This would significantly improve performance for repeated access, at the expense
            // of not being responsive to changes in the underlying environment variables,
            // configuration files, etc.

            switch (folder)
            {
                case SpecialFolder.UserProfile:
                case SpecialFolder.MyDocuments: // same value as Personal
                    return home;
                case SpecialFolder.ApplicationData:
                    return GetXdgConfig(home);
                case SpecialFolder.LocalApplicationData:
                    // "$XDG_DATA_HOME defines the base directory relative to which user specific data files should be stored."
                    // "If $XDG_DATA_HOME is either not set or empty, a default equal to $HOME/.local/share should be used."
                    string? data = GetEnvironmentVariable("XDG_DATA_HOME");
                    if (string.IsNullOrEmpty(data) || data[0] != '/')
                    {
                        data = Path.Combine(home, ".local", "share");
                    }
                    return data;

                case SpecialFolder.Desktop:
                case SpecialFolder.DesktopDirectory:
                    return ReadXdgDirectory(home, "XDG_DESKTOP_DIR", "Desktop");
                case SpecialFolder.Templates:
                    return ReadXdgDirectory(home, "XDG_TEMPLATES_DIR", "Templates");
                case SpecialFolder.MyVideos:
                    return ReadXdgDirectory(home, "XDG_VIDEOS_DIR", "Videos");

#if TARGET_OSX
                case SpecialFolder.MyMusic:
                    return Path.Combine(home, "Music");
                case SpecialFolder.MyPictures:
                    return Path.Combine(home, "Pictures");
                case SpecialFolder.Fonts:
                    return Path.Combine(home, "Library", "Fonts");
                case SpecialFolder.Favorites:
                    return Path.Combine(home, "Library", "Favorites");
                case SpecialFolder.InternetCache:
                    return Path.Combine(home, "Library", "Caches");
#else
                case SpecialFolder.MyMusic:
                    return ReadXdgDirectory(home, "XDG_MUSIC_DIR", "Music");
                case SpecialFolder.MyPictures:
                    return ReadXdgDirectory(home, "XDG_PICTURES_DIR", "Pictures");
                case SpecialFolder.Fonts:
                    return Path.Combine(home, ".fonts");
#endif
            }

            // No known path for the SpecialFolder
            return string.Empty;
        }
```

以上代码地址： <https://github.com/dotnet/runtime/blob/v6.0.26/src/libraries/System.Private.CoreLib/src/System/Environment.GetFolderPathCore.Unix.cs>

在 dotnet 8 的 [Fix some incorrect SpecialFolder entries for Unix by Miepee · Pull Request #68610 · dotnet/runtime](https://github.com/dotnet/runtime/pull/68610 ) 的更改里面，优化了各路径的读取方法，从而更改了 MyDocuments 的返回值路径

详细文档请看 [.NET 8 中断性变更：Unix 上的 GetFolderPath 行为 - .NET Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/core/compatibility/core-libraries/8.0/getfolderpath-unix )

以上不仅变更了在 Linux 上的行为也变更了在安卓 macOS 等的行为
