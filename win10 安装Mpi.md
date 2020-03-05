# win10 安装Mpi

<!--more-->
<!-- CreateTime:2020/3/5 9:26:17 -->


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




