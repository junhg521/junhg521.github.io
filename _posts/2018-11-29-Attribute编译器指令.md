---
layout: post
title: __attribute__属性
---
# __attribute__属性

`__attribute__`是一套编译器指令，被GNU和LLVM编译器所支持，允许对于`__attribute__`增加一些参数，做一些高级检查和优化。

## objc_subclassing_restricted
`objc_subclassing_restricted`属性表示被修饰的类不能被其他类继承，否则会报下面的错误

```
__attribute__((objc_subclassing_restricted))
@interface TestObject : NSObject
@property (nonatomic, strong) NSObject *object;
@property (nonatomic, assign) NSInteger age;
@end

@interface Child : TestObject
@end

错误信息：
Cannot subclass a class that was declared with the 'objc_subclassing_restricted' attribute
```
##  objc_requires_super

`objc_requires_super`属性表示子类必须调用被修饰的方法super，否则报黄色警告。

```
@interface TestObject : NSObject
- (void)testMethod __attribute__((objc_requires_super));
@end

@interface Child : TestObject
@end

警告信息：(不报错)
Method possibly missing a [super testMethod] call
```
## constructor / destructor
`constructor`属性表示在`main`函数执行之前，可以执行一些操作。`destructor`属性表示在`main`函数执行之后做一些操作。`constructor`的执行时机是在所有`load`方法都执行完之后，才会执行所有`constructor`属性修饰的函数。

```
__attribute__((constructor)) static void beforeMain() {
    NSLog(@"before main");
}

__attribute__((destructor)) static void afterMain() {
    NSLog(@"after main");
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSLog(@"execute main");
    }
    return 0;
}

执行结果：
debug-objc[23391:1143291] before main
debug-objc[23391:1143291] execute main
debug-objc[23391:1143291] after main
```
在有多个constructor或destructor属性修饰的函数时，可以通过设置优先级来指定执行顺序。格式是__attribute__((constructor(101)))的方式，在属性后面直接跟优先级。

```
__attribute__((constructor(103))) static void beforeMain3() {
    NSLog(@"after main 3");
}

__attribute__((constructor(101))) static void beforeMain1() {
    NSLog(@"after main 1");
}

__attribute__((constructor(102))) static void beforeMain2() {
    NSLog(@"after main 2");
}
```
## overloadable
`overloadable`属性允许定义多个同名但不同参数类型的函数，在调用时编译器会根据传入参数类型自动匹配函数。这个有点类似于C++的函数重载，而且都是发生在编译期的行为。

```
__attribute__((overloadable)) void testMethod(int age) {}
__attribute__((overloadable)) void testMethod(NSString *name) {}
__attribute__((overloadable)) void testMethod(BOOL gender) {}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        testMethod(18);
        testMethod(@"lxz");
        testMethod(YES);
    }
    return 0;
}
```
## objc_runtime_name
`objc_runtime_name`属性可以在编译时，将`Class`或`Protocol`指定为另一个名字，并且新名字不受命名规范制约，可以以数字开头。

```
__attribute__((objc_runtime_name("TestObject")))
@interface Object : NSObject
@end

NSLog(@"%@", NSStringFromClass([TestObject class]));

执行结果：
TestObject
```
## cleanup
通过cleanup属性，可以指定给一个变量，当变量释放之前执行一个函数。指定的函数执行的时间，是在dealloc之前的。在指定的函数中，可以传入一个形参，参数就是cleanup修饰的变量，形参是一个地址

```
static void releaseBefore(NSObject **object) {
    NSLog(@"%@", *object);
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        TestObject *object __attribute__((cleanup(releaseBefore))) = [[TestObject alloc] init];
    }
    return 0;
}
```
## unused
还有一个属性很实用，在项目里经常会有未使用的变量，会报一个黄色警告。有时候可能会通过其他方式获取这个对象，所以不想出现这个警告，可以通过unused属性消除这个警告。

```
NSObject *object __attribute__((unused)) = [[NSObject alloc] init];
```
