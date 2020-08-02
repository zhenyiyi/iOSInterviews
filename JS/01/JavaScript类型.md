

- 为什么有的编程规范要求用 void 0 代替 undefined？
- 字符串有最大长度吗？
- 0.1 + 0.2 不是等于 0.3 么？为什么 JavaScript 里不是这样的？
- ES6 新加入的 Symbol 是个什么东西？
- 为什么给对象添加的方法能用在基本类型上？



JavaScript 语言的每一个值都属于某一种数据类型

1. Undefined；任何变量在赋值前是 Undefined 类型、值为 undefined
2. Null；
3. Boolean；
4. String；
5. Number；
6. Symbol；
7. Object。





### Symbol

Symbol 是 ES6 中引入的新类型，它是一切非字符串的对象 key 的集合，在 ES6 规范中，整个对象系统被用 Symbol 重塑。





- List 和 Record： 用于描述函数传参过程。
- Set：主要用于解释字符集等。
- Completion Record：用于描述异常、跳出等语句执行过程。
- Reference：用于描述对象属性访问、delete 等。
- Property Descriptor：用于描述对象的属性。
- Lexical Environment 和 Environment Record：用于描述变量和作用域。
- Data Block：用于描述二进制数据。



如果我们不用原生的 Number 和 parseInt，用 JavaScript 代码实现 String 到 Number 的转换

