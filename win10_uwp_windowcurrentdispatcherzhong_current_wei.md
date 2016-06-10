# win10 uwp Window.Current.Dispatcher中Current为null


我们可以在修改属性使用

```
    public abstract class NotifyPropertyChangedBase : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected async void OnPropertyChanged([CallerMemberName] string propName = "")
        {
            await Window.Current.Dispatcher.RunAsync(CoreDispatcherPriority.High,
                () =>
                {
                    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propName));
                });
        }
    }
 ```

在老周博客：http://www.cnblogs.com/tcjiaan/p/5511419.html

但是我发现在HttpRequest中的函数出发了OnPropertyChanged，这时发现Current is null
 
并且：public event PropertyChangedEventHandler PropertyChanged;中PropertyChanged也是null

老周：由于线程出现嵌套，在Get请求回调的时候，窗口线程已由系统调整。就按你的做法，用主视图层上的调度对象来调用，应用程序级别的视图线程一般不会改变。
要么改用HttpClient类的异步方法来请求，是Windows.Web.Http下面的类，非.net core类型

![](image/

简单方法：
```
await CoreApplication.MainView.CoreWindow.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () => {  });
```
