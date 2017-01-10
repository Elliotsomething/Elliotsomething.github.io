---
layout:     post
title:      "iOS之UIImage对象优化"
subtitle:   "UIImage optimization"
date:       2016-3-1
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - UIImage
    - 缓存优化
---

### 一、imageNamed和imageWithContentsOfFile的区别

在iOS开发中生成一个UIImage对象的方法通常有两种

1. 利用`imageNamed`方法
2. 使用`imageWithContentsOfFile`方法

下面介绍这两中方法的区别:

#### imgeNamed

```objective_c
[UIImage imageNamed:@"hearderImage"]
```
使用这个方法生成的`UIImage`对象,会在应用的`bundle`中寻找图片,如果找到则`Cache`到系统缓存中,作为内存的`cache`。
而程序员是无法操作`cache`的,只能由系统自动处理,如果我们需要重复加载一张图片,那这无疑是一种很好的方式，
因为系统能很快的从内存的`cache`找到这张图片。
但是试想，如果加载很多很大的图片的时候，内存消耗过大的时候，就会会强制释放内存，即会遇到内存警告(`memory warnings`)。
由于在iOS系统中释放图片的内存比较麻烦,所以冲易产生内存泄露。

**小结**

`imageNamed`只适合用于小的图片的读取，或重复使用一张图片的时候,而当加载一些比较大的图片文件的时候
我们应当尽量避免使用这个方法.

#### imageWithContentsOfFile

```objective_c
NSString *filePath = [[NSBundle mainBundle] pathForResource:fileName ofType:extension];
    UIImage *image = [UIImage imageWithContentsOfFile:filePath];
```
相比上面的`imageNamed`这个方法要写的代码多了几行,使用`imageWithContentsOfFile`的方式加载的图片，
图片会被系统以数据的方式进行加载,在使用完成之后系统会直接释放,并不会缓存下来，所以一般不会因为加载图片的方法遇到内存问题.

**小结**

当有些图片在应用中只使用比较少的次数的，就可以用这样的方式，相比`imageNamed`会降低内存消耗,避免一些内存问题.

### 二、优化方案
这两种方法都有各自的优缺点，那么有什么办法能够把其优点结合起来呢，答案是`hook`;

利用两个API各自的特点进行`hook`；

首先hook `imageNamed`方法，然后在我们自己的方法中进行如下判断：

1. 定义一个本地缓存图片的最大size
2. 判断本地Bundle中是否存在该图片，不存在直接返回到原`imageNamed`方法，存在则继续
3. 从缓存中直接读取，如果存在该图片，直接返回，否则继续
3. 判断该图片是否大于size，如果大于则调用原`imageWithContentsOfFile`方法返回`UIImage`
4. 如果该图片不大于size则返回，并存入缓存

接下来hook `imageWithContentsOfFile`方法，步骤也是类似，就不多做阐述，直接看代码；

代码如下：

```objective_c
+ (void)startCache{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method orgMethod = class_getClassMethod(self, @selector(imageWithContentsOfFile:));
        Method extMethod = class_getClassMethod(self, @selector(hookImageWithContentsOfFile:));
        method_exchangeImplementations(orgMethod, extMethod);

        Method orgMethod2 = class_getClassMethod(self, @selector(imageNamed:));
        Method extMethod2 = class_getClassMethod(self, @selector(hookImageNamed:));
        method_exchangeImplementations(orgMethod2, extMethod2);

    });
}
+ (UIImage *)hookImageNamed:(NSString *)path{

    if (path.length == 0) {
        return [self hookImageNamed:path];
    }

    NSString *type = @"png";
    if ([path hasSuffix:@".png"]) {
        type = nil;
    }
    NSString *filePath = [[NSBundle mainBundle] pathForResource:path ofType:type];
    if (filePath) {
        return [self quaryImageCacheWithPath:filePath];
    }else{
        NSString *appString = @"@2x";
        if (SCREEN_NATIVE_SCALE > 2) {
            appString = @"@3x";
        }
        if (! type) {
            path = [path stringByReplacingCharactersInRange:NSMakeRange(path.length-4, 4) withString:@""];
            type = @"png";
        }
        filePath = [[NSBundle mainBundle] pathForResource:[path stringByAppendingString:appString] ofType:type];
        if (filePath) {
            return [self quaryImageCacheWithPath:filePath];
        }
        return [self hookImageNamed:path];
    }

    // UIImage *image = [self hookImageWithContentsOfFile:path];
    // return image;
}
+ (UIImage *)hookImageWithContentsOfFile:(NSString *)path{
    return [self quaryImageCacheWithPath:path];

    // UIImage *image = [self hookImageWithContentsOfFile:path];
    // return image;
}

#define kMaxCacheSize (15*1024*1024)
#define kMaxCacheImage (640*640*4)// 位宽一般=4

+ (UIImage *)quaryImageCacheWithPath:(NSString *)path{
    if (path == nil){
        return nil;
    }
    /*
     @"times":
     @"time":
     @"size":
     @"image":
     */

    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        imagesDict = [NSMutableDictionary dictionaryWithCapacity:100];
        mapDateToPath  = [NSMutableDictionary dictionaryWithCapacity:100];
    });
    @synchronized(imagesDict){
        // NSLog(@"debug: cache count:%d", imagesDict.count);
        NSMutableDictionary *config = imagesDict[path];
        if (config){
            NSDate *now = [NSDate date];
            [mapDateToPath removeObjectForKey:config[@"time"]];
            config[@"time"] = now;
            mapDateToPath[now] = path;
            config[@"times"] = @([config[@"times"] integerValue] + 1);
            return config[@"image"];
        }else{
            UIImage *image = [self hookImageWithContentsOfFile:path];
            if (image) {
                NSInteger size = [self cacleImageMemorySize:image];
                if (size < kMaxCacheImage) {// 只缓存 200*200像素以下的图片
                    NSDate *now = [NSDate date];
                    config = [NSMutableDictionary dictionaryWithDictionary:@{@"times":@(1),
                                                                             @"time":now,
                                                                             @"size":@(size),
                                                                             @"image":image
                                                                             }];
                    imagesDict[path] = config;
                    mapDateToPath[now] = path;
                    currentTotolSize += size;
                    if (currentTotolSize > kMaxCacheSize){
                        [self freeCacheToSize:(kMaxCacheSize*0.7)];
                    }
                }
            }
            return image;
        }
    }
}
```
