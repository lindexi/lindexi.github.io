
# Unity3D 判断点击命中物体对象和命中环境

我想要做到点击物体的时候显示一些内容，而点击环境或其他游戏物体的时候隐藏一些内容

<!--more-->


<!-- 发布 -->

本文的脚本都是附加到预置体 Prefab 里面

一个大的空对象包含很多小的物体，在大的空对象上面添加一个脚本，这个脚本核心就是通过 Update 方法里面拿到鼠标点击，判断当前是不是点击这个对象

如下代码放在 SolideCubeElement.cs 代码文件里面

```csharp
    public bool IsClickedSelf { set; get; }

    // Update is called once per frame
    void Update()
    {
        if (Input.GetMouseButtonDown(0))
        {
            if (IsClickedSelf)
            {
            	// 点击到物体
            }
            else
            {
                // 点击到环境
            }

            IsClickedSelf = false;
        }
    }
```

接下来在对应的能响应点击的物体上面添加如下脚本

```csharp
    public GameObject mainElement;

    private void OnMouseDown()
    {
        var element = mainElement.GetComponent<SolideCubeElement>();
        element.IsClickedSelf = true;
    }
```

分别绑定脚本和给物体上面的脚本设置 mainElement 为这个大的空对象

在用户点击物体可以触发 OnMouseDown 方法，在这个方法里面设置 SolideCubeElement 的 IsClickedSelf 属性，刚好 OnMouseDown 会比 Update 方法先进入鼠标点击，此时就可以让空物体判断当前是不是点击到物体

[unity中的获取点击对象_游戏_liuzhongchao123的博客-CSDN博客](https://blog.csdn.net/liuzhongchao123/article/details/82151296 )

[【Unity 3D学习】获取鼠标点击所对应的GameObject_游戏_既然选择了远方，便只顾风雨兼程！-CSDN博客](https://blog.csdn.net/u011601165/article/details/54317390 )

[Unity在游戏中鼠标点击选中GameObject物体并打印其名字_游戏_w8ed-CSDN博客](https://blog.csdn.net/MASILEJFOAISEGJIAE/article/details/84205633?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase )

[unity中使用SetActive()和gameobject实例化的陷坑总结_游戏_wdear0401的博客-CSDN博客](https://blog.csdn.net/wdear0401/article/details/62043171 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。