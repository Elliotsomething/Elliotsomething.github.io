---
layout:     post
title:      "CoreFoundation 学习笔记"
subtitle:   "CoreFoundation Note"
date:       2017-5-16
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - CoreFoundation
    - Note
---
**版权声明：本文为博主原创文章，未经博主允许不得转载；如需转载，请保持原文链接。**

### CoreFoundation 学习笔记

本文仅做个人学习用

### CFStringRef 入门

```
#ifdef __CONSTANT_CFSTRINGS__
#define CFSTR(cStr)  ((CFStringRef) __builtin___CFStringMakeConstantString ("" cStr ""))
#else
#define CFSTR(cStr)  __CFStringMakeConstantString("" cStr "")
#endif
```

#### 使用CFSTR()快速创建CFStringRef

```objective_c
CFSTR("yes")
```
#### 使用CFShowStr()打印CFString的详细信息

```objective_c
/*
Use CFShowStr() to printf detailed info about a CFString
*/
CFShowStr(CFSTR("yes"));
```
#### CFStringRef 转 char *

```objective_c
CFStringRef aClassStr = CFSTR("BSBacktraceLogger");
const char *ccc = CFStringGetCStringPtr(aClassStr, kCFStringEncodingUTF8);
```
#### char * 转 CFStringRef

```objective_c
char * firstName_c = "BSBacktraceLogger";
CFStringRef tmpFirstName =CFStringCreateWithCString(NULL, firstName_c, kCFStringEncodingUTF8);
```


### CFArrayRef 学习

#### 使用 CFArrayCreate() 创建CFArrayRef

代码例子：

```
int main(int argc, const char * argv[]) {
    const void *values[] = {CFSTR("A"),CFSTR("B"),CFSTR("C")};
    CFArrayRef array = CFArrayCreate(kCFAllocatorDefault, values, 3, &kCFTypeArrayCallBacks);
    CFShow(array);
    CFRelease(array);
    return 0;
}
```

其他用法：

```
CFArrayRef myCFArrayRef = NULL;
CFStringRef strs[2];
strs[0] = CFSTR("BSBacktraceLogger");
strs[1] = CFSTR("banana");
myCFArrayRef = CFArrayCreate(NULL,(void *)strs,2,&kCFTypeArrayCallBacks);
CFStringRef key = CFSTR("apple");
CFRange myRange = CFRangeMake (0,1);
if (CFArrayContainsValue (myCFArrayRef,myRange,key)){
	printf("YES");
}else{
	printf("NO");
}
CFShow(myCFArrayRef);
CFStringRef aClassStr = CFArrayGetValueAtIndex(myCFArrayRef, 0);
```

#### CFArrayRef 入门

代码例子：

```
CFIndex CFArrayGetLastIndexOfValue(CFArrayRef array, CFRange range, const void *value) {
    CFIndex idx;
    __CFGenericValidateType(array, __kCFArrayTypeID);
    __CFArrayValidateRange(array, range, __PRETTY_FUNCTION__);
    CHECK_FOR_MUTATION(array);
    const CFArrayCallBacks *cb = CF_IS_OBJC(CFArrayGetTypeID(), array) ? &kCFTypeArrayCallBacks : __CFArrayGetCallBacks(array);
    for (idx = range.length; idx--;) {
	const void *item = CFArrayGetValueAtIndex(array, range.location + idx);
	if (value == item || (cb->equal && INVOKE_CALLBACK2(cb->equal, value, item)))
	    return idx + range.location;
    }
    return kCFNotFound;
}
```

#### CFMutableArray 入门

代码例子：

```
int main(int argc, const char * argv[]) {
    CFMutableArrayRef array;
    array = CFArrayCreateMutable(kCFAllocatorDefault,
                                 0,
                                 &kCFTypeArrayCallBacks);
    CFArrayAppendValue(array, CFSTR("A"));
    CFArrayAppendValue(array, CFSTR("B"));
    CFArrayAppendValue(array, CFSTR("C"));
    CFShow(CFSTR("print all"));
    CFShow(array);
    CFShow(CFSTR("Remove Value At index 1"));
    CFArrayRemoveValueAtIndex(array, 1);
    CFShow(array);
    CFShow(CFSTR("Remove All Value"));
    CFArrayRemoveAllValues(array);
    CFShow(array);
    return 0;
}
```

### CFNumber 学习

代码例子：

```
int main(int argc, const char * argv[]) {
    printf("CFIndex value \n");
    CFIndex value = 123;
    CFNumberRef number = CFNumberCreate(kCFAllocatorDefault, kCFNumberCFIndexType, &value);
    printf(" number type => %ld \n", CFNumberGetType(number));
    printf(" byteSize => %ld \n", CFNumberGetByteSize(number));
    printf(" isFloatType => %d \n", CFNumberIsFloatType(number));
    NSInteger getValue;
    CFNumberGetValue(number, kCFNumberCFIndexType, &getValue);
    printf(" getValue => %ld \n",getValue);
    printf("Float value \n");
    float floatValue = 0.123;
    CFNumberRef floatNumber = CFNumberCreate(kCFAllocatorDefault, kCFNumberFloatType, &floatValue);
    printf(" number type => %ld \n", CFNumberGetType(floatNumber));
    printf(" byteSize => %ld \n", CFNumberGetByteSize(floatNumber));
    printf(" isFloatType => %d \n", CFNumberIsFloatType(floatNumber));
    float getFloatValue;
    CFNumberGetValue(floatNumber, kCFNumberFloatType, &getFloatValue);
    printf(" getValue => %f \n",getFloatValue);
    CFRelease(number);
    CFRelease(floatNumber);
    return 0;
}
```

