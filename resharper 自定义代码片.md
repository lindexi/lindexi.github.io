# resharper 自定义代码片

我们在做一件事前，需要先做工具，工具好，最后我们做事也快。这个过程需要的时间也是值得的。

我们在C#下使用的工具，有一个神器，Resharper，他可以帮修改代码、重构，做很多重复的事。

而Resharper 虽然强大，但是还不能全和我们需要的一样，如代码片，有些需要或经常使用的代码还是没有，需要自己去写，他自带的代码片叫 Live Template.

Resharper的代码预知和 VisualStudio 的代码片相似，但是他可以知道当前输入
是变量还是属性，这样就比原来的好用。

本文主要：如何修改Resharper代码片，自定义代码片

<!--more-->

<div id="toc"></div>

原本我的 VisualStudio 也是可以自定义代码片，在工具选择代码片，导入自己写的代码片。

安装了 Resharper 2016.2 会隐藏 VisualStudio 的代码片。

resharper 提供了很有用的代码片，但是我们还是觉得不够，这时我们需要自己编辑 Resharper 代码片。

我将会告诉大家，如果在 resharper 定义自己需要的 代码片 。

打开 Resharper > Tool > Templates Explor

![](http://7xqpl8.com1.z0.glb.clouddn.com/76a67ab5-7429-4e23-8bd2-6d6d68755c8e2016122205413.jpg)

选择语言

![](http://7xqpl8.com1.z0.glb.clouddn.com/76a67ab5-7429-4e23-8bd2-6d6d68755c8e2016122205450.jpg)

选择一个修改的代码片，选择编辑

![](http://7xqpl8.com1.z0.glb.clouddn.com/76a67ab5-7429-4e23-8bd2-6d6d68755c8e2016122205827.jpg)

可以添加新的代码片，我们新建一个，接下来准备做一个简单的东西。

和vs的一样，除了不变的文字，对于需要改变的变量，使用`$变量$`。在输入的时候，对于变量相同，会在输入之后换为相同单词，而不同的变量，可以按 Enter 跳到下一个，当然一旦按 Enter 就是确定这个单词。

例如我们想写一个ViewModel经常写的代码

```csharp
        public string Url
        {
            set
            {
                _url = value;
                OnPropertyChanged();
            }
            get
            {
                return _url;
            }
        }

        private string _url;

```

其中所有的属性`public`是固定的，但是类型不是，我们给类型一个变量`$string$`，
可以看到 Url 是变量名，不同的，我们给一个变量，$name$

可以看到，这变量，有 Url 需要我们写三遍，而且还需要写set、get，所以我们需要写一个简单的模板，直接使用。

接下来我就直接写出一个可以使用的

```csharp
public $string$ $name$
{
    set
    {
         _$name$=value;  
		 OnPropertyChanged();       
    }
    get
    {
         return _$name$;
    }
}

private $string$ _$name$$END$;

```

所有输入的`$string$`都会代换为一个单词，`$name$`也代换为一个单词，这个单词就是用户输入

写完我们设置按键

![](http://7xqpl8.com1.z0.glb.clouddn.com/136fe646-e19f-446e-99e9-0159fa8e5fca2016123193729.jpg)

这一个就是在代码按 ps 就会使用属性加上`OnPropertyChanged();`


还有特殊的变量`$END$`，变量作用在用户写完就是跳到END位置。

有定义一些常用的变量，这变量不会让用户改变。

我们先看下有哪些。

 - `$SELECTION$`就是选择放在地方，这代码用在的是surround templat，关于这个我们刚才没有说，刚才说的是快速输入代码，而包围代码是我们选择了一段代码，然后让模板把代码包围。

 - `$SELSTART$`

 - `$SELEND$ ` 选择一段字符结束，和上面的合起就是选择一段

我们可以使用之前Vs写的代码。

其实上面代码，我们不能让命名有下划线小写

要让变量名小写，我们可以使用`macr`

在我们写出一个变量，可以在左边出现 mar

我们修改下模板

```csharp
public $string$ $name$
{
    set
    {
         _$field$=value;  
		 OnPropertyChanged();       
    }
    get
    {
         return _$field$;
    }
}

$END$

private $string$ _$field$;

```

![](http://7xqpl8.com1.z0.glb.clouddn.com/76a67ab5-7429-4e23-8bd2-6d6d68755c8e2016122213645.jpg)

点击属性选择，我们可以让输入的变量，修改范围

![](http://7xqpl8.com1.z0.glb.clouddn.com/76a67ab5-7429-4e23-8bd2-6d6d68755c8e2016122213830.jpg)

输入Name是`Suggest name variable`输入名称为变量名

然后field是在Name前第一个小写

选择上下就是输入变量的前后，第一个是第一输入

https://www.jetbrains.com/help/resharper/2016.2/Templates__Creating_and_Editing_Templates.html

写好，我们选快捷键

保存

在一个新建工程输入快捷键，就可以输入我们写的

## 常用功能

接下来介绍一下功能

[Resharper 如何把类里的类移动到其他文件](Resharper 如何把类里的类移动到其他文件.md)

如何在 Resharper  忽略文件？

有一些文件需要忽略，不让他分析，因为这文件太多错误，但是是必要的，可以打开Resharper 设置

选择 Code Inspection ，设置，添加例外文件。

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017413103255.jpg)

添加例外文件可以指点哪些文件忽略，可以忽略某个文件，很简单，如果有不懂请告诉我。



如何显示空格

显示有多少空格，有些代码可能存在看不见的字`\u0012`，如果有这些，你以为是空格。

所以需要显示空格，按 ctrl+r+w

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017413103548.jpg)



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。



