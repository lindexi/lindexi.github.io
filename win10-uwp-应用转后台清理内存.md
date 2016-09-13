#win10 uwp 应用转后台清理内存

我在写小说阅读器，把每个打开的文件的内容读到内存，因为小说都很小，所以放在内存不怕太大，但是我如果打开了一本小说，再打开一本，我不会把先打开的小说的内容清除掉，在内存。所以一旦我打开多小说的时候，内存就会用比较多，这样觉得不好，不过垃圾wr给我们一个事件，这个我会在下面说。

我们很多应用会在前台用很多资源，例如我们的界面，在转入后台可以清理很多资源，如果判断用户不是马上就转回的，我们可以用新的`EnteredBackground`使用简单。在我的小说里面有写，我来看看我是怎么写。

在我们的App()

```
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

```
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

`AppMemoryUsageLimitChanging`在我们应用转入后台，我们的内存限制就会变化，然而还有很多诡异的也会让我们内存限制变化，我们可以根据内存变化，清理缓存



