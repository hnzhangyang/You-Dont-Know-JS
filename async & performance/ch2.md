# You Don't Know JS: Async & Performance
## 第二章 Callbacks
在第一章，我们了解了 javaScript 的异步语法，和 javaScript 的单线程事件
流 event loop。我们还体验了在 javaScript 中的各种各样的并发问题，以及解决方式。

第一章中，我们普遍采用函数作为基本单元，每个函数就时每个功能，用函数去完成
需求，其实函数能做的远远不止于此。

在本章中，函数又叫做 callback，可以理解为它是每次 event loop 的迭代所调用的
回调函数。callback 更多是用在异步编程中，作为相应异步请求的回调单元。

不管是简单的还是复杂的 javaScrpit 代码，多多少少会有异步编程，而 callback
 作为异步编程不可或缺的基本单元，被普遍使用。

 callback 也有缺点，很多开发者希望使用 Promise 来编写异步编程的代码，以获得更好的体验。
 但是使用前最好充分理解什么是 Promise 以及它的工作模式。

 在本章中，我们会深入讨论这点，讨论为什么需要诸如 Promise 之类的新异步 API ，
 以及它们的优缺点。