# Unity3d 入门 控制相机移动

本文将告诉大家如何在 Unity3D 中通过键盘或鼠标进行控制 Unity3D 的相机移动。在 Unity3D 中的相机相当于人的视角，通过移动相机可以用来修改咱界面看到的画面

<!--more-->
<!-- CreateTime:2021/5/15 17:10:45 -->


在 Unity3D 中，通过创建脚本的方式，让脚本附加到某个物体上，即可让脚本作用到某个物体上。而这个物体可以是相机。咱先创建一个空白的 Unity3D 项目，接下来创建一个 C# 脚本，当然了，这几个步骤还需要大家自己去摸索一下界面或者看一下新手教程视频哈

新建一个 Move.cs 的 C# 脚本文件，接下来右击此文件使用 C# 项目打开，在 VisualStudio 中添加如下代码

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Animations;

public class Move : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {

    }

    public float Speed = 10;

    // Update is called once per frame
    void Update()
    {
        var horizontalAsixName = "Horizontal";
        var horizontal = Input.GetAxis(horizontalAsixName);

        var verticalAsixName = "Vertical";
        var vertical = Input.GetAxis(verticalAsixName);

        var z = 0f;

        if (Input.GetButton("Fire1"))
        {
            z = 1f;
        }
        else if (Input.GetButton("Fire2"))
        {
            z = -0.1f;
        }

        transform.Translate(new Vector3(horizontal, vertical, z) * Time.deltaTime * Speed, Space.World);
    }
}
```

以上代码即可实现通过上下左右或 wasd 键盘控制物体上下左右移动，然后通过鼠标的左键和右键进行前进和后退

通过 Input.GetAxis 传入参数，即可获取水平或垂直方式的移动量，因为不同的业务下需要的速度不相同，因此咱以上还添加了 Speed 字段用于配置速度。在 Unity3D 里面的规矩标准和 dotnet 通用的有一点不相同的是允许公开 Behavior 等的字段

而 Z 轴方向，用来控制前进和后退的，就通过 Fire1 和 Fire2 分别获取左键和右键，通过此方式即可完成代码。更多请参阅 [Unity3d 连续按键处理和单次按键处理](https://blog.lindexi.com/post/Unity3d-%E8%BF%9E%E7%BB%AD%E6%8C%89%E9%94%AE%E5%A4%84%E7%90%86%E5%92%8C%E5%8D%95%E6%AC%A1%E6%8C%89%E9%94%AE%E5%A4%84%E7%90%86.html )

下一步就是将此 Move.cs 文件拖放到相机上即可

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
