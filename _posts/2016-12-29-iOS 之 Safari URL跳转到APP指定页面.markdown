---
layout:     post
title:      "iOS 之 Safari Or Web URL跳转到APP指定界面"
subtitle:   "Safari URL Schemes PushViewController"
date:       2016-12-29
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - URL
    - Schemes
---

**版权声明：本文为博主原创文章，未经博主允许不得转载；如需转载，请保持原文链接。**

### iOS 之 Safari Or Web URL跳转到APP指定页面

有时候需要从文章中分享一个链接直接跳到APP的指定页面，这个怎么做呢?

其实很简单；只要是`iphone`自带的浏览器，都会自带一个识别APP的`URL Schemes`的功能，所以只要是特定的URL，都能被web识别，所以不管是在项目内部还是在`Safari`中，都可以实现URL跳转到指定页面的需求、

只要是符合特定格式的URL，都会跳入指定的APP中，然后再加上一些参数，即可实现从APP中跳入指定的页面

**注：如果是其他APP内部的web页面，则不会进入跳转；那么有一个办法是引导用户分享到Safari去，然后再触发URL Schemes跳转**

我的格式为：`"url schemes"://"Bundle id"/mod="viewContrllerID"?id="params"`

比如`testUrlSchemes://com.testBundleID.YH/mod=appstore?id="params"`

这种方式还可以做更多的事，比如用作`Route`等等；这里只是做一个简述，更多精彩等大家去实现！！！

步骤如下：

首先进入APP是会进入这个回调方法中的：

```objective_c
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    BOOL ret = [ExtensionHandle handleOpenURL:url];
    if (ret) {
        return ret;
    }else{
        //handle other share URL
    }
}
```

接下来在`ExtensionHandle` 的 `handleOpenURL:`中处理改URL及参数；

```objective_c
#define URL_SCHEME_PREFIX     @"URL Schemes://" //这个是 URL Schemes
+ (BOOL)handleOpenURL:(NSURL *)url{
    BOOL ret = [url.absoluteString hasPrefix:URL_SCHEME_PREFIX];
    if (ret) {
        NSString *bundelID = [[[NSBundle mainBundle] bundleIdentifier] lowercaseString];
        if ([[url.absoluteString lowercaseString] rangeOfString:bundelID].location!=NSNotFound) {
            NSArray *urlComponents = [url.absoluteString componentsSeparatedByString:[NSString stringWithFormat:@"%@/",bundelID]];
            NSString *query = urlComponents.lastObject;

            /*analyze  params*/
            NSDictionary *params = [self analyzeQuery:query];

            if ([params[@"mod"] length] != 0) {
                NSString *modStr =  params[@"mod"];
                if (url.query) {
                    modStr = [modStr stringByAppendingString:url.query];
                }
                [PageTransferHelper appJumpToPageWithModString:modStr];
            }
        }
    }
    return ret;
}
```

然后是解析参数出来，其实就是对字符串处理，分离处后面的参数

```objective_c
+(NSDictionary *)analyzeQuery:(NSString *)query
{
    NSMutableDictionary *keyValues = [NSMutableDictionary dictionary];
    NSArray *queryArray = [query componentsSeparatedByString:@"&"];
    for (NSString *oneParma in queryArray) {
        NSArray *keyValue = [oneParma componentsSeparatedByString:@"="];
        if (keyValue.count >=2) {
            keyValues[keyValue[0]] = keyValue[1];
        }
    }
    return keyValues;
}
```

然后在`PageTransferHelper`的`appJumpToPageWithModString:`方法中处理界面跳转；

首先是处理参数中要跳转的类以及参数分离；

注：讲道理，`string == appstore?id="params"`

```objective_c
+ (void)appJumpToPageWithModString:(NSString *)string{
    if (string.length==0) {
        return;
    }
    /*URL 解码*/
    string = [string stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    NSArray* paragamArr = [string componentsSeparatedByString:@"?id="];
    classKey = [paragamArr firstObject];
    //handle params
    id obj = params;

    NSInteger type;
    NSDictionary *dict = [self pageTransferStringDict];
    for (NSNumber *key in [dict allKeys]) {
        if ([dict[key] isEqualToString:classKey]) {
            type = [key integerValue];
            break;
        }
    }
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [self appJumpToPage:type andParagam:obj];
    });
}
```

这里配置要跳转的界面

```objective_c
+ (NSDictionary*)pageTransferStringDict
{
    return @{@(ePageTransferTypeAppStore):@"appstore"};
}
+ (NSDictionary*)pageTransferClassDict
{
    return @{@(ePageTransferTypeAppStore):[AppStoreViewController class]};
}
```

接下来是处理真正的跳转界面以及传参数

注：`type == ePageTransferTypeAppStore;
		 obj == "params";`

```objective_c
+ (void)appJumpToPage:(NSInteger)type andParagam:(id)obj,...{
    [AppDelegate dismissAllViewController];
    AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
    Class class = [self pageTransferClassDict][@(type)];
    [appDelegate.rootViewController setSelectedIndex:0];
    UIViewController *viewController = [[class alloc]init];
    viewController.params = obj;
    [appDelegate.rootViewController.navigationController pushViewController:viewController animated:NO];
}
```

在web中点击URL之后的回调是：

```objective_c
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;
```

接下来的解析URL跟上面一样
