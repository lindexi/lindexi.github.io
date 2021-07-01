# dotnet C# 调用委托的 GetInvocationList 的对象分配

本文也叫跟着 Stephen Toub 大佬学性能优化系列，这是我从 Stephen Toub 大佬给 WPF 框架做性能优化学到的知识，在热路径下，也就是频繁调用的模块，如果调用了委托的 GetInvocationList 方法，那么将视委托的大小，每次创建不同大小的新数组对象，而在频繁调用的模块，将会创建大量的对象

<!--more-->
<!-- CreateTime:2021/6/29 8:41:11 -->

<!-- 发布 -->

如以下代码的一个委托，当然对于事件来说也是如此

```csharp
            Action action = Foo;
            for (int i = 0; i < 10; i++)
            {
                action += Foo;
            }

            static void Foo()
            {

            }
```

如果调用了 action 的 GetInvocationList 方法，那么在每次调用都会申请一些内存，如使用以下代码进行测试

```csharp
            for (int i = 0; i < 100; i++)
            {
                var beforeAllocatedBytesForCurrentThread = GC.GetAllocatedBytesForCurrentThread();
                var invocationList = action.GetInvocationList();
                var afterAllocatedBytesForCurrentThread = GC.GetAllocatedBytesForCurrentThread();
                Console.WriteLine(afterAllocatedBytesForCurrentThread - beforeAllocatedBytesForCurrentThread);
            }
```

上面代码的 GetAllocatedBytesForCurrentThread 是一个放在 GC 层面的方法，可以用来获取当前线程分配过的内存大小，这是一个用来辅助调试的方法。详细请看 [dotnet 使用 GC.GetAllocatedBytesForCurrentThread 获取当前线程分配过的内存大小](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-GC.GetAllocatedBytesForCurrentThread-%E8%8E%B7%E5%8F%96%E5%BD%93%E5%89%8D%E7%BA%BF%E7%A8%8B%E5%88%86%E9%85%8D%E8%BF%87%E7%9A%84%E5%86%85%E5%AD%98%E5%A4%A7%E5%B0%8F.html)

可以看到运行时的控制台输出如下

```
312
112
112
112
112
112
112
112
112
112
112
112
// 不水了
```

这是因为在底层的实现，调用 GetInvocationList 方法的代码如下

```csharp
    public override sealed Delegate[] GetInvocationList()
    {
      Delegate[] delegateArray;
      if (!(this._invocationList is object[] invocationList))
      {
        delegateArray = new Delegate[1]{ (Delegate) this };
      }
      else
      {
        delegateArray = new Delegate[(int) this._invocationCount];
        for (int index = 0; index < delegateArray.Length; ++index)
          delegateArray[index] = (Delegate) invocationList[index];
      }
      return delegateArray;
    }
```

可以看到每次都需要重新申请数组，然后给定数组里面的元素。如果在调用频繁的模块里面，不断调用 GetInvocationList 方法，将会有一定的性能损耗。如在 WPF 的移动鼠标等逻辑里面

一个优化的方法是，如果指定的委托或事件的加等次数比调用 GetInvocationList 的次数少，如 WPF 的 PreNotifyInput 等事件，此时可以通过在加等的时候缓存起来，这样后续的调用就不需要重新分配内存

以上优化的细节请看 [Avoid calling GetInvocationList on hot paths by stephentoub · Pull Request #4736 · dotnet/wpf](https://github.com/dotnet/wpf/pull/4736)

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6ed312e74e286d581e3d609ed555447474259ae4/FairhojafallJeeleefuyi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6ed312e74e286d581e3d609ed555447474259ae4/FairhojafallJeeleefuyi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6ed312e74e286d581e3d609ed555447474259ae4
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 FairhojafallJeeleefuyi 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
