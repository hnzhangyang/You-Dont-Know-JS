# You Don't Know JS : Async & Performance
## 第一章
在一段时间中怎么去表达和操纵程序行为是很重要也很容易被误解的。
这不仅仅是关于for循环的细节以及for循环发生的时间，这是关于你的哪一部分程序先运行，哪一部分程序稍后运行，以及这两者的时间差。
实际上，很少有程序定义了某种方法让你控制上述情况。不管是等待用户交互，从服务器获取数据，或者是发送请求给服务器等待应答，你的程序都必须去管理上述状态。
事实上，在你的程序中，程序片段是现在运行还是推迟运行是异步编程的核心。
异步编程在JS诞生初始就存在，但是大多数JS开发人员并没有仔细思考这些是怎么或者这些是如何发生在程序中的。也不知道怎么去处理它。有一个足够好的方法是回掉函数（callback），并且很多人也认为用回掉函数处理异步问题已经足够了。
但是 js 变得越来越强大且完整，面对日渐丰富的需求，处理异步的问题也变得越来越痛苦。
所以我们呼吁处理异步的方法能更完善更强大。
当然，这些听起来很抽象，我保证我们会更具体的去了解上述情况。我们会体验关于js异步编程的各种不同的方式，在后面几个章节中。
在这之前，我们先要去了解到底什么是异步编程以及异步编程在js中的处理方法。
## 代码块
你可能将你的JS写在一个.JS文件里，通常情况下你的程序由几个代码块组成，在这几个代码块中，只有一个是正在执行，剩下的会延迟执行。代码块最普通的单元就是函数 function。
现在大部分JS程序员面临的问题是那些延迟执行的代码块并不是严格的在当前代码块执行完毕后就立即执行。换句话说，任务不会马上完成，而是异步完成。它并不会像你期望的那样阻塞代码执行，直至它完成。

考虑如下情况
``` javaScript
// ajax(..) is some arbitrary Ajax function given by a library
var data = ajax( "http://some.url.1" );

console.log( data );
// Oops! `data` generally won't have the Ajax results
```
你可能意识到标准的 Ajax 请求不会同步完成，这意味着 ajax(...) 函数并不会马上返回值给 data 变量。如果 ajax(...) 能够阻塞执行，直到收到服务器回应，那么 data = ... 这个赋值语句才会有效。

我们生成了一个异步 Ajax 请求，Ajax 请求并没有阻塞程序执行直到收到服务器返回的数据。
最简单的等待服务器请求的方法是使用函数 function，一般我们叫回掉函数 callback。
``` javaScript
// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", function myCallbackFunction(data){

	console.log( data ); // Yay, I gots me some `data`!

} );
```
警告，你也许听过同步请求。这在技术上来说是可行的，但是你永远不要用它，因为同步请求会锁死浏览器 UI ，阻止任何交互，这不是一个好方法，你应该避免使用它。

在你提出异议之前，你渴望只是为了获取数据而导致的混乱无章的 callback。

考虑下面的情况。
``` javaScript
function now() {
	return 21;
}

function later() {
	answer = answer * 2;
	console.log( "Meaning of life:", answer );
}

var answer = now();

setTimeout( later, 1000 ); // Meaning of life: 42
```
在程序中有两个代码块，其中一个现在运行，另一个稍后运行。执行顺序显而易见。但是还是让我们详细解释下。

now:
``` javaScript
function now() {
	return 21;
}

function later() { .. }

var answer = now();

setTimeout( later, 1000 );
```
later:
``` javaScript
answer = answer * 2;
console.log( "Meaning of life:", answer );
```
在你执行程序的时候，Now 代码块正确运行，但是 setTimeout(...) 设置了一个延迟事件。导致 later 函数体会在 100ms 之后运行。
任何时候你指定一段函数在某一个事件点执行（计时器，鼠标点击，Ajax 异步请求等等），你就编写了一个延迟函数，也就是异步函数。

## console
这里不想也不会解释 **console** 这个方法的运行机制，因为它不是 javaScript 的规范，实际上， **console** 这个方法是由宿主环境，也就是浏览器提供的（详细请看 Types & Grammar 一节）。

**console** 方法的行为取决于浏览器爸爸，他怎么高兴怎么来，这容易导致一些功能上的分歧，或许会使你困惑。

有时候，在一些浏览器，或者满足某种条件下， **console.log(...)** 不会立即输出你想要的值。主要原因是因为 I/O 接口非常缓慢，它阻塞了程序的执行（不单单是阻塞JS），为了避免这种情况，浏览器爸爸选择异步执行 **console.log(...)** 方法，这件事以前大概你没听过（反正我是没听过）。

有一种不是很普遍，但是有可能发生的情况是，情况下面代码。
``` javaScript
var a = {
	index: 1
};

// later
console.log( a ); // ??

// even later
a.index++;
```


