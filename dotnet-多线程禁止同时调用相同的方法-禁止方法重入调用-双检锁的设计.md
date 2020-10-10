
# dotnet 多线程禁止同时调用相同的方法 禁止方法重入调用 双检锁的设计

大家在使用多线程的时候，是否有关注过线程安全的问题。如果咱的代码在使用多线程时，在相同的时间有多个线程同时执行相同的方法，此时也许就存在数据安全的问题，如多个线程之间对相同的内存进行同时的读取和修改。而让方法在多线程调用中，相同的时间会被多个线程同时执行某段代码逻辑的技术称为方法重入调用技术，而禁止方法被同时调用也就是禁止方法重入调用。在 dotnet 里面有多个方式可以做到禁止方法重入调用，本文将告诉大家如何做到禁止方法重入调用

<!--more-->


<!-- CreateTime:2020/10/9 18:01:23 -->

<!-- csdn -->

执行代码逻辑的重入是一个很泛的领域，本文仅仅只和大家聊多线程同时执行某段代码逻辑时的重入

在开始之前，我需要告诉大家，本文不聊递归的方法。递归就是方法自身调用方法自身，或者说方法间接调用了自身，如下面代码

```csharp
public void Foo()
{
    Foo();
}
```

以及间接调用如下面代码

```csharp
        private void A()
        {
            B();
        }

        private void B()
        {
            A();
        }
```

以上代码的递归重入不在本文讨论范围内。因为在一个线程执行过程里面，所有的逻辑都是顺序执行的，除非是递归的重入，否则不会在相同的时间调用方法两次

而对多线程的应用，多个线程同一时刻是可以访问相同的方法执行相同的代码逻辑，如果想要让多线程每次只能有一个线程执行，那么需要使用到锁等方法。可以使用的方法有很多，下面让我告诉大家如何做到禁止方法重入调用

## 锁定方法

在 C# 里面可以使用关键词 lock 加上一个对象作为锁定，在进入 lock 的逻辑，只能有一个线程获取锁，因此在 lock 里面的代码只被一个线程同时执行

如以下代码就是标准的锁定方法的代码

```csharp
        private void FooLock()
        {
            lock (_locker)
            {
                // 代码
            }
        }

        private readonly object _locker = new object();
```

需要注意的细节是创建一个空白的对象 `_locker` 作为字段，使用字段而不是局部变量的原因在于 lock 只有在使用相同的对象才能做到多个线程进入时，只有一个线程执行，其他线程等待。如果是局部变量，那么多个线程都会创建自己的局部变量，因此就做不到让一个线程执行，其他线程等待

其次是这个 `_locker` 应该是私有的，采用私有的可以让整个锁的功能在自己内部的完全控制的代码下使用，而不会担心被其他业务使用。基于这个原因可以了解到使用 `lock(this)` 是不推荐的，因为 this 将会被其他类所使用，此时就无法完全了解这个锁使用的对象使用的地方。尽管自己在开发的时候可以关注到，但是在后续更改中不一定能了解这些细节，因此也许就会因此出现相互等待的锁的坑

最后是这个对象应该是 `readonly` 不可变的，原因在于也许在线程进入锁的时候，如果是可变的字段，将也许有其他业务在其他线程下更改了这个对象，也就让其他线程依然可以执行相同的逻辑

而多创建一个对象也用不了多少内存，关于对象使用的内存请看 [C# CLR 聊聊对象的内存布局 一个空对象占用多少内存](https://blog.lindexi.com/post/C-CLR-%E8%81%8A%E8%81%8A%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%86%85%E5%AD%98%E5%B8%83%E5%B1%80-%E4%B8%80%E4%B8%AA%E7%A9%BA%E5%AF%B9%E8%B1%A1%E5%8D%A0%E7%94%A8%E5%A4%9A%E5%B0%91%E5%86%85%E5%AD%98.html )

## 通过特性

在 dotnet 里面可以使用 `MethodImpl` 特性表示当前这个方法只能让一个线程进入，其他线程将需要等待

```csharp
        [MethodImpl(MethodImplOptions.Synchronized)]
        private void F1()
        {

        }
```

使用 MethodImplOptions.Synchronized 的本质就和上文的定义 `_locker` 对象的方法类似，只是具体实现机制由 CLR 决定

当前的 CLR 将会在实例方法，也就是非静态的方法，使用 `this` 作为锁定对象。在静态方法使用对象的 Type 作为锁定的对象

如果这个类型不是私有的类型，那么尽量不要使用 MethodImpl 这个方法禁止冲入。原因是在实例方法使用 `this` 作为锁定对象，而其他代码也许也会将这个实例作为锁定的对象，此时也许如下面代码所示有两个线程在相互等待

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var program = new Program();
            var autoResetEvent = new AutoResetEvent(false);
            var manualResetEvent = new ManualResetEvent(false);

            var task1 = Task.Run(() =>
            {
                lock (program)
                {
                    // 用于让 task1 执行到这里才让 task2 执行
                    autoResetEvent.Set();

                    // 用于等待 task2 执行完成
                    manualResetEvent.WaitOne();
                }
            });

            var task2 = Task.Run(() =>
            {
                // 用于等待 task1 执行
                autoResetEvent.WaitOne();

                // 调用禁止冲入的方法
                program.F1();

                // 如果上面代码调用返回，那么让 tas1 继续执行
                manualResetEvent.Set();
            });

            Task.WaitAll(task1, task2);
        }

        [MethodImpl(MethodImplOptions.Synchronized)]
        private void F1()
        {
            Console.WriteLine("执行逻辑");
        }
    }
