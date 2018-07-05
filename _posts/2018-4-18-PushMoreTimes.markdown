---
layout:     post
title:      "iOS 之 防止多次push同一个页面"
subtitle:   "Push Page More Times"
date:       2018-4-18
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
---

**版权声明：本文为博主原创文章，未经博主允许不得转载**

#### iOS 之 防止多次push同一个页面

场景：快速多次点击cell跳转到另一个页面，另一个页面被push多次。

原因：push后的页面有耗时操作或者刚好push到另一个页面时，另一个页面正好在reloadData卡住主线程。造成点击cell时卡住了。

解决方法：重写导航控制器的push方法。

```objective_c
#import "DemoNavViewController.h"  
@interface DemoNavViewController () <UINavigationControllerDelegate>  
// 记录push标志  
@property (nonatomic, getter=isPushing) BOOL pushing;  
@end  
@implementation DemoNavViewController  
- (void)viewDidLoad {  
    [super viewDidLoad];  

    self.delegate = self;  
}  
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated  
{  
    if (self.pushing == YES) {  
        NSLog(@"被拦截");  
        return;  
    } else {  
        NSLog(@"push");  
        self.pushing = YES;  
    }  
    [super pushViewController:viewController animated:animated];  
}    
#pragma mark - UINavigationControllerDelegate  
-(void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animated  
{  
    self.pushing = NO;  
}    
@end  
```
