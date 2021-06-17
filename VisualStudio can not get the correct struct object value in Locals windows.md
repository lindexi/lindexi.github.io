# VisualStudio can not get the correct struct object value in Locals windows

VisualStudio 的局部变量窗口里面看到某些结构体的值和实际上的结构体调用 ToString 方法返回的值是不同的

When I use the unsafe struct with the special size and I override the ToString method, then I can find the actual ToString value and the value in VisualStudio Locals windows are different

<!--more-->
<!-- CreateTime:2021/6/15 18:59:37 -->

<!-- 发布 -->

尝试如下代码

Try to run this code:

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var str = "Hello";

            GC.TryStartNoGCRegion(1000000);
            unsafe
            {
                fixed (char* sp = str)
                {
                    var a = new ByValStringStructForSizeMAX_PATH();
                    Buffer.MemoryCopy(sp, &a, 260 * 2, 5 * 2);
              
                    Console.WriteLine(a);
                }
            }
        }
    }

    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode, Size = 260 * sizeof(char))]
    public unsafe struct ByValStringStructForSizeMAX_PATH
    {
        char _firstChar;
        //char _foo;

        /// <inheritdoc/>
        public override string ToString()
        {
            fixed (char* charPtr = &_firstChar)
            {
                //var t = charPtr;
                //StringBuilder str = new StringBuilder();
                //int i = 0;
                //while (true)
                //{
                //    char c = *t;
                //    t++;

                //    if (c == 0)
                //    {
                //        break;
                //    }

                //    i++;

                //    str.Append(c);
                //}

                //str.Append(" ").Append(i).Append(";").Append(GetHashCode()).Append(";").Append(_firstChar.GetHashCode());

                //fixed (void* p = &this)
                //{
                //    str.Append(";").Append(((IntPtr) (&p)));
                //}

                //return str.ToString();

                return new string(charPtr);
            }
        }

        /// <summary>
        /// 
        /// </summary>
        /// <param name="val"></param>
        public static implicit operator string(ByValStringStructForSizeMAX_PATH val) => val.ToString();
    }
```

以上代码放在 GitHub 上： [https://github.com/lindexi/lindexi_gd/tree/52c6016997220b13641c21e470f3aec9069a87a6/GewelhalllemniGafihaja](https://github.com/lindexi/lindexi_gd/tree/52c6016997220b13641c21e470f3aec9069a87a6/GewelhalllemniGafihaja)

You can find the code in github: https://github.com/lindexi/lindexi_gd/tree/52c6016997220b13641c21e470f3aec9069a87a6/GewelhalllemniGafihaja

此时可以看到在 VisualStudio 的局部变量窗口里面，显示的 a 变量的值是 `H` 然而实际上调用 ToString 在控制台将会输出 `Hello` 字符串

And you can find the value in VisualStudio Locals windows is `H` but the actual value is `Hello`

这个看起来是 VisualStudio 的锅，在调试的时候，使用的是拷贝的结构体的值，但是在拷贝时，丢失了结构体的 Size 属性

Seem that the VisualStudio use the copy object and miss the size

如果给 ByValStringStructForSizeMAX_PATH 再加一个字段，将如上代码的 `_foo` 字段去掉注释，如下面代码

```csharp
    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode, Size = 260 * sizeof(char))]
    public unsafe struct ByValStringStructForSizeMAX_PATH
    {
        char _firstChar;
        char _foo;

        // 忽略代码
    }
```

那么此时的 VS 调试可以看到是 `He` 两个字符

本文只是用来给 VisualStudio 报 Bug 的，请看 [https://developercommunity.visualstudio.com/t/VisualStudio-can-not-get-the-correct-str/1450433](https://developercommunity.visualstudio.com/t/VisualStudio-can-not-get-the-correct-str/1450433)

特别感谢 [lsj](https://blog.sdlsj.net) 的协助

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
