# dotnet 删除文件夹方法

使用 C# 可以在 dotnet 一句话删除文件夹，但是这个方法坑，本文给大家一个好用的方法删除文件夹

<!--more-->
<!-- CreateTime:2020/2/2 22:11:51 -->

<!-- 发布 -->

下面代码可以复制在你的项目使用

```csharp
        public static void DeleteFolder(string dirPath)
        {
            if (!Directory.Exists(dirPath))
            {
                return;
            }

            var directory = dirPath;
            var subdirectories = new Stack<string>();
            subdirectories.Push(directory);

            var exceptionList = new List<Exception>();

            var folderList = new List<string>();

            // 尽可能地删除目录中的文件。

            // 如果出现异常也不需要记录
            while (subdirectories.Any())
            {
                var dir = subdirectories.Pop();
                folderList.Add(dir);

                try
                {
                    var files = Directory.GetFiles(dir);
                    foreach (var file in files)
                    {
                        try
                        {
                            File.SetAttributes(file, FileAttributes.Normal);
                            File.Delete(file);
                        }
                        catch (Exception e)
                        {
                            exceptionList.Add(e);
                        }
                    }
                }
                catch (Exception e)
                {
                    exceptionList.Add(e);
                }

                try
                {
                    var subdirs = Directory.GetDirectories(dir);
                    foreach (var subdir in subdirs)
                    {
                        subdirectories.Push(subdir);
                    }
                }
                catch (Exception e)
                {
                    exceptionList.Add(e);
                }
            }

            // 删除目录结构。
            try
            {
                for (var i = folderList.Count - 1; i >= 0; i--)
                {
                    try
                    {
                        Directory.Delete(folderList[i], true);
                    }
                    catch (Exception e)
                    {
                        exceptionList.Add(e);
                    }
                }

                if (Directory.Exists(directory))
                {
                    Directory.Delete(directory, true);
                }
            }
            catch (Exception e)
            {
                exceptionList.Add(e);
            }

            if (exceptionList.Any())
            {
                Console.WriteLine("Error when DeleteFolder");
                Console.WriteLine(new AggregateException(exceptionList));
            }
        }

```

注意我这里没有输出错误，也许你需要修改`Console.WriteLine(new AggregateException(exceptionList));`代码

删除文件之前设置 `File.SetAttributes` 不一定符合你的需求，用这个方法删除速度比较慢，建议不要在主线程使用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
