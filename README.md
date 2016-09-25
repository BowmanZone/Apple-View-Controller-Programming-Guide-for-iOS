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

#Preserving and Restoring State
视图控制器在状态保存和恢复过程中扮演了一个很重要的角色。状态保存记录你的应用程序在被暂停之前的配置，以便在后续应用程序启动的时候能够快速恢复。返回应用程序之前的配置状态可以为用户节约时间，并且提供更好地用户体验。

这个保存和恢复的过程大多是自动完成的，但是你需要告诉iOS，你应用程序的哪一部分需要被保存。保存你的应用程序视图控制器步骤如下：

* （必须的）为你想要保存的视图控制器定义恢复标记 restoration identifiers。see `Tagging view Controllers for Preservation`
* （必须的）在启动的时候告诉iOS怎样创建或者加载加载试图控制对象。see `Restoring View Controllers at Launch Time`
* （可选的）为每一个视图控制器存储任何需要的特殊配置数据，返回视图控制器原来的配置。see `Encoding and Decoding Your View Controller's State`

查看存储和恢复的过程，参照 `App Programming Guide for iOS`.

##Tagging View Controllers for Preservation
UIKit只会存储那些你告诉它要存储的视图控制器。每一个视图控制器都有一个`restorationIdentifier` 属性，这个属性的值默认是 `nil`。设置一个可用的字符串给这个属性，告诉UIKit这个视图控制器和它的视图应该被存储。你可以在storyboard文件中，或者通过代码形式来定义 `restoration identifier`。

当定义 `restoration identifier`的时候，在视图控制器视图层级结构中的所有视图控制器也必须有 `restoration identifier`(想要存储的那个视图控制器所在的视图层级结构中的视图控制器都必须要配置这个属性)。在存储的过程中，UIKit从window的 root view Controller开始遍历视图控制器视图层级结构。如果在这个视图层级结构中的视图控制器没有一个 `restoration identifier`，这个视图控制器和所有它的子视图控制器、父视图控制器会被忽略。（看下图）

### Choosing Effective Restoration Identifiers
UIKit稍后会使用你的 `resotration identifier`字符串来重新创建视图控制器，因此选择简单的可标记的字符串到你的代码中。如果UIKit不能自动创建你的视图控制器，它要求你来创建，通过为你提供视图控制器,以及它的父视图控制器的 `restoration identifier`。这一串identifiers展示了视图控制器的 `restoration path`，并且展示了你是如何定义被要求保存的视图控制器。这个 `restoration path`从根视图控制器开始，并且包括向上（视图层级）的每一个视图控制器，还包括这个需要被保存的试图控制器。

`restoration identifier`通常就是视图控制器的类名。如果你在很多地方使用相同的类，你可能想要定义为更有意义的值。例如，你可以定义一个基于视图控制器管理的数据字符串。

每一个视图控制器的`restoration path`必须是唯一的。如果一个容器视图控制器有两个子控制器，这个容器必须为每一个子视图控制器定义一个唯一的 `restoration identifier`。一些UIKit中的视图控制器自动地消除了它们子视图控制器的歧义，允许你对每个子视图控制器使用相同的 `restoration identifier`。例如，UINavigationController类为每一个子视图控制器添加信息就是基于它们在导航栈中位置。想要了解更多视图控制器的行为，查看相应的类接口。
###Excluding Groups of View Controller
为了在恢复过程中不包含一整组的视图控制器，设置父视图控制器的 `restoration identifier`为 `nil`。下图展示了设置 `restoration identifier`为 `nil`对视图控制器视图层级结构的影响。缺少保存数据会阻止视图控制器稍后被存储。

7-1：

![自动保存过程不包括视图控制器7-1](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/state_vc_caveats_2x.png)

不包含一个或者多个视图控制器在稍后的恢复过程中不会将它们一起移除。在启动的时候，你的应用程序默认建立的部分视图控制器仍然会被创建，见下图。这些视图控制器依然会使用默认配置来被重新创建，但是它们仍然被创建了。

7-2：

![加载一组默认视图控制器](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/state_vc_caveats_2_2x.png)

在自动保存过程中不包含一个视图控制器不会阻止你手动地保存它。在恢复 视图控制器和它的状态信息保存的归档过程中保留视图控制器的引用。例如，如果7-1的中应用程序的代理保存了三个导航控制器的子视图控制器，它们的状态会被保存。在恢复过程中，应用程序代理会重新创建这些视图控制器，并且把它们推入导航控制器的栈中。

