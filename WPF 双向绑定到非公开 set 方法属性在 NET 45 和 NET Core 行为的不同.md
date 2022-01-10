# WPF 双向绑定到非公开 set 方法属性在 NET 45 和 NET Core 行为的不同

本文记录 WPF 在 .NET Framework 4.5 和 .NET Core 3.0 或更高版本对使用 Binding 下的 TwoWay 双向绑定模式绑定到非公开的 set 属性上的行为变更

<!--more-->
<!-- CreateTime:2022/1/8 17:15:41 -->

<!-- 发布 -->
<!-- 博客 -->

在 .NET Framework 4.5 下，可以使用 Binding 下的 TwoWay 双向绑定模式，绑定到非公开的 set 属性，如 `private set` 私有设置的属性上，实现双向更改，效果上和公开的 set 方法一样，可以成功写入

但是在 .NET Core 3.0 开始，此绑定将会提示 XamlParseException 而抛出异常

如以下的 ViewModel 代码，包含了一个 Name 属性，此属性的 set 方法是私有的

```csharp
    class ViewModel : INotifyPropertyChanged
    {
        public string Name
        {
            get => _name;
            private set
            {
                _name = value;
                OnPropertyChanged();
            }
        }

        private string _name;

        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
```

在 XAML 使用如下代码双向绑定，期望在 TextBox 输入的内容可以写入到 Name 属性

```xml
   <TextBox Text="{Binding Name,Mode=TwoWay,UpdateSourceTrigger=PropertyChanged}"></TextBox>
```

以上代码是能在 .NET Framework 4.5 如预期工作。然而在 .NET Core 3.0 或更高版本，将会抛出异常

```
System.Windows.Markup.XamlParseException: '“设置属性“System.Windows.Controls.TextBox.Text”时引发了异常。”'

InvalidOperationException: 无法对“GogeeceldeaLabacheleabe.ViewModel”类型的只读属性“Name”进行 TwoWay 或 OneWayToSource 绑定。
```

对应的英文异常如下

```
System.InvalidOperationException: 'A TwoWay or OneWayToSource binding cannot work on the read-only property 'Name' of type 'GogeeceldeaLabacheleabe.ViewModel'.'
```

根据 [WPF: After Visual Studio 2017 Update, "A TwoWay or OneWayToSource binding cannot work on the read-only property" - Visual Studio Feedback](https://developercommunity.visualstudio.com/t/wpf-after-visual-studio-2017-update-a-twoway-or-on/171772 ) 的描述，其实这是 .NET Framework 4.5 的坑，在 .NET Framework 4.7 就修复了。经过我的考古，在 .NET Framework 4.6 下的行为就和 .NET Core 3.0 版本相同，是会抛出异常

**敲黑板，使用双向绑定到非公开 set 方法的属性上的行为变更，不是 .NET Framework 和 .NET Core 的差别行为变更，而仅仅是 .NET Framework 4.5 和后续版本的差别**

以下是原文：

> So, this was a BUG in framework V4.5, when most of the code was written, and "FIXED" in V4.7

在 WPF 官方从 .NET Framework 拷贝代码到 .NET Core 开源时，也遇到此坑，请看 [Removed HandleTwoWayBindingToPropertyWithNonPublicSetter compat flag by ojhad · Pull Request #1502 · dotnet/wpf](https://github.com/dotnet/wpf/pull/1502 )

> Two-way binding to properties with non-public setters was being allowed because we took on behavior that was caused by a bug in .NET Framework 4.5. Back then, a compatibility flag was introduced due to this 4.5 bug. We no longer want to support this scenario because we want correct behavior in .NET Core and we want the behavior to be on parity with net472/8. So there is no longer need for the compat flag.

在 .NET Core 3.0 的更新里，也提到了这个坑，参阅 [August Update for WPF on .NET Core 3.0 · Issue #1731 · dotnet/wpf](https://github.com/dotnet/wpf/issues/1731 )

此问题我也报告给官方，请看 [Binding non-public property behavior changed between dotnet core 3.1 and net45 · Issue #5923 · dotnet/wpf](https://github.com/dotnet/wpf/issues/5923 )

我认为，如果 ViewModel 设置了属性的 set 为私有，那也就是从设计上不要让其他逻辑进行设置，自然在 XAML 里对非公开设置的属性进行写入也是非预期的，抛出异常符合设计

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/01bb068fd7f714313e44cdbcfdf5d0b5630f1bac/GogeeceldeaLabacheleabe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/01bb068fd7f714313e44cdbcfdf5d0b5630f1bac/GogeeceldeaLabacheleabe) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 01bb068fd7f714313e44cdbcfdf5d0b5630f1bac
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 GogeeceldeaLabacheleabe 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
