---
layout:     post
title:      "Tableview优化实践"
subtitle:   "Tableview Optimization!"
date:       2016-7-20
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - Tableview
    - 优化
---

### Tableview优化实践

这里首先说一个概念，为什么iOS的滑动频率FPS是60。 iOS 的显示系统是由 `VSync` 信号驱动的，`VSync` 信号由硬件时钟生成，每秒钟发出 60 次（这个值取决设备硬件，比如 `iPhone` 真机上通常是 59.97）。iOS 图形服务接收到 `VSync` 信号后，会通过 `IPC` 通知到 `App` `内。App` 的 `Runloop` 在启动后会注册对应的 `CFRunLoopSource` 通过 `mach_port` 接收传过来的时钟信号通知，随后 `Source` 的回调会驱动整个 App 的动画与显示。


**首先交代一下背景：** 列表文字超过5000，图片最多9张，还有一些其他`UILabel`，所以如果不做优化的话，列表卡出翔；

### 优化思路：
1. 高度缓存
2. `cell`内容异步绘制
3. 提前计算滑动`cell`位置，提前绘制，缓存
4. 如果滑动过快，可以只绘制前后3个`cell`，中间的`cell`空白忽略
5. `cell`重用不用说，大家都知道
6. 减少透明图层
7. 减少离屏渲染

到达上面说的基本列表就很流畅了 。

### 具体实施

#### 1、高度缓存

高度缓存是空间换时间，但是性价比很高，计算高度的方法一般是比较耗时的，频繁的计算高度更加耗时，所以高度缓存能以很小的代价换取性能的提升，非常划算；那么怎么缓存高度呢？很简单，给每个`cell`都缓存高度到一个字典或者数组就行了，取值是按照每个`cell`特定的标识去取，必须要保证每个`cell`唯一对应一个值。

```objective_c
static NSMutableDictionary *heightCache;
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{

    if (!heightCache) {
        heightCache = [@{}mutableCopy];
    }
    /*唯一标识*/
    NSString *key = [NSString stringWithFormat:@"%@_%@", cell.sid,cell.svn];

    if (heightCache[key]&&report.svn != 0) {
        return [heightCache[key] doubleValue];
    }

    CGFloat cellHeight = [cellText sizeWithFont:[UIFont systemFontOfSize:16]
    constrainedToSize:CGSizeMake(CELL_CONTENT_WIDTH, 100000) lineBreakMode:NSLineBreakByWordWrapping];
    heightCache[key] = @(cellHeight);

    return cellHeight;
}
```
#### 2、cell内容异步绘制

异步绘制`cell`内容这个是大幅度提升fps的技巧，像微博，朋友圈等这些列表都是异步绘制的，所以这个优化非常重要。
由于UI是在主线程执行的，当在主线程进行耗时操作时就会阻塞主线程，这时UI就会卡顿；而异步绘制是把耗时操作放到后台（子）线程中去进行，当完成之后再将结果返回主线程显示出来，这样就不会阻塞主线程。上述基本思路

**注：异步绘制缺点是，当后台（子）线程没有完成时，界面空白，但是基本在可接受范围内。**

一般情况下绘制是通过coretext将内容绘制在画布`context`上，然后直接生成`image`，返回给主线程，这样做的原因是ios允许`color`，`font`，`image`等这些UI在子线程中操作，（`coretext`和`CoreGraphics`都是线程安全的）

##### 线程实现

线程的实现也是有技巧的，如果只是简单的用`dispatch`将线程放入后台队列，这样做的后果就是如果不断的滑动，并且耗时较长的情况下，会开n个线程，这样每个线程占x内存，很快就会内存警告挂掉。所以如果一定要用`dispatch`的话，建议用多个异步串行队列（我用的5个），这样最多只会开5个线程，不至于内存警告，同时也保证了流畅。

**具体实现如下：**
（更正：队列数是根据当前最大的活跃内核数决定的，`iphone`手机是双核，所以最多为2个队列）

**文档描述：** The number of active processing cores available on the computer. (read-only)

