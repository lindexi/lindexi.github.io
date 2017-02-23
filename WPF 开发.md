# WPF 开发

本文：我遇到的WPF的坑

<!--more-->

## 非托管使用托管委托

如果有一个 C++ 写的dll，他需要一个函数指针，在C#使用，就可以传入委托。

那么简单的方法是这样写：


```csharp
    private static void Func(){}
    public void C()
    {
        c(Func);
    }
```

其中c就是C++写的函数，传进去看起来好像正常。

但是有时候程序不知道怎么就炸了。

因为这样写是不对的。

传入的不是函数地址，传入的是把函数隐式转换委托，然后转换的委托是局部变量，会被gc，所以在C++拿到的是一个被回收的委托，调用时就会炸。

这里无法用catch，所以用这个会让程序退出。

调用C#的函数，使用委托，是隐式转换，上面代码可以写成下面的


```csharp
    private static void Func(){}
    public void C()
    {
         var temp=new delegate(){Func};
         c(temp);
    }

```

于是在函数完就把temp放到gc在调用时找不到委托。

一个好的做法


```csharp
    private static void Func(){}
    private delegate Temp=new delegate(){Func};
    private void C()
    {
        c(Temp);
    }
```

放在静态变量不会gc调用不会空，可以这样不会出现上面问题。




