#关于 Swift Error 的分类

##Swift 错误类型的种类

###Simple domain error

简单的，显而易见的错误。这种错误的最大特点是我们不需要关心原因，只需要知道错误发生，并且想要进行处理。用来表示这种错误发生的方法一般就是返回一个 `nil` 值。在 Swift 中，这类错误最常见的情况就是将某个字符串转换为整数，或者在字典尝试用某个不存在的 `key` 获取元素：

```swift
// Simple Domain Error 的例子

let num = Int("hello world") // nil

let element = dic["key_not_exist"] // nil
```

在使用层面 (或者说应用逻辑) 上，这类错误一般用 `if let` 的可选值绑定或者是 `guard let` 提前进行返回处理即可，不需要再在语言层面上进行额外处理。

###Recoverable error

正如其名，这类错误应该是被容许，并且是可以恢复的。可恢复错误的发生是正常的程序路径之一，而作为开发者，我们应当去检出这类错误发生的情况，并进一步对它们进行处理，让它们恢复到我们期望的程序路径上。

这类错误在 Objective-C 的时代通常用 `NSError` 类型来表示，而在 Swift 里则是 `throw` 和 `Error` 的组合。一般我们需要检查错误的类型，并作出合理的响应。而选择忽视这类错误往往是不明智的，因为它们是**用户正常使用过程中可能会出现的情况，我们应该尝试对其恢复**，或者至少向用户给出合理的提示，让他们知道发生了什么。像是网络请求超时，或者写入文件时磁盘空间不足：

```swift
// 网络请求
let url = URL(string: "https://www.example.com/")!
let task = URLSession.shared.dataTask(with: url) { data, response, error in
    if let error = error {
        // 提示用户
        self.showErrorAlert("Error: \(error.localizedDescription)")
    }
    let data = data!
    // ...
}

// 写入文件
func write(data: Data, to url: URL) {
    do {
        try data.write(to: url)
    } catch let error as NSError {
        if error.code == NSFileWriteOutOfSpaceError {
            // 尝试通过释放空间自动恢复
            removeUnusedFiles()
            write(data: data, to: url)
        } else {
            // 其他错误，提示用户
            showErrorAlert("Error: \(error.localizedDescription)")
        }
    } catch {
        showErrorAlert("Error: \(error.localizedDescription)")
    }
}
```

###Universal error

这类错误理论上可以恢复，但是由于语言本身的特性所决定，我们难以得知这类错误的来源，所以一般来说也不会去处理这种错误。这类错误包括类似下面这些情形：

```swift
// 内存不足
[Int](repeating: 100, count: .max)

// 调用栈溢出
func foo() { foo() }
foo()
```

我们可以通过设计一些手段来对这些错误进行处理，比如：检测当前的内存占用并在超过一定值后警告，或者监视栈 frame 数进行限制等。但是一般来说这是不必要的，也不可能涵盖全部的错误情况。更多情况下，这是由于代码触碰到了设备的物理限制和边界情况所造成的，一般我们也不去进行处理（除非是人为造成的 bug）。

**在 Swift 中，各种被使用 fatalError 进行强制终止的错误一般都可以归类到 Universal error。
**


###Logic failure

**逻辑错误是程序员的失误所造成的错误**，它们应该在开发时通过代码进行修正并完全避免，而不是等到运行时再进行恢复和处理。

常见的 Logic failure 包括有：

```swift
// 强制解包一个 `nil` 可选值
var name: String? = nil
name!

// 数组越界访问
let arr = [1,2,3]
let num = arr[3]

// 计算溢出
var a = Int.max
a += 1

// 强制 try 但是出现错误
try! JSONDecoder().decode(Foo.self, from: Data())
```

这类错误在实现中触发的一般是 `assert` 或者 `precondition`。

###断言的作用范围和错误转换
和 `fatalError` 不同，`assert` 只在进行编译优化的 `-O` 配置下是不触发的，而如果更进一步，将编译优化选项配置为 `-Ounchecked` 的话，`precondition` 也将不触发。此时，各方法中的 `precondition` 将被跳过，因此我们可以得到最快的运行速度。但是相对地代码的安全性也将降低，因为对于越界访问或者计算溢出等错误，我们得到的将是不确定的行为。

函数 | fatalError | precondition | assert
---|---| --- | --- |
-Onone |	触发 |	触发 |	触发
-O	| 触发 | 触发 |	 
-Ounchecked |	 触发 |	 	 

对于 `Universal error` 一般使用 `fatalError`，而对于 `Logic failure` 一般使用 `assert` 或者 `precondition`。遵守这个规则会有助于我们在编码时对错误进行界定。而有时候我们也希望能尽可能多地在开发的时候捕获 Logic failure，而在产品发布后尽量减少 crash 比例。这种情况下，相比于直接将 Logic failure 转换为可恢复的错误，我们最好是使用 `assert` 在内部进行检查，来让程序在开发时崩溃。


本文转载自喵神的 [Blog](https://onevcat.com/2017/10/swift-error-category/) 



