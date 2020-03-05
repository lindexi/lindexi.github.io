# ParaView

ParaView是一种开放源代码，多平台的数据分析和可视化应用程序。 ParaView用户可以快速建立可视化，分析其数据使用定性和定量方法。勘探的数据可以做三维交互或编程方式使用ParaView的批处理能力

<!-- 不发布 -->
<!--more-->
<!-- CreateTime:2019/9/2 12:57:37 -->


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

