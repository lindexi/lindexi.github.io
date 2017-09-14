# win10 uwp 获得Slider拖动结束的值


本文讲的是如何获得Slider移动结束的值，也就是触发移动后的值。如果我们监听ValueChanged，在我们鼠标放开之前，只要拖动不放，那么就不停触发，而我们可能要得到事件是拖动结束后，那么我们如何获得。
<!--more-->

<div id="toc"></div>
<!-- csdn -->

在WPF，我们可以使用`Thumb.DragCompleted`，连接：[http://stackoverflow.com/questions/723502/wpf-slider-with-an-event-that-triggers-after-a-user-drags](http://stackoverflow.com/questions/723502/wpf-slider-with-an-event-that-triggers-after-a-user-drags)，这个在UWP没有，所以我们没法使用这个。

但是可以使用鼠标放开的值，在 UWP 把触摸放开、鼠标这些叫


