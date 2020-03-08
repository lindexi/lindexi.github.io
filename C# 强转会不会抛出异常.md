# C# 强转会不会抛出异常

最近遇到一个小伙伴问我，从一个很大的数强转，会不会抛出异常。实际上不会出现异常。

<!--more-->
<!-- CreateTime:2018/7/29 14:24:01 -->


最简单的代码是使用一个比 maxvalue 大的数，然后用它强转

```csharp
         long tathkDucmmsc = int.MaxValue ;
            tathkDucmmsc *= 2;
            int kuplStqfbbmx = (int) tathkDucmmsc; // -2
```

结果没有异常，只是拿到的值是 -2

但是因为默认是 `unchecked` 如果把上面的代码添加`checked`，那么就会出异常

```csharp
           checked
            {
                long tathkDucmmsc = int.MaxValue;
                tathkDucmmsc *= 2;
                int kuplStqfbbmx = (int) tathkDucmmsc;
            }
```

```csharp
System.OverflowException:“Arithmetic operation resulted in an overflow.”
```

但是对于 float ，他的值就不是这样了

```csharp
            checked
            {
                double hcmzgSsby = float.MaxValue;

                hcmzgSsby *= 2;

                float djmmmkvawSswu = (float)hcmzgSsby;
            }    
```

可以看到 djmmmkvawSswu 的值是 Infinity 不会出现异常，所以对于浮点数不要通过异常来判断强转是不是超过最大值。

那么如何判断 Infinity ？ 使用`float.IsInfinity`或 `double.IsInfinity` 都可以判断。

![](https://i.loli.net/2018/04/08/5ac9ff8833a18.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
