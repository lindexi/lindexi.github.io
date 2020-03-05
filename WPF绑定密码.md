# WPF 绑定密码

我们发现我们无法绑定密码框的密码，PasswordBox 的 Password 不能绑定。

我们想做 MVVM ，我们需要绑定密码，不能使用前台 xaml.cs 监听 密码改变得到密码的值，传到 ViewModel 。

本文提供一个简单方法来绑定 WPF 的 PasswordBox 的 Password 。这种方法不仅在 WPF 可以使用，在 UWP 也可以使用。关于 UWP 绑定密码，可以在我博客 [win10 uwp 绑定密码](http://lindexi.oschina.io/lindexi/post/win10-uwp-%E7%BB%91%E5%AE%9A%E5%AF%86%E7%A0%81/) 查看。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

我在网上找的很多大神给出的可以解决绑定密码的方法，下面是我找的一个简单方法。

首先需要新建一个类 PasswordHelper ，他是一个静态类，当然不是静态也没关系，但是一般写静态的可以让我们少犯错，因为我们所有属性等都是需要静态的。
		

```csharp
    public static class PasswordHelper
    {
        public static readonly DependencyProperty PasswordProperty =
            DependencyProperty.RegisterAttached("Password",
            typeof(string), typeof(PasswordHelper),
            new FrameworkPropertyMetadata(string.Empty, OnPasswordPropertyChanged));

        public static readonly DependencyProperty AttachProperty =
            DependencyProperty.RegisterAttached("Attach",
            typeof(bool), typeof(PasswordHelper), new PropertyMetadata(false, Attach));

        private static readonly DependencyProperty IsUpdatingProperty =
           DependencyProperty.RegisterAttached("IsUpdating", typeof(bool),
           typeof(PasswordHelper));


        public static void SetAttach(DependencyObject dp, bool value)
        {
            dp.SetValue(AttachProperty, value);
        }

        public static bool GetAttach(DependencyObject dp)
        {
            return (bool)dp.GetValue(AttachProperty);
        }

        public static string GetPassword(DependencyObject dp)
        {
            return (string)dp.GetValue(PasswordProperty);
        }

        public static void SetPassword(DependencyObject dp, string value)
        {
            dp.SetValue(PasswordProperty, value);
        }

        private static bool GetIsUpdating(DependencyObject dp)
        {
            return (bool)dp.GetValue(IsUpdatingProperty);
        }

        private static void SetIsUpdating(DependencyObject dp, bool value)
        {
            dp.SetValue(IsUpdatingProperty, value);
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


写完我们就可以使用他，使用很简单，在我们需要密码框的页面的xaml 上写两行新的代码就好。
		

```xml
<PasswordBox local:PasswordHelper.Attach="True" 
                             local:PasswordHelper.Password="{Binding Password, Mode=TwoWay}" 
                             Width="180" Style="{DynamicResource PasswordBoxStyle}"/>

```

其中，Password 是 ViewModel 的PassWord，很简单我们把PasswordBox 绑定到ViewModel。


PASSWORDPROPERTY是附加属性，REGISTERATTACHED 就是注册附加。

我们附加属性是回调，当属性变化使用函数。

我们需要设置Attach，设置时调用`static void Attach(DependencyObject sender,           DependencyPropertyChangedEventArgs e)`

在 Attach 触发，首先要判断设置的 sender 是不是 Password
		

```csharp
            PasswordBox passwordBox = sender as PasswordBox;

            if (passwordBox == null)
            {
                return;
            }

```
判断改变的值，Old是true还是false，如果是true，那么之前用了事件，我们要把事件
		

```csharp
passwordBox.PasswordChanged -= PasswordChanged;

```
如果之前是false，那么没绑定，我们不能删除。

判断要改变的，如果是true，我们就
		

```csharp
passwordBox.PasswordChanged += PasswordChanged;

```

如果不是，我们就不使用。

我们使用了是否存在密码修改就使用PasswordChanged函数。也就是设置了刚才的就可在密码变化使用PasswordChanged。

我们在PasswordChanged判断输入是不是PasswordBox，把密码传进PasswordProperty。

参见：http://www.wpftutorial.net/PasswordBox.html

还有一个简单方法

<script src="https://gist.github.com/taylorleese/468331.js"></script>

		

```csharp
using System.Windows;
using System.Windows.Controls;

namespace CustomControl
{
    public class BindablePasswordBox : Decorator
    {
        /// <summary>
        /// The password dependency property.
        /// </summary>
        public static readonly DependencyProperty PasswordProperty;

        private bool isPreventCallback;
        private RoutedEventHandler savedCallback;

        /// <summary>
        /// Static constructor to initialize the dependency properties.
        /// </summary>
        static BindablePasswordBox()
        {
            PasswordProperty = DependencyProperty.Register(
                "Password",
                typeof(string),
                typeof(BindablePasswordBox),
                new FrameworkPropertyMetadata("", FrameworkPropertyMetadataOptions.BindsTwoWayByDefault, new PropertyChangedCallback(OnPasswordPropertyChanged))
            );
        }

        /// <summary>
        /// Saves the password changed callback and sets the child element to the password box.
        /// </summary>
        public BindablePasswordBox()
        {
            savedCallback = HandlePasswordChanged;

            PasswordBox passwordBox = new PasswordBox();
            passwordBox.PasswordChanged += savedCallback;
            Child = passwordBox;
        }

        /// <summary>
        /// The password dependency property.
        /// </summary>
        public string Password
        {
            get { return GetValue(PasswordProperty) as string; }
            set { SetValue(PasswordProperty, value); }
        }

        /// <summary>
        /// Handles changes to the password dependency property.
        /// </summary>
        /// <param name="d">the dependency object</param>
        /// <param name="eventArgs">the event args</param>
        private static void OnPasswordPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs eventArgs)
        {
            BindablePasswordBox bindablePasswordBox = (BindablePasswordBox) d;
            PasswordBox passwordBox = (PasswordBox) bindablePasswordBox.Child;

            if (bindablePasswordBox.isPreventCallback)
            {
                return;
            }

            passwordBox.PasswordChanged -= bindablePasswordBox.savedCallback;
            passwordBox.Password = (eventArgs.NewValue != null) ? eventArgs.NewValue.ToString() : "";
            passwordBox.PasswordChanged += bindablePasswordBox.savedCallback;
        }

        /// <summary>
        /// Handles the password changed event.
        /// </summary>
        /// <param name="sender">the sender</param>
        /// <param name="eventArgs">the event args</param>
        private void HandlePasswordChanged(object sender, RoutedEventArgs eventArgs)
        {
            PasswordBox passwordBox = (PasswordBox) sender;

            isPreventCallback = true;
            Password = passwordBox.Password;
            isPreventCallback = false;
        }
    }
}

```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。