---
layout:     post
title:      "iOS 之 使用@synchronized加锁的坑"
subtitle:   "error_of_using_synchronized"
date:       2017-6-23
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - synchronized
    - Lock
---

**版权声明：本文为博主原创文章，未经博主允许不得转载；如需转载，请保持原文链接。**


### 使用@synchronized加锁的坑

首先让我们来复习一下`@synchronized`的内部实现

首先一个简单的测试代码

```objective_c
- (void) testddd{
	@synchronized (arr) {

	}
}
```
然后看下汇编实现：

<img src="https://Elliotsomething.GitHub.io/images/error_of_using_synchronized1.png">

可以看到`synchronized`会对加锁的对象进行`retain`，但是这个不是重点，关键看`objc_sync_enter`和`objc_sync_exit`这两个函数，去源码里面找找：

```objective_c
// Begin synchronizing on 'obj'.
// Allocates recursive mutex associated with 'obj' if needed.
// Returns OBJC_SYNC_SUCCESS once lock is acquired.  
int objc_sync_enter(id obj)
{
    int result = OBJC_SYNC_SUCCESS;

    if (obj) {
        SyncData* data = id2data(obj, ACQUIRE);
        assert(data);
        data->mutex.lock();
    } else {
        // @synchronized(nil) does nothing
        if (DebugNilSync) {
            _objc_inform("NIL SYNC DEBUG: @synchronized(nil); set a breakpoint on objc_sync_nil to debug");
        }
        objc_sync_nil();
    }
    return result;
}

// End synchronizing on 'obj'.
// Returns OBJC_SYNC_SUCCESS or OBJC_SYNC_NOT_OWNING_THREAD_ERROR
int objc_sync_exit(id obj)
{
    int result = OBJC_SYNC_SUCCESS;
    if (obj) {
        SyncData* data = id2data(obj, RELEASE);
        if (!data) {
            result = OBJC_SYNC_NOT_OWNING_THREAD_ERROR;
        } else {
            bool okay = data->mutex.tryUnlock();
            if (!okay) {
                result = OBJC_SYNC_NOT_OWNING_THREAD_ERROR;
            }
        }
    } else {
        // @synchronized(nil) does nothing
    }
    return result;
}
```
可以看出:

1、`synchronized`是使用的递归mutex来做同步。`@synchronized(nil)`不起任何作用（这样就避免了多次嵌套@synchronized导致死锁）

2、只对同一对象加锁，如果对象不同，则不起作用，这样就会有个问题，如果我在`@synchronized`里面对该对象重新赋值了，其他线程还是会进来，不会起到同步阻塞作用；

ok，看到这里基本可以知道我要说的问题了，就不继续深入了；

回到主题上，首先来看下问题代码，我把该代码简化了：
看代码：

```objective_c
	for (int i = 0; i < 10000; i++) {
		dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
		[self testddd];
		});
	}

- (void)testddd{
	if (selectorAndClassArray.count==0) {
		NSMutableArray *emergencyArray = [NSMutableArray array];
		selectorAndClassArray = emergencyArray;

	}
}
```
上面的代码，这种多线程赋值操作很容易出现野指针Crash，我加了10000次循环，几乎野指针Crash是必现的，具体是因为对象的多次释放导致的，这里不细讲，可以去看下这篇博客；

那么该问题处理办法就是加锁了，重点来了

```objective_c
- (void) testddd{
	if (selectorAndClassArray.count==0) {
		NSMutableArray *emergencyArray = [NSMutableArray array];
		@synchronized (selectorAndClassArray) {
			selectorAndClassArray = emergencyArray;

		}
	}
}
```
我这样加锁之后，发现并没有什么用，还是会出现野指针Crash，只能慢慢调试了，下断点在`@synchronized`中，发现有很多线程还是进来了，如下图：
<img src="https://Elliotsomething.GitHub.io/images/error_of_using_synchronized2.png">
<img src="https://Elliotsomething.GitHub.io/images/error_of_using_synchronized3.png">

回到我们开头所讲，发现原来如此，因为`@synchronized`只会锁住同一对象，而`@synchronized`内部改变之前标记的对象，所以就相当于没有加锁一样其他线程都可以进去了，这样就达不到阻塞同步代码的作用，还是和上面没加锁的代码一样会引起野指针Crash；

找到原因之后就好修改了，直接使用`NSLock`就行，代码如下：
```objective_c
- (void) testddd{
	if (selectorAndClassArray.count==0) {
		NSMutableArray *emergencyArray = [NSMutableArray array];
		[self.lock lock];
		selectorAndClassArray = emergencyArray;
		[self.lock unlock];
	}
}
```

总结：需要慎用`@synchronized`，在加锁的场景下要考虑清楚该用什么锁，不然一不留神就给自己挖了个坑；
本文完；
