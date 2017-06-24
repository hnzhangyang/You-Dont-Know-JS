# You Don't Know JS : Async & Performance
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
这里不想也不会解释 console 这个方法的运行机制，因为它不是 javaScript 的规范，实际上， console 这个方法是由宿主环境，也就是浏览器提供的（详细请看 Types & Grammar 一节）。

console 方法的行为取决于浏览器爸爸，他怎么高兴怎么来，这容易导致一些功能上的分歧，或许会使你困惑。

有时候，在一些浏览器，或者满足某种条件下， console.log(...) 不会立即输出你想要的值。主要原因是因为 I/O 接口非常缓慢，它阻塞了程序的执行（不单单是阻塞JS），为了避免这种情况，浏览器爸爸选择异步执行 console.log(...) 方法，这件事以前大概你没听过（反正我是没听过）。

有一种不是很普遍，但是有可能发生的情况是，请看下面代码。
``` javaScript
var a = {
	index: 1
};

// later
console.log( a ); // ??

// even later
a.index++;
```
通常情况下，我们希望 console.log(...) 能准确的输出 a 的值({index: 1})。事实上，由于后面的 a.index++ 修改了 a 的值，我们实际上看到的是 {index: 2}。

为什么呢？上面说到浏览器的 console.log(...) 采用的是异步处理，又因为 I/O 接口比较慢，正当浏览器后台处理这个 I/O 接口的时候，a.index++ 执行了... 所以输出来的值就是 {index: 2}。

有时候在 debug 的时候，如果出现了 console.log(...) 与预期不符的情况，就考虑下是不是因为 异步 I/O 接口缓慢，从而执行了其他的代码导致的。（我就碰到过这事，一直调试到7点半...）。

注意：如果你真的遇到了这种情况，有两条路供你选择，其一，使用断点代替 console.log(...)，其二，使用 JSON.stringify(...) 代替 console.log(...)， 输出对象的 JSON 字符串。

## Event loop
怎么说，其实不管你的异步代码看起来多么的清晰直接，其实直到现在的 ES6，在 javaScript 的内部并没有相关的异步的语法概念。

其实 javaScript 引擎只做意见事，给定一个时间点，让它执行一段代码块。

谁给?浏览器爸爸。

其实 javaScript 引擎并不是单独运行，它依赖于运行的环境，对于大部分开发者（就像我）来说，这个环境是浏览器爸爸。当然现在 node 的出现让 javaScript 有能力在服务端运行，这是后话了。。。

浏览器爸爸（下面简称 js环境）有自己的机制去控制何时触发 js引擎，这个机制叫 event loop。 javaScript 把整个代码切分成一个个单独的代码块，供 js环境 调用，每个单独的代码块我们用 event 表示。

举个栗子，当你的 javaScript 程序制造了一个 ajax 请求去服务器获取数据，
通常情况下你会写一个 callback，让后告诉 js环境，‘嘿，这段代码你先
不要执行，等到我的 ajax 请求获取到数据之后了，你再执行’。

然后 js环境 就会监听这个请求，获取到数据之后将之前悬置的 callback 
加入 event loop。

event loop 怎么玩？请看下面代码。
``` javaScript
// `eventLoop` is an array that acts as a queue (first-in, first-out)
var eventLoop = [ ];
var event;

// keep going "forever"
while (true) {
	// perform a "tick"
	if (eventLoop.length > 0) {
		// get the next event in the queue
		event = eventLoop.shift();

		// now, execute the next event
		try {
			event();
		}
		catch (err) {
			reportError(err);
		}
	}
}
```
你可以看到，我设置了一个永远不会结束循环的 while 循环，在 while 循环
 中，每一次迭代我们叫一次 tick，每个 tick ，如果 eventLoop 里面
 有 event，取出来执行。上面说的你设置的 ajax 请求的 callback 
 就是加入到当前的 eventLoop 末尾。

 所以单独把 event 加入到 eventLoop 中，event 可能并不会马上执行。
 如果在 event 之上还有 20 个 event，那么你的 callback 将会等到
 那 20 个  event 执行完毕了之后再执行。这就解释了 setTimeout(...) 
 为什么有时候并不是准确的按时间执行，因为在它定义的 callback 执行之前，可能
 还有别的程序正在执行。使用 setTimeout(...) 你只能保证它在某个时间点之前不
 会执行，不能保证在某个时间点一定会执行。

 你编写的 javaScript 代码，被分成若干个 event 之后加入到 event loop
 中来，按  event loop 的顺序执行。

 注意：我们刚才提到的 ES6,它有一个 API Promise 提供了更好的 event loop 管理
 能力，它将 event loop 的控制权从浏览器爸爸手中抢了回来，这个在第三章再说。

 ## Parallel Threading