### CFBoolean 学习

代码例子：

```
int main(int argc, const char * argv[]) {

    CFShow(kCFBooleanTrue);
    CFShow(kCFBooleanFalse);
    printf("true => %d \n",CFBooleanGetValue(kCFBooleanTrue));
    printf("false => %d \n",CFBooleanGetValue(kCFBooleanFalse));
    printf("CFBooleanGetTypeID = %ld \n",CFBooleanGetTypeID());
    return 0;
}
```

#### CFNumberCompare 比较

代码例子：

```
int main(int argc, const char * argv[]) {

    CFIndex one = 1, two = 2;
    CFNumberRef number1 = CFNumberCreate(kCFAllocatorDefault, kCFNumberCFIndexType, &one);
    CFNumberRef number2 = CFNumberCreate(kCFAllocatorDefault, kCFNumberCFIndexType, &two);

    CFComparisonResult compResult = CFNumberCompare(number1, number2, NULL);
    printf("compResult %ld \n",compResult);

    compResult = CFNumberCompare(number2, number2, NULL);
    printf("compResult %ld \n",compResult);

    compResult = CFNumberCompare(number2, number1, NULL);
    printf("compResult %ld \n",compResult);

    return 0;
}

```
// 执行結果
compResult -1
compResult 0
compResult 1

#### CFStringCompare 比较

代码例子：

```
int main(int argc, const char * argv[]) {
    printf("a : b %ld \n", CFStringCompare(CFSTR("a"), CFSTR("b"), 0));
    printf("b : b %ld \n", CFStringCompare(CFSTR("b"), CFSTR("b"), 0));
    printf("b : a %ld \n", CFStringCompare(CFSTR("b"), CFSTR("a"), 0));
    printf("A : a %ld \n", CFStringCompare(CFSTR("A"), CFSTR("a"), kCFCompareCaseInsensitive));
    return 0;
}
```
// 执行結果
a : b -1
b : b 0
b : a 1
A : a 0

#### CFBoolean 比较

代码例子：

```
printf("kCFBooleanTrue : kCFBooleanFalse %d \n", kCFBooleanTrue == kCFBooleanFalse);
    printf("kCFBooleanFalse : kCFBooleanFalse %d\n", kCFBooleanFalse == kCFBooleanFalse);
```

### CF 命名的共同点
比如：
CFArray

CFArrayCreate

CFArrayCreateCopy

CFArrayCreateMutable

CFArrayCreateMutableCopy

CFString

CFStringCreate

CFStringCreateCopy

CFStringCreateMutable

CFStringCreateMutableCopy

#### CoreFoundation的Mutable型

| Immutable     | Mutable      |
| ------------- |:-------------:|
|CFArray			|CFMutableArray|
|CFAttributeString	|CFMutableAttributeString|
|CFBag				|CFMutableBag|
|CFBitVector		|CFMutableBitVector|
|CFCharcterSet	|CFMutableCharcterSet|
|CFData			|CFMutableData|
|CFDictionary		|CFMutableDictionary|
|CFSet				|CFMutableSet|
|CFString			|CFMutableString|


### Compare 比较排序

代码例子：

```
int main(int argc, const char * argv[]) {

    CFMutableArrayRef array;
    array = CFArrayCreateMutable(kCFAllocatorDefault, 0, &kCFTypeArrayCallBacks);
    CFIndex one = 1, two = 2, three = 3;
    CFNumberRef number1, number2, number3;
    number1 = CFNumberCreate(kCFAllocatorDefault, kCFNumberCFIndexType, &one);
    number2 = CFNumberCreate(kCFAllocatorDefault, kCFNumberCFIndexType, &two);
    number3 = CFNumberCreate(kCFAllocatorDefault, kCFNumberCFIndexType, &three);

    CFArrayAppendValue(array, number2);
    CFArrayAppendValue(array, number3);
    CFArrayAppendValue(array, number1);

    CFRelease(number1);
    CFRelease(number2);
    CFRelease(number3);

    CFShow(CFSTR("print all"));
    CFShow(array);

    CFArraySortValues(array, CFRangeMake(0, CFArrayGetCount(array)), (CFComparatorFunction)CFNumberCompare, NULL);
    CFShow(CFSTR("sorted array"));
    CFShow(array);

    CFRelease(array);

    return 0;
}

```

// 执行結果
print all
(
    2,
    3,
    1
)
sorted array
(
    1,
    2,
    3
)
