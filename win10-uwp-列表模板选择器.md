本文主要讲ListView等列表可以根据内容不同，使用不同模板的列表模板选择器，DataTemplateSelector

好啦，我们先来说下我们在什么下需要使用，其实就是当我们的数据有多样。

例如我们做了一个类，叫做 人，这时我们继承人做出来 男生 和女生，那么男生的属性可能和女生的不同

假设我们的 人有个属性叫做名称，而男生有个属性叫身高，女孩有个属性叫年龄，当然女生年龄放出来并不好，不过我也没找到别的。

好啦，我们在ViewModel放一个ObservableCollection&lt;Human&gt; HumanWord,这是我们发现，在前台不好弄，如何让列表显示男生和女孩，因为他们的属性不同。

这时就需要我们做选择器，这个可以根据我们传入选择模板。

首先我们建立一个类

这个类是ListViewDataTemplateSelector选择我们的模板，模板我们会在xaml，不会写cs

我们类继承DataTemplateSelector

我们属性

```
 public DataTemplate MaleData { set; get; }

 public DataTemplate FemaleData { set; get; }

```

然后我们判断我们是否传进来是男生，如果是就返回MaleData 

我们需要override SelectTemplateCore，这时有两个，如果我们的 `ItemsPanel` 是 `ItemsStackPanel` 或 ItemsWrapGrid 我们就需要选择

