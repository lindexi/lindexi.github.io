
# ParaView

ParaView是一种开放源代码，多平台的数据分析和可视化应用程序。 ParaView用户可以快速建立可视化，分析其数据使用定性和定量方法。勘探的数据可以做三维交互或编程方式使用ParaView的批处理能力
<!-- 不发布 -->

<!--more-->



ParaView-5.0.1-Qt4-OpenGL2-MPI-Linux-64bit.tar.gz

链接：http://pan.baidu.com/s/1kVxW89D 密码：stpl

`MobaXterm_Personal_7.6`

链接：http://pan.baidu.com/s/1c2nv95Y 密码：hk4k

```
./bin/pvpython lib/paraview-5.0/site-packages/paraview/web/pv_web_visualizer.py --content ./share/paraview-5.0/www --data-dir ../ParaView-5.0.0/ --port 125

    
```

```
./bin/pvpython lib/paraview-5.0/site-packages/paraview/web/pv_web_visualizer.py --content ./share/paraview-5.0/www --data-dir ../ParaView-5.0.0/ --port 125
2016-04-22 16:42:03+0800 [-] Log opened.
ERROR: In /home/kitware/dashboards/buildbot/paraview-debian6dash-linux-shared-release_opengl2_qt4_superbuild/build/paraview/src/paraview/VTK/Rendering/OpenGL2/vtkXOpenGLRenderWindow.cxx, line 288

vtkXOpenGLRenderWindow (0x5359980): bad X server connection. DISPLAY=Aborted

```

```
--use-offscreen-rendering

pv_web_visualizer.py: error: unrecognized arguments: --use-offscreen-rendering
```
https://www.kitware.com/blog/home/post/999

http://www.paraview.org/Wiki/ParaView/ParaView_And_Mesa_3D


```
autoreconf -fi
libtoolize: putting auxiliary files in AC_CONFIG_AUX_DIR, `bin'.
libtoolize: copying file `bin/ltmain.sh'
libtoolize: putting macros in AC_CONFIG_MACRO_DIR, `m4'.
libtoolize: copying file `m4/libtool.m4'
libtoolize: copying file `m4/ltoptions.m4'
libtoolize: copying file `m4/ltsugar.m4'
libtoolize: copying file `m4/ltversion.m4'
libtoolize: copying file `m4/lt~obsolete.m4'
```

```
./configure \
    CXXFLAGS="-O2 -g -DDEFAULT_SOFTWARE_DEPTH_BITS=31" \
    CFLAGS="-O2 -g -DDEFAULT_SOFTWARE_DEPTH_BITS=31" \
    --disable-xvmc \
    --disable-glx \
    --disable-dri \
    --with-dri-drivers="" \
    --with-gallium-drivers="swrast" \
    --enable-texture-float \
    --disable-shared-glapi \
    --disable-egl \
    --with-egl-platforms="" \
    --enable-gallium-osmesa \
    --enable-gallium-llvm=yes \
    --with-llvm-shared-libs \
    --prefix=/HOME/nsfc2015_88/lindexi.Mesa
```
```
make -j2 
make -j4 install
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。