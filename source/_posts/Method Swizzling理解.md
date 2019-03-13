---
title: Method Swizzling 理解
date: 2016-08-01 14:48:38
tags:
categories: 小知识 #分类
---

# Method Swizzling
顾名思义方法交换,这是一种利用了runtime机制，在运行时通过修改类分发表中slector对应的函数来修改函数的实现。该机制的用途可以是在某些场景下，比如:我们需要在当前的工程中，每一个同样的方法中做埋点（在日志追踪系统中会在一些指定的地方发送一些数据），但不影响本身方法的实现，在其具体实现中并不掺入任何埋点相关的代码。或者监听哪些方法被调用多少次，我是在一次检测内存中对方法交换产生了兴趣,于是手撸了一遍代码。

# Method Swizzling 的实现
Mattt Thompson写过一篇Method Swizzling的[Blog](https://github.com/chausson/CHAnimationDemo)里面实现了一套标准交换方法的代码实现

``` obj-c
#import <objc/runtime.h>

@implementation UIViewController (Tracking)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];

        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        // ...
        // Method originalMethod = class_getClassMethod(class, originalSelector);
        // Method swizzledMethod = class_getClassMethod(class, swizzledSelector);

        BOOL didAddMethod =
            class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));

        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark - Method Swizzling

- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}

@end
```
首先我们先来看
``` obj-c
Method class_getInstanceMethod(Class aClass, SEL aSelector)
Method originalMethod = class_getClassMethod(class, originalSelector);
BOOL didAddMethod =
            class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));
            class_replaceMethod(class,
                            swizzledSEL,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
```
1.class_getInstanceMethod这个官方解释是说会从父类中找到对象方法的实现，但不会从class_copyMethodList里面去找，如果找到则返回该实现方法，没有的话则会返回NULL。

2.class_getClassMethod则是从类的方法中去寻找。

3.class_addMethod则会为该类添加一个方法，但不会去重写已有实现的方法，如果已经有相同方法名称则会返回NO添加失败。

4.class_replaceMethod的话看名称就知道这个是为替换方法开的接口，如果方法不存在则会先调用method_setImplementation。

# Method Swizzling 总结
该机制是利用了运行时的特性，使用的时候请注意避免一些坑
1.不要在替换方法中实现类的原有方法，这样会形成死循环。
2.所有替换的方法必须加一些前缀用来区别替换的方法。
3.不调用苹果提供的方法实现，可能会影响到程序的其他部分。

# Github Demo
[Method Swizzling Demo](https://github.com/chausson/MethodSwizzlingDemo)
