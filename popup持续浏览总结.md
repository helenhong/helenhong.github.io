##一、About UI 关于UI

K歌的UI是通过代码搭建起来的，区别于以往的使用Storyboard依赖于Autolayout还原视觉设计的经验。因此，在搭建视图的时候，要注意：

####1.UIView 层级
要清楚其superviews以及其subviews,因为frame的数值是相对于superview的。
####2.UI 初始化和刷新
初始化的各个属性都要确定好，尽可能减少对外开放刷新UI的interface，保证简洁性。在创建popup页底部信息栏KSContintueBrowsingBottomView的合唱样式时，复用了详情页已有的KSTimelineChorusView。因为实际需要展示的内容略有区别，且减少代码变动，所以是新添加了刷新UI的接口。5.0版本详情页改版，需要对KSTimelienChorusView的UI进行改版，可以考虑使用工厂模式展现样式上类似，但是内容上有区别的UIView。

[此处有图，待更新]

其中用户展示信息的KSContinueBrowsingBottomView有两种样式，用于播放视频的view有9:16和1:1样式。因此，在成功切换视频后，需要重新绘制视图。绘制底部信息栏时，不仅要考虑内部布局，也考虑frame变化导致的和superview或者是同层级的视图之间的布局关系。

##二、Data 数据处理
####1.获取网络数据
#####协议 protocol
使用公司的维纳斯(WNS)网络通道，实现https数据收发和二进制数据收发。定义好收发数据使用的请求cmd，请求参数和处理数据回包的block。request和response都使用JCE协议进行编解码。目前项目里，有使用KSProtocolBase直接，也有定义KSProtocolBase的子类封装request和response收发数据的方式，不够规范化。
#####服务 services
使用不同的manager(都是单例)定义接口，用于处理不同场景下单业务数据收发。统一收归在Class/Services目录下。
#####模型 model
由于返回的数据类型是遵循JCE协议的模型，把数据抛给业务处理Class的时候要最好是转换成Objective-C类型。

[此处有图，待更新]
####2.使用Delegate处理数据
popup浏览页的视图层级分KSVideoChannelVC,KSVideoChannelCell和镶嵌在Cell里的其他subviews,消息传递通定义的各种Delegate protocol，把作用在其他subviews的事件发送给KSVideoChannelCell,再由Cell委托控制器KSVideoChannel去完成。

#####设计思路：
这样做的目的是tableView里的subviews只负责处理UI和接受事件，处理数据的逻辑就交给了Controller。因为NavigationController能处理跳转以及维护Controller对象里全局的datasource。

另一种设计方法是给subviews的接口传递参数，对象自己处理数据，把处理结果抛给Controller。这种设计方法通常应用在APP里和特定Controller业务相关性不强的、复用率高的面板、浮层、弹窗等，比如送礼、评论、分享、充值提示等。保证类的封装性，能提高后续版本的开发和维护效率，也能保证代码质量。

###3.使用NSNotification
发送通知处理数据的方式通常应用在APP内与全局性相关性较强的业务，比如用户信息。或者是两个跳转路径相距较远的类的消息互通，比如动态Feed里显示的礼物鲜花数据和在其他地方进入的详情页之间的消息互通。要注意：NSNotification的observer收到post出来的消息说同步执行的，在执行对变量的读写前，要对模型的生命周期有清晰的认识，确定没有后续异步操作（网络请求回调，缓存回调、异步线程）对指针指向的同一个对象重复赋值，确定读取数据时获取到的信息的准确性。

后面部分有刷新送礼和送花数据针对这部分的理解。

##三、Design and Policy 设计和策略
###1.封装性
尽可能的减少Delegate、头文件中定义的方法和属性，把对外的属性设置为可读，以传递的参数定义在初始化方法里。提高类的重用性，降低类之间的耦合程度。类里定义的全局变量、宏定义也要注意其作用域，popup浏览方式里关于连击送花操作的数据处理这里定义了较多的全局变量，消息传递方式做的不够好，后续可以优化。
###2.保护逻辑
保护逻辑最基本的是要对可能造成crash的地方进行判断处理。特别是要注意越界、nil、或者是状态切换时的边界情况。
###3.Debug
去验证代码功能和定位问题的时候，debug能验证数据的准确。通过操作路径一步一步查看不同对象的数据处理情况，明确BUG产生的时机。比如请求网络数据的过程中，1.可能request的对象有数据缺失，造成的服务器返回异常；2.可能是转化数据模型的过程中，变量精确性丢失，或转化缺失字段；3.有可能是服务器返回数据缺失；4.或模型生命周期中有其他操作导致数值变动。

##四、坑
####表情解析：
KSDrawItemEmotion（继承于KSDrawItem）使用的GIF动图通过url获取的，初始化KSDrawItemEmotion默认enableAnimation=YES使用动态表情。在绘制表情的时候会先加载本地默认静态图片,再根据enableAnimation的布尔值确定是否要加载动图。目前的动态表情带有锯齿，白色背景下显示正常。建议其他背景色的表情使用静态表情。
####礼物榜单刷新：
在送礼成功的回调里马上请求后台礼物榜单的数据是送礼成功前的数据。因此做了一个2s延迟后再去请求数据，保证数据的准确性。送礼面板赠送礼物或者鲜花成功后，调用Delegate方法并发送NSNotification。由于此时存在单例类型的详情页对象，并且监听了这个通知，所以会对礼物数据进行操作先赋值缓存，在赋值网络请求回来后的数据。因为popup浏览的指针和详情页的指针指向同样的礼物对象，会反复给礼物对象执行赋值操作，导致变量值变化。
####多手势处理：
popup视频区域可以响应单击，连击，横向滑动动作。为了能够响应连击事件，使用了touchesEnded:withEvent:API。但是，有个地方要特别注意：从参数touches里获取到在触屏事件为长按（longPress）事件时，tapCount=0，所以区分单击，连击和长按要做成三个判断分支，tapCount=0,tapCount =1和tapCount >1的区分。
####横竖屏切换：
KSBaseViewController和KSongBaseScrollVIewController在初始化UITableView的时候设置autoresizingMask = UIViewAutoresizingFlexibleHeight（枚举类型），这个属性会让UITableView自动根据父视图调整自身高度。当viewController屏幕旋转后frame发生变化，此时调用UITableView的reloadData方法会计算UITableViewCell的高度，若cell的高度与viewController的frame相关，则计算失准，导致UITableView的contentSize失准和contentOffset错位。

解决：autoresizingMask 重新设置回 系统默认的UIViewAutoresizingNone，子视图的frame则不会受父视图变化的影响。如果转屏后调用reloadData方法是不可避免的，则要重新设计计算UITableViewCell高度的方法。

##五、其他
####新版loading动画：
老的loading动画只是简单的使用系统样式，视觉效果不好，所以在4.5版本后期增加了视频缓冲时的loading动画。在videoChannelCell初始化progressView的时候作为subView放在一个视图层级里。在KSVideoChannel(Playback)监听到播放器状态变化的时候，调用startLoadingAnimation去展示动画和stopLoadingAnimation隐藏动画。
####工具和流程：
1.版本维护使用svn，mac下使用cornerstone，后台协议也在svn上更新；日常交流使用rtx、企业微信；后台协议转JCE使用在线jce_tool；数据上报在腾讯罗盘里观察；

2.流程：需求评审、开发、体验（可能需求变动）、测试（改BUG)、灰度测试、发布。需求、缺陷需要在tapd上svn上关联到代码。





















