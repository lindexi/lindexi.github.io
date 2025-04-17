
# 在 windows 上运行的 podman 默认的挂载相对路径是什么

我在 windows 运行 podman 当成 docker 的代替品，从网上抄了 ollama 的部署命令，发现里面存在一个相对路径的挂载文件夹。我期望拿到 ollama 的下载内容，需要寻找到 podman 默认的挂载路径，但在网上找了一圈，可能是我的关键词问题，没有找到，于是记录本文期望能帮到大家

<!--more-->


<!-- CreateTime:2024/04/25 07:13:43 -->

<!-- 发布 -->
<!-- 博客 -->

如下面命令

```
podman run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

使用了 `-v ollama:/root/.ollama` 参数将本机的 ollama 文件夹挂载到容器里面的 `/root/.ollama` 文件夹

那默认情况下的本机 ollama 文件夹是在哪？在 podman 里面挂载相对路径是什么

在 podman 里面挂载相对路径是在 WSL 里面的 `~/.local/share/containers/storage/volumes/` 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。