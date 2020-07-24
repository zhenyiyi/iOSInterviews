

**“Never”**是一个约定，表示一件事在过去或未来的任何时段都不会发生。它是时间轴上的一种逻辑上的不可能，在任何方向延展开去都没有可能。这就是为什么在代码中看到 [这样的注释](https://github.com/search?q="this+will+never+happen"&type=Code) 会特别让人不安。

```swift
// this will never happen
```

所有编译器的教科书都会告诉你，这样一句注释不能也不会对编译出的代码产生任何影响。[墨菲定理](https://en.wikipedia.org/wiki/Murphy's_law) 告诉你并非如此，注释以下的代码一定会被触发。

那 Swift 是如何在这种无法预测的、混乱的开发过程中保证安全呢？答案难以置信：“**什么都不做**”，以及“**崩溃**”。



使用 `Never` 替换 `@noreturn` 修饰符

从 Swift 3 开始，`fatalError` 和它的相关函数都被声明为返回 `Never` 类型。

```swift
// Swift >= 3.0
func fatalError(_ message: @autoclosure () -> String = String(),
                     file: StaticString = #file,
                     line: UInt = #line) -> Never
```

作为一个注释的替代品，它肯定是很复杂的，对吗？NO！事实上，恰恰相反，`Never` 可以说是整个 Swift 标准库中最简单的一个类型：

```swift
enum Never {}
```



# 无实例类型（Uninhabited Types）

`Never` 是一个 *无实例*（*Uninhabited*）类型，也就是说它没有任何值。或者换句话说，无实例类型是无法被构建的。

在 Swift 中，没有定义任何 `case` 的枚举是最常见的一种无实例类型。跟结构体和类不同，枚举没有初始化方法。跟协议也不同，枚举是一个具体的类型，可以包含属性、方法、泛型约束和嵌套类型。正因如此，Swift 标准库广泛使用无实例的枚举类型来做诸如 [定义命名空间](https://github.com/apple/swift/blob/a4230ab2ad37e37edc9ed86cd1510b7c016a769d/stdlib/public/core/Unicode.swift#L918) 以及 [标识类型的含义](https://github.com/apple/swift/blob/a6952decab6f918a9df3c6fa342153a9f9204f8e/stdlib/public/core/MemoryLayout.swift#L43) 之类的事情。

但 `Never` 并不这样。它没有什么花哨的东西，它的特别之处就在于，它就是它自己（或者说，它什么都不是）。

# 消除泛型中的不可能状态

[Matt Diephouse](https://github.com/mdiep) 在提案中解释了为什么让这个令人费解的类型去遵守 `Equatable` 和其他协议



> `Never` 在表示不可能执行的代码方面非常有用。大部分人熟悉它，是因为它是 `fatalError` 等方法的返回值，但 `Never` 在泛型方面也非常有用。比如说，一个 `Result` 类型可以使用 `Never` 作为它的 `Value`，表示某种东西一直是错误的，或者使用 `Never` 作为它的 `Error`，表示某种东西一直不是错误的。





比如，下面的代码假定数组一定不为空，

```swift
let array: [Int]
let firstIem = array.first!
```

为了避免强制解包，你可以使用带条件赋值的 `guard` 语句：

```swift
let array: [Int]
guard let firstItem = array.first else {
    fatalError("array cannot be empty")
}
```



> https://nshipster.cn/never/