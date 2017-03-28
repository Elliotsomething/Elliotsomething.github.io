---
layout:     post
title:      "iOS日志上传很简单(二)搭建HTTPServer升级篇"
subtitle:   "LogFile Upload Is Very Simple(2) -- Build A Simple HTTP Server"
date:       2017-2-28
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - HTTP Server
    - CFNetwork
    - KVO
---
**版权声明：本文为博主原创文章，未经博主允许不得转载；如需转载，请保持原文链接。**

### iOS日志上传很简单(二)搭建HTTPServer升级篇

上一篇介绍了一下怎么搭建一个简易的HttpServer，也就是直接在客户端上通过IP加上端口号直接访问APP的沙箱文件内容；
没看过的可以移步看一下[搭建简易HTTPServer](https://elliotsomething.github.io/2017/02/25/%E6%97%A5%E5%BF%97%E4%B8%8A%E4%BC%A0%E5%BE%88%E7%AE%80%E5%8D%95(%E4%B8%80)%E6%90%AD%E5%BB%BA%E7%AE%80%E6%98%93%E7%9A%84HTTP%E6%9C%8D%E5%8A%A1%E5%99%A8/);

#### 概述

这一篇作为进阶篇，也就是在上一篇的基础上加一些复杂的逻辑，比如文件的上传、下载、删除等；

实现的大概思路很简单，只是实现稍微复杂

1. 首先客户端发起请求，也就是IP地址加端口号（这个在上一篇已经讲过了）
2. 服务端返回响应数据，这个上一篇基本也讲了，不同的是服务端返回的数据是一个网页，其中包含APP的沙箱文件列表，以及一些操作按钮
3. 客户端点击操作按钮，比如下载、删除等
4. 服务端接收数据，处理客户端请求，比如method=upload，并返回响应数据

基本就是这样一个实现思路，其中比较难处理的是第二步，和第四步；不过整体思路理解了还是不难实现的，只是时间问题而已

#### 具体实现

首先第一步客户端发起请求，这个忽略；我们从第二步开始讲起；

**服务端返回网页数据**
服务端返回数据，这个在上一篇的响应数据已经讲了，接下来只要把响应的数据替换为网页数据即可；

代码如下：

```objective_C
static const NSString *html = @"<!DOCTYPE html PUBLIC \"-//W3C//DTD XHTML 1.0 Transitional//EN\" \"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd\">\r\n<html xmlns=\"http://www.w3.org/1999/xhtml\">\r\n<head>\r\n<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\" />\r\n<title>文件浏览器</title>\r\n</head>\r\n\r\n<body>\r\n**text**\r\n<table width=\"790\" height=\"30\" border=\"1\" align=\"center\">\r\n  <tr>\r\n    <td width=\"450\" height=\"30\"><div align=\"center\">名称</div></td>\r\n    <td width=\"160\"><div align=\"center\">编辑时间</div></td>\r\n    <td width=\"80\"><div align=\"center\">大小</div></td>\r\n    <td width=\"100\"><div align=\"center\">操作</div></td>\r\n</tr>\r\n**table**\r\n</table>\r\n</body>\r\n</html>";
```
上面的代码是返回一个基本的网页，这种简单的html就不讲了；

但是只是返回一个基本网页是不够的，我们还需要在网页里面能够点击操作，所以我加了一个简单的form表单，然后加了上传、替换、下载、删除四个提交表单方法；

代码如下

```objective_c
NSString *path = [currentPath stringByAppendingPathComponent:name];
NSString *sizeStr = [fileType isEqualToString:NSFileTypeRegular]? sizeDescBlock([info[NSFileSize] longLongValue]): @"";
NSString *delUrl = [NSString stringWithFormat:@"%@?function=delete", path];
NSString *downUrl = [NSString stringWithFormat:@"%@?function=download", path];
static const NSString *oprHtml = @"<a href=\"%@\">%@</a>";
static const NSString *html = @"<tr>\r\n    <td height=\"30\"><a href=\"%@\">%@</a></td>\r\n    <td><div align=\"center\">%@</div></td>\r\n    <td><div align=\"center\">%@</div></td>\r\n    <td><div align=\"center\">**opr**</div></td>\r\n</tr>";
NSMutableArray *opr = [NSMutableArray array];
if([fileType isEqualToString:NSFileTypeRegular]) {
	[opr addObject:[NSString stringWithFormat:(NSString *)oprHtml, delUrl, @"删除"]];
} else {
	[opr addObject:[NSString stringWithFormat:(NSString *)oprHtml, downUrl, @"下载"]];
	[opr addObject:[NSString stringWithFormat:(NSString *)oprHtml, delUrl, @"删除"]];
}
NSString *oprStr = [opr componentsJoinedByString:@" / "];
NSString *htmlStr = [html stringByReplacingOccurrencesOfString:@"**opr**" withString:oprStr];
NSString *retStr = [NSString stringWithFormat:htmlStr, path, name, dateStr, sizeStr];

static const NSString *upload = @"<form action=\"%@\" method=\"post\" enctype =\"multipart/form-data\" runat=\"server\"> \r\n<div align=\"center\">\r\n<input id=\"file\" runat=\"server\" name=\"uploadfile\" type=\"file\" /> \r\n<input type=\"submit\" name=\"upload\" value=\"%@\" id=\"upload\" />\r\n</form>\r\n<form action=\"%@\" method=\"post\" enctype =\"multipart/form-data\" runat=\"server\"> \r\n<input id=\"file\" runat=\"server\" name=\"replace\" type=\"file\" /> \r\n<input type=\"submit\" name=\"replace\" value=\"%@\" id=\"replace\" />\r\n</div>\r\n</form>\r\n";
NSString *uploadUrl = [NSString stringWithFormat:@"%@?function=upload", currentPath];
NSString *replaceUrl = [NSString stringWithFormat:@"%@?function=replace", currentPath];
NSString *uploadForm = [NSString stringWithFormat:(NSString *)upload, uploadUrl, @"上传文件", replaceUrl, @"替换目录"];

NSString *str = [content componentsJoinedByString:@"\r\n"];
str = [html stringByReplacingOccurrencesOfString:@"**table**" withString:str];
```
上面的代码就是组合四个form提交表单方法，基本的html知识即可看懂，这里也不细讲；当我们把这些数据返回给客户端之后，基本这一步就完成了；

**客户端点击操作按钮，发起下载，删除等请求**

客户端点击操作按钮，发起请求，其实就是html的表单提交，只要懂html就能理解，我们在上一步的时候已经把这个一步的步骤都完成了，HTTPServer返回的html表单包含了4个方法，只需要在客户端点击相应的方法发起请求即可；

**服务端处理下载、删除的请求，返回响应数据**

HTTPServer处理请求，返回响应数据，这里同样也是和前面一样，只是稍微加几个if的条件判断，然后分别处理不同的请求，然后返回不同的响应数据就搞定了

首先我们需要分别定义四个处理表单请求的方法

```objective_c
selectorForMethod = @{@"GET": @{@"download": NSStringFromSelector(@selector(dealDownloadFunction:andClientHandle:)),
@"delete": NSStringFromSelector(@selector(dealDeleteFunction:andClientHandle:)),},
@"POST": @{@"replace": NSStringFromSelector(@selector(dealReplaceFunction:andClientHandle:)),
@"upload": NSStringFromSelector(@selector(dealUploadFunction:andClientHandle:)),}
};
```
然后这四个方法分别处理对用的下载、删除、替换、上传这四个请求，基本的HTTPServer就此完成了；由于代码有点多，这里就不贴出来了，还有一些具体的实现细节大家可以去github上下载demo自己看；基本的实现细节就讲到这里，由于水平不够，可能有很多地方没讲清楚，大家如果有不懂的可以直接提问

#### 结束

这一篇是进阶篇，基本就是模拟了HTTP的服务器处理请求数据并返回响应数据；只要理解了整个思路，基本自己就慢慢实现出来，这里只讲了一些大概的实现，具体的实现细节大家可以去看[demo](https://github.com/Elliotsomething/HTTPServerDemo)（别忘记star一下哈）