多线程和异步是很容易混淆的事情，即使他们有很大的不同，一般而言，**异步是指一段代码块何时运行**，而**多线程是几个不同的代码块一起运行**。
  
对于多线程而言，我们一般想到线程（processes）和进程（threads），他们可以独立运行。
有时候几个线程甚至几台计算机都一起运行。几个进程一起运行的时候他们可以共享一个
线程的资源。

用 event loop 做对比比， event loop 每次只能干一件事，而多线程可以同时干几件事。他们是有很大区别的。

举个栗子。
``` javaScript
function later() {
	answer = answer * 2;
	console.log( "Meaning of life:", answer );
}
```
想象以上代码是 event loop 中的一个 event，这个 event 执行起来先要取到 answer 的值，然后
才能执行 answer = answer * 2，然后才能打印 console.log( "Meaning of life:", answer )。一切
都需要按顺序来。

在单线程环境，这不会有什么问题。因为没有程序可以打乱一个程序的执行（它每次只能执行一个程序片段嘛），
但是到了多线程，我们有几个程序片段同时运行，并且共享资源。这就会导致一些不可预期的行为了。
``` javaScript
var a = 20;

function foo() {
	a = a + 1;
}

function bar() {
	a = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
在 javaScript 的单线程环境中，以上代码如果 foo 先执行， a = 42，反之 a = 41。

假设，如果 javaScript 拥有多线程的能力，foo 于 bar 可以同时运行，并且共享资源。那么事情
就有点难办了。我把 foo 与 bar 运行时的大概步骤写出来了， X 于 Y 均是临时变量，考虑下如果这两个流程一起运行，将会
发生什么事情。

foo
``` javaScript
 foo():
  a. load value of `a` in `X`
  b. store `1` in `Y`
  c. add `X` and `Y`, store result in `X`
  d. store value of `X` in `a`
``` 

bar 
``` javaScript
bar():
  a. load value of `a` in `X`
  b. store `2` in `Y`
  c. multiply `X` and `Y`, store result in `X`
  d. store value of `X` in `a`
```
当 foo 与 bar 一起运行的时候，如果按一下流程进行，a 最后取什么值？
``` javaScript
1a  (load value of `a` in `X`   ==> `20`)
2a  (load value of `a` in `X`   ==> `20`)
1b  (store `1` in `Y`   ==> `1`)
2b  (store `2` in `Y`   ==> `2`)
1c  (add `X` and `Y`, store result in `X`   ==> `22`)
1d  (store value of `X` in `a`   ==> `22`)
2c  (multiply `X` and `Y`, store result in `X`   ==> `44`)
2d  (store value of `X` in `a`   ==> `44`)
```
好了 a = 44,我们换一种方式。
 ``` javaScript
1a  (load value of `a` in `X`   ==> `20`)
2a  (load value of `a` in `X`   ==> `20`)
2b  (store `2` in `Y`   ==> `2`)
1b  (store `1` in `Y`   ==> `1`)
2c  (multiply `X` and `Y`, store result in `X`   ==> `20`)
1c  (add `X` and `Y`, store result in `X`   ==> `21`)
1d  (store value of `X` in `a`   ==> `21`)
2d  (store value of `X` in `a`   ==> `21`)
 ```
 最后 a = 21。

 可以看出，在多线程编程中，如果你不阻止以上问题的发生，那么你就有可能发生问题 :) 

 javaScript 从来就没有多线程的能力，所以以上情况不会发生。但是这不代表这类问题
 永远不可能在 javaScript 编程中发生，比如说之前举得栗子，两个 ajax 请求因为触发的
 callback 顺序不同，返回的结果还是有不同滴。

 注意：如果你还没有遇到此类情况，也不要紧张，其实有时候这些情况并不会导致问题，或者
 有时候这些情况的确会导致问题... 呃，接下去看把。

 ## Run-to-Completion
 
 因为 javaScript 是单线程，foo 与 bar 作为 其中的两个代码块，当 foo 执行的时候，bar 不能打扰它，
 它会一直运行到代码块结束。反之亦然，这叫做不打扰原则（Run-to-Completion）。
 
 现在我给 bar 与 foo 加一点内容，让不打扰原则体现的更明显。
 ``` javaScript
