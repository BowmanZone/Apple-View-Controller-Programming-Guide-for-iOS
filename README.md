安装上了zsh，貌似能够让控制台界面更醒目，可是应用主题的时候始终无法应用上，完全就搞不懂这个东西，好心塞。http://ohmyz.sh

Markdown语法： http://wowubuntu.com/markdown/index.html#img Ubuntu的一些资讯

学习方向：

* iOS运行流程。软件启动，视图生命周期等
* iOS开发的框架
* iOS测试指南。黑盒、白盒测试，性能、UI测试等
* iOS发布指南
* iOS人机交互指南。最新的指南可以说已经对设计师量身定做
* swift语言。包括swift的语法和使用swift开发iOS应用程序这两部分
* Auto Layout。屏幕适配，超哥的屏幕适配思路和Auto Layout

注意：切记不要浮躁，针对应用功能和开发条件选择开发框架进行学习，心无旁骛。切记不要杂而不精。

UIPresentationController
=
##基本概念
摘自：<http://www.15yan.com/story/jlkJnPmVGzc/>
>
先讲一些presentation基础知识，在iPad的设置页面，可以通过popOver弹出一个UIViewController，这个弹出的，可以和用户交互的Controller叫做PresentedViewController，而后面那个被部分遮挡的UIViewController叫做PresentingViewController，而在UIPresentationController中，PresentedViewController是presentation的content，而PresentingViewController叫做Chrome。如下图：

[presentedViewController、presentingViewController](http://ob7zbqpa6.qnssl.com/kg9l1xv6wlvuxs18us31vzsjhztrtc2l.jpg!content)

[content](http://ob7zbqpa6.qnssl.com/0s89tcp3kin0memvexrxu14mc8mesrs4.jpg!content)

[chrome](http://ob7zbqpa6.qnssl.com/2f6ofboq6xrq53drt9x1bjioy1hohob7.jpg!content)

所有的UIViewController的presentation都是由UIPresentationController管理的。在UIPresentationController中可以定义content和chrome的动画，可以根据大小的变化来改变content大小，可以根据系统的不同，来改变展示方式，UIPresentationController也是可复用的，可以很容易的拿到其他的UIViewController中去使用。
>

UIPresentationController对象为presented view controllers提供了先进的视图和过渡管理系统。