```

上面代码的 AutoResetEvent 和 ManualResetEvent 仅是为了让两个线程按照如下顺序执行和相互等待，线程1将会拿到 Program 实例，用这个实例作为锁定的对象。然后线程1需要等待线程2执行完成之后才会退出锁定。而线程2在线程1执行进入锁定之后才会开始执行，开始执行的时候调用了 F1 方法，调用之后执行完成

而在上面代码里面，调用 F1 执行的过程，在当前 CLR 的实现，将会尝试拿到自身作为锁定对象。而 F1 的自身也就是 Program 的实例，此时被线程1作为锁定对象，因此线程2需要等待线程1不再将 Program 的实例作为锁定的对象之后才会执行 F1 方法。而此时的线程1在等待线程2执行完成才会退出锁定，而线程2在等待线程1退出锁定才会执行完成。因此两个线程在相互等待

这样的逻辑代码是在 F1 方法定义的时候无法了解的，这就是为什么不建议使用 `MethodImpl` 的原因。即使在开发的时候采用的是私有的类，但是后续更改的时候也许就将他开放了，而后续有逗比开发者参与开发，将某个对象作为锁定的对象

## 双检锁

太子说以下的误导性特别高，请小伙伴在大人们的指导下观看

双检锁又称双险锁（也许是没有 双险锁 这个名字的），本质上是让方法在多线程下只执行一次，和上文的用途有点不相同。上文的方法是只有一个线程执行，其他线程等待。而双检锁是让一个线程执行，其他线程不执行的代码设计方法

双检锁有多个不同的写法，采用双检锁仅仅只是为了提升性能，而如果不为了提升性能，可以采用如下更直观的实现方法，尽管准确来说以下不是双检锁的写法

```csharp
        private void F2()
        {
            lock (_locker)
            {
                if (_isDoing)
                {
                    return;
                }

                _isDoing = true;

                // 执行代码
            }
        }

        private bool _isDoing;
        private readonly object _locker = new object();
```

这个方法就是用来判断是否执行的逻辑，如果执行过了，那么将不再执行。上面方法在使用 `lock (_locker)` 可以让方法里面的代码只有一个线程同时执行，此时对 `_isDoing` 的读取和修改将会是线程安全的，因此可以通过此判断而解决重入问题

但上面方法因为默认需要进入 `lock (_locker)` 一次锁定，而 lock 尽管性能已经足够好了，但是依然在性能敏感的逻辑上，会影响整体的性能。为什么 lock 的性能已经足够好了，因为默认的 lock 是一个混合锁，也就是一个会使用用户态和内核态的锁。在进入 lock 时，此时将会使用自旋锁，在等待一段时候之后才会进行线程锁等。在开始进入自旋锁，此时的逻辑大概就是 `while (true)` 的循环判断逻辑。进入自旋锁可以做到没有线程上下文切换，也就是当前线程依然在执行中。如果这段代码很快就能进入执行，此时的速度是非常快的。相当于在循环里面做判断布尔

当然，如果在 lock 一直没有进入执行，那么将会从自旋锁退出进入线程锁，而线程锁将会涉及到线程上下文的切换，此时的速度将会比较慢

当然了我很难用几句话描述清楚 lock 的底层原理，以上描述，就当看着玩

为了更好的提升性能，也就是一段代码其实大部分时候进入的时候都是被执行过的，不需要再次被执行，此时可以采用双检锁的写法。先判断布尔值，然后再进入锁定，再进行判断，请看代码

```csharp
        private void F2()
        {
            if (_isDoing)
            {
                return;
            }

            lock (_locker)
            {
                if (_isDoing)
                {
                    return;
                }

                _isDoing = true;

                // 执行代码
            }
        }

        private bool _isDoing;
        private readonly object _locker = new object();
