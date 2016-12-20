---
layout:     post
title:      "尝试XcodeExtension"
subtitle:   "XcodeExtension Learning!"
date:       2016-12-19
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - XcodeExtension
    - Xcode8
---

**版权声明：本文为博主原创文章，未经博主允许不得转载；如需转载，请保持原文链接。**

### 尝试实现Xcode的Extension扩展

本文旨在学习如何实现一个XcodeExtension扩展

其实网上有大神出了博客，直接参考大神的博客就可以弄出来

**[从头构建你的第一个 Xcode 扩展](https://xcoder.tips/build-your-first-xcode-extension/)**
**[Xcode 插件集：xTextHandler](https://zhuanlan.zhihu.com/p/21380769)**

但是出于想要折腾的心理，还是自己再造个轮子，其实代码都挺简单的，官方开放的接口不多，很容易上手

**一个插件能做下面的事情：**

1. 获取 Xcode 正在编辑的文本
2. 获取所有的选中区域
3. 替换 Xcode 正在编辑的文本
4. 选中 Xcode 正在编辑的文本
5. 在 Xcode 的 Editor 菜单里面给你的插件生成一个子菜单，用于调用插件
6. 可以在 Xcode 的 Key Binding 里面给插件分配一个快捷键


**注：Extension必须使用Xcode8以上；**

### 动手写一个Extension

#### 创建Extension
打开`Xcode8`，新建一个工程选择`Cocoa application`

<img src="https://Elliotsomething.GitHub.io/images/post-XcodeExtension-01.png">

填写工程名称：`StrHandleExtension`，去掉 `test` 相关选项

<img src="https://Elliotsomething.GitHub.io/images/post-XcodeExtension-02.png">

然后选中工程文件，新建一个 `Target`，从工程模板中选择 `Xcode Source Editor Extension`，填写 `Target` 名为：`StrHandle`

<img src="https://Elliotsomething.GitHub.io/images/post-XcodeExtension-03.png">

这个时候`Xcode`会自动生成两个类`SourceEditorExtension`和`SourceEditorCommand`
此时的目录结构是这样的：

<img src="https://Elliotsomething.GitHub.io/images/post-XcodeExtension-04.png">

这个时候，其实这个`Extension`已经可以运行了；在运行对象框中选中 `StrHandle` 对象，`Command R `运行，它需要附属到一个应用程序上，请注意选择 `Xcode 8`。运行后，在新起的灰色的 `Xcode 8` 中打开一个项目，在代码编辑器中，打开 `Editor` 菜单，不出意外的可以看到我们的新扩展程序的菜单了


但是有一点坑的是，运行的target必须要有signing Team

<img src="https://Elliotsomething.GitHub.io/images/post-XcodeExtension-05.png">

不然`Extension`是不会加载的；

**注：如果你运行的是 EI Capitan 系统，请在运行此扩展前在终端中输入以下命令并重启电脑：`sudo /usr/libexec/xpccachectl`**

#### 细节实现

两个类都继承自`NSObject`；

**1、`SourceEditorExtension`服从`XCSourceEditorExtension`协议，这个协议有两个方法**

```objective-c
//Extension启动成功后调用
- (void)extensionDidFinishLaunching
{
    // If your extension needs to do any work at launch, implement this optional method.
}
//返回数组配置command，可以配置多个；
- (NSArray <NSDictionary <XCSourceEditorCommandDefinitionKey, id> *> *)commandDefinitions
{
    return @[];
}
```

**第二个方法可以直接在Plist里面配置：**

<img src="https://Elliotsomething.GitHub.io/images/post-XcodeExtension-06.png">

`XCSourceEditorCommandDefinitions`是一个数组，定义了一系列这个扩展程序支持的命令，每个命令包含一个 `Identifier`， 一个入口类名，及命令名，我们可以将命令名修改为自己喜欢的名字，我将名字改为`StrHandle1`，并新加了一个item命名为`StrHandle2`;

**2、`SourceEditorCommand`服从`XCSourceEditorCommand`协议**

主要的逻辑在`XCSourceEditorCommand`类中，当插件被触发之后，你有机会在代理方法里面拦截到这个消息（`XCSourceEditorCommandInvocation`），做出处理之后将内容返回给 Xcode；

```objective-c
- (void)performCommandWithInvocation:(XCSourceEditorCommandInvocation *)invocation completionHandler:(void (^)(NSError * _Nullable nilOrError))completionHandler
{
    // Implement your command here, invoking the completion handler when done. Pass it nil on success, and an NSError on failure.

    completionHandler(nil);
}
```

`invocation(XCSourceEditorCommandInvocation)`，它提供了源代码文件的文本内容相关的信息，对它的修改将直接生效
`completionHandler`，这是一个回调 Block，用于通知 Xcode，扩展程序的操作已经完成


`XCSourceEditorCommandInvocation` 里面会存放一些 meta 数据，其中最重要的是 `identifier` 和 `buffer`，`identifier` 就是用来做区分的，`buffer` 则是整个插件中最最重要的概念


XCSourceTextBuffer 最重要的环节有两个，

```objective-c
/** The lines of text in the buffer, including line endings. Line breaks within a single buffer are expected to be consistent. Adding a "line" that itself contains line breaks will actually modify the array as well, changing its count, such that each line added is a separate element. */
@property (readonly, strong) NSMutableArray <NSString *> *lines;

/** The text selections in the buffer; an empty range represents an insertion point. Modifying the lines of text in the buffer will automatically update the selections to match. */
@property (readonly, strong) NSMutableArray <XCSourceTextRange *> *selections;
```

分别表示了传递过来的文本，以及选中区域。所以要做的事情已经一目了然了，流程如下：

1. 在菜单或快捷键触发插件
2. `XCSourceEditorCommand` 拦截了消息
3. 从 `invocation `中拿到 `buffer`
4. 在 `buffer` 中根据当前行，获取到你要的数据，把数据替换后塞回去

为了方便操作，我们可以为扩展命令添加键盘绑定。如下图，在 `Xcode` 中打开偏好，在 `Key Bindings` 下找到我们的新菜单命令，添加快捷键：`Control Shift S`

<img src="https://Elliotsomething.GitHub.io/images/post-XcodeExtension-07.png">

####具体逻辑

这里就可以写自己想要实现的功能了，我这里写的是字符串大写或小写替换功能；

**代码比较简单，就不放到github上去了**

```objective_c
typedef id(*_IMP) (id, SEL, ...);

- (void)performCommandWithInvocation:(XCSourceEditorCommandInvocation *)invocation completionHandler:(void (^)(NSError * _Nullable nilOrError))completionHandler
{
	NSString *identiifer = invocation.commandIdentifier;

	if ([identiifer isEqualToString:@"uppercaseString"])
	{
		[self uppercaseStringHandle:invocation];
	}
	else if ([identiifer isEqualToString:@"lowercaseString"])
	{
		[self lowercaseStringHandle:invocation];
	}else
	{
		[self capitalizedStringHandle:invocation];
	}

    completionHandler(nil);
}

- (void)uppercaseStringHandle:(XCSourceEditorCommandInvocation *)invocation{
	[self handleSelector:@selector(uppercaseStringWithLocale:) withObject:invocation];
}
- (void)lowercaseStringHandle:(XCSourceEditorCommandInvocation *)invocation{
	[self handleSelector:@selector(lowercaseStringWithLocale:) withObject:invocation];
}
- (void)capitalizedStringHandle:(XCSourceEditorCommandInvocation *)invocation{
	[self handleSelector:@selector(capitalizedStringWithLocale:) withObject:invocation];
}

- (void)handleSelector:(SEL)selector withObject:(XCSourceEditorCommandInvocation *)invocation{
	Method m2 = class_getInstanceMethod([NSString class], selector);
	_IMP imp = (_IMP)method_getImplementation(m2);

	for (XCSourceTextRange *range in invocation.buffer.selections) {
		for (NSInteger line = range.start.line; line <= range.end.line; line++) {
			NSString *text = invocation.buffer.lines[line];
			if (line == range.start.line && line == range.end.line){
				NSString *subText = [text substringWithRange:NSMakeRange(range.start.column, range.end.column-range.start.column)];
				subText = imp(subText,selector,[NSLocale currentLocale]);
				text = [text stringByReplacingCharactersInRange:NSMakeRange(range.start.column, range.end.column-range.start.column) withString:subText];
			}
			else if (line == range.start.line)
			{
				NSString *subText = [text substringFromIndex:range.start.column];
				subText = imp(subText,selector,[NSLocale currentLocale]);
				text = [text stringByReplacingCharactersInRange:NSMakeRange(range.start.column, text.length-range.start.column) withString:subText];
			}
			else if (line == range.end.line)
			{
				NSString *subText = [text substringToIndex:range.end.column];
				subText = imp(subText,selector,[NSLocale currentLocale]);
				text = [text stringByReplacingCharactersInRange:NSMakeRange(0, range.end.column) withString:subText];
			}
			else
			{
				text = [text uppercaseStringWithLocale:[NSLocale currentLocale]];
			}

			invocation.buffer.lines[line] = text;

		}
	}
}

```
