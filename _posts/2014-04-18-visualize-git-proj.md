---
layout: post
title: "Visualize Git Projects with Ubigraph"
description: ""
category:
tags: [Data]
---
{% include JB/setup %}

一个比较大的项目一般都由一群人协作开发，开发人员可能活动于各个模块之间。前两天突然想起如果把一个工程的所有commit数据提取出来，然后按时间顺序动态演示出来可能会比较好玩。从这个过程中我们可以看到一个项目是如何进化的，各个开发者到底在折腾哪些模块。比如这是一个多个开发者参与的一个项目展示图，其实是3D动态的。

<img src="/images/gitshow.png" alt="gitshow" class="img-center"/>

我写了两个脚本来做这件事情，代码放在[这里了](https://github.com/chenyukang/VisualizeGitProj)。第一个脚本是Ruby写的gitstat.rb，用来提取git的commit数据，这些信息包括：提交者名字，日期，增加的行数，删减的行数，相关的模块。所有这些数据都按照提交的时间排序，然后输出到一个文本文件里。
使用方法是:

{% highlight sh %}
$./gitstat.rb -l eventmachine,tinyrb -o log.txt
{% endhighlight %}
-l后面是模块名字列表，如果不加-l脚本会自己检测出当前文件夹下所有的.git，每一个目录当做一个模块。log.txt的格式看起来像这个样子：

{% highlight sh %}
Francis     2008-07-28T16:57:15+00:00   1   1   eventmachine
Francis     2008-07-28T17:03:46+00:00   2   0   eventmachine
Francis     2008-07-29T23:34:53+00:00   3   1   eventmachine
Macournoyer 2008-07-31T23:34:52+00:00   13  47  tinyrb
Macournoyer 2008-08-01T00:36:27+00:00   32  0   tinyrb
{% endhighlight %}

另外一个脚本就是gitshow.py用来从文件中读取数据，然后发送给Ubigraph渲染。

Ubigraph可以从官方网站上下载，解压后会看到一个example目录，里面有几种语言的示例。使用方式是：

{% highlight sh %}
$./bin/ubigraph_server  [在Ubigraph目录启动服务端]
$./gitshow.py log.txt
{% endhighlight %}

这里开发者用圆球表示，模块用多边形球表示，并且颜色加以区分。另外加入了一点效果就是当开发者有提交的时候，其颜色闪红一下，同时开发者和模块之间加上一条虚线。并且开发者和模块的体积会随着代码改变量而增大，这样也能看出哪些模块工作量比较大(当然用行数来衡量工作量本身并没有多大参考价值，只是为了效果)。

对于一个多人参与的项目也可以看出一些好玩的信息来，如果一个开发者贡献大其体积越大，而且离项目的节点越近，比如eventmachine的演示图如下：

<img src="/images/eventmachine.png" alt="gitshow" class="img-center"/>


有一个类似的开源的C++项目叫做: [Gource](https://github.com/acaudwell/Gource)，效果做得很漂亮。
