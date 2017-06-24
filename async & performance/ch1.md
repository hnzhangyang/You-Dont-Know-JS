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
这里不想也不会解释 **console** 这个方法的运行机制，因为它不是 javaScript 的规范，实际上， **console** 这个方法是由宿主环境，也就是浏览器提供的（详细请看 Types & Grammar 一节）。

**console** 方法的行为取决于浏览器爸爸，他怎么高兴怎么来，这容易导致一些功能上的分歧，或许会使你困惑。

有时候，在一些浏览器，或者满足某种条件下， **console.log(...)** 不会立即输出你想要的值。主要原因是因为 I/O 接口非常缓慢，它阻塞了程序的执行（不单单是阻塞JS），为了避免这种情况，浏览器爸爸选择异步执行 **console.log(...)** 方法，这件事以前大概你没听过（反正我是没听过）。

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
通常情况下，我们希望 **console.log(...)** 能准确的输出 a 的值({index: 1})。事实上，由于后面的 **a.index++** 修改了 a 的值，我们实际上看到的是 {index: 2}。

为什么呢？上面说到浏览器的 **console.log(...)** 采用的是异步处理，又因为 **I/O** 接口比较慢，正当浏览器后台处理这个 **I/O** 接口的时候，**a.index++** 执行了... 所以输出来的值就是 {index: 2}。

有时候在 debug 的时候，如果出现了 **console.log(...)** 与预期不符的情况，就考虑下是不是因为 异步 **I/O** 接口缓慢，从而执行了其他的代码导致的。（我就碰到过这事，一直调试到7点半...）。

注意：如果你真的遇到了这种情况，有两条路供你选择，其一，使用断点代替 **console.log(...)**，其二，使用 **JSON.stringify(...)** 代替 **console.log(...)**， 输出对象的 JSON 字符串。

## Event loop
怎么说，其实不管你的异步代码看起来多么的清晰直接，其实直到现在的 **ES6**，在 **javaScript** 的内部并没有相关的异步的语法概念。

其实 **javaScript** 引擎只做意见事，给定一个时间点，让它执行一段代码块。

谁给?浏览器爸爸。

其实 **javaScript** 引擎并不是单独运行，它依赖于运行的环境，对于大部分开发者（就像我）来说，这个环境是浏览器爸爸。当然现在 **node** 的出现让 **javaScript** 有能力在服务端运行，这是后话了。。。

浏览器爸爸（下面简称 **js环境**）有自己的机制去控制何时触发 **js引擎**，这个机制叫 **event loop**。 **javaScript** 把整个代码切分成一个个单独的代码块，供 **js环境** 调用，每个单独的代码块我们用 **event** 表示。

举个栗子，当你的 **javaScript** 程序制造了一个 **ajax** 请求去服务器获取数据，
通常情况下你会写一个 **callback**，让后告诉 **js环境**，‘嘿，这段代码你先
不要执行，等到我的 **ajax** 请求获取到数据之后了，你再执行’。

然后 **js环境** 就会监听这个请求，获取到数据之后将之前悬置的 **callback** 
加入 **event loop**。

**event loop** 怎么玩？请看下面代码。
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
你可以看到，我设置了一个永远不会结束循环的 **while 循环**，在 **while 循环**
 中，每一次迭代我们叫一次 **tick**，每个 **tick** ，如果 **eventLoop** 里面
 有 **event**，取出来执行。上面说的你设置的 **ajax** 请求的 **callback** 
 就是加入到当前的 **eventLoop** 末尾。

 所以单独把 **event** 加入到 **eventLoop** 中，**event** 可能并不会马上执行。
 如果在 **event** 之上还有 20 个 **event**，那么你的 **callback** 将会等到
 那 20 个  **event** 执行完毕了之后再执行。这就解释了 **setTimeout(...)** 
 为什么有时候并不是准确的按时间执行，因为在它定义的 **callback** 执行之前，可能
 还有别的程序正在执行。使用 **setTimeout(...)** 你只能保证它在某个时间点之前不
 会执行，不能保证在某个时间点一定会执行。

 你编写的 **javaScript** 代码，被分成若干个 **event** 之后加入到 **event loop**
 中来，按  **event loop** 的顺序执行。

 注意：我们刚才提到的 **ES6**,它有一个 API **Promise** 提供了更好的 **event loop** 管理
 能力，它将 **event loop** 的控制权从浏览器爸爸手中抢了回来，这个在第三章再说。

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



