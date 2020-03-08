# dotnet Blazor 用 C# 控制界面行为

微软很久就在做 Blazor 但是我现在才开始创建一个测试项目，我想用 C# 去控制 HTML 界面。小伙伴也许会问现在前端不是烂大街么，为什么还需要 Blazor 来做。可能原因只有一个，就是可以使用 C# 写脚本，代码比较清真

<!--more-->
<!-- CreateTime:2019/11/20 18:26:25 -->

<!-- csdn -->

用 VisualStudio 创建一个默认的 Blazor 项目，在创建完成之后，可以看到有很多例子文件，这样可以降低入手成本。我是从睡醒3点开始创建项目，同时一边水群，一边看 B 站，然而我在 3 点半左右就做出下图效果，虽然大部分逻辑都不理解

![](https://i.loli.net/2019/11/09/nD1pvHRa7biPZqT.gif)

所以本文不是教程，而是在告诉大家又有一个新坑

## 路由

在 Blazor 里面，用页面第一句代码 `@page` 说明这个页面的路由，也就是默认首页的是 `/` 可以这样写

```csharp
@page "/"
```

也就是无论页面命名为什么，只需要设置了路由，就能更改默认页面

## 页面就是字符串

在我用 Blazor 的理解，整个页面除了代码就是字符串，而这个页面可以使用字段变量作为占位符替换。也就是整个页面的显示内容包括样式都是可以使用局部变量替换的，所以本文上面的逻辑就是通过让按钮的样式绑定变量，通过在代码修改变量的方式修改界面

默认有 Counter.razor 页面，在这个页面里面，默认的代码如下

```csharp
<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code 
{

    int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
    }
}
```

也就是在按钮点击时触发 IncrementCount 方法，而在上面有文本绑定 `currentCount` 在代码会被修改，此时可以看到点击按钮时将会修改文本的值

现在尝试将按钮的样式也进行绑定，将按钮的字体和 margin 进行绑定

```html
<button class="btn btn-primary" style="
     font-size: @fontSize; 
     margin-left: @(marginLeft)px;
" @onclick="IncrementCount">Click me</button>
```

当然此时需要在 code 里面添加 fontSize 和 marginLeft 的定义。在上面代码，因为需要拼接 xx px 字符串，而 marginLeft 的值和 px 中间没有空格隔开，也就是需要 `margin-left: 10px;` 才是需要的值，如果连一起写就是 `margin-left: @marginLeftpx;` 也就是无法识别为 `@marginLeft` 变量和 `px` 还是需要 `@marginLeftpx` 变量，此时就可以使用括号

修改代码让点击按钮触发事件，修改按钮样式

```csharp
@code 
{
    
	string fontSize = "large";

	int marginLeft = 1;

    int currentCount = 0;

    void IncrementCount()
    {
	   if(fontSize == "large")
	   {
	       fontSize = "initial";
	   }
	   else
	   {
	       fontSize = "large";
	   }
        marginLeft++;
    }
}
```

这样就是上文的效果，如果有前端小伙伴协助写界面样式，此时让我来写 C# 业务逻辑，应该是可以快速上手的。这就是 Blazor 的优点

大部分项目都可以忽略性能和并发和流量，所以一些玩具可以使用 Blazor 开发

使用 HTML 写界面对我来说还是不清真，现在有 UNO 项目，这个项目是通过 XAML 写界面的，可以作为 WebAssembly 发布，这个玩具会更清真

微软的技术还是很强，可惜这些技术都不在实处，大概作为玩具还不错。期待这些玩具能真的作为产品级使用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
