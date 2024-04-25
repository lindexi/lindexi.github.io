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