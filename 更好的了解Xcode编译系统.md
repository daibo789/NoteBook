#【译】更好的了解Xcode构建系统

原文：https://medium.com/flawless-app-stories/xcode-build-system-know-it-better-96936e7f52a

作者：Varun Tomar



一个程序在运行到一台设备之前是经历了很多转换的步骤的。和其它的编程语言处理系统一样，Xcode构建系统为了确保执行顺序和各种依赖库，需要运行很多命令行指令，传递各种各样的参数。整个构建过程分为以下五个阶段：

1、预处理

2、编译

3、汇编

4、链接

5、加载



## 预处理

预处理的目的是把我们的程序转换成能被编译器识别的形式。它用宏的定义替换宏，发现依赖关系并解析预处理器指令。如果Swift编译器没有预处理器，我们则不能在Swift项目中定义宏命令。在Xcode8中是允许我们通过`Build Setting`中的`SWIFT_ACTIVE_COMPILATION_CONDITIONS`定义预处理器标记位的。它跟OC中的预处理器宏定义是一致的。

![](https://raw.githubusercontent.com/zhangferry/Images/master/blog/20200624180433.png)



## 编译器

编译是整个过程中非常重要的一环。编译器是一个程序，它把高级语言像是Swift&Objective C转换成低级语言，像是object文件。在iOS中有两类编译器『Clange & swiftc』。该过程的抽象描述为：

![img](https://miro.medium.com/max/450/1*sryDiLA0zu5EhrPCUjZBew.png)

**注意**：编译器包含两个主要部分：前端和后端。Clange是C/C++/Objective-C的编译前端，swiftc是Swift的编译前端，LLVM是后端。这些乱七八糟的东西是什么，LLVM是从哪里来的?

![img](https://miro.medium.com/max/500/1*cb10Q6m1P8eHbU74uqyocQ.gif)

不要担心😉，我将会对它进行简明扼要的介绍，尽管这可能要单独出一篇文章说明其中细节（之后的文章，我会就这方面写一篇文章）。

> LLVM(*Low Level Virtual Machine*)是一个后端编译器，用于在其上构建编译器。它处理优化和生产适应目标架构(ARM、x86)的代码。CLang/Swiftc是一个前端编译器，可以解析C、c++、Objective C和Swift代码，并将其转换为适合LLVM的中间表示(IR)。在本文后面，您将更好地理解它。



让我们更细粒度的理解下Swift语言的编译过程，参考下面的图示：

![img](https://miro.medium.com/max/2058/1*wzgnQ32uL0GGi3XrX4rlTA.png)



**接下来介绍Swift编译器示如何一步一步工作的：**

1、Swift代码解析成AST **(Abstract Syntax Tree抽象语法树)**。AST是一个代表源码结构的抽象语法树，树的每个节点代表一个结构。它是什么样的呢，让我们通过一个例子来理解它。下面是我们将要进行分析的Swift代码：

```swift
//
//  Test.swift
//
//  Created by Varun Tomar
//  Copyright © 2020 Varun Tomar. All rights reserved.
//

import Foundation

class Bird {
  func fly() { }
}

func isFlyHigh(bird: Bird) -> Bool { return false }

class Sparrow: Bird {
  override func fly() { }
  func add(x: Int, y: Int) -> Int { return x + y }
}
```

我们可以将这段代码转换成抽象语法树格式内容，这需要运行`xcrun swiftc -dump-ast Test.swift`。