var a = 1;
var b = 2;

function foo() {
	a++;
	b = b * a;
	a = b + 3;
}

function bar() {
	b--;
	a = 8 + b;
	b = a * 2;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
 ```
因为 foo 与 bar 谁也不能妨碍谁的执行，所以最后的结果取决与两个 ajax 谁先获取到数据。
但是如果提供了多线程的能力，foo 与 bar 一起执行，那么问题就很多了。

下面的代码 Chunk 1 是同步执行，Chunk 2 和 Chunk 3 是异步执行。

Chunk 1:
``` javaScript
var a = 1;
var b = 2;
```
Chunk 2 （foo）：
``` javaScript
a++;
b = b * a;
a = b + 3;
```
Chunk 3 （bar）：
``` javaScript
b--;
a = 8 + b;
b = a * 2;
```
因为 foo 与 bar 的发生先后顺序有可能不同，所以可能出现以下两种情况。

情况 1：
``` javaScript
var a = 1;
var b = 2;

// foo()
a++;
b = b * a;
a = b + 3;

// bar()
b--;
a = 8 + b;
b = a * 2;

a; // 11
b; // 22
```
情况 2：
``` javaScript
var a = 1;
var b = 2;

// bar()
b--;
a = 8 + b;
b = a * 2;

// foo()
a++;
b = b * a;
a = b + 3;

a; // 183
b; // 180
```
这两个情况何时出现是不确定的。但是这是在程序上的不确定，而不是在线程上的不确定，比起线程，
其实还是清晰很多。

以 javaScript 来说， 这种情况叫做竞争条件（race condition）。foo 与 bar 竞争
谁先触发。根据竞争结果，得出两个不同的值。

注意：上面说的不打扰原则（run-to-completion），在 ES6 的新 API Generators 中是
不适用的，这个以后再说。

## Concurrency
想象有一个需求：
- 1、展示一个信息列表
- 2、当用户往下翻的时候，信息列表自动加载更多

要实现这个需求，我们需要以上两个功能同时进行。一个用来监听 onscroll
 事件，当 onscroll 事件触发时发送 ajax 请求至服务器，请求数据。
 另一个用来展示数据，当浏览器收到服务器传回来的值时，渲染新值。

 显然，如果用户疯狂往下滑，并且服务器又不给力，那就会同时触发 N 个
 onscroll 事件， N 个 ajax 请求。 ajax 请求到数据以后渲染，相互交错，
 相辅相生...

 功能 1：
``` javaScript
onscroll, request 1
onscroll, request 2
onscroll, request 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
onscroll, request 7
```
功能 2：
``` javaScript
response 1
response 2
response 3
response 4
response 5
response 6
response 7
```
很有可能两个方法同时发生：
``` javaScript
onscroll, request 1
onscroll, request 2          response 1
onscroll, request 3          response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6          response 4
onscroll, request 7
response 6
response 5
response 7
```
但是就像之前提到的，javaScript 一次只能干一件事情，这两个方法必定执行顺序有
先后。具体顺序看 event loop。
``` javaScript
onscroll, request 1   <--- Process 1 starts
onscroll, request 2
response 1            <--- Process 2 starts
onscroll, request 3
response 2
response 3
onscroll, request 4
onscroll, request 5
onscroll, request 6
response 4
onscroll, request 7   <--- Process 1 finishes
response 6
response 5
response 7            <--- Process 2 finishes
```
可能在浏览器层面，两个方法可以同时添加进 event loop ，具体执行顺序还是得看
event loop 中事件的顺序。

但是有一点你发现没有，response 6 和 response 5 顺序似乎颠倒了。为什么？

## Noninteracting
当多个代码片段交叉运行时，如果这几个代码片段没有交互，那一般就没什么问题。

举个栗子
``` javaScript
var res = {};

function foo(results) {
	res.foo = results;
}

function bar(results) {
	res.bar = results;
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
foo 与 bar 是并行的 ajax 请求的回调函数，我们不知道那个 ajax 请求会
先收到数据，所以也就不知道 foo 或 bar 哪个先触发，但是这没什么问题，我们
根本不用担心这个，因为这两个回调函数独立运行，并没有交互。他们没有竞争问题
（race condition）。 

## Interaction
当两个方法共享资源的时候，就容易出现问题。当然不单单是指上述的 ajax 与 dom 的
交互。一旦程序有可能出现这种问题，需要实现协调。

请看下面代码：
``` javaScript
var res = [];

function response(data) {
	res.push( data );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```
两个 ajax 请求的回调函数都是将服务器返回的值放进 res 数组，但是我们没办法
预料数组中两个返回值的先后顺序，因为两个 ajax 不知道哪个先触发回调。

一般处理方法是添加条件判断：
``` javaScript
var res = [];

function response(data) {
	if (data.url == "http://some.url.1") {
		res[0] = data;
	}
	else if (data.url == "http://some.url.2") {
		res[1] = data;
	}
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```
这样一来，不管哪个 ajax 请求先触发回调，res 中的资源顺序总是确定的，这样处理
后在一定程度上就解决了两个 ajax 请求的竞争问题。

同样的方法也适用于当多个回调函数共享一个 dom 节点时，比如一个回调函数是更新
节点内容，另一个更新节点 style 或者 attributes 。一般情况下，你最好判断下
当多个回调都触发时，在渲染节点，这样效果更好。

再看一个栗子：
``` javaScript
var a, b;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(y) {
	b = y * 2;
	baz();
}

function baz() {
	console.log(a + b);
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
上面的代码中，不管 foo 或者 bar 哪个回调先触发，他们都会最终触发 baz。
如果不做处理，不管哪个先，都会出现问题， a 或者 b 总有一个值时 undefined。

对于以上情况，有一个简单的处理办法：
``` javaScript
var a, b;

function foo(x) {
	a = x * 2;
	if (a && b) {
		baz();
	}
}

function bar(y) {
	b = y * 2;
	if (a && b) {
		baz();
	}
}

function baz() {
	console.log( a + b );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
我们在函数体中加了一个条件判断 a && b，这样就控制了只有两个 ajax 请求
都成功回调时，才触发最终的 baz 函数。

有时候你希望几个 ajax 请求中，只有最先回调的 ajax 请求有效，其他的无效，
,如果不做处理的话，会出问题。

请看下面代码：

``` javaScript
var a;

function foo(x) {
	a = x * 2;
	baz();
}

function bar(x) {
	a = x / 2;
	baz();
}

function baz() {
	console.log( a );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
不管 foo ，bar 哪个先触发，一定会后面的回调值覆盖前一个回调值，这于我们的
初衷相悖，我们可以做以下处理：
``` javaScript
var a;

function foo(x) {
	if (a == undefined) {
		a = x * 2;
		baz();
	}
}

function bar(x) {
	if (a == undefined) {
		a = x / 2;
		baz();
	}
}

function baz() {
	console.log( a );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", foo );
ajax( "http://some.url.2", bar );
```
a == undefined 条件让我们忽略掉了后续出现的回调，这样就解决了问题。

## Cooperation
还有一个就是合作问题，它不是关于多个函数共享一份数据，而是在多个函数中
，当一个函数运行事件较长时，会影响体验。

再来个栗子：
``` javaScript
var res = [];

// `response(..)` receives array of results from the Ajax call
function response(data) {
	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `data` values doubled
		data.map( function(val){
			return val * 2;
		} )
	);
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```
不管哪个 ajax 回调，都会触发 data.map() 方法，当 data 的成员较少时，并不会有什么
问题，但是如果 data 返回的数据有几千万条，那么 data.map() 方法就需要一段
相当可观的时间去运行。

结合之前说的，当一个回调运行时，其他的函数都不能运行，此时，一切交互都无效，
这事就有点麻烦了。

为了友好的交互，你当然需要处理这些问题，比如：
``` javaScript
var res = [];

// `response(..)` receives array of results from the Ajax call
function response(data) {
	// let's just do 1000 at a time
	var chunk = data.splice( 0, 1000 );

	// add onto existing `res` array
	res = res.concat(
		// make a new transformed array with all `chunk` values doubled
		chunk.map( function(val){
			return val * 2;
		} )
	);

	// anything left to process?
	if (data.length > 0) {
		// async schedule next batch
		setTimeout( function(){
			response( data );
		}, 0 );
	}
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```
以上代码中，每次切割一部分 data 的数据运行，剩下的放入下一个事件流中执行，
这个方法也在一定程度上缓解了当 data 数据量过多时，会堵塞交互的问题。

## Jobs
之前有提到说 event loop 队列，它表示浏览器执行 JS 的顺序，在 ES6 中
加入了新的 API Promise , Promise 操控一个叫 "Job" 的队列。

原作者在这里写了一大段话，简单来说就是 Job 于 event 共处于 一个 loop 中， 
Job 是 VIP 它在不管在队列中哪个位置，它总是优先运行。

而 Promise 返回的就是一个 job 事件，添加到 event loop 中。

作者还举了一个简单的栗子：
``` javaScript
console.log( "A" );

setTimeout( function(){
	console.log( "B" );
}, 0 );

// theoretical "Job API"
schedule( function(){
	console.log( "C" );

	schedule( function(){
		console.log( "D" );
	} );
} );
```
在寻常情况中，你可能会以为输出的是 A B C D 。但是实际上是 A C D B，因为
Job 事件是 VIP 嘛。

详细细节再第三章中会详细讨论。

## Statement Ordering
这里有一个有趣的事情。

js 引擎不一定会按我们声明语句的格式去执行我们声明的语句... 呃，有点拗口
，有点陌生，没关系，下面将会详细讲到。

在这之前，需要强调的是，在 javaScript 中语法有详细的规定，我们接下来说的东西
在你的代码中是起作用，但是却是观察不到的。

考虑以下代码：
``` javaScript
var a, b;

a = 10;
b = 30;

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```
上面代码中没有异步语法，正常情况下，它是从上到下一行一行执行过来的。

但是到了执行的时候，js 引擎为了更高的效率，会预编译你的代码（详情见 Scope & Closures 一章）。

例如变成：
``` javaScript
var a, b;

a = 10;
a++;

b = 30;
b++;

console.log( a + b ); // 42
```
或者：
``` javaScript
var a, b;

a = 11;
b = 31;

console.log( a + b ); // 42
```
甚至：
``` javaScript
// because `a` and `b` aren't used anymore, we can
// inline and don't even need them!
console.log( 42 ); // 42
```
所有的这些，都是 js 引擎采取的安全的编译，所以它运行效率更高，而且结果并不会改变。

但是有一些情况，这些编译变变得有点问题。
``` javaScript
var a, b;

a = 10;
b = 30;

// we need `a` and `b` in their preincremented state!
console.log( a * b ); // 300

a = a + 1;
b = b + 1;

console.log( a + b ); // 42
```
还有一些情况，编译会造成可见的影响。
``` javaScript
function foo() {
	console.log( b );
	return 1;
}

var a, b, c;

// ES5.1 getter literal syntax
c = {
	get bar() {
		console.log( a );
		return 1;
	}
};

a = 10;
b = 30;

a += foo();				// 30
b += c.bar;				// 11

console.log( a + b );	// 42
```
如果没有 console.log(...) 语句，一切都很好，但是如果有。
``` javaScript
// ...

a = 10 + foo();
b = 30 + c.bar;

// ...
```

## Review
javaScript 代码会分成几个代码块按顺序执行，几个代码块共享整个程序的资源，前一个
代码块的执行可能会对后面的代码块有影响。

event loop 总是按添加顺序从上到下执行，每次迭代叫一次 “tick”。

js 引擎每次只能执行一个任务，每次执行的时候，可能会添加更多的任务进入 event loop。

当一个或多个任务交互运行且共享资源时，可能会出现一系列并发问题，此时需要人为优化。
