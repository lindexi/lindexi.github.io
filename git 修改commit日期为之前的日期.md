# git 修改commit日期为之前的日期

我在之前修改了一个文件，但是没有commit，现在我想要commit，日期为那天的日期

git 修改日期的方法很简单，因为有一个命令`--date` 可以设置 git 提交时间。

默认的 git 的提交时间会受到系统的时间的影响，如果想要系统的时间不会影响到 git 的提交时间，请使用本文的方式，自己指定提交的时间

<!--more-->
<!-- CreateTime:2019/11/19 8:53:16 -->


使用git自定义时间的提交格式：

```csharp
git commit --date="月 日 时间 年 +0800" -am "提交"
```

如果我要把日期修改为 2016.5.7 那么我可以使用下面代码

```csharp
git commit --date="May 7 9:05:20 2016 +0800" -am "提交"
```

其中我希望大家知道的：

各个月份的缩写，不然每次都需要去百度一下

```csharp
January, Jan.
February, Feb.
March, Mar.
April, Apr.
May, May.
June, Jun.
July, Jul.
August, Aug.
September, Sep.
October, Oct.
November, Nov.
December, Dec.
```

当然，如果你想写为程序，那么我还可以送你一点代码

```csharp
            new List<string>()
            {
                "Jan","Feb","Mar","Apr","May","Jun","Jul","Aug",
                "Sep","Oct","Nov","Dec"
            };
```

如果需要使用 C# 调用 git ，可以使用我之前写过的代码：

```csharp
    public class GitControl
    {
        public GitControl(string fileDirectory)
        {
            FileDirectory = fileDirectory;
        }

        /// <summary>
        ///     git的文件夹
        /// </summary>
        public string FileDirectory
        {
            set;
            get;
        }

        public string Branch
        {
            set;
            get;
        }

        public string Origin
        {
            set;
            get;
        }

        public string Add(string file = ".")
        {
            string str = "add " + file;
            return Control(str);
        }

        private string ConvertDate(DateTime time)
        {
            //1.  一月     January      （Jan）2.  二月      February   （Feb）
            //3.  三月      March        （Mar） 
            //4.  四月      April           （Apr）
            //5.  五月      May           （May）
            //6.  六月      June           （Jun）
            //7.  七月      July             （Jul）
            //8.  八月      August        （Aug）
            //9.  九月      September  （Sep）
            //10.  十月     October      （Oct） 
            //11.  十一月   November （Nov）12.  十二月   December （Dec）
            List<string> temp = new List<string>()
            {
                "Jan","Feb","Mar","Apr","May","Jun","Jul","Aug",
                "Sep","Oct","Nov","Dec"
            };

            //StringBuilder str = new StringBuilder();
            //            git commit --date = "月 日 时间 年 +0800" - am "提交"

            //git commit --date = "May 7 9:05:20 2016 +0800" - am "提交"
            return $"--date=\"{temp[time.Month - 1]} {time.Day} {time.Hour}:{time.Minute}:{time.Second} {time.Year} +0800\" ";
        }

        public string Commit(string str = null, DateTime time = default(DateTime))
        {
            string commit = " commit";
            if (time != (default(DateTime)))
            {
                commit += " " + ConvertDate(time);
            }

            if (string.IsNullOrEmpty(str))
            {
                if (time == default(DateTime))
                {
                    time = DateTime.Now;
                }
                str = time.Year + "年" + time.Month + "月" +
                      time.Day + "日 " +
                      time.Hour + ":" +
                      time.Minute + ":" + time.Second;
            }
            commit += " -m " + "\"" + str + "\"";
            //commit = FileStr() + commit;
            return Control(commit);
        }

        public string Push()
        {
            //git push origin master
            if (string.IsNullOrEmpty(Branch))
            {
                Branch = "master";
            }

            if (string.IsNullOrEmpty(Origin))
            {
                Origin = "origin";
                string str = $"push {Origin} {Branch}";
                return Control(str);
            }
            else
            {
                string str = $"push {Origin} {Branch}";
                str= Control(str);
                if (Origin != "origin")
                {
                    Origin = "origin";
                    str += Control($"push {Origin} {Branch}");
                }
                return str;
            }
        }

        private string _gitStr = "git -C {0}";

        private string FileStr()
        {
            return string.Format(_gitStr, FileDirectory) + " ";
        }

        private string Control(string str)
        {
            str = FileStr() + str;
            // string str = Console.ReadLine();
            //System.Console.InputEncoding = System.Text.Encoding.UTF8;//乱码

            Process p = new Process();
            p.StartInfo.FileName = "cmd.exe";
            p.StartInfo.UseShellExecute = false; //是否使用操作系统shell启动
            p.StartInfo.RedirectStandardInput = true; //接受来自调用程序的输入信息
            p.StartInfo.RedirectStandardOutput = true; //由调用程序获取输出信息
            p.StartInfo.RedirectStandardError = true; //重定向标准错误输出
            p.StartInfo.CreateNoWindow = true; //不显示程序窗口
            p.StartInfo.StandardOutputEncoding= Encoding.UTF8;//Encoding.GetEncoding("GBK");//乱码
            p.Start(); //启动程序

            //向cmd窗口发送输入信息
            p.StandardInput.WriteLine(str + "&exit");

            p.StandardInput.AutoFlush = true;
            //向标准输入写入要执行的命令。这里使用&是批处理命令的符号，表示前面一个命令不管是否执行成功都执行后面(exit)命令，如果不执行exit命令，后面调用ReadToEnd()方法会假死
            //同类的符号还有&&和||前者表示必须前一个命令执行成功才会执行后面的命令，后者表示必须前一个命令执行失败才会执行后面的命令

            
            //获取cmd窗口的输出信息
            string output = p.StandardOutput.ReadToEnd();
            output += p.StandardError.ReadToEnd();

            p.WaitForExit(); //等待程序执行完退出进程
            p.Close();

            return output + "\r\n";
        }
    }

```

如果修改过程需要修改上一次提交的日期，可以添加 `--amend` 的参数，如果要修改不是上一次的提交，而是很久的提交，我暂时没找到如何做，如果你知道怎么做，请告诉我

本文用的时间是 RFC 2822 格式，这个格式的时间是 `月 日 时间 年 +0800` 而除了这个格式，还可以使用 ISO 8601 格式，如下面代码

```csharp
2005-04-07T22:13:13
```

在 C# 可以使用 `DateTime.UtcNow.ToString("s")` 将时间输出

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  