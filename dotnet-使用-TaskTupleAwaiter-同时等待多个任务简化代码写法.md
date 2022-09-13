
# dotnet 使用 TaskTupleAwaiter 同时等待多个任务简化代码写法

在某些业务逻辑下，需要同时等待多个任务执行完成，才能继续往下执行后续逻辑。等待任务执行的逻辑，大部分情况下需要使用到 Task.WhenAll 方法，代码行数不少。另外，在需要获取多个异步任务的返回值的逻辑上，整体的逻辑代码量看起来也不少。本文将和大家介绍 TaskTupleAwaiter 库，通过 TaskTupleAwaiter 库可以方便等待多个任务执行完成，且方便获取各个异步任务的返回值

<!--more-->


<!-- CreateTime:2022/9/9 19:25:14 -->


<!-- 发布 -->
<!-- 博客 -->

假定有两个异步任务方法，如以下代码，期望等待这两个方法执行完成，获取到结果，再执行后续逻辑

```csharp
Task<string> GetFoo1Async() => Task.Run(() => "Foo1");

Task<string> GetFoo2Async() => Task.Run(() => "Foo2");
```

传统方法是通过 Task.WhenAll 等待任务完成，再获取对应的值，如以下代码

```csharp
var task1 = GetFoo1Async();
var task2 = GetFoo2Async();

await Task.WhenAll(task1, task2);

var (foo1, foo2) = (task1.Result, task2.Result);
```

但千万不要先等待第一个任务执行完成，再等待第二个任务执行完成哦，如果是如以下代码的写法，自然会没有充分利用资源，第二个任务还在等待中

```csharp
var foo1 = await GetFoo1Async();
var foo2 = await GetFoo2Async();
```

在异步任务超过 3 个之后，代码逻辑的长度自然就很长了。接下来看看本文介绍的 TaskTupleAwaiter 库的优化后的写法

使用 TaskTupleAwaiter 库之后的可以简化为如下代码

```csharp
var (foo1, foo2) = await (GetFoo1Async(), GetFoo2Async());
```

可以看到一行就实现上面大概用了 4 行才能完成的任务，随着异步任务的数量的增加，优化力度也会更加大，同时也能解决在返回值相同的时候，不小心写过等待的任务的坑

按照惯例，使用 TaskTupleAwaiter 库的第一步就是安装 NuGet 包，对于 SDK 格式的 csproj 项目文件，可以在 csproj 里面添加如下代码用来安装

```xml
  <ItemGroup>
      <PackageReference Include="TaskTupleAwaiter" Version="2.0.0" />
  </ItemGroup>
```

这个库的使用方法十分简单，只是创建一个扩展类，里面就对 ValueTuple 扩展了 GetAwaiter 方法，根据 [C＃ await 高级用法](https://blog.lindexi.com/post/C-await-%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.html ) 博客可以了解到，对于 `await` 等待来说，只需要等待的类型存在 GetAwaiter 方法且此 GetAwaiter 方法返回一个实现了等待相关方法的类型的对象即可

例如对于由三个 Task 任务组成的 ValueTuple 加上可等待的功能的扩展方法可以是如下代码

```csharp
	public static TupleTaskAwaiter<T1, T2, T3> GetAwaiter<T1, T2, T3>(this (Task<T1>, Task<T2>, Task<T3>) tasks) =>
		new(tasks);

	public readonly record struct TupleTaskAwaiter<T1, T2, T3> : ICriticalNotifyCompletion
	{
		private readonly (Task<T1>, Task<T2>, Task<T3>) _tasks;
		private readonly TaskAwaiter _whenAllAwaiter;

		public TupleTaskAwaiter((Task<T1>, Task<T2>, Task<T3>) tasks)
		{
			_tasks = tasks;
			_whenAllAwaiter = Task.WhenAll(tasks.Item1, tasks.Item2, tasks.Item3).GetAwaiter();
		}

		public bool IsCompleted =>
			_whenAllAwaiter.IsCompleted;

		public void OnCompleted(Action continuation) =>
			_whenAllAwaiter.OnCompleted(continuation);

		[SecurityCritical]
		public void UnsafeOnCompleted(Action continuation) =>
			_whenAllAwaiter.UnsafeOnCompleted(continuation);

		public (T1, T2, T3) GetResult()
		{
			_whenAllAwaiter.GetResult();
			return (_tasks.Item1.Result, _tasks.Item2.Result, _tasks.Item3.Result);
		}
	}
```

在 `GetAwaiter` 扩展方法，给 `(Task<T1>, Task<T2>, Task<T3>)` 扩展了可等待的方法，如此即可使用 `await` 进行等待

通过 TupleTaskAwaiter 实现具体的等待逻辑，核心实现依然是 Task.WhenAll 进行等待，只是对此进行封装

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/37c7473807581cde1215374856e5fd8f285c21a9/JahawciceyainalljoHeneeqearhi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/37c7473807581cde1215374856e5fd8f285c21a9/JahawciceyainalljoHeneeqearhi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 37c7473807581cde1215374856e5fd8f285c21a9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 37c7473807581cde1215374856e5fd8f285c21a9
```

获取代码之后，进入 JahawciceyainalljoHeneeqearhi 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。