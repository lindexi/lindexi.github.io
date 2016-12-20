# WPF 绑定密码

我们发现我们无法绑定密码框的密码，PasswordBox的Password不能绑定。

<!--more-->

我在网上找的很多大神给出的可以解决，我找了一个简单方法。

新建一个类PasswordHelper
		
```
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


然后在xaml
		
```
<PasswordBox local:PasswordHelper.Attach="True" 
                             local:PasswordHelper.Password="{Binding Password, Mode=TwoWay}" 
                             Width="180" Style="{DynamicResource PasswordBoxStyle}"/>

```

其中，Password是ViewModel的PassWord，很简单我们把PasswordBox 绑定到ViewModel。


PASSWORDPROPERTY是附加属性，REGISTERATTACHED就是注册附加。

我们附加属性是回调，当属性变化使用函数。

我们需要设置Attach，设置时调用`static void Attach(DependencyObject sender,           DependencyPropertyChangedEventArgs e)`

判断设置的是不是Password
		
```
            PasswordBox passwordBox = sender as PasswordBox;

            if (passwordBox == null)
            {
                return;
            }

```
判断改变的值，Old是true还是false，如果是true，那么之前用了事件，我们要把事件
		
```
passwordBox.PasswordChanged -= PasswordChanged;

```
如果之前是false，那么没绑定，我们不能删除。

判断要改变的，如果是true，我们就
		
```
passwordBox.PasswordChanged += PasswordChanged;

```

如果不是，我们就不使用。

我们使用了是否存在密码修改就使用PasswordChanged函数。也就是设置了刚才的就可在密码变化使用PasswordChanged。

我们在PasswordChanged判断输入是不是PasswordBox，把密码传进PasswordProperty。

参见：http://www.wpftutorial.net/PasswordBox.html

还有一个简单方法

<script src="https://gist.github.com/taylorleese/468331.js"></script>

		
```
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