
# win10 安装Mpi


<!--more-->



<div id="toc"></div>

首先我们需要安装Bash

		

```csharp
$ sudo apt-get install libcr-dev mpich2 mpich2-doc


```

写hello.c
		

```c
# include <mpi.h>
# include <stdio.h>

int main (int argc, char* argv[])
{
  int rank, size;

  MPI_Init (&argc, &argv);      /* starts MPI */
  MPI_Comm_rank (MPI_COMM_WORLD, &rank);        /* get current process id */
  MPI_Comm_size (MPI_COMM_WORLD, &size);        /* get number of processes */
  printf( "Hello world from process %d of %d\n", rank, size );
  MPI_Finalize();
  return 0;
}

```

使用mpicc ，`[[INVALID],INVALID] ORTE_ERROR_LOG: A system-required executable either could not be found or was not executable by this user in file ess_singleton_module.c at line 231`

		

```csharp
$ mpicc.mpich2 hello.c -o h

$ mpiexec.mpich -np <num> ./h
```

I use `mpiexec.mpich` run the code.`mpicc.mpich2 hello.c -o h` `mpiexec.mpich -np <num> ./h`

可以安装mpi，从http://download.csdn.net/detail/lindexi_gd/9714817 下载，使用
    

```csharp
tar zvf  openmpi-1.6.5.tar.gz  
cd openmpi-1.6.5
./config
sudo make & make install

```








<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。