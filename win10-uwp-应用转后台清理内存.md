# win10 uwp 应用转后台清理内存

我在写小说阅读器，把每个打开的文件的内容读到内存，因为小说都很小，所以放在内存不怕太大，但是我如果打开了一本小说，再打开一本，我不会把先打开的小说的内容清除掉，在内存。所以一旦我打开多小说的时候，内存就会用比较多，这样觉得不好，不过垃圾wr给我们一个事件，这个我会在下面说。
<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

我们很多应用会在前台用很多资源，例如我们的界面，在转入后台可以清理很多资源，如果判断用户不是马上就转回的，我们可以用新的`EnteredBackground`使用简单。在我的小说里面有写，我来看看我是怎么写。

在我们的App()

```csharp
        public App()
        {
            this.InitializeComponent();
            this.Suspending += OnSuspending;

            EnteredBackground += App_EnteredBackground;
            LeavingBackground += App_LeavingBackground;
        }

        private void App_LeavingBackground(object sender, LeavingBackgroundEventArgs e)
        {
            //应用离开后台
            _areBackground = false;
        }

        private void App_EnteredBackground(object sender, EnteredBackgroundEventArgs e)
        {
            //应用进入后台
            _areBackground = true;
        }

        private bool _areBackground;


```

如果你应用没有EnteredBackground ，更新sdk 14393

如果安装了，可以修改你的文件`<TargetPlatformVersion>10.0.14393.0</TargetPlatformVersion>`

![这里写图片描述](http://img.blog.csdn.net/20160913190120214)

我们判断在我们进入后台清理我们的内存，因为小说经常不是后台就不用，我们就把我们现在使用的小说打开，其他打开小说放在内存资源全删，很简单，但是我们看官方建议是不`GC.Collect();`

```csharp
        private void App_EnteredBackground(object sender, EnteredBackgroundEventArgs e)
        {
            //应用进入后台
            _areBackground = true;
            //清理
            var account = AccountGoverment.View.Account;//我们把所有的用户的放在用户管理
            if (account != null)
            {//我们把我们现在打开的小说除掉，其他都清空
                foreach (var temp in account.File.Where(temp => temp != AccountGoverment.View.File))
                {
                    //AccountGoverment.View.File我们打开的小说
                    //account.File 所有小说
                    temp.Str = null;
                    //我们会在加载的时候，点击小说，EaddressModel.Read()
                }
            }
        }
```

我们对于内存，还有在我们使用缓存，判断我们应用使用内存，垃圾wr给我们两个事件，需要我们用sdk 14393，我觉得垃圾wr这样不好，才半年就改

`MemoryManager`给我们几个属性，`AppMemoryUsage` 获取应用程序当前的内存使用率, `AppMemoryUsageLevel` 获取应用程序当前的内存使用率级别,`AppMemoryUsageLimit` 获取应用程序当前的内存使用率限制,都是只读，我们可以获取这些值来得到我们应用是不是占用太多内存。

`AppMemoryUsageLimitChanging`应用限制最大内存，在我们应用转入后台，我们的内存限制就会变化，然而还有很多诡异的也会让我们内存限制变化，我们可以根据内存变化，清理缓存

`AppMemoryUsageIncreased`我们在我们开始缓存需要我们的内存应用等级，这个事件是内存使用等级，假如我们的内存等级从小到大，那么发生，一旦发生我们就要检查我们是否清理

`AppMemoryUsageDecreased`我们应用内存等级下降，在我们使用内存从大到小使用，这个在我们清理很多缓存可以让我们知道不用清理，一般用是在`AppMemoryUsageIncreased` 有个任务CleanTask，把我们的缓存清理，然后我们有个bool，一旦`AppMemoryUsageDecreased`我们就设为true，那么我们的CleanTask判断true就停下。


源代码：https://github.com/lindexi/NovelRead

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。






