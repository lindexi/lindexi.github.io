
# C# dotnet 使用 TaskCompletionSource 实现暂停功能

在 C# 里面，可以使用 dotnet 的 TaskCompletionSource 方法自己实现一个异步任务，这个任务可以方便设置任务是否完成等做到让等待的过程继续或等待。根据这个功能可以解决在复杂的功能里面通过异步等待的方法实现暂停。做法就是等待一个异步任务，如果这个功能处于暂停，那么只需要让这个异步任务不结束，此时整个功能就会在等待，直到设置了异步任务完成

<!--more-->


<!-- 发布 -->

用异步等待的方式实现暂停有什么优势？

最大的优势就是可以让代码写起来简单，调试也简单。因为异步的代码可以通过 `await` 关键字实现，此时就可以和同步代码一样向下写，而此时的暂停逻辑可以通过 await 的方式，出让执行给现成池回收，提升整体性能

大概的功能加上暂停功能的写法如下

```csharp
        static async Task Foo(IContext context)
        {
            var n = 0;
            var n1 = 1;
            var n2 = 1;
            while (n1 > 0)
            {
                F1();
                await context.WaitForContinue();
                n1 = F2(n, n1);
                await context.WaitForContinue();
                n = F3(n1, n2);
                await context.WaitForContinue();
                await Task.Delay(10);
            }
        }
```

这个写法特别适合在存在大量局部需要的变量和状态的时候需要支持暂停的功能，如上面的代码有三个过程，如 F1 和 F2 和 F3 这三个，在每个方法执行完成之后都需要进行一次判断当前过程是否需要暂停

这里的暂停是其他业务通过 context 设置的，假设咱不使用异步的写法来做这个功能，那么请问有啥好主意来做这个事情？我需要在 F1 和 F2 和 F3 三个方法，在每一个方法执行完成的时候判断是否需要暂停，如果需要暂停那么进入暂停。但是暂停之后又可以在当前暂停处恢复

其实除了本文的方法，通过状态机也是可以实现的，通过状态机进行调度的方法，每次调度完成一个方法就进入判断是否暂停，如果暂停就等待暂停完成由业务的调用继续的时候触发后续的方法调度。但是用状态机对于有大量局部状态，如上面代码的 n 和 n1 和 n2 变量的存储就不清真了，从原本内部状态需要修改为外部状态，特别是这些状态涉及到了大量的相关变量

先不管 F1 和 F2 和 F3 的功能，请试试用状态机或其他方法改写这部分逻辑，需要在每个功能调用完成之后判断业务是否设置暂停，同时需要支持暂停之后可以在业务设置继续之后继续执行

请先思考一下哈

是不是会发现代码玩起来不够简单，在考虑其他小伙伴接手以及后续调试时，会发现逻辑实际上有点绕。当然这里欢迎小伙伴评论更好的方法哈

回到本文，上面的 `await context.WaitForContinue()` 是如何做到暂停功能？实际上暂停功能就是，如果业务暂停那么调用 `await context.WaitForContinue()` 就会持续等待，不会执行后面的代码。在业务设置继续的时候，就从 await 继续执行

那么这个方法是如何实现的？在 dotnet 里面提供了 TaskCompletionSource 类，这个类可以在创建完成调用 await 关键字等待，直到调用 SetResult 等方法设置完成

简单的使用例子如下，有两个方法在两个不同的线程，此时在 A 线程调用 A 方法，而在 B 线程调用 B 方法。在A方法等待任务完成才继续往下走，而在 B 方法则设置任务完成

```csharp
            var taskCompletionSource = new TaskCompletionSource<bool>();
            Task.Run(async () => await A(taskCompletionSource));

            Task.Run(async () => await B(taskCompletionSource)).Wait();

        private static async Task A(TaskCompletionSource<bool> taskCompletionSource)
        {
            Console.WriteLine("A 开始");
            await taskCompletionSource.Task;
            Console.WriteLine("A 完成");
        }

        private static async Task B(TaskCompletionSource<bool> taskCompletionSource)
        {
            await Task.Delay(1000);
            Console.WriteLine("B 开始");
            taskCompletionSource.SetResult(true);
            Console.WriteLine("B 完成");
        }
```

