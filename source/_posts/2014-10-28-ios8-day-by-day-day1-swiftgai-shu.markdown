---
layout: post
title: "iOS8 Day-by-Day :: Day1 :: Swift概述"
date: 2014-10-28 20:38:03 +0800
comments: true
categories: 
- iOS8 Day-by-Day
---

本文译自 : [iOS8 Day-by-Day :: Day 1 :: Blagger's Guide to Swift](http://www.shinobicontrols.com/blog/posts/2014/07/17/ios8-day-by-day-day-1-blaggers-guide-to-swift)


+ list element with functor item
{:toc}


## 简介

今年的WWDC令人印象深刻，除了宣布`iOS8`，苹果还向我们介绍了一门新的编程语言--`Swift`。`Swift`和`objective-C`有很大的区别，它是强类型(`Strongly-typed`)的，并且包含有很多现代编程语言的特性。

出于对新兴事物强大的热情和兴趣，这个博客系列将全程使用Swift。现在市面上已经有了很丰富的资源向我们介绍如何使用Swift，以及它是如何与Cocoa框架交互的。事实上，通过阅读如下官方文档开始你的Swift之旅是一个非常明智的选择:

<!-- more -->

+ [The Swift Programming Language](https://itunes.apple.com/us/book/the-swift-programming-language/id881256329?mt=11&ls=1)
+ [Using Swift with Cocoa and Objective-C](https://itunes.apple.com/us/book/using-swift-cocoa-objective/id888894773?mt=11&ls=1)

你同时应该关注官方的[Swift Blog](https://developer.apple.com/swift/blog/)，以及Apple提供的一些其他的官方[资源](https://developer.apple.com/swift/resources/)。

由于已经有了很多很好的资源向我们介绍如何使用`Swift`，因此接下来我们不会涉及过多的基础内容，但我们会着重向你介绍在初次使用Swift的过程中可能会遇到的一些重要的陷阱和潜在的痛点，特别是与系统的框架相关的。

我会在一个`Xcode 6 playground`里向你展示以下所介绍的示例代码，你可以通过`ShinobiControls`的Github[得到它](https://github.com/ShinobiControls/iOS8-day-by-day)。

如果对本文有任何疑问，或者建议添加一些其他相关内容，你可以通过在下边添加评论或通过[@iwantmyrealname](https://twitter.com/iwantmyrealname)在`twitter`上告诉我，我会在本系列文章中对其持续跟进。

## 初始化(Initialisation)

Swift正式引入了与对象初始化相关的概念，包括`designated initialiser`(指定初始化方法)和`convenience initialiser`(便利初始化方法)，以及在对一个对象初始化的过程中以指定的顺序调用一系列初始化方法。近期我将会通过本系列文章中的一篇来详细介绍Swift的初始化是如何工作的，以及这将会对你所写的objective-C部分的代码造成哪些影响，敬请关注。

Swift和objective-C在初始化方面的另一个重要区别是返回值和初始化失败的情况。在objective-C中的初始化方法大概像这样：

```objective-c
- (instancetype)init {
    self = [super init];
    if (self) {
        // Do some stuff
    }
    return self;
}
```

在Swift中是这样：

```swift
init {
    variableA = 10
    ...
    super.init()
}
```

注意，objective-C的初始化方法负责创建(Create)并且返回`self`，但在Swift对应的场景中却不存在`return`语句。这就意味着，在Swift中你无法像objC那样在初始化失败时返回一个`nil`对象。

或许Swift在不久后正式发布时会解决掉这个问题，但现在唯一可以绕过这个问题的解决方案是使用一个返回可选类型(`optional type`)的类方法(`class method`)：

```swift
class MyClass {
    class func myFactoryMethod() -> MyClass? {
        ...
    }
}
```

有趣的是，objective-C`APIs`中的工厂方法(`factory method`)在Swift中被转换成了初始化方法(`initialiser`)，因此上述解决方案并不完美。但是在Swift语言解决这个问题之前，这是处理初始化失败的情况的唯一选择。


## 可变性(Mutability)

对于Cocoa开发者来说，(不)可变性(`(im)mutability`)并不是一个新的概念。我们已经能够在适当的场景使用`NSArray`或其对应的可变类型`NSMutableArray`，而且我们也知道应该尽可能得优先选用相应的不可变(`immutable`)类型。但Swift把这一概念提升到了一个新的高度，而且把不可变性(`immutability`)作为一个基本概念融入到了这门语言的血液。

Swift中用关键字`let`来定义一个不可变变量(`immutable variable`)，即你不能改变其所代表的内容。
举个栗子：

```swift
let a = MyClass()
a = MySecondClass() // 不允许
```

这就意味着你不能对一个由关键字`let`定义的变量重新赋值(`redefine`)。取决于对象的类型，对象自身也可能是不可变的(`immutable`)。如果是值类型(`value type`)，比如`struct`，它也将是不可变的。如果是引用类型(`reference type`)，比如`class`，那么它将是可变的。(译者注：这里所说的对象自身是指对象内部的属性)

为了说明这一点，我们来看如下结构体(`struct`)：

```
struct MyStruct {
    let t = 12
    var u: String
}
```

如果我们用关键字`var`来定义一个变量`struct1`，可以看到如下结果：

```
var struct1 = MyStruct(t: 15, u: "Hello")
struct1.t = 13 // 错误: t 是一个不可变的属性
struct1.u = "GoodBye"
struct1 = MyStruct(t: 10, u: "You")
```

你可以改变`u`属性，因为它是由`var`定义的，你也可以重新给变量`struct1`赋值(`redefine`)，同样是因为它是由`var`定义的。但是属性`t`是不能被改变的，因为它被定义为`let`。下面我们来看下用关键字`let`来定义一个结构体(`struct`)的实例将会发生什么：

```
let struct2 = MyStruct(t: 12, u: "World")
struct2.u = "Planet" // 错误: struct2 是不可变的
struct2 = MyStruct(t: 10, u: "Defeat") // 错误: struct2 是不可变的引用
```

这里不仅不能改变`struct2`变量，也不能改变这个结构体的内容(比如其属性`u`)。这是因为结构体(`struct`)是**值类型**(`value type`)。

同样的场景，对于一个类(`class`)来说就有一些微妙的差别：

```
class MyClass {
    let t = 12
    var u: String
    
    init(t: Int, u: String) {
        self.t = t
        self.u = u
    }
}
```

根据使用objective-C的经验，将变量定义为`var`所表现出来的行为或许你并不感到奇怪：

```
var class1 = MyClass(t: 15, u: "Hello")
class1.t = 13 // 错误: t 是一个不可变的属性
class1.u = "GoodBye"
class1 = MyClass(t: 10, u: "You")
```

你不仅可以改变引用本身，而且可以改变所有使用`var`定义的变量，但是却不能改变任何一个使用`let`定义的变量。接下来用`let`来定义一个类实例，我们来对比一下他们的表现有何不同：

```
let class2 = MyClass(t: 12, u: "World")
class2.u = "Planet" // 正确
class2 = MyClass(t: 11, u: "Geoid") // 错误: class2 是一个不可改变的引用
```

这里你不能改变这个引用(`reference`)自身，但是**可以**改变这个类中的任何一个使用`var`定义的属性。这是因为类是**引用类型**(`reference type`)。

上述现象很容易理解，并且在该语言的参考书籍有详尽的解释。但是当我们把目光转向Swift的集合类型(`collection types`)时，上述概念就容易混淆了。

`NSArray`是引用类型。也就是说当你创建一个`NSArray`的实例时，你创建了一个对象，并且代表该对象的变量是一个指向该数组在内存中的地址的指针，也因此在objective-C中定义该变量时要在前边标注一个星号。如果你回顾一下之前所学的关于`var`和`let`的引用类型和和值类型的语义(`semantics`)，可能会想象出他们会有怎样的行为表现。事实上，如果你需要一个NSArray的可变的版本，那不得不用另一个类 -- `NSMutableArray`。

Swift的数组并不像这样，它们是值类型而非引用类型。这就意味着它们的行为更像是结构体(`struct`)而不是类(`class`)。因此，关键字`var`或`let`不仅指定了变量能否被重新定义(`redefined`)，还决定了所创建的数组是否是可变的。

一个用`var`定义的数组不仅可以被重新赋值(`reassigned`)，还可以改变其元素：

```
var array1 = [1,2,3,4]
array1.append(5)  // [1,2,3,4,5]
array1[0] = 27    // [27,2,3,4,5]
array1 = [3,2]    // [3,2]
```

但是对于一个被定义为`let`的数组，既不能对其重新赋值，也不能改变其元素：

```
let array2 = [4,3,2,1]
array2.append(0) // 错误: array2 是不可变的
array2[2] = 36   // 错误: array2 是不可变的
array2 = [5,6]   // 错误: 一个不可变的引用不能被再次赋值
```

这是一个非常容易引起混淆的地方。它不仅完全改变了我们对集合的可变性(`mutability`)的理解，同时也混淆了之前明显不同的两个概念。也许在不久之后的Swift的正式发行版本会改变这种行为，让我们拭目以待。

由此导致的一个必然结果是，由于这里的数组是值类型，因此它们传递的是一个副本(`copy`)。`NSArray`实例总是通过引用(`reference`)传递，因此一个拿到某个`NSArray`类型指针的方法将指向和之前相同的一块内存空间。如果你给某方法传一个Swift数组，它拿到的将是该数组的一份拷贝。数组中所存放对象的类型决定是深拷贝还是浅拷贝。写代码时要警惕这里！


## 强类型(`Strong Typing`)和`AnyObject`

强类型(`Strong Typing`)被视为Swift的一个重大特性，它能顾及到更加安全的代码，因为objective-C中运行时才可能抛出的异常如今在编译阶段就能将其捕获。

非常棒，但是，当Swift和objective-C系统框架一起使用时，你会发现很多`AnyObject`类型。它在Swift中相当于objective-C中的`nil`。在很多方面，`AnyObject`给人的感觉不太像Swift的风格(`un-Swift-like`)。它允许你调用**任何**它能找到的方法，但这样会导致运行时异常。实际上，它的作用几乎和objective-C中的`id`一模一样。不同的是如果某些属性或不含参数的方法不存在于某`AnyObject`，将会返回`nil`：

```
let myString: AnyObject = "hello"
myString.cornerRadius // Returns nil
```

为了以一种更加Swift的方式与`Cocoa APIs`协同工作，你会经常看到如下形式的代码：

```
func someFunc(parameter: AnyObject!) -> AnyObject! {
    if let castedParameter = parameter as? NSString {
        // Now I know I have a string :)
        ...
    }
}
```

如果你清楚的知道将要接收的是一个String类型的参数，就不必那么小心翼翼地对这个传参进行安全的类型转换了：

```
let castedParameter = parameter as NSString
```

其实对一个数组类型的传参进行安全的类型转换很简单。由于`NSArray`不支持泛型(`generics`)，所以你从Cocoa框架接收来的数组都是`[AnyObject]`类型的。然而，几乎无一例外的，objective-C中同一个数组中的元素都是相同的类型，而且在当前情形下还都是已知的类型。你可以用如下语法对整个传参过来的数组进行处理：

```
func someArrayFunc(parameter: [AnyObject]!) {
    let newArray = parameter as [String]
    // Do something with your strings :)
}
```


## 协议的一致性(Protocol Conformance)

Swift中的`protocol`(协议)很好理解，以如下方式定义：

```
protocol MyProtocol {
    func myProtocolMethod() -> Bool
}
```

我们经常需要检测一个对象是否符合某个特定的协议(protocol)，你可以这样做：

```
if let class1AsMyProtocol = class1 as? MyProtocol {
    // We're in
}
```

然而这样做有一个错误，因为要想检查一个协议的一致性(`conformance`)，这个协议必须是一个objective-C的协议，并且在开头用`@objc`标注：

```
@objc protocol MyNewProtocol {
    func myProtocolMethod() -> Bool
}

if let class1AsMyNewProtocol = class1 as? MyNewProtocol {
    // We're in
}
```

这里可能比你想象的要麻烦，因为为了给一个协议标记为`@objc`，它所有的属性和方法返回值类型都要能够被objective-C识别。这就意味着即使你要加载一些你认为只需要专注于Swift的类，那可能依然需要将其标注为`@objc`。


## 枚举(Enums)

Swift中的枚举(`Enums`)异常强大。它不仅可以有关联的(`associated`)值(并且他们可以是不同的类型)，而且还可以定义函数。

```
enum MyEnum {
    case FirstType
    case IntType (Int)
    case StringType (String)
    case TupleType (Int, String)
    
    func prettyFormat() -> String {
        switch self {
            case .FirstType:
                return "No params"
            case .IntType(let value):
                return "One param: \(value)"
            case .StringType(let value):
                return "One param: \(value)"
            case .TupleType(let v1, let v2):
                return "Some params: \(v1), \(v2)"
            default:
                return "Nothing to see here"
        }
    }
}
```

这点确实强大。可以像如下这样使用它：

```
var enum1 = MyEnum.FirstType
enum1.prettyFormat() // "No params"
enum1 = .TupleType(12, "Hello")
enum1.prettyFormat() // "Some params: 12, Hello"
```

我们需要通过一些练习来探索一下怎样才能很好的利用这些强大之处。具体你能用它来做什么这里提供一个参考 --- Swift中的可选系统(`optionals system`)就是以枚举为基础实现的。


## 总结

Swift真的很强大。我们需要一段时间的练习来达到最佳实践，并且通过Swift中某些特性和花样来解决一些在objective-C的限制下所解决不了的问题。本文简要介绍了一些在从objective-C向Swift过渡的过程中通常容易引起混淆的地方，不要被这些吓到。本系列博文的所有相关工程都是用Swift写的，而且在大多数情况下都很容易理解。本文所提及的所有示例代码都包含在我们所提供了一个`playground`文件中，你可以通过[github.com/ShinobiControls/iOS8-day-by-day](https://github.com/ShinobiControls/iOS8-day-by-day)获取它。
如果你对本文所涉及的内容有任何疑问，或者有添加或修改的建议，请联系我。你可以在下边给我留言，或者twitte我[@iwantmyrealname](https://twitter.com/iwantmyrealname)。








