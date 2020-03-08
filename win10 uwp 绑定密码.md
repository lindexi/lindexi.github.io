# win10 uwp 绑定密码

win10 下，密码框无法绑定到ViewModel，Password是不可以绑定。

我们可以自己使用简单方法去绑定。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

我们之前在WPF 使用[绑定密码框](http://lindexi.oschina.io/lindexi/post/WPF-绑定密码/)，我写了一篇，关于如何绑定，我提供一个我自己试了可以的类。

首先，我们新建一个类，这个类是让 PasswordBox 可以绑定Password。

UWP让 PasswordBox 可以绑定Password 的一个方法，其实我使用的和之前一样。

我们新建静态类，附件属性，只是和之前有的小不一样。

我们先写一个函数`PasswordChanged`这个函数是 Password 变化使用的，我们先判断sender 是不是 PasswordBox ，是的话我们就通知密码改变。

通知使用`SetPassword(passwordBox, passwordBox.Password);`我们要先更新password，然后更新界面，但是我们设置password，会自动更新界面，一旦界面更新又更新 password，这样不好，我们需要设置 Updateing，如果是 true，就是我们界面更新，直接更新 password 。如果是 false，那么是 password 更新界面。于是我们在 PasswordChanged ，修改依赖属性，必须添加是否后台修改密码 IsUpdating ，如果是后台添加就需要设置他为 true，目的是可以让前台修改通知，后台修改忽略，不然出现无限循环就不好。
		

```csharp
                SetIsUpdating(passwordBox, true);
                SetPassword(passwordBox, passwordBox.Password);
                SetIsUpdating(passwordBox, false);

```

然后写` private static void OnPasswordPropertyChanged(DependencyObject sender,        DependencyPropertyChangedEventArgs e)`

这个函数是我们定义的一个属性变化时，判断 sender 是不是 PasswordBox ，是的话，因为我们绑定是双向，所以先把`passwordBox.PasswordChanged -= PasswordChanged`取消，然后判断是不是更新了，如果更新了，也就是完成更新`if (!(bool)GetIsUpdating(passwordBox))`我们就把新的Value给`passwordBox.Password`，不管有没更新，我们需要`passwordBox.PasswordChanged += PasswordChanged`
	

```csharp
        private static void OnPasswordPropertyChanged(DependencyObject sender,
         DependencyPropertyChangedEventArgs e)
        {
            PasswordBox passwordBox = sender as PasswordBox;
            if (passwordBox != null)
            {
                passwordBox.PasswordChanged -= PasswordChanged;

                if (!(bool)GetIsUpdating(passwordBox))
                {
                    passwordBox.Password = (string)e.NewValue;
                }
                passwordBox.PasswordChanged += PasswordChanged;
            }
        }

```

我们还需要一个`Attach`判断用户是不是要绑定，如果是 false ，就是和原来，不绑定

我们需要判断 sender 是 PasswordBox ，好像不是的话我们不需要做下，因为都是静态，使用事件绑定，用的是 sender ，关于 Sender 和 Origin 其实是不同的，不过在这里就使用 sender，不是路由事件。

我们判断，在使用 OldValue 是不是true，如果是的话，我们先把`passwordBox.PasswordChanged -= PasswordChanged`，不是的话不能`passwordBox.PasswordChanged -= PasswordChanged` ，因为之前不是已经添加了，所以不可以再移除，即使添加了，也不会出错。这样在修改密码就可以通过函数知道密码已经修改，通过已经修改的密码更新到附加属性，就可以让附加属性得到密码，绑定附加属性，就可以让vm得到值，于是这个核心就是这么简单。

判断NewValue，如果是true，`passwordBox.PasswordChanged += PasswordChanged;`，这时就是绑定了，以后修改了密码就可以知道了。

代码可以复制到一个文件，注意需要使用他所在的name，使用xmlns引用需要用到
		

```csharp
    public static class PasswordBoxHelper
    {
        public static readonly DependencyProperty PasswordProperty = DependencyProperty.RegisterAttached(
            "Password", //属性
            typeof(string),//属性的类型
             typeof(PasswordBoxHelper), //属于的类，我们做的是静态，所以需要这个让附加属性可以知道他所在，我们到时可以使用 sender 拿到实例，所以需要知道他的类可以转
             new PropertyMetadata(default(string),//默认值
            OnPasswordPropertyChanged));//属性改变调的函数

        public static void SetPassword(DependencyObject element, string value)
        {
            element.SetValue(PasswordProperty, value);
        }

        public static string GetPassword(DependencyObject element)
        {
            return (string) element.GetValue(PasswordProperty);
        }

        public static readonly DependencyProperty AttachProperty = DependencyProperty.RegisterAttached(
            "Attach", typeof(bool), typeof(PasswordBoxHelper), new PropertyMetadata(default(bool),Attach));

        public static void SetAttach(DependencyObject element, bool value)
        {
            element.SetValue(AttachProperty, value);
        }

        public static bool GetAttach(DependencyObject element)
        {
            return (bool) element.GetValue(AttachProperty);
        }

        public static readonly DependencyProperty IsUpdatingProperty = DependencyProperty.RegisterAttached(
            "IsUpdating", typeof(bool), typeof(PasswordBoxHelper), new PropertyMetadata(default(bool)));

        public static void SetIsUpdating(DependencyObject element, bool value)
        {
            element.SetValue(IsUpdatingProperty, value);
        }

        public static bool GetIsUpdating(DependencyObject element)
        {
            return (bool) element.GetValue(IsUpdatingProperty);
        }

        private static void OnPasswordPropertyChanged(DependencyObject sender,
         DependencyPropertyChangedEventArgs e)
        {
            PasswordBox passwordBox = sender as PasswordBox;//sender就是实例
            if (passwordBox != null)
            {
                passwordBox.PasswordChanged -= PasswordChanged; //在WPF绑定密码有说为何这样做
                //我们需要修改的是在更改，所以不能让他继续 PasswordChanged 使用了会无限循环 所以先去掉，在后面加上。

                if (!(bool)GetIsUpdating(passwordBox))
                {
                    passwordBox.Password = (string)e.NewValue;
                }
                passwordBox.PasswordChanged += PasswordChanged;
            }
        }

        private static void Attach(DependencyObject sender,
            DependencyPropertyChangedEventArgs e)
        {
            PasswordBox passwordBox = sender as PasswordBox;

            if (passwordBox == null)
            {
                 return;
            }
            //e.OldValue 改变前的值
            if ((bool)e.OldValue)//如果之前有绑定，我们就解绑
            {
                passwordBox.PasswordChanged -= PasswordChanged;
            }
            //e.NewValue 改变的值
            if ((bool)e.NewValue)
            {
                passwordBox.PasswordChanged += PasswordChanged;
            }
        }

        private static void PasswordChanged(object sender, RoutedEventArgs e)
        {
            PasswordBox passwordBox = sender as PasswordBox;
            if (passwordBox != null)
            {
                SetIsUpdating(passwordBox, true);//设置我们修改的是UI绑定的修改，那么不更改PasswordBox.password
                //设置是false会修改，我们通知修改密码，然后他修改后台password又通知PasswordChanged 这样会炸
                SetPassword(passwordBox, passwordBox.Password);
                SetIsUpdating(passwordBox, false);
            }
        }
    }

```

我们的 ViewModel 有一个属性 password ，注意我们使用 Binding 把他绑定到 PasswordBox 的密码。绑定的过程

不需要去做修改，直接加上`view:PasswordBoxHelper.Attach="True" view:PasswordBoxHelper.Password="{Binding Password,Mode=TwoWay}"`

代码很简单，也就是把 不可绑定的 Password 改为下面的属性
		

```csharp
 <PasswordBox  view:PasswordBoxHelper.Attach="True"
               view:PasswordBoxHelper.Password="{Binding Password,Mode=TwoWay}"
                 >

```

注意需要引用命名，好像属性比较长，但是我也没有好方法让他比较好看

附加属性使用的比较多，是不是需要我来写一个博客说下什么是附加属性。

对于 WPF 几乎和rt一样的 附加属性，如果没有找到 好的博客，直接去看 wpf就好啦，我在使用发现不需要去学新的东西就可以使用。

<script src="https://gist.github.com/lindexi/e4809b4b54a36db6aa166524c89fcebb.js"></script>

本文同时发在[九幽专栏](http://www.win10.cm/?p=1220)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 



