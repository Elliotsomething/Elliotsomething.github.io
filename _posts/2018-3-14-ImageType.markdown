---
layout:     post
title:      "iOS 之 图片数据类型"
subtitle:   "Image Type"
date:       2018-3-14
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - 图片
---

**版权声明：本文为博主原创文章，未经博主允许不得转载**

#### iOS 之 图片数据类型

通过二进制数据判断图片类型

一般来说图片的二进制数据都是用来判断类型的，比如，jpeg、png、webp等。

比如：二进制数据的前1个字节：
0xFF为jpeg类型
0x89为png类型
0x47为gif类型
0x49、0x4D为tiff类型
0x52为webp类型

通过查看图片的数据类型如下：

<img src="https://Elliotsomething.GitHub.io/images/imagetype-01.png">
<img src="https://Elliotsomething.GitHub.io/images/imagetype-02.png">

通过图片的二进制流判断图片类型的代码如下：
```objective_c
+ (NSString *)sd_contentTypeForImageData:(NSData *)data {
    uint8_t c;
    [data getBytes:&c length:1];
    switch (c) {
        case 0xFF:
            return @"image/jpeg";
        case 0x89:
            return @"image/png";
        case 0x47:
            return @"image/gif";
        case 0x49:
        case 0x4D:
            return @"image/tiff";
        case 0x52:
            // R as RIFF for WEBP
            if ([data length] < 12) {
                return nil;
            }
            NSString *testString = [[NSString alloc] initWithData:[data subdataWithRange:NSMakeRange(0, 12)] encoding:NSASCIIStringEncoding];
            if ([testString hasPrefix:@"RIFF"] && [testString hasSuffix:@"WEBP"]) {
                return @"image/webp";
            }
            return nil;
    }
    return nil;
}

```
