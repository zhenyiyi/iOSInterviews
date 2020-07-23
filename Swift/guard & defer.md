## guard

`guard` 是一个要求表达式的值为 `true` 从而继续执行的条件语句。如果表达式为 `false`，则会执行必须提供的 `else` 分支。

`guard` 语句中的 `else` 分支必须退出当前的区域，通过使用 `return` 来退出函数，`continue`或者 `break` 来退出循环，或者使用像 `fatalError(_:file:line:)` 这种返回 [`Never`](https://nshipster.com/never) 的函数。

`guard` 语句和 optional 绑定组合在一起非常好用。在 `guard` 语句的条件里进行的 optional 绑定可以在函数或闭包其后的部分使用。

对比一下 `guard-let` 语句和 `if-let` 语句中的 optional 绑定：

```swift
var name: String?

if let name = name {
    // name 在这里面不是 optional（类型是 String）
}
// name 在外面是 optional（类型是 String?）


guard let name = name else {
    return
}

// name 从这里开始都不是 optional 了（类型是 String）
```

### 使用 guard 来避免过多的缩进和错误

我们来对比一下使用 `guard` 关键字之后能如何改善代码且帮助我们避免错误。

比如，我们要实现一个 `readBedtimeStory()` 函数：

```swift
enum StoryError: Error {
    case missing
    case illegible
    case tooScary
}

func readBedtimeStory() throws {
    if let url = Bundle.main.url(forResource: "book",
                               withExtension: "txt")
    {
        if let data = try? Data(contentsOf: url),
            let story = String(data: data, encoding: .utf8)
        {
            if story.contains("👹") {
                throw StoryError.tooScary
            } else {
                print("Once upon a time... \(story)")
            }
        } else {
            throw StoryError.illegible
        }
    } else {
        throw StoryError.missing
    }
}
```



使用 `guard` 语句组织代码可以让代码读起来更加的线性



```swift
func readBedtimeStory() throws {
    guard let url = Bundle.main.url(forResource: "book",
                                  withExtension: "txt")
    else {
        throw StoryError.missing
    }

    guard let data = try? Data(contentsOf: url),
        let story = String(data: data, encoding: .utf8)
    else {
        throw StoryError.illegible
    }

    if story.contains("👹") {
        throw StoryError.tooScary
    }

    print("Once upon a time ...\(story)")
}
```



## defer

在错误处理方面，`guard` 和新的 `throw` 语法之间，Swift 鼓励用尽早返回错误（这也是 NSHipster 最喜欢的方式）来代替嵌套 if 的处理方式。尽早返回让处理更清晰了，但是已经被初始化（可能也正在被使用）的资源必须在返回前被处理干净。

`defer` 关键字为此提供了安全又简单的处理方式：声明一个 block，当前代码执行的闭包退出时会执行该 block。

看看下面这个包装了系统调用 `gethostname(2)` 的函数，用来返回当前系统的[主机名称](https://zh.wikipedia.org/zh-cn/主機名稱)：

```swift
import Darwin

func currentHostName() -> String {
    let capacity = Int(NI_MAXHOST)
    let buffer = UnsafeMutablePointer<Int8>.allocate(capacity: capacity)

    guard gethostname(buffer, capacity) == 0 else {
        buffer.deallocate()
        return "localhost"
    }

    let hostname = String(cString: buffer)
    buffer.deallocate()

    return hostname
}
```



这里有一个在最开始就创建的 `UnsafeMutablePointer<UInt8>` 用于存储目标数据，但是我**既要**在错误发生后销毁它，**又要**在正常流程下不再使用它时对其进行销毁。

这种设计很容易导致错误，而且不停地在做重复工作。

通过使用 `defer` 语句，我们可以排除潜在的错误并且简化代码：

```swift
func currentHostName() -> String {
    let capacity = Int(NI_MAXHOST)
    let buffer = UnsafeMutablePointer<Int8>.allocate(capacity: capacity)
    defer { buffer.deallocate() }

    guard gethostname(buffer, capacity) == 0 else {
        return "localhost"
    }

    return String(cString: buffer)
}
```



尽管 `defer` 紧接着出现在 `allocate(capacity:)` 调用之后，但它要等到当前区域结束时才会被执行。多亏了 `defer`，`buffer` 才能无论在哪个点退出函数都可以被释放。

考虑在任何需要配对调用的 API 上都使用 `defer`，比如 `allocate(capacity:)` / `deallocate()`、`wait()` / `signal()` 和 `open()` / `close()`。这样的话，你不仅可以消除一种程序员易犯的错误，

### 经常 defer

如果在同一个作用域内使用多个 `defer` 语句，它们会根据出现顺序反过来执行——像栈一样。这个反序是非常重要的细节，保证了被延迟的代码块创建时作用域内存在的东西，在代码块执行同样存在。

举个例子，执行这段代码会得到下面的输出：

```
func procrastinate() {
    defer { print("wash the dishes") }
    defer { print("take out the recycling") }
    defer { print("clean the refrigerator") }

    print("play videogames")
}
```

​        play videogames
​        clean the refrigerator
​        take out the recycling
​        wash the dishes
​      

