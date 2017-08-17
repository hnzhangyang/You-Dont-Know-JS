# You Don't Know JS: Async & Performance
## Chapter 4: Generators
在第二章中，我们阐述了关于使用 callback 作为异步手段的两个不利之处。
- callback 的使用方式与我们平常的思考方式不符。
- 由于使用 callback 不可避免的会转移控制权给第三方插件，这会导致代码不可信赖。

而在第三章中，我们讨论了针对上述问题，怎么使用 Promises 从第三方插件收回我们的代码控制权，使代码可信赖。

在本章中，我们主要讨论怎么使用 Generator 构造看起来像同步代码的异步代码 : )

## Breaking Run-to-Completion
在第一章中，我们讨论了在 javaScript 开发中的一个基本定理，一旦一个函数开始执行，就会执行到函数结束，其中没有办法打断其执行。

有趣的是，在 ES6 中，这个概念还要加一个条件，**除非函数自己中断执行**。可中断执行的函数，就是今天我们讨论的 Generator。