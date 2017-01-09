---
layout:     post
title:      "学习swift笔记Tips"
subtitle:   "Swift Learning Tips"
date:       2016-8-1
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - swift
    - Tips
---

**学习swift笔记Tips**

该篇文章是学习`swift 2.2`官方文档时，通过和OC对比总结的30个tips；官方swift文档更新后，可能有些地方有一些出入，所以最好还是直接看官方文档；如有不到位的地方，欢迎指正；

**1、** `nil`在`swift`中是一个确定的对象，任何可选对象都可以设置为`nil`；而在oc中`nil`代表一个空指针，表示对象不存在。

**2、** 在`swift`中不允许对`nil`发消息，会`crash`；而`oc`中对`swift`发消息是允许的。

**3、** 在`swift`中集合对象（`array`，`set`，`dictionary`）可以存入基本数据类型（`int`，`float`），而在oc中不允许。（在`swift`中一切皆对象）

**4、** `a ?? b`代表的含义是`a != nil ? a! : b`。

**5、**

```objective_c
As an optimization, Swift may instead capture and store a copy of a value if that
value is not mutated by a closure, and if the value is not mutated after the closure
is created.
Swift also handles all memory management involved in disposing of variables when they
are no longer needed.
```

**6、** 类和结构体，在`swift`中可以直接对类中的结构体的某一个变量赋值。而不像在`oc`中必须对结构体赋值然后才能对结构体中的变量赋值。

```objective_c
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}

let someResolution = Resolution()
let someVideoMode = VideoMode()

someVideoMode.resolution.width = 1280
print("The width is \(someVideoMode.resolution.width)")
// Prints "The width is 1280"
```

**7、** 结构体和枚举是值类型，值类型当它被分配给一个变量或常数，或者当它被传递给一个函数时，它的值被复制。

```objective_c
/*结构体*/
let hd = Resolution(width: 1920, height: 1080)
var cinema = hd
cinema.width = 2048
print("cinema is now \(cinema.width) pixels wide")
// Prints "cinema is now 2048 pixels wide"
print("hd is still \(hd.width) pixels wide")
// Prints "hd is still 1920 pixels wide"

/*枚举*/
enum CompassPoint {
    case North, South, East, West
}
var currentDirection = CompassPoint.West
let rememberedDirection = currentDirection
currentDirection = .East
if rememberedDirection == .West {
    print("The remembered direction is still .West")
}
// Prints "The remembered direction is still .West"
```

**8、** 类是引用类型的，所以只传指针过去，值会跟着一起改变 用`===`判断类的实例是否相等；

**9、** `swift`中不需要用*来创建指针

**10、** 在`swift`中很多基础数据类型，像`string`，`array`，`dictionary`等都是值类型，意味着每次都会被`copy`，而在oc中则是引用类型

**11、** 全局常量和变量总是懒加载，局部常量和变量都不用懒加载；

**12、** 每个属性都会有`willset`和`didset`两个方法，`willset`在属性设置值之前会调用，`didset`在属性设置值之后调用

**13、** 结构体和枚举中的方法如果想改变其内部的属性值，必须在`func`前面加`mutating`关键字；如果加了`mutating`关键字，结构体或枚举不能是常量类型

**14、** 在`swift`中可以在结构体，枚举，类定义类层次的方法，在`oc`中只能在类中定义；类方法的定义在`func`前面加上`static`关键字，而在类中如果你想让子类重写该方法，则将`static`替换成`class`

**15、** 类，结构体，枚举都可以自定义下标，使用`subscript`关键字即可（用法和数组字典一样[]取值）

**16、** `swift`的类可以不需要继承，如果创建一个类不指定父类的话，则自动转为基类（oc必须要继承）

**17、** 父类的属性，方法，下标（`subscript`）以及类方法（非静态），如果不想被子类继承，则必须在关键字前面加上`final`关键字；`private`关键字是保护其不被外部发现，但是子类还是可以继承。(`final`不可和`static`一起使用);

**18、** 当类被定义为`final`时，该类不允许被继承