通过控制台可以看到先输出 `A 开始` 然后等待一秒输出 `B 开始` 之后调用了 SetResult 方法，然后输出 `A 完成` 也就是 A 方法在 `await taskCompletionSource.Task` 等待直到 B 调用 `taskCompletionSource.SetResult(true)` 方法才继续往下

根据这个功能可以做到在 `await context.WaitForContinue()` 根据业务设置是暂停还是继续返回不同的 Task 进行等待

在 dotnet 里面可以通过返回 Task.CompletedTask 表示一个完成的任务，此时等待此任务将会立刻返回。而在业务设置了暂停，此时可以返回 taskCompletionSource 等待，在业务重新设置继续的时候，设置 SetResult 方法，此时就可以做到让功能继续

用这个方法在等待 TaskCompletionSource 可以出让执行让线程回到线程池，提升总体效率。同时在暂停之后继续可以保持原有的调用堆栈和局部变量。通过保存原有的调用堆栈可以方便进行调试，通过保存原因的局部变量，可以让代码写起来简单

下面是这个 Context 的具体实现方法

```csharp
    class Context : IContext
    {
        public async Task WaitForContinue()
        {
            await (CurrentTask?.Task ?? Task.CompletedTask);
        }

        private TaskCompletionSource<bool> CurrentTask { set; get; }

        public void SetState(State state)
        {
            switch (state)
            {
                case State.Continue:
                    CurrentTask?.TrySetResult(false);
                    CurrentTask = null;
                    break;
                case State.Pause:
                    CurrentTask ??= new TaskCompletionSource<bool>();
                    break;
                default:
                    throw new ArgumentOutOfRangeException();
            }
        }
    }
```

在 WaitForContinue 方法里面，将会通过是否存在 TaskCompletionSource 判断是否需要等待完成的任务

在 SetState 里面通过 TrySetResult 可以解决多次设置继续，通过在 State.Pause 使用判断 CurrentTask 是否是空解决多次连续调用暂停的方法，上面代码的 `??= ` 是新的语法特性，和下面代码等价

```csharp
if (CurrentTask is null)
{
	CurrentTask = new TaskCompletionSource<bool>();
}
```

如果用户设置过暂停但是还没有设置继续，那么 CurrentTask 是存在的，也就是不会重新被创建

在用户重来没有暂停设置继续的时候，因为 CurrentTask 是空，继续执行

在用户设置过暂停之后设置继续的时候，将调用 TrySetResult 方法让 CurrentTask 完成

这里有一个细节就是在 TrySetResult 方法设置任务完成的时候，将会在当前线程继续执行等待 TaskCompletionSource 的代码。如上面例子的 A 和 B 两个方法，在 B 方法调用 SetResult 的时候，将会在 B 方法所在线程执行 A 方法的输出 `A 完成` 然后再继续

如果此时还需要实现中断的功能，很简单，可以通过抛 Exception 的方法，因为业务中断本来就属于非预期的过程

在 TaskCompletionSource 可以通过 CurrentTask.SetException 的方法，让等待这个任务的代码抛出传入的值

本来我是使用 WPF 做的，但是这里不好控制比较复杂的测试，所以修改为控制台。如果使用 WPF 做效果会比较好，可以新建两个按钮，分别是继续和暂停，修改下面代码可以看到在点击暂停的时候，执行等待任务的代码就会暂停直到点击继续按钮

下面是通过输出 F1 和 F2 和 F3 方法的调用，可以根据前面的时间看到暂停的功能

```csharp
07:28:09 F1
07:28:12 F2
07:28:12 F3
07:28:12 F1
07:28:12 F2
07:28:12 F3
07:28:12 F1
07:28:12 F2
07:28:12 F3
07:28:12 F1
07:28:12 F2
07:28:12 F3
07:28:12 F1
07:28:12 F2
07:28:12 F3
07:28:12 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:13 F2
07:28:13 F3
07:28:13 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
07:28:17 F1
07:28:17 F2
07:28:17 F3
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a0c74d13a44883e1f1f982ca4d972ccf4dfb9cc4/CallnernawbawceKairwemwhejeene) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。