# 自动更新所有 Git 仓库

我在本地添加了很多开源项目，我写了一个脚本可以每天自动从开源项目更新代码

<!--more-->
<!-- CreateTime:2019/8/4 14:44:59 -->

<!-- csdn -->

我在写开源项目的时候，在需要写之前更新项目，但是每次更新都需要等很久，能否可以自动更新开源项目的代码，这样每次需要更新的就很少，提高开发的速度。我找到了 Git 的命令可以更新项目，本文主要就是通过写一个程序自动使用 Git 命令更新

可以通过找到所有 Git 文件夹，执行 `git fetch --all` 命令更新项目

所以步骤就是写 cmd 命令行调用，然后写 Git 命令，接着是找到磁盘的所有 Git 文件夹，然后调用 Git 命令更新

写一个类执行 cmd 命令

```csharp
    public static class Control
    {
        static Control()
        {
            Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
        }

        public static string Command(string str)
        {
            // string str = Console.ReadLine();
            //System.Console.InputEncoding = System.Text.Encoding.UTF8;//乱码

            Process p = new Process
            {
                StartInfo =
                {
                    FileName = "cmd.exe",
                    UseShellExecute = false,  //是否使用操作系统shell启动
                    RedirectStandardInput = true,  //接受来自调用程序的输入信息
                    RedirectStandardOutput = true,  //由调用程序获取输出信息
                    RedirectStandardError = true, //重定向标准错误输出
                    CreateNoWindow = true, //不显示程序窗口
                    StandardOutputEncoding = Encoding.GetEncoding("GBK") //Encoding.UTF8
                    //Encoding.GetEncoding("GBK");//乱码
                }
            };
           
            p.Start(); //启动程序

            //向cmd窗口发送输入信息
            p.StandardInput.WriteLine(str + "&exit");

            p.StandardInput.AutoFlush = true;
            //p.StandardInput.WriteLine("exit");
            //向标准输入写入要执行的命令。这里使用&是批处理命令的符号，表示前面一个命令不管是否执行成功都执行后面(exit)命令，如果不执行exit命令，后面调用ReadToEnd()方法会假死
            //同类的符号还有&&和||前者表示必须前一个命令执行成功才会执行后面的命令，后者表示必须前一个命令执行失败才会执行后面的命令

            bool exited = false;

            // 超时
            Task.Run(() =>
            {
                Task.Delay(TimeSpan.FromMinutes(1)).ContinueWith(_ =>
                {
                    if (exited)
                    {
                        return;
                    }

                    try
                    {
                        if (!p.HasExited)
                        {
                            Console.WriteLine($"{str} 超时");
                            p.Kill();
                        }
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine(e);
                    }
                });
            });

            //获取cmd窗口的输出信息
            string output = p.StandardOutput.ReadToEnd();
            //Console.WriteLine(output);
            output += p.StandardError.ReadToEnd();
            //Console.WriteLine(output);

            //StreamReader reader = p.StandardOutput;
            //string line=reader.ReadLine();
            //while (!reader.EndOfStream)
            //{
            //    str += line + "  ";
            //    line = reader.ReadLine();
            //}

            p.WaitForExit(); //等待程序执行完退出进程
            p.Close();

            exited = true;

            return output + "\r\n";
        }
    }

```

然后需要有一个类可以用于通过命令行调用 Git 执行代码

```csharp
    public class GitCommand
    {
        /// <inheritdoc />
        public GitCommand(DirectoryInfo repo)
        {
            Repo = repo;
        }

        public DirectoryInfo Repo { get; }

        public void FetchAll()
        {
            Control("fetch --all");
        }

        private string Control(string str)
        {
            str = FileStr() + str;
            Log(str);
            str = RegeejairhemFurwhurrahuki.Control.Command(str);

            Log(str);
            return str;
        }

        private static void Log(string str)
        {
            Console.WriteLine(str);
        }

        private string FileStr()
        {
            return string.Format(GitStr, Repo.FullName);
        }

        private const string GitStr = "git -C {0} ";
    }

```

不知道有没小伙伴找到一个好用的执行 Git 命令的库

然后找到磁盘所有 Git 文件夹修改项目

```csharp
        static void Main(string[] args)
        {
            List<DirectoryInfo> deviceList = FindDevice();

            foreach (var device in deviceList)
            {
                Log($"开始读取{device}盘符内容");
                try
                {
                    FindDirectory(device, 10);
                }
                catch (Exception e)
                {
                    Log(e.ToString());
                }
            }

        }

        /// <summary>
        /// 寻找文件夹
        /// </summary>
        /// <param name="directory"></param>
        /// <param name="deep">寻找深度</param>
        private static void FindDirectory(DirectoryInfo directory, int deep)
        {
            try
            {
                Log($"开始读取 {directory.FullName} 文件夹，当前递归深度{deep}");
                deep--;
                var gitFolder = new DirectoryInfo(Path.Combine(directory.FullName, GitFolder));
                if (CheckIsGitFolder(gitFolder))
                {
                    Log($"找到{gitFolder.FullName}文件夹");
                    try
                    {
                        UpdateGitFile(directory);
                    }
                    catch (Exception e)
                    {
                        Log(e.ToString());
                    }
                }
                else
                {
                    //Log($"因为{directory.FullName}不是");
                    if (deep == 0)
                    {
                        Log($"达到递归深度{directory.FullName}不再继续寻找子文件夹");

                        return;
                    }

                    var subDirectoryList = directory.GetDirectories();

                    Log($"在{directory.FullName}找到{subDirectoryList.Length}个文件夹");

                    foreach (var temp in subDirectoryList)
                    {
                        FindDirectory(temp, deep);
                    }
                }
            }
            catch (Exception e)
            {
                Log(e.ToString());
            }
        }

        private static void UpdateGitFile(DirectoryInfo directory)
        {
            var gitCommand = new GitCommand(directory);
            gitCommand.FetchAll();
        }

        private const string GitFolder = ".git";

        /// <summary>
        /// 判断当前是否一个 git 文件夹
        /// </summary>
        /// <param name="directory"></param>
        private static bool CheckIsGitFolder(DirectoryInfo directory)
        {
            return directory.Exists;
        }


        private static void Log(string str)
        {
            Console.WriteLine(str);

            //File.AppendAllText("log.txt", str + "\r\n");
        }


        /// <summary>
        /// 找到所有驱动器
        /// </summary>
        /// <returns></returns>
        private static List<DirectoryInfo> FindDevice()
        {
            var deviceList = new List<DirectoryInfo>();
            for (int i = 0; i < 'Z' - 'A' + 1; i++)
            {
                var device = (char)('A' + i) + ":\\";
                if (Directory.Exists(device))
                {
                    deviceList.Add(new DirectoryInfo(device));
                }
            }

            return deviceList;
        }
```

代码请看 [github](https://github.com/lindexi/lindexi_gd/blob/353776a0d7b44c017b1a037a94fbd29718f2303f/RegeejairhemFurwhurrahuki/RegeejairhemFurwhurrahuki/Program.cs )

![](http://image.acmx.xyz/lindexi%2F20198414445357)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
