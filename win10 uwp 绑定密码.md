# win10 uwp 绑定密码

win10 下，密码框无法绑定到ViewModel，Password是不可以绑定。

我们可以自己使用简单方法去绑定。

<!--more-->

我们之前在WPF 使用[绑定密码框](http://lindexi.oschina.io/lindexi/post/WPF-绑定密码/)，我写了一篇，关于如何绑定，我提供一个我自己试了可以的类。

首先，我们新建一个类，这个类是让PasswordBox 可以绑定Password。

UWP让PasswordBox 可以绑定Password 的一个方法，其实我使用的和之前一样。

我们新建静态类，附件属性，只是和之前有的小不一样。

我们先写一个函数`PasswordChanged`这个函数是Password 变化使用的，我们先判断sender是不是PasswordBox，是的话我们就通知密码改变。

通知使用`SetPassword(passwordBox, passwordBox.Password);`我们要先更新password，然后更新界面，但是我们设置password，会自动更新界面，一旦界面更新又更新password，这样不好，我们需要设置Updateing，如果是true，就是我们界面更新，直接更新password。如果是false，那么是password更新界面。于是我们在PasswordChanged，使用
		
```
                SetIsUpdating(passwordBox, true);
                SetPassword(passwordBox, passwordBox.Password);
                SetIsUpdating(passwordBox, false);

```

然后写` private static void OnPasswordPropertyChanged(DependencyObject sender,        DependencyPropertyChangedEventArgs e)`

这个函数是我们定义的一个属性变化时，判断sender是不是PasswordBox，是的话，因为我们绑定是双向，所以先把`passwordBox.PasswordChanged -= PasswordChanged`取消，然后判断是不是更新了，如果更新了，也就是完成更新`if (!(bool)GetIsUpdating(passwordBox))`我们就把新的Value给`passwordBox.Password`，不管有没更新，我们需要`passwordBox.PasswordChanged += PasswordChanged`
	
```
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

我们还需要一个`Attach`判断用户是不是要绑定，如果是false，就是和原来，不绑定

我们需要判断sender是PasswordBox，好像不是的话我们不需要做下，因为都是静态，使用事件绑定，用的是sender

我们判断，在使用OldValue是不是true，如果是的话，我们先把`passwordBox.PasswordChanged -= PasswordChanged`，不是的话不能`passwordBox.PasswordChanged -= PasswordChanged`

判断NewValue，如果是true，`passwordBox.PasswordChanged += PasswordChanged;`

总的
		
```
    public static class PasswordBoxHelper
    {
        public static readonly DependencyProperty PasswordProperty = DependencyProperty.RegisterAttached(
            "Password", typeof(string), typeof(PasswordBoxHelper), new PropertyMetadata(default(string),OnPasswordPropertyChanged));

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

        private static void Attach(DependencyObject sender,
            DependencyPropertyChangedEventArgs e)
        {
            PasswordBox passwordBox = sender as PasswordBox;

            if (passwordBox == null)
                return;

            if ((bool)e.OldValue)
            {
                passwordBox.PasswordChanged -= PasswordChanged;
            }

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
                SetIsUpdating(passwordBox, true);
                SetPassword(passwordBox, passwordBox.Password);
                SetIsUpdating(passwordBox, false);
            }
        }
    }

```

我们的ViewModel有一个属性password，注意我们使用Binding

		
```
 <PasswordBox  view:PasswordBoxHelper.Attach="True"
               view:PasswordBoxHelper.Password="{Binding Password,Mode=TwoWay}"
                 >

```