###Preserving a View Controller's Views
一些视图有额外的状态信息,这些信息与视图相关，与父视图控制器无关。例如，scroll view有一个你可能想要保存的滑动位置。当视图控制器负责提供scroll view的内容的时候，scroll view本身就负责保存它自己的可视内容。

为了保存一个视图的状态，按照下面这样做：

* 为视图的 `restorationIdentifier` 属性定义一个可用的字符串。
* 使用视图控制器中的视图，这样有一个可用的 `restoration identifier`。
* 对于表单视图和集合视图，定义一个遵循 `UIDataSourceModelAssociation`协议的数据源。

为视图定义一个 `restoration identifier`，告诉UIKit，它应该把视图的状态写入到保存归档中。当视图控制器稍后被恢复的时候，UIKit也会恢复任何拥有 `restoration identifier` 的视图的状态。

##Restoring View Controllers at Launch Time
在启动的时候，UIKit尝试恢复你的应用程序之前的状态。在这个时候，UIKit告诉你的应用程序创建（或者定位）视图控制器对象来组成你保存的用户界面。当你尝试定位视图控制器的时候，UIKit按照下面的顺序进行搜索：

此图来自<http://useyourloaf.com/blog/state-preservation-and-restoration/>网站,里面还有相关例子上传到Github上。

