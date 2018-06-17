---
layout:     post
title:      "iOS 11 contentoffset 坑"
subtitle:   "iOS 11 contentoffset"
date:       2017-11-25
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - contentoffset
    - iOS 11
---

**版权声明：本文皆摘抄自网络，仅用于学习参考；如有侵权，请随时联系。**

#### iOS 11 contentoffset 坑

发现在iOS11系统上tableView动画有异常，在其他系统的设备上都是正常的，动画的操作是观察tableView的contentOffset变化后执行的，异常动画发生在tableView reloadData之后，也就是说tableView reloadData之后，tableView的contentOffset发生了几次变化。查了下资料发现原因是iOS11中默认开启了Self-Sizing。

Self-Sizing在iOS11下是默认开启的，Headers, footers, and cells都默认开启Self-Sizing，所有estimated 高度默认值从iOS11之前的 0 改变为UITableViewAutomaticDimension：
```objective_c
@property (nonatomic) CGFloat estimatedRowHeight NS_AVAILABLE_IOS(7_0); // default is UITableViewAutomaticDimension, set to 0 to disable
```

如果目前项目中没有使用estimateRowHeight属性，在iOS11的环境下就要注意了，因为开启Self-Sizing之后，tableView是使用estimateRowHeight属性的，这样就会造成contentSize和contentOffset值的变化，如果是有动画是观察这两个属性的变化进行的，就会造成动画的异常，因为在估算行高机制下，contentSize的值是一点点地变化更新的，所有cell显示完后才是最终的contentSize值。因为不会缓存正确的行高，tableView reloadData的时候，会重新计算contentSize，就有可能会引起contentOffset的变化。
iOS11下不想使用Self-Sizing的话，可以通过以下方式关闭：（前言中提到的问题也是通过这种方式解决的）
```objective_c
tableview.estimatedRowHeight = 0;
tableview.estimatedSectionHeaderHeight = 0;
tableview.estimatedSectionFooterHeight = 0;
```

iOS11下，如果没有设置estimateRowHeight的值，也没有设置rowHeight的值，那contentSize计算初始值是 44 * cell的个数，如下图：rowHeight和estimateRowHeight都是默认值UITableViewAutomaticDimension 而rowNum = 15；则初始contentSize = 44 * 15 = 660；

[image]
<img src="https://Elliotsomething.GitHub.io/images/iOS11ContentOffset-01.png">
