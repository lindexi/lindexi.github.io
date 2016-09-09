#win10 uwp如何使用DataTemplate

这是数据模板，一般用在数组的绑定，显示数组中的元素。

我们使用`Binding`和WPF其实没有多少不同，在Mode只有`OneWay`,`OneTime`,`TwoWay`。我们使用的`x:bind`在DataTemplate才和原来有一些不同。




##转换

如果我们的数据需要转换，转换需要我们的变量，在我上次使用win10 uwp 进度条使用了是静态的数值，这样不好，那么我们需要做一个简单使用我们类数据转换器，在我们常用的是把它写staticResource