```

可以对比上面代码，使用双检锁的标准写法里面，就是先判断布尔字段的值，然后再进入锁。在大部分进入的时候方法都执行完成时，此时的判断布尔值就能让方法返回，而不需要进入锁，可以提升不少的性能

而在刚好第一次执行的时候，多个线程如果都进入判断布尔值时，此时判断不是线程安全的。但是没关系，因为后续会进入 `lock (_locker)` 然后再次判断，这就是 双检锁 这个名字的原因了

而如大家所见，上面代码的复杂度确实比较高，也需要占用两个本地字段。更加优雅但是比较难理解的禁止方法重入多次调用的写法可以使用 Interlocked 类的方法，在 Interlocked 类的 Exchange 方法提供了对 int 等基础类型的原子修改，可以在将某个值进行原子修改之后返回原先的值。而原子修改是线程安全的，也就是多个线程如果同时进入原子修改，此时不会存在线程安全问题

使用 Interlocked 的写法如下

```csharp
        private void F2()
        {
            var doingCount = Interlocked.Exchange(ref _doingCount, 1);

            if (doingCount == 0)
            {
                // 执行代码
                Console.WriteLine("执行逻辑");
            }
        }

        private int _doingCount;
```

可以看到，上面代码每次都进入 `Interlocked.Exchange` 的逻辑，而只有一次能返回 0 的值，因此也就只能执行一次。这个方法的性能将会更好，但是写法上会比较难以理解，需要了解 Interlocked 以及原子修改的原理才比较好理解上面的写法。但实际上用了 Interlocked 就不算双检锁了，只是思想上和双检锁差不多。使用 Interlocked 的方法可以获取极高的性能

如果你想要将如上代码用于对象的初始化，那么上面两个写法其实有本质的不同，不同之处在于用 双检锁 的写法可以让线程阻塞，在首次对象初始化过程中，其他线程能使用到执行线程的执行结果。而使用 Interlocked 是只让一个线程执行，其他线程跳过，而不能用到对象初始化的结果。因此在 Interlocked 的用法上面，不适合用来让对象初始化一次的业务

## 更复杂的需求

如果我要求限制执行某个方法的线程数量，要求只能让两个线程去执行某个方法或任务，那么此时我将和你推荐我的开源库 [dotnet-campus/AsyncWorkerCollection: 高性能的多线程异步工具库](https://github.com/dotnet-campus/AsyncWorkerCollection )

这是一个在 GitHub 完全开源的库，基于非常友好的 MIT 开源协议，请看 [https://github.com/dotnet-campus/AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection)

如上文的需求，限制执行某个方法的数量，其实就是生产者消费者模式，可以使用 [AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection) 库的 AsyncQueue 类实现这个功能，详细请看 [dotnet 使用 AsyncQueue 创建高性能内存生产者消费者队列](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-AsyncQueue-%E5%88%9B%E5%BB%BA%E9%AB%98%E6%80%A7%E8%83%BD%E5%86%85%E5%AD%98%E7%94%9F%E4%BA%A7%E8%80%85%E6%B6%88%E8%B4%B9%E8%80%85%E9%98%9F%E5%88%97.html)

如果我要求执行方法的时候，如果有多个线程调用，那么在方法执行过程中，多次进来的线程都不做实际的执行，而是等待当前在执行方法的线程执行完成之后，取出执行的返回值作为其他线程的执行方法的返回值。此时可以使用 KeepLastReentrancyTask 类

如果需要支持本机内多线程调用某一确定的任务的执行，任务仅执行一次，多次调用均返回相同结果。此时可以使用 [ExecuteOnceAwaiter](https://blog.lindexi.com/post/C-dotnet-%E9%AB%98%E6%80%A7%E8%83%BD%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%B7%A5%E5%85%B7-ExecuteOnceAwaiter-%E5%8F%AA%E6%89%A7%E8%A1%8C%E4%B8%80%E6%AC%A1%E7%9A%84%E4%BB%BB%E5%8A%A1.html ) 类

欢迎小伙伴关注 [https://github.com/dotnet-campus/AsyncWorkerCollection](https://github.com/dotnet-campus/AsyncWorkerCollection) 开源库，也欢迎小伙伴贡献代码

## 更多博客

在使用多线程的时候，将会遇到很多锁的问题，在 dotnet 里面提供了大量不同功能的锁。尽管 lock 基本上能搞定一切，但是有些复杂的业务或比较底层库还是需要了解更多的细节，如下是我写的一些锁的博客

- [C# dotnet 使用 AsyncEx 库的 AsyncLock 异步锁](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-AsyncEx-%E5%BA%93%E7%9A%84-AsyncLock-%E5%BC%82%E6%AD%A5%E9%94%81.html)
- [C# dotnet 的锁 SemaphoreSlim 和队列](https://blog.lindexi.com/post/C-dotnet-%E7%9A%84%E9%94%81-SemaphoreSlim-%E5%92%8C%E9%98%9F%E5%88%97.html)
- [dotnet 使用 SemaphoreSlim 可能的内存泄露](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-SemaphoreSlim-%E5%8F%AF%E8%83%BD%E7%9A%84%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2.html)
- [dotnet 里的那些锁 AutoResetEvent 用法](https://blog.lindexi.com/post/dotnet-%E9%87%8C%E7%9A%84%E9%82%A3%E4%BA%9B%E9%94%81-AutoResetEvent-%E7%94%A8%E6%B3%95.html)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。