---
layout:     post
title:      "iOS之webView加载文件 及 文件操作"
subtitle:   ""
date:       2015-3-13
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - webView
    - 文件操作
---

这几天在做`webView`浏览附件的功能，研究了一下，正好可以记下来，用`webView`可以打开各种附件（`.doc .pdf` 等）

### `webView`加载文件

**加载路径的第一个方式**

```objective_c
NSString *path1 = [[NSBundle mainBundle] pathForResource:@"文件名" ofType:nil];
NSURL *url = [NSURL fileURLWithPath:path1];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
[self.WebView loadRequest:request];
```

**加载路径的第二个方式**

```objective_c
NSURL *url = [[NSBundle mainBundle] URLForResource:@"文件名" withExtension:nil];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
[self.WebView loadRequest:request];
```

加载路径的第三个方式，以二进制数据流加载，`webview`加载本地文件，可以使用加载数据的方式
- 第一个诶参数是一个`NSData`， 本地文件对应的数据
- 第二个参数是`MIMEType`
- 第三个参数是编码格式
- 相对地址，一般加载本地文件不使用，可以在指定的`baseURL`中查找相关文件。
- 以二进制数据的形式加载沙箱中的文件，
- 加载`.doc`文件 `TYPE`为`application/vnd.openxmlformats-officedocument.wordprocessingml.document`

```objective_c
NSData *data = [NSData dataWithContentsOfFile:path];
[self.webView loadData:data MIMEType:@"application/vnd.openxmlformats-officedocument.wordprocessingml.document" textEncodingName:@"UTF-8" baseURL:nil];  
```

步骤基本一样的，都是先获取文件所在路径，然后转化成`URL`，然后`webView`加载这个URL请求。
第三种方法有点特别就是他要获取文件的类型，这个可以让服务器一起发送过来。

**怎么让webView缩放：**

```objective_c
[self.WebView setScalesPageToFit:YES];
```

### 文件操作

默认情况下，iOS 的沙盒下都有三个文件夹，功能基本如下：

1. Documents：苹果建议将程序创建产生的文件以及应用浏览产生的文件数据保存在该目录下，iTunes备份和恢复的时候会包括此目录
2. Library：存储程序的默认设置或其它状态信息；Library/Caches：存放缓存文件，保存应用的持久化数据，用于应用升级或者应用关闭后的数据保存，不会被itunes同步，所以为了减少同步的时间，可以考虑将一些比较大的文件而又不需要备份的文件放到这个目录下。
3. tmp：提供一个即时创建临时文件的地方，但不需要持久化，在应用关闭后，该目录下的数据将删除，也可能系统在程序不运行的时候清除。

**获取沙盒的根路径：**

```objective_c
NSString *dirHome=NSHomeDirectory();
NSLog(@"app_home: %@",dirHome);
```
**获取document文件夹目录：**

```objective_c
//document文件目录
NSString *pathDocument=[ NSSearchPathForDirectoriesInDomains ( NSDocumentDirectory , NSUserDomainMask , YES ) objectAtIndex : 0 ];
```
**获取library目录：**

```objective_c
//library文件目录
NSString *pathLib=[ NSSearchPathForDirectoriesInDomains ( NSLibraryDirectory , NSUserDomainMask , YES ) objectAtIndex : 0 ];
```
**获取cache目录：**

```objective_c
cache文件目录
NSString *pathCache=[ NSSearchPathForDirectoriesInDomains ( NSCachesDirectory , NSUserDomainMask , YES ) objectAtIndex : 0 ];
```
**获取temp目录：**

```objective_c
NSString *pathTemp = NSTemporaryDirectory();
```
获取了路径了，以上的文件夹都是默认存在的，接下来是自己创建自己的文件夹，文件夹创建，删除，读写操作等都需要 `NSFileManager` 这个单例类支持：
以`temp`目录为例，在`temp`目录下建立`attachment`文件夹：

```objective_c
NSFileManager *fileManager = [NSFileManager defaultManager];
//temp文件目录
NSString *path1 = NSTemporaryDirectory();
path1=[path1 stringByAppendingPathComponent : @"attachment" ]; //拼接路径 temp/attachment
[fileManager createDirectoryAtPath:path1 withIntermediateDirectories:YES attributes:nil error:nil]; //创建 文件夹
[objc] view plain copy
path1=[path1 stringByAppendingPathComponent : @"我的文件.doc"];  //再次拼接路径
[fileManager createFileAtPath:path1 contents:nil attributes:nil];  //创建  文件
```
**判断文件是否存在：**

```objective_c
[fileManager fileExistsAtPath:path1]
```
