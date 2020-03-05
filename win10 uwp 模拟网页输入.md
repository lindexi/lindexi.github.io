# win10 uwp 模拟网页输入

有时候需要获得网页的 js 执行后的源代码，或者模拟网页输入，如点按钮输入文字。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


如果需要实现，那么就需要用 WebView ，使用方法很简单。

首先创建一个 WebView ，接下来的所有输入都需要在 NavigationCompleted 之后才可以使用。

所以我就在构造方法使用下面代码

```csharp
    webView.Navigate(new Uri("https://www.bing.com/"));
    webView.NavigationCompleted += webView_NavigationCompletedAsync;
```

在模拟输入之前，如果需要在 UWP 使用 Webview 获取网页源代码，那么需要在 加载完成的函数 使用下面的代码来 获得加载完成网页的源代码。

```csharp
private async void webView_NavigationCompletedAsync(WebView sender, WebViewNavigationCompletedEventArgs args)
{
    str = await webView.InvokeScriptAsync("eval", new string[] { "document.documentElement.outerHTML;" });
}
```

用到的方法就是 webView.InvokeScriptAsync 使用 js 代码。

如果需要在指定的文本框输入文字，可以使用下面代码

```csharp
private async void EnterTextAsync(string text,string enterText)
{
    var functionString = string.Format(@"document.getElementsByClassName('{0}')[0].innerText = '{1}';",text, enterText);
    await webView.InvokeScriptAsync("eval", new string[] { functionString });
}
```

看起来这些都是 js 的知识，难度不高。

点击按钮可以使用下面代码

```csharp
private async void SimulateClickAsync(string button)
{
    var functionString = string.Format(@"document.getElementsByClassName('{0}')[0].click();",button);
    await webView.InvokeScriptAsync("eval", new string[] { functionString });
}
```

如果需要填写表单 form 那么前面使用的`innerText`需要修改为value，建议打开 edge 在控制命令输入，尝试一个正确的输入

更多的请去了解 js 的知识

### UWP webView 模拟登陆 csdn

下面给大家一个叫简单方法模拟登陆csdn

```csharp
          GeekWebView.Navigate(new Uri("http://passport.csdn.net/"));

            GeekWebView.NavigationCompleted += OnNavigationCompleted;


            F = async () =>
            {

                var functionString = string.Format(@"document.getElementsByName('username')[0].value='{0}';", "lindexi_gd@163.com");
                await GeekWebView.InvokeScriptAsync("eval", new string[] { functionString });
                functionString = string.Format(@"document.getElementsByName('password')[0].value='{0}';", "密码");
                await GeekWebView.InvokeScriptAsync("eval", new string[] { functionString });

                functionString = string.Format(@"document.getElementsByClassName('logging')[0].click();");
                await GeekWebView.InvokeScriptAsync("eval", new string[] { functionString });
            };

        private Action F { set; get; }

        private void OnNavigationCompleted(WebView sender, WebViewNavigationCompletedEventArgs args)
        {
            F();
        }
```

## 使用 cookie

如果需要使用 cookie 那么请加上下面的代码

```csharp
            Windows.Web.Http.Filters.HttpBaseProtocolFilter filter = new Windows.Web.Http.Filters.HttpBaseProtocolFilter();

```

只要写上这句话就好了

参见：https://stackoverflow.com/questions/44685469/programatically-press-a-button-in-a-website-from-an-app-in-a-phone/44692971

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  