```objective_c
//这里创建多个异步串行队列，按顺序每次返回一个串行队列
dispatch_queue_t YHAsyncViewGetDisplayQueue() {

#define MAX_QUEUE_COUNT 5
    static int queueCount;
    static dispatch_queue_t queues[MAX_QUEUE_COUNT];
    static dispatch_once_t onceToken;
    static int32_t counter = 0;

    dispatch_once(&onceToken, ^{
        /*获取当前活跃的内核数，iphone手机是双核，所以最多为2个队列*/
        queueCount = (int)[NSProcessInfo processInfo].activeProcessorCount;
        queueCount = queueCount < 1 ? 1 : queueCount > MAX_QUEUE_COUNT ? MAX_QUEUE_COUNT : queueCount;
        if ([UIDevice currentDevice].systemVersion.floatValue >= 8.0) {
            for (NSUInteger i = 0; i < queueCount; i++) {
                dispatch_queue_attr_t attr = dispatch_queue_attr_make_with_qos_class(DISPATCH_QUEUE_SERIAL, QOS_CLASS_USER_INITIATED, 0);
                queues[i] = dispatch_queue_create("com.yh.sanfor.MOA.render", attr);
            }
        } else {
            for (NSUInteger i = 0; i < queueCount; i++) {
                queues[i] = dispatch_queue_create("com.yh.sanfor.MOA.render", DISPATCH_QUEUE_SERIAL);
                dispatch_set_target_queue(queues[i], dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0));
            }
        }
    });
    int32_t cur = OSAtomicIncrement32(&counter);
    if (cur < 0) cur = -cur;
    return queues[(cur) % queueCount];
#undef MAX_QUEUE_COUNT

}

dispatch_async(YHAsyncViewGetDisplayQueue(), ^{
    //coretext draw view;
    dispatch_async(dispatch_get_main_queue(), ^{
    //back main thread;
    };
};
```

还有要注意的地方就是取消任务了，`dispatch`是没有取消`block`的用法，只能自己去判断

```objective_c
NSInteger drawFlag;//实例变量

- (void)cancleBlock{
    drawFlag = arc4random();
}

- (void)asynDisplay{
NSInteger flag = drawFlag
dispatch_async(YHAsyncViewGetDisplayQueue(), ^{

    //coretext draw view;
    if (drawFlag!=flag) return;
    dispatch_async(dispatch_get_main_queue(), ^{
    if (drawFlag!=flag) return;
    //back main thread;
    };
};

}
```


如果用`NSOperasion`就比较简单了，直接可以设置最大线程数，以及是否取消任务。

#### 3、提前计算滑动cell位置，提前绘制，缓存

当手指滑动列表时，是可以提前知道列表会滑动到哪里，这样就可以提前计算出要显示的`cell`。然后提前绘制并缓存起来，当`cell`展示时，直接读缓存，如果缓存没有再进行异步绘制。（我这里只提前绘制前后两个`cell`） 代码如下：

```objective_c

static NSMutableDictionary ContentImgCache;

- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView{
    [ContentImgCache removeAllObjects];
}

/*提前渲染 cell的内容*/
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset{


    if (!ContentImgCache) {
        ContentImgCache = [@{}mutableCopy];
    }
//这里是计算出列表停下时，将要展示的cell
        NSArray *temp = [self.tableView indexPathsForRowsInRect:CGRectMake(0, targetContentOffset->y, self.tableView.width, self.tableView.height)];

        NSMutableArray *arr = [NSMutableArray arrayWithArray:temp];

        if (velocity.y<0) {
            NSIndexPath *indexPath = [temp lastObject];
            if (indexPath.section == 0) {
                return;
            }
            if (indexPath.row+2<dataCount) {

                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row+1 inSection:indexPath.section]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row+2 inSection:indexPath.section]];

            }
        } else {
            NSIndexPath *indexPath = [temp firstObject];
            if (indexPath.section == 0) {
                return;
            }
            if (indexPath.row>2) {
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-1 inSection:indexPath.section]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-2 inSection:indexPath.section]];
            }

        }

        for (NSIndexPath * indexPath in arr) {
            if (indexPath.section == 0) {
                continue;
            }

            UIImage *img = //asyn coretext draw

            NSString *key = [NSString stringWithFormat:@"%@_%@_ContentImg", cell.sid,cell.svn];
            ContentImgCache[key] = img;

        }

//  }
}
```


#### 4、如果滑动过快，可以只绘制前后3个cell，中间的cell空白忽略

如果滑动很快的，每个`cell`都会提交一个任务去后台线程，如果不取消不需要的任务的话，很快会内存警告挂掉，虽然用上述方法最多5个线程，但是会造成其他线程等待的结果；如果取消不需要的任务，毕竟已经提交过任务了，还是会造成不必要的开销。所以最好是直接不提交任务，只提交需要绘制的任务。（当然这是在滑动很快的情况下才使用，一般情况下不需要这样）。 代码如下：

