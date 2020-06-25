# C# dotnet 使用 TaskCompletionSource 让事件转异步方法

咱今天来聊聊一个毁灭世界的故事，哦，不，是一个使用 TaskCompletionSource 让偷核武器，哦，又说错了，是让事件转换为异步的方法，让咱可以在一个方法里面顺序写下毁灭世界的逻辑

<!--more-->
<!-- CreateTime:6/23/2020 3:25:38 PM -->

<!-- 发布 -->

故事的背景是这个世界上的核导弹的发射是只要有密码就能发射，而刚好咱有一个强大的黑客团队可以窃取到密码。咱想要写一个方法，这个方法可以按照顺序发布一些指令，包括让黑客团队窃取密码，然后发射导弹，等待世界毁灭

因为黑客团队都很神秘，请动黑客团队去窃取密码之后，不会从原先的方法返回。而是黑客团队为了安全性（世界都要毁灭了，哪来的安全性） 会通过比特信告诉你，拿到密码了。等等，什么是比特信？这是一个超级高加密的匿名的邮箱 P2P 服务，反正对咱程序来说就是一个事件，定义如下

```csharp
    static class BtcMessage
    {
        public static event EventHandler KeyReceived;
    }

    static class HackTeam
    {
        public static void PeekKey()
        {

        }
    }
```

如上面的定义，黑客团队 HackTeam 有一个公开的方法是窃取密码，具体做啥，神秘的黑客可不会告诉咱。只是知道在完成之后，咱 BtcMessage 的 KeyReceived 事件将会触发。咱可以进行下一步的行动

下一步就是发射导弹了，尽管发射的过程很复杂，但对咱来说也是一个函数调用的事情

```csharp
    static class Missiles
    {
        public static void Fire()
        {

        }
    }
```

发射完成导弹之后，是不是这个世界就毁灭了？还不是，还需要等待导弹落地，爆炸，然后等待生态环境凉凉…… 没关系，最后这个世界会告诉咱世界已经毁灭了，因为这个世界是有[意识](https://github.com/lindexi/lindexi.github.io/blob/master/从人工思维加上二阶熵猜测世界存在意志思维.md)的

```csharp
    static class World
    {
        public static event EventHandler Broke;
    }
```

好，那么咱的逻辑可以如何组织。第一步让黑客团队获取密码，等待 BtcMessage 事件回调。在 BtcMessage 事件触发之后调用导弹发射。然后等待世界触发毁灭事件

按照最简单的逻辑应该是这样写的，本来是想做个 WPF 程序，点击按钮就执行毁灭世界的。不过看看还是控制台简单

```csharp
        static void Main(string[] args)
        {
            World.Broke += (sender, eventArgs) =>
            {
                Console.WriteLine("Hello World!");
            };

            BtcMessage.KeyReceived += (sender, eventArgs) =>
            {
                Missiles.Fire();
            };

            HackTeam.PeekKey();
        }

```

可以看到代码不清真，因为都是倒过来写的，不倒过来也不成，因为如果在动作执行之后再监听事件，说不定事件就执行完成了

这就需要标题的 TaskCompletionSource 出场了，这个类的作用就是支持等待，同时可以被设置完成，很不好理解。写写代码就知道了

添加了 TaskCompletionSource 的写法如下

```csharp
            HackTeam.PeekKey();

            await btcReceivedTask.Task;

            Missiles.Fire();

            await worldBrokeTask.Task;

            Console.WriteLine("Hello World!");
```

是的按照顺序了写下来了，但是 btcReceivedTask 和 worldBrokeTask 是什么？这是先准备好的 TaskCompletionSource 对象

```csharp
            var btcReceivedTask = new TaskCompletionSource<bool>();
            var worldBrokeTask = new TaskCompletionSource<bool>();

            BtcMessage.KeyReceived += (sender, eventArgs) =>
            {
                btcReceivedTask.SetResult(true);
            };

            World.Broke += (sender, eventArgs) =>
            {
                worldBrokeTask.SetResult(true);
            };
```

为什么需要使用 `TaskCompletionSource<bool>` 是因为 TaskCompletionSource 只有泛形的版本，而 `SetResult` 方法一调用，将会让等待的代码继续往下执行

也就是在代码执行到 `await btcReceivedTask.Task;` 的时候，将会进入等待。在 `btcReceivedTask.SetResult` 方法被调用之后，才会继续执行 `await btcReceivedTask.Task;` 之后的代码

于是在 TaskCompletionSource 的辅助之后的代码，写毁灭世界的逻辑请看来就清真了

当然，一开始的代码还可以封装一下，咱可以封装出等待任意事件的触发作为异步的代码

例如封装一个世界被毁灭的等待任务

```csharp
    public class WorldBrokeTask
    {
        public WorldBrokeTask()
        {
            World.Broke += World_Broke;
        }

        private void World_Broke(object sender, EventArgs e)
        {
            World.Broke -= World_Broke;
            TaskCompletionSource.SetResult(true);
        }

        public Task WaitWorldBroke()
        {
            return TaskCompletionSource.Task;
        }

        private TaskCompletionSource<bool> TaskCompletionSource { get; } = new TaskCompletionSource<bool>();
    }

```

这个封装可以协助咱准备监听是哪一次的世界毁灭，这样咱就不需要处理具体的事件监听逻辑。如一开始的代码其实存在一个坑就是当在毁灭世界之后，下一次世界毁灭的时候，又会触发事件。如果不是创建方法，那么很难做到只监听一次

通过封装之后的使用如下

```csharp
    var worldBrokeTask = new WorldBrokeTask();

    HackTeam.PeekKey();

    await btcReceivedTask.Task;

    Missiles.Fire();

    await worldBrokeTask.WaitWorldBroke();

    Console.WriteLine("Hello World!");
```

可以看到创建出来 WorldBrokeTask 然后接着等待就可以了，代码很简单

通过本文的例子相信大家也掌握了毁灭世界，哦，不，使用 TaskCompletionSource 封装事件为异步的方法

当然本文也回答了一个问题，是否使用 await 就存在线程的切换。其实可以不用切换线程

更多请看

- [.NET 中使用 TaskCompletionSource 作为线程同步互斥或异步操作的事件_walterlv - 吕毅-CSDN博客_wpf taskcompletionsource](https://blog.csdn.net/WPwalter/article/details/85526459 )

- [# dotnet 使用 TaskCompletionSource 实现暂停功能](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-TaskCompletionSource-%E5%AE%9E%E7%8E%B0%E6%9A%82%E5%81%9C%E5%8A%9F%E8%83%BD.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
