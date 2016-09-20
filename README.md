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
> 先讲一些presentation基础知识，在iPad的设置页面，可以通过popOver
> 弹出一个UIViewController，这个弹出的，可以和用户交互的
> Controller叫做PresentedViewController，而后面那个被部分遮挡的
> UIViewController叫做PresentingViewController，而在
> UIPresentationController中，PresentedViewController是
> presentation的content，而PresentingViewController叫做
> Chrome。如下图：
> ![presentedViewController、presentingViewController](http://ob7zbqpa6.qnssl.com/kg9l1xv6wlvuxs18us31vzsjhztrtc2l.jpg!content)
> ![content](http://ob7zbqpa6.qnssl.com/0s89tcp3kin0memvexrxu14mc8mesrs4.jpg!content)
> ![chrome](http://ob7zbqpa6.qnssl.com/2f6ofboq6xrq53drt9x1bjioy1hohob7.jpg!content)
>
> 所有的UIViewController的presentation都是由
> UIPresentationController管理的。在UIPresentationController
> 中可以定义content和chrome的动画，可以根据大小的变化来改变content
> 大小，可以根据系统的不同，来改变展示方式，
> UIPresentationController也是可复用的，可以很容易的拿到其他的
> UIViewController中去使用。
> 

## UIPresentationController
UIPresentationController对象为presented view controllers提供了先进的视图和过渡管理系统。从一个view controller被presented的时间一直到它dismiss的时间，UIKit使用一个presentation controller来管理view controller在各个方向的演示过程。presentation controller可以在那些被提供的动画师对象上添加它自己的动画，它可以响应size变化，还可以管理view controller如何被presented到屏幕上的其他方面。

### 概要
当你使用` present(_:animated:completion:)`方法来展示一个view controller的时候，UIKit一直管理这个展示(presentation)的过程。这个过程涉及到为给定的presentation style创建必要的presentation controller。对于内置样式（像pageSheet style），UIKit定义并且创建所需要的presentation Controller对象。你的应用唯一一次可以提供一个自定义的presentation Controller是在设置你的view Controller的modalPresentationStyle属性的时候。在你想要为presented view controller的下面添加一个阴影视图或者装饰视图的时候，或者是当你想要在其他方面修改presentation行为的时候，你可以提供一个自定义的presentation Controller。

通过你视图控制器的` transitioning delegate`来公开发布你的自定义presentation Controller对象。当presented view controller在屏幕上的时候，UIKit会保持对你的presentation Controller对象的引用。想要了解更多关于transitioning delegate和它提供的对象的信息，查阅[UIViewControllerTransitioningDelegate](https://developer.apple.com/reference/uikit/uiviewcontrollertransitioningdelegate)

#### The Presentation Process
被一个presentation controlller管理的the presentation process涉及到三个方面：

* The presentation phase(阶段)涉及通过一系列的transition animations将新的view controller移动到屏幕上。
* The management phase涉及到当新的view controller在屏幕上时响应环境的变化（例如：设备旋转）。
* The dismissal phase涉及到通过一系列的transition animations来将这个新的view controller移除屏幕。

在所有这些阶段中，the presentation controller的角色就是管理它自己自定义的视图和状态信息。在the presentation 和 dismissal phase ，如果可能的话，the presentation Controller向视图层级结构中添加它的自定义视图，并且为这些视图创建任何合适的transition animations。视图控制器的视图添加在屏幕上的动画仍然被一个animator对象管理，这个对象遵循`UIViewControllerAnimatedTransitioning`协议。UIKit在presentation的开始、结束，和dismissal phase的时候单独调用presentation Controller的方法，以便presentation Controller可以执行任何所需要的清理工作。

#### Adding Custom Views to a Presentation
当要presenting一个view controller的时候UIPresentationController class定义了操作视图层级结构的特殊接口。当一个view controller将要被presented的时候，UIKit调用presentation controller的`presentationTransitionWillBegin()`方法，你可以使用这个方法向视图层级结构中添加视图，并且为这些视图设置动画相关。在the presentation phase最后，UIKit调用`presentationTransitionDidEnd(_:)`方法来让你知道`transition 过渡`完成了。

当dismissing view controller的时候，使用` dismissalTransitionWillBegin()`方法来配置任何动画，并且使用` dismissalTransitionDidEnd(_:)`方法来从视图层级中移除任何自定义视图。

#### Adapting to Size Class Changes
size class变化将会大规模的改变你的应用程序应该如何展示它的内容。presentation controller通过为它们的presented controller调整presentation style来管理size class的改变（如果有必要）。这种调整仅仅在当前presentation style在新的环境中不起作用的时候、无意义的时候才进行调整。例如，当size class从水平常规变化到水平紧凑的时候，popover变成一个全屏的presentation，对于在新环境中起作用的presentation style，就像全屏的presentation style，没有自适应变化。

当the presented view controller的size class从水平常规变化到水平紧凑的时候，the presentation controller 调用它`adaptivePresentationStyle`方法来确定哪一个新的style（如果有）应该被应用。当需要一个新的style的时候，the presentation Controller为新的style初始化一个transition，也可能会涉及到将当前的presentation controller对象替换一个新的对象，因为presentation controller对象可能改变了，始终从presented view controller的presentationController属性检索presentation controller。

当presentation style改变的时候，presentation controller也会给你机会来指定一个新的presented view Controller。在初始化transition之前，presentation controller调用它代理对象的`presentationController(_:viewControllerForAdaptivePresentationStyle:)`方法。如果你实现了这个方法，你就可以用它为展示的内容presented content做出或大或小的调整，一个大的调整或许会将当前的view controller替换为一个全新的为特定的size class所特别设计的view Controller。一个小的调整可能会将当前的view Controller加载到一个navigation Controller中来为新的size class生成navigation。

#### Responding to Size Changes
size的改变代表了视图控制器的视图的width或height的小改变。通常，这类改变在设备水平和垂直方向旋转的时候发生。当size改变的时候，UIKit调用presentation Controller的`viewWillTransition(to:with:)`方法。在一个custom presentation中，使用这个方法来修改你的presentation Controller的自定义视图或者改变视图层级结构。例如，你可以更换自定义装饰视图来更好地适应新的size。

在通知你presentation Controller的一个定义的size变化后，UIKit开始正常的布局过程。应用程序使用autolayout而不用其他的，因为autolayout根据需要重新调整大小。但是，如果一个custom presentation Controller需要特殊的布局改变，可以在它的` containerViewWillLayoutSubviews()` 和 ` containerViewDidLayoutSubviews()`方法中这样做。这些方法与UIViewController class的`viewWillLayoutSubviews()` 和 `viewDidLayoutSubviews()`方法相等，同样的使用方法。UIKit在调用视图层级结构中的视图的`layoutSubViews()`方法之前和之后调用这些方法。

#### Subclssing Notes
对于custom presentation style，你应该子类化UIPresentationController并且覆盖重写它的一些方法来实现你的custom pres 行为。使用你的custom presentation Controller来自定义the pres process。

对于custom presentation Controllers，你必须在初始化过程中调用`init(presentedViewController:presenting:)`方法。这个方法是类的设计初始化器（designated initializer），UIKit使用它来为所有的presentation Controller对象执行必须初始化(required initialization)。

#### Methods to Override
如果你的UIPresentationController子类管理了一个或者更多的自定义视图，你应该重写下面的方法：

* 使用` presentationTransitionWillBegin()`方法将你的自定义视图添加到视图层级结构中，并且动画出现的视图。如果view Controller没有被presented，必要的时候可以在` presentationTransitionDidEnd(_:)`方法中移除这些视图。
* 使用` dismissalTransitionWillBegin()`方法来执行和the presented view Controller的dismissal相关的动画。在`dismissalTransitionDidEnd(_:)`方法被调用之前都不要从你的视图层级结构中删除自定义视图。
* 使用`viewWillTransition(to:with:)`方法来对你的自定义视图做出任何size相关的变化。

子类为了提供自定义的行为，如果有必要，必须要覆盖重写这个类的其他方法。例如，你可能需要重写`shouldPresentInFullscreen` 或 ` frameOfPresentedViewInContainerView`方法，并且返回不同的值，而不是提供默认的实现。