```objective_c

//清空needLoadArr数组，判断如果为空也不进行该操作
- (void)scrollViewWillBeginDragging:(UIScrollView *)scrollView{
    [needLoadArr removeAllObjects];
}

//按需加载 - 如果目标行与当前行相差超过指定行数，只在目标滚动范围的前后指定3行加载。
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset{
    NSIndexPath *ip = [self indexPathForRowAtPoint:CGPointMake(0, targetContentOffset->y)];
    NSIndexPath *cip = [[self indexPathsForVisibleRows] firstObject];
    //滑动的距离如果超过8个才进行该操作
    NSInteger skipCount = 8;
    if (labs(cip.row-ip.row)>skipCount) {
        NSArray *temp = [self indexPathsForRowsInRect:CGRectMake(0, targetContentOffset->y, self.width, self.height)];
        NSMutableArray *arr = [NSMutableArray arrayWithArray:temp];
        if (velocity.y<0) {
            NSIndexPath *indexPath = [temp lastObject];
            if (indexPath.row+3<datas.count) {
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row+1 inSection:0]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row+2 inSection:0]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row+3 inSection:0]];
            }
        } else {
            NSIndexPath *indexPath = [temp firstObject];
            if (indexPath.row>3) {
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-3 inSection:0]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-2 inSection:0]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-1 inSection:0]];
            }
        }
        //需要绘制的cell，如果cell不在needLoadArr中则不绘制
        [needLoadArr addObjectsFromArray:arr];
    }
}
```

#### 5、cell重用，忽略
大家都知道，这里就不介绍了！

#### 6、减少透明图层

透明图层对渲染性能会有一定的影响,因为混合(`blending`)是渲染中最慢的操作,系统会将透明图层与下面的视图混合起来计算并绘制图层属性,减少透明图层并使用不透明的图层来替代它们,在不透明的视图里标明 opaque 属性以避免无用的 `Alpha` 通道合成,可以大量提高 GPU 的计算速度. 可以通过 `instrument` 的 `Core Animation` 中开启 `Color Blended Layers` ,然后红色的部分就是我们需要重点消灭的区域.

#### 7、减少离屏渲染

`CALayer` 的 `border`、圆角、阴影、遮罩(`mask`),`CAShapeLayer` 的矢量图形显示,通常会触发离屏渲染(`offscreen rendering`).与之相对的是当前屏幕渲染(`On-Screen Rendering`),指的是渲染操作是用于在当前屏幕显示的缓冲区进行.离屏渲染的概念来自于 `OpenGL` 中 `GPU` 渲染屏幕的两种方式： `On-Screen Rendering`（当前屏幕渲染）`Off-Screen Rendering`（离屏渲染）。 指的是：在当前屏幕以外新开辟一个缓冲区进行渲染操作。离屏渲染造成卡顿的原因是：离屏渲染需要多次切换上下文环境，先是从当前屏幕(`On-Screen`)切换到离屏(`Off-Screen`)，等到离屏渲染结束以后,将离屏缓冲区的渲染结果显示到屏幕上又需要将上下文环境从离屏切换到当前屏幕,而上下文环境的切换是一项高开销的动作。 通常我们会一个 `view` 设置阴影会使用 `shadowoffset`

```objective_c
UIView *diamondView = [[UIView alloc] init];
diamondView.layer.shadowOffset = CGSizeMake(1.0f, 1.0f);
diamondView.layer.shadowRadius = 5.0f;
diamondView.layer.shadowOpacity = 0.5;
```

但是这种方式会触发离屏渲染造成不必要的开销,那么既要实现阴影图层,又要减少离屏渲染,提高性能的话.有什么更好的方式么?

```objective_c
UIView *diamondView = [[UIView alloc] init];
diamondView.layer.shadowPath = [UIBezierPath bezierPathWithRect:CGRectMake(diamondView.bounds.origin.x + 1,
diamondView.bounds.origin.y + 1, diamondView.bounds.size.width, diamondView.bounds.size.height)].CGPath;
imageView.layer.shadowOpacity = 0.5;
```

但是 `shadowPath` 只适用于给规则的矩形生成阴影路径.如果我们迫不得已要使用 `shadowoffset`,可以尝试开启 `CALayer`.`shouldRasterize` 属性, 图像将会被缓存起来并绘制到实际图层的 `contents` 和子图层.将原本在 `GPU` 中的一些工作让 `CPU` 来做,让两者达到一个平衡.但是这并不是有一个全优解,因为光栅化原始图像需要时间,而且会消耗额外的内存.所以一定要避免在内容不断变动的图层上使用,不然缓存的优势将荡然无存. 最完美的解决方案是使用 `Core Graphics` 绘制圆角 `UIImage` 设置给 `UIImageView` 然后插入到 `UIView` 中去。

### 完！