![流程图](http://useyourloaf.com/assets/images/2013/StateRestoration.png)

1. 如果视图控制器有一个 `restoration class`，UIKit要求这个类提供视图控制器。
	
	UIKit调用相关的 `restoration class` 的`viewControllerWithRestorationIdentifierPath:coder:` 方法来重新得到视图控制器。如果这个方法返回 `nil`，它就会假设应用程序不想重新创建视图控制器，并且UIKit停止查找它。
	
2. 如果视图控制器没有一个 `restoration class`，UIKit要求应用程序的代理提供这个视图控制器。
	
	UIKit调用你的应用程序代理的 ` application:viewControllerWithRestorationIdentifierPath:coder:` 方法来查找没有 `restoration class` 的视图控制器。如果这个方法返回 `nil`，UIKit会尝试隐式地查找视图控制器。
	
3. 如果视图控制器已经存在正确的 `restoration path`，UIKit会使用这个对象。
	如果你的应用程序在启动的时候创建视图控制器（不管是代码形式或是通过一个storyboard加载它们），并且这些视图控制器有 `restoration identifiers`，UIKit会基于它们的 `restoration paths`隐式地找到它们。
	
4. 如果视图控制器是最初从storyboard文件加载，UIKit使用存储的storyboard信息来定位并且创建它们。
	UIKit把视图控制器的storyboard信息保存在恢复归档中。在恢复的时候，如果视图控制器没有被其他方式发现，UIKit使用这些信息来定位相同的storyboard文件，并且实例化相应的视图控制器。

为视图控制器定义一个 `restoration clss` 阻止UIKit隐式地搜索视图控制器。使用 `restoration class`给你更多的控制而不是是否你真的想要创建一个视图控制器。例如，你的类决定这个视图控制器不应该被重新创建，你的` viewControllerWithRestorationIdentifierPath:coder: `方法返回一个 `nil`。当这里没有 `restoration class`的时候，UIKIt会做它能够做的一切来找到或者创建一个视图控制器，并恢复它。

当使用 `restoration class`的时候，你的 `viewControllerWithRestorationIdentifierPath:coder:`方法应该创建一个新的类实例，完成最小化的初始化，并且返回结果对象。下面展示了一个例子，你如何使用这个方法来从一个storyboard中加载一个视图控制器。因为这个视图控制器原本就是从一个storyboard加载的，这个方法使用 `UIStateRestorationViewControllerStoryboardKey` 键来从归档中得到storyboard。注意这个方法不会尝试配置视图控制器的数据信息，这一步会在稍后视图控制器的状态被解码的时候发生。

	 + (UIViewController*) viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder {
  
	MyViewController* vc;

	UIStoryboard* sb = [coder decodeObjectForKey:UIStateRestorationViewControllerStoryboardKey];
   
	if (sb) {
   
	vc = (PushViewController*)[sb instantiateViewControllerWithIdentifier:@"MyViewController"];
      
	vc.restorationIdentifier = [identifierComponents lastObject];
      
	vc.restorationClass = [MyViewController class];
      
	}
   
	return vc;
    
	}

当手动创建视图控制器的时候，重新分配 `restoration identifier` 和 `restoration class` 是一个应该养成的好习惯。恢复 `restoration identifier`最简单的方式就是获取 `identifierComponents`数组中的最后一项，并且把它定义给你的视图控制器。

在启动的时候，被你的应用程序的 `main storyboard` 文件创建的对象不会创建任何对象的新实例。让UIKit隐式地找到这些对象或者使用你的应用程序的代理的 `application:viewControllerWithRestorationIdentifierPath:coder:` 方法来找到存在的对象。

##Encoding and Decoding Your View Controller's State
计划为每一个对象保存，UIKit调用对象的 `encodeRestorableStateWithCoder:` 方法来给它机会保存它的状态。在恢复的过程中，UIKit调用匹配的 `decodeRestorableStateWithCoder: ` 方法来解码状态，并且把它应用到对象。这些方法的实现是可选的，但是对你的视图控制器建议使用这些方法。你可能使用这些方法来保存和恢复下面的类型信息：

* 引用任何需要显示的数据（不是数据本身）
* 对于容器类视图控制器，引用它们的子视图控制器
* 当前选择的信息
* 对于用户可配置视图的视图控制器，关于这个视图的当前配置信息

在你的编码和解码方法中，你可以编码对象和任何支持编码的数据类型。对于除了视图和视图控制器的所有对象，对象必须遵循 `NSCoding协议` 并且使用协议方法来编写它的状态。对于视图和视图控制器，编码不需要使用 `NSCoding协议` 来保存对象的状态 。相反，编码保存对象的 `restoration identifier`,并且把它添加到可保存的对象列表中，这是因为对象的 `encodeRestorableStateWithCoder:`方法会被调用。

视图控制器的 `encodeRestorableStateWithCoder:` 和 ` decodeRestorableStateWithCoder:` 方法必须在它的实现中调用 `super`。调用 `super` 给父类有机会保存和恢复任何额外的信息。下面展示了这些方法的简单实现，保存一个数字值来标记特殊的视图控制器。

	- (void)encodeRestorableStateWithCoder:(NSCoder *)coder {
		[super encodeRestorableStateWithCoder:coder];
 
		[coder encodeInt:self.number forKey:MyViewControllerNumber];
	}
 
	- (void)decodeRestorableStateWithCoder:(NSCoder *)coder {
		[super decodeRestorableStateWithCoder:coder];
 
		self.number = [coder decodeIntForKey:MyViewControllerNumber];
	}

在编码和解码的过程中，代码对象不会被分享。每一个有可保存状态的对象接收它自己的代码对象。使用唯一的代码意味着你不需要担心命名空间和你的键冲突。但是，不要使用下面这些键来命名：
	
	UIApplicationStateRestorationBundleVersionKey, 这个键是一个`NSString`对象，当状态信息被保存的时候，标记了你当前的应用程序的版本（和你的应用程序中的`info.plist`文件中的`CFBundleVersion key`所包含的一样）。你可以使用使用这个键的值来决定在状态恢复过程怎么进行下去。例如，如果这个状态表示应用程序是一个老版本，你可能想要避免恢复之前的状态或者更为显著地修改恢复过程。
	
	UIApplicationStateRestorationUserInterfaceIdiomKey, 这个键的值是一个`NSNumber`对象,包含了`UIUserInterfaceIdiom`枚举中的一个值。这个值反映了被保存的界面是iPad，还是iPhone。	
	
	UIStateRestorationViewControllerStoryboardKey，这个键的值是一个`UIStoryboard`对象，代表一个storyboard，它包含了一个被初始化的视图控制器。在状态保存过程中，每一个视图控制器自动地把这个键写入到编辑器（coder）中。
	
这些键被UIKIt使用来保存视图控制器的额外状态信息。

查看更多视图控制器实现的编码和解码方法，查看 UIViewController Class Reference

##Tips for Saving and Restoring Your View Controllers
当你在视图控制器中添加了支持状态保存和恢复功能，考虑下面的指导：

* 记住你不会想要保存所有的视图控制器。有时候，保存一个视图控制器是没有意义的，如果应用程序正在显示一个变化，你可能想要取消功能，并且将应用程序恢复到之前的状态。例如，你不会保存视图控制器来询问新的密码信息。
* 在恢复过程中避免交换视图控制器类。状态保存系统编码它保存的视图控制器类。在恢复过程中，如果identifier应用程序返回一个对象类，这个类不匹配（或者不是一个子类）它原来的类，系统不会要求视图控制器来解码任何状态信息。因此，交换一个旧的视图控制器和一个完全不一样的类不会保存对象完整的状态。
* 状态保存系统希望你有目的的使用视图控制器。恢复过程依赖视图控制器的控制关系来构建界面。如果没有正确使用容器类视图控制器，保存系统不会发现你的视图控制器。例如，不要把一个视图控制器的视图嵌套到另一个不同的视图中，除非相应的控制器之间有一个包含关系。


#UIPresentationController

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
