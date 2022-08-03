
# WPF 解决 ObservableCollection 提示 Cannot change ObservableCollection during a CollectionChanged event 异常

本文告诉大家在使用 ObservableCollection 时，抛出 InvalidOperationException 异常，提示 Cannot change ObservableCollection during a CollectionChanged event 内容，的原因和解决方法

<!--more-->



<!-- 发布 -->
<!-- 博客 -->

准确来说，这个异常和 WPF 是没有任何关系的。这个异常是 ObservableCollection 类型抛出的，而 ObservableCollection 类型是在 dotnet runtime 定义的，放在 System.ObjectModel 里，而且此异常可以在除 WPF 的其他框架，比如控制台或者 UWP 上复现

想要解决此问题，还请先了解一下此异常抛出的原因

在 ObservableCollection 的设计上，是可以了解列表的变更。而在列表的变更了解，是通过 CollectionChanged 事件实现。然而事件的触发，稍微了解 C# 语法的开发者都知道，是每个方法独立执行。这就让 ObservableCollection 存在一个设计上需要解决的问题，那就是如果事件 CollectionChanged 被加等两次，意味着有两次方法的调用。如果在第一次调用方法时，在此方法内再次修改了 ObservableCollection 列表的元素，那么将会让第二个方法进入的时候，所获取的状态和第一个方法所获取的一定不相同

这个设计上的问题，是很难解决的。既然很难解决，那就不解决了，将问题交给开发者好了，在 ObservableCollection 判断如果 CollectionChanged 事件被加等大于 1 次，同时在事件触发的过程中，进行集合的变更，将会抛出 InvalidOperationException 异常，提示 Cannot change ObservableCollection during a CollectionChanged event 内容

这就是从设计上的原因。那为什么只加等 1 次时不抛出呢？那是因为既然只有一次，那改不改都影响不了当前的进入的方法的状态

由于 CollectionChanged 事件加等的次数决定了 InvalidOperationException 是否抛出，从而让一些开发者拿到错误的结论： 在 CollectionChanged 事件里面修改集合本身是安全的。或者反过来，在 CollectionChanged 事件里面修改集合本身是不安全的

正确的行为是： 当 CollectionChanged 事件加等的委托在 1 个以内时，在 CollectionChanged 事件里面修改集合本身是安全的。如果 CollectionChanged 事件加等的委托大于 1 个时，在 CollectionChanged 事件里面修改集合本身是不安全的

从代码上，在 ObservableCollection 的各个更改集合的函数，例如 InsertItem ClearItems RemoveItem 等，都会调用 CheckReentrancy 方法，判断是否存在重入。在 CheckReentrancy 方法的实现如下

```csharp
        /// <summary> Check and assert for reentrant attempts to change this collection. </summary>
        /// <exception cref="InvalidOperationException"> raised when changing the collection
        /// while another collection change is still being notified to other listeners </exception>
        protected void CheckReentrancy()
        {
            if (_blockReentrancyCount > 0)
            {
                // we can allow changes if there's only one listener - the problem
                // only arises if reentrant changes make the original event args
                // invalid for later listeners.  This keeps existing code working
                // (e.g. Selector.SelectedItems).
                if (CollectionChanged?.GetInvocationList().Length > 1)
                    throw new InvalidOperationException(SR.ObservableCollectionReentrancyNotAllowed);
            }
        }
```

上面代码的 `_blockReentrancyCount` 是在 OnCollectionChanged 方法和 BlockReentrancy 方法使用的。在没有重写 ObservableCollection 的情况下，可以认为 `_blockReentrancyCount` 只有在 OnCollectionChanged 方法更改

```csharp
        protected virtual void OnCollectionChanged(NotifyCollectionChangedEventArgs e)
        {
            NotifyCollectionChangedEventHandler? handler = CollectionChanged;
            if (handler != null)
            {
                // Not calling BlockReentrancy() here to avoid the SimpleMonitor allocation.
                _blockReentrancyCount++;
                try
                {
                    handler(this, e);
                }
                finally
                {
                    _blockReentrancyCount--;
                }
            }
        }
```

也就是说，在 CollectionChanged 事件触发进入的方法里面，一定是判断 `if (_blockReentrancyCount > 0)` 通过的。也就说接下来只需要看 `if (CollectionChanged?.GetInvocationList().Length > 1)` 判断即可。这里的 GetInvocationList 是 CollectionChanged 事件对应的委托的数量，只要超过 1 个就炸

了解了原因，那么解决方法也很简单。要么是在 CollectionChanged 事件里面修改集合时确保只让 CollectionChanged 加等一个委托。要么就是继承 ObservableCollection 类型，重写 OnCollectionChanged 方法，不要修改 `_blockReentrancyCount` 字段。要么就是等待 CollectionChanged 事件触发完成之后，通过 `Dispatcher` 的 InvokeAsync 方法调度出去执行




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。