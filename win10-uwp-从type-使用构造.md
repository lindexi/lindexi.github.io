# win10 uwp 从Type使用构造

本文主要：如何从Type new一个对象

<!--more-->

原本从WPF，要想new一个对象从type，可以使用`type.Assembly.CreateInstance(type.FullName);`

但是在UWP，需要使用`type.GetConstructor(Type.EmptyTypes).Invoke(parameters);`

多谢durow，找了很久在他写的http://www.cnblogs.com/durow/p/4883556.html 刚好有这个