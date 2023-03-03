# 修复 GitLab 的 CI Runner 提示找不到 pwsh 执行文件

本文告诉大家如何修复使用 GitLab 的 Runner 做 CI 时提示 "pwsh": executable file not found in %PATH% 错误

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

有两个方法，第一个方法就是安装 pwsh 命令，安装方法是在 PowerShell 里输入以下代码安装

```
winget install Microsoft.PowerShell
```

如果嫌弃 winget 输入太慢，可以从他的输出里面找到 PowerShell 的下载地址，换个快速的下载器去下载即可

输入以上命令之后，相信你看界面就会了

第二个方法就是将 `pwsh` 修改为 `powershell` 代码，编辑 gitlab-runner.exe 所在文件夹下的 config.toml 文件，将里面的  `pwsh` 修改为 `powershell` 如以下代码

```csharp
[[runners]]
  name = "xxxxx"
  url = "https://xxxxxx/"
  id = 1363
  token = "h_h-xxx-"
  token_obtained_at = 2023-03-02T11:00:05Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "shell"
  shell = "powershell" // 只需要关键这句话就可以了。原本是 `shell = "pwsh"` 这句话
```

两个方法选一个即可，我比较推荐使用第一个方法