**19、** 函数的参数分为局部参数和外部参数，默认第一个为局部参数，在调用时可以省略（也可以自己定义为外部参数），后面的参数为外部参数，调用时不可省略，如果想要省略的话将参数名改为下划线_

**20、** 默认的初始化都是`init()`，但是结构体会有一个特殊的初始化方法，（`Memberwise Initializers`）如果自定义初始化方法的话，会自动生成

```objective_c
struct Size {
    var width = 0.0, height = 0.0
}
/*成员逐一初始化*/
let twoByTwo = Size(width: 2.0, height: 2.0)
```

**21、** 类的初始化构造器： 为了简化指定构造器和便利构造器之间的调用关系，`Swift` 采用以下三条规则来限制构造器之间的代理调用：（注意是类初始化，值类型的不一样）指定构造器必须调用其直接父类的的指定构造器。 便利构造器必须调用同一类中定义的其它构造器。 便利构造器必须最终以调用一个指定构造器结束。

**一个更方便记忆的方法是：**

指定构造器必须总是向上代理 便利构造器必须总是横向代理

子类指定构造方法一定要调用父类构造方法，并且必须在子类存储属性初始化之后调用父类构造方法

**22、** 如果你使用闭包来初始化属性的值，请记住在闭包执行时，实例的其它部分都还没有初始化。这意味着你不能够在闭包里访问其它的属性，就算这个属性有默认值也不允许。同样，你也不能使用隐式的`self`属性，或者调用其它的实例方法。

**23、** 引用计数只应用在类的实例。结构体(`Structure`)和枚举类型是值类型，并非引用类型，不是以引用的方式来存储和传递的。

**24、** `Swift`有如下约束：只要在闭包内使用`self`的成员，就要用`self.someProperty`或者`self.someMethod`（而非只是`somePropert`y或`someMethod`）。这可以提醒你可能会不小心就占有了`self`。

**25、** 用类型检查操作符(`is`)来检查一个实例是否属于特定子类型。类型检查操作符返回 `true` 若实例属于那个子类型，若不属于返回 `false` 。用类型转换操作符(`as`)向下转型。 使用`is`和`as`操作还可以检查协议一致性，并转换为一个指定的协议。

**26、** `as`关键字转换没有真的改变实例或它的值。潜在的根本的实例保持不变；只是临时地把它作为它被转换成的类来使用。

**27、** `Swift`为不确定类型提供了两种特殊类型别名：

1. `AnyObject`可以代表任何`class`类型的实例。
2. `Any`可以表示任何类型，除了方法类型（`function types`）。
**注意：只有当你明确的需要它的行为和功能时才使用`Any`和`AnyObject`。在你的代码里使用你期望的明确的类型总是更好的。**

**28、** 扩展就是向一个已有的类、结构体或枚举类型添加新功能（`functionality`）。这包括在没有权限获取原始源代码的情况下扩展类型的能力（即逆向建模）。扩展和 `Objective-C` 中的分类（`categories`）类似。（不过与`Objective-C`不同的是，`Swift` 的扩展没有名字。）

**注意：如果你定义了一个扩展向一个已有类型添加新功能，那么这个新功能对该类型的所有已有实例中都是可用的，即使它们是在你的这个扩展的前面定义的。**

**29、** `Swift` 中的扩展可以：

- 添加计算型属性和计算静态属性
- 定义实例方法和类型方法
- 提供新的构造器
- 定义下标
- 定义和使用新的嵌套类型
- 使一个已有类型符合某个接口

**30、** 在`Swift`语言中，访问修饰符有三种，分别为`private`，`internal`和`public`。同时，`Swift`对于访问权限的控制，不是基于类的，而是基于文件的。其区别如下：

1. `private`: `private`访问级别所修饰的属性或者方法只能在当前的Swift源文件里可以访问。
2. `internal`（默认访问级别，`internal`修饰符可写可不写）: `internal`访问级别所修饰的属性或方法在源代码所在的整个模块都可以访问。 如果是框架或者库代码，则在整个框架内部都可以访问，框架由外部代码所引用时，则不可以访问。 如果是App代码，也是在整个App代码，也是在整个App内部可以访问。
3. `public`: 可以被任何人使用，一般都是第三方库接口给别人调用时才定义的
