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

