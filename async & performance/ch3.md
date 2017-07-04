#You Don't Know JS: Async & Performance
## Chapter 3: Promises
在第二章中，我们讨论了使用 callback 作为异步回调的两个主要缺陷。

第一个问题是控制权的问题，有时候我们把程序的控制权交给第三方插件，等待某一个时刻，第三方插件
会调用我们写好的 callback。一般情况下，我们并不会去仔细阅读第三方插件的源码，所以说一般这种
时候，程序的运行并不在我们的掌握之中，（对我来说，这事实在是不太舒服）。

现在有一个替代的选项，可以不交出控制权，取而代之的是当第三方插件完成了它自己的任务之后，返回
一个状态让我们知道，我们自己处理剩下的逻辑，这样听起来是不是就很好了？

Promise 就是干这事的。

Promise 推出就引发了轰动，开发者终于可以不用忍受难看的 “callback hell”。事实上，越来越多的
异步 API 开始建立在 Promise 之上（截至目前，Promise 支持得越来越好，特别是移动端），所以现在
是时候开始学习 Promise 了。

## What Is a Promise?
你有没有过这样的体验，公司要求你学习一门新技术，不管三七二十一，先看源码，看了源码就开始练习，
边练习边体会新技术，很多人都是这么干的（比如说我）。

但是有时候你又会感到非常之困惑，有些技术的 API 实在是太多，太抽象，太枯燥乏味。在你搞懂它具体的思路或者为什么会出现之前，
基本上看源码就处于懵逼的状态。 Promise 就属于这样一门技术。

所以在真正介绍 Promise 之前，原文作者为了帮助大家理解 Promise 的思想，特地举了很多的贴近生活的栗子，让我们一起往下看吧。

### Future Value
因为国情不同，本栗子做了适当的国产化~~

一天中午，你饥肠辘辘，搞完手头的工作后终于可以去吃饭。你来到了最喜欢的黄焖鸡米饭，点了一份大分加辣。

如果不是奸商的话，黄焖鸡米饭当然不会马上送过来（奸商才在一开始就做一大锅给客人吃剩的）。一般情况下，你给钱之后会得到一个号码牌，方便服务员送餐。

这个号码牌就相当于一个 Promise，它是黄焖鸡的凭证。

过个十几分钟，服务员将你的黄焖鸡送过来，拿走了号码牌。

或者服务员过几分钟来告诉你，今儿不好意思，黄焖鸡卖完了，要不来个黄焖排骨？

可见，你拿了号码牌代表未来会有一个结果，这个结果可能好可能坏，可能是黄焖鸡，也可能是黄焖排骨，但不管怎么样，服务员都会来通知你。

### Values Now and Later

你可能觉得上面这个栗子跟你敲代码没有半毛钱关系，没关系，继续往下看。

**Promise 是关于未来的断言**。在真正解释 Promise 之前，我们先回顾下前两章的栗子，温故知新，我们用 callback 导出 Promise 的特性。

想象当你写一个表达式时，比如 + 操作符。其实你已经确定 + 操作符两边的变量都已经时确定值。

``` javaScript
var x, y = 2;

console.log( x + y ); // NaN  <-- because `x` isn't set yet
```
如上所说， 在使用 + 操作符的时候时假设其两边的变量都以确定（resolved）。

如果不是，那就有点奇怪了，假设其中有一个变量是已经确定，而另一个是在未来某一个时候才确定，你能想象 + 操作符会等待另一个值确定的时候再将
两个值相加吗？

又比如说你怎么看待两个值其中有一个或两个都是现在不确定的值，假设第二个值的确定依赖第一个值，如果第一个值确定了之后，第二个值根据第一个值
推倒得出。或者第一个值获取失败，相对的，第二个值也获取失败。

两个值相互依赖，是不是有点像第一章举的栗子？

回到上面这个栗子，我们现在的条件是，执行 x + y。如果 x 或者 y 有一个不确定，请等待其确定了之后再执行相加，你会怎么写代码？（用 callback 的形式）
``` javaScript
function add(getX,getY,cb) {
	var x, y;
	getX( function(xVal){
		x = xVal;
		// both are ready?
		if (y != undefined) {
			cb( x + y );	// send along sum
		}
	} );
	getY( function(yVal){
		y = yVal;
		// both are ready?
		if (x != undefined) {
			cb( x + y );	// send along sum
		}
	} );
}

// `fetchX()` and `fetchY()` are sync or async
// functions
add( fetchX, fetchY, function(sum){
	console.log( sum ); // that was easy, huh?
} );
```
好吧，上面的异步代码看起来比较不优美。我们将 x ，y 都看作是异步的值。而 add(...) 函数并不关心他们是异步还是同步，只有当两个值都确定时，
才会输出他们的和。

甚至可以将 x，y 都看做是异步的值，尽管他们可能是同步值，反正 add(...) 不 care。这样让我们更容易读懂代码。

当然，像上面那样使用基于 callback 的方法处理异步值看起来很粗糙，但是它不妨碍我们理解**像使用同步值一样使用异步值**的优势。

### Promise Value
关于 Promise 的细节我们会在文章的稍后部分读到，现在有点困惑没关系，先扫一眼下面的代码，看用 Promise 怎么处理上面的 x + y 的栗子。
``` javaScript
function add(xPromise,yPromise) {
	// `Promise.all([ .. ])` takes an array of promises,
	// and returns a new promise that waits on them
	// all to finish
	return Promise.all( [xPromise, yPromise] )

	// when that promise is resolved, let's take the
	// received `X` and `Y` values and add them together.
	.then( function(values){
		// `values` is an array of the messages from the
		// previously resolved promises
		return values[0] + values[1];
	} );
}

// `fetchX()` and `fetchY()` return promises for
// their respective values, which may be ready
// *now* or *later*.
add( fetchX(), fetchY() )

// we get a promise back for the sum of those
// two numbers.
// now we chain-call `then(..)` to wait for the
// resolution of that returned promise.
.then( function(sum){
	console.log( sum ); // that was easier!
} );
```
上面代码中有两个方面需要注意。

第一，我们直接调用 fetchX() 于 fetchY()，将两个函数的返回值（Promise值）直接传入 add(...)。这两个返回值或许是同步，或许是异步，没关系。

第二，add(...) 函数体内返回了 Promise.all([...]) 语句，Promise.all([...]) ，Promise.all([...])执行完成之后会调用其后的 then(...) 语句。当 add(...) 函数执行完毕之后，
“x” 和 “y” 就已经取到值了，我们可以直接拿来用。可能你也发现了，add(...) 函数隐藏了等待 “x” 和 “y” 的逻辑。

注意：在 add(...) 函数体内，Promise。all([...]) 语句也创造了一个 Promise，这个 Promise 等待 promiseX 和 promiseY 的完成。同样的.then(...) 语句也会创造一个新的 Promise，这个
Promise 返回 values[0] + values[1]。同理，add(...) 函数体外跟随的 .then 也会返回一个 Promise。关于这部分的细节将在下文中详细讲解。

就像黄焖鸡米饭，一个 Promise 返回的结果也有两种，成功（rejection）或失败（fulfillment）。

在 Promise 中，then(...) 语句有两个函数参数。第一个参数是 Promise 成功时的回调，第二个参数是 Primise 失败时的回调。
``` javaScript
add( fetchX(), fetchY() )
.then(
	// fullfillment handler
	function(sum) {
		console.log( sum );
	},
	// rejection handler
	function(err) {
		console.error( err ); // bummer!
	}
);
```
上面的代码中，如果基于某种不明原因获取 X 或 Y 失败，promise（add(...)函数）的状态变为 rejected。.then 语句的第二个参数函数将会被调用。

**Promise 有自己的内部状态，初始状态:正在运行（pending），以及结束状态：成功（fulfillment）或者失败（rejection）。一旦状态由初始状态变成结束状态（不管是 fulfillment 还是 rejection），其状态都不可再被改变**。

理解这一点对理解 Promise 很重要，因为同样的特性，用 callback 实现起来会显得特别冗长乏味，不是一个明知的选择。

### Completion Event
跟上面说到的一样， Promise 是一个关于未来的值。你也将 Promise 理解为一个控制多个异步的流程控制器。

想象我们调用一个函数 foo(...) 去执行一个任务。我们不需要知道也不需要关心 foo(...) 函数体的任何细节。它可能是同步也可能是异步，这些都不重要，我们只需要确定的是，当它执行完成的时候能给我们一个信息，好让我们去做接下来该做的事。

在传统 javaScript 中，如果你需要去监听一个事件，最容易联想到的就是事件绑定机制（比如给 dom 节点绑定 click 事件）。那么我们监听一个事件也可以看作是一个事件机制，只不过这个事件是由 foo(...) 函数发出，而不是用户点击了 dom 节点。

思考下面代码。
``` javaScript
foo(x) {
	// start doing something that could take a while
}

foo( 42 )

on (foo "completion") {
	// now we can do the next step!
}

on (foo "error") {
	// oops, something went wrong in `foo(..)`
}
```
我们调用 foo(...) 函数，并且设定了两个事件监听，一个是监听 "completion" 事件，另一个是监听 "error" 事件。调用哪个事件取决于 foo(...) 的执行结果。事实上，foo(...) 函数不需要关心是否在其之后订阅了事件，事件的订阅与 foo(...) 函数的执行是分离开来的。

当然，这种语法是不是看起来很怪异，你没写过这样的语法，JS 引擎也没有这种语法，这是原作者虚构出来的语法 : )

为了让代码看起来更加自然，作者又优化成了以下代码。

``` javaScript
function foo(x) {
	// start doing something that could take a while

	// make a `listener` event notification
	// capability to return

	return listener;
}

var evt = foo( 42 );

evt.on( "completion", function(){
	// now we can do the next step!
} );

evt.on( "failure", function(err){
	// oops, something went wrong in `foo(..)`
} );
```
经过改造后，foo(...) 函数返回了一个有能力接收订阅事件的对象，这个对象接收了两个事件，“completion”，“failure”。

跟 callback 的写法有点不同，foo(...) 函数不需要我们将 callback 作为参数传递给它，取而代之的是。它返回了一个可以接受 callback 函数的对象。

在上一章提到过说使用 callback 就意味这代码控制权的转移，而现在我们没有使用 callbcak  而是使用可接受监听事件的对象，这样一来控制权还是在我们手上（表达的可能不大清楚，各位仔细理解下）。

我们的代码现在可以监听 foo(...) 函数的执行，当 foo(...) 执行完成的时候我们可以立即调用之前注册的监听的回调函数。

``` javaScript
var evt = foo( 42 );

// let `bar(..)` listen to `foo(..)`'s completion
bar( evt );

// also, let `baz(..)` listen to `foo(..)`'s completion
baz( evt );
```
这样做的好处就是 bar(...) 与 baz(...) 不需要关注 foo(...) 是怎样执行，foo(...) 也不需要关注是不是有 bar(...)，baz(...) 监听他的执行。

evt 对象就像一个中立的第三方控制器一样联接着我们的代码。

### Promise "Events"

可能你也注意到了，evt 对象监听事件的能力有点像 Promise。关于这部分的细节将在下文中详细讲解。

在 Promise 中，上面的代码要改写成 foo(...) 函数返回一个 Promise 实例，然后将这个实例当作参数传递给 bar(...) 和 baz(...)。

注意：Promise 监听事件和上面所说的 event 监听事件还是有区别的（虽然他们的行为看起来很像），Promise 不会监听 “completion” 和 “error” 事件，取而代之的是 Promise 使用 then(...)
去注册一个 “then” 事件。更准确的说，即使没有明确的代码表示，但是其实 then(...) 是注册了两个事件，一个是 “fulfillment”，另一个是 “rejection”。

考虑如下代码：
``` javaScript
function foo(x) {
	// start doing something that could take a while

	// construct and return a promise
	return new Promise( function(resolve,reject){
		// eventually, call `resolve(..)` or `reject(..)`,
		// which are the resolution callbacks for
		// the promise.
	} );
}

var p = foo( 42 );

bar( p );

baz( p );
```
注意：new Promise( function(..){ .. } ) 是一个调用一个构造函数，这个构造函数接受两个参数，一个是 “resolve”，还有一个是 “reject”。这两个参数是用来改变 Promise
的状态的，执行 resolve(...) 将 Promise 的状态改变为 “fulfillment”，执行 reject(...) 将 Promise 的状态改变为 “rejection”。

在 bar(...) 和 baz(...) 内部是这样的：
``` javaScript
function bar(fooPromise) {
	// listen for `foo(..)` to complete
	fooPromise.then(
		function(){
			// `foo(..)` has now finished, so
			// do `bar(..)`'s task
		},
		function(){
			// oops, something went wrong in `foo(..)`
		}
	);
}

// ditto for `baz(..)`
```
Promise 可以像上面这样，仅仅作为联接两段代码的联接器。

思考下面代码：
``` javaScript
function bar() {
	// `foo(..)` has definitely finished, so
	// do `bar(..)`'s task
}

function oopsBar() {
	// oops, something went wrong in `foo(..)`,
	// so `bar(..)` didn't run
}

// ditto for `baz()` and `oopsBaz()`

var p = foo( 42 );

p.then( bar, oopsBar );

p.then( baz, oopsBaz );
```
注意：如果你仔细阅读了先前的栗子的话，上面代码的最后两行，你或许会写成 p.then( .. ).then( .. )。这个是需要注意的地方，p.then(..); p.then(..) 与 p.then( .. ).then( .. )
有很大的不同，这个会在稍后解释。

相对于将 Promise 当作参数传递给 bar(...)，baz(...)，我们现在使用 Promise 来控制 bar(...) 与 baz(...) 的触发，这两个方式有一个区别就是，当 Promise 出现问题时，后者可以更方便地做相应处理。

在上面的第一段代码中，不管 foo(...) 函数运行完成还是运行失败， bar(...) 都会被触发，如果需要处理 foo(...) 运行失败的逻辑，需要将代码卸载 bar(...) 函数内部。baz(...)
同理。

第二个栗子，只有当 foo(...) 成功运行时，才会触发 bar(...)，如果 foo(...) 运行出现错误，会触发 oopsBar(...)，同理 baz(...)。

上面两种方式都是对的，具体采用哪种方式需要参考那时的代码环境决定。

不论什么情况，foo(...) 返回的 Promise 都是用来处理 foo(...) 运行完之后的逻辑。

此外，一旦 Promise 的状态被决定，可以按需求多次调用 Promise 实例的 then 方法，Promise 方法的状态是不变的。

### Thenable Duck Typing
有一点需要注意的是，如何区分一个对象是否是 Promise 对象？还是这个对象只是一个跟 Promise 对象行为很相似的值？

我们是通过构造函数 Promise 来创建 Promise 实例的（new Promise()），或许你的第一反应就是通过 p instanceof Promise 来判断一个对象是不是 Promise 对象。

想法很好，但是（总是有但是），有特殊情况，我们知道**任何构造函数都是当前框架下的 window 对象的属性**。这也就是说如果当一个页面里面有两个框架（比如用了 iframe），那么两个页面的 widnow.Promise
其实是不同的，这也就是说如果你的 promise 对象是从父框架传过来的，因为两个框架的 Promise 不同，所以此时 p instanceof Promise 返回的结果也是 false，即使 p
的确是一个 Promise 实例。

此外，因为支持 Promise 的浏览器还不是很全，在 Promise 普及过程中，一些第三方框架难免要自己写一个类 Promise 的对象，以用在一些不兼容 Promise 的浏览器上。

在文章的稍后部分我们详细解释 Promise 运行机制的时候，你就会发现鉴别 类Promise 对象是很重要的，现在先不管细节，只需要知道这很重要...

一般而言，我们鉴别 类Promise 对象的方法就是看一个对象是否拥有 then 方法，如果有，那么它就是一个 类Promise 对象。这种检测的方法称为 “duck typing”，鸭子检测...
如果一个动物长的像鸭子，并且叫声也像鸭子，那么，它就是鸭子。所以我们的鉴别方法可以写成下面这样：
``` javaScript
if (
	p !== null &&
	(
		typeof p === "object" ||
		typeof p === "function"
	) &&
	typeof p.then === "function"
) {
	// assume it's a thenable!
}
else {
	// not a thenable
}
```
请原谅这么粗糙的检测方法，实际上，它不只粗糙，还有很多问题。

显而易见，像上面那样的检测方法会把一些不是 类Promise 的对象当成是 类Promise 对象。只要这个对象拥有 then 方法。

比如说下面：
``` javaScript
var o = { then: function(){} };

// make `v` be `[[Prototype]]`-linked to `o`
var v = Object.create( o );

v.someStuff = "cool";
v.otherStuff = "not so cool";

v.hasOwnProperty( "then" );		// false
```
v 肯定不是 类Promise 对象吧？就是因为 v 继承了 o 对象的原型链，所以他也具有 then 方法， 也被当成了 Promise 对象。

有时候你甚至不会察觉到某个对象会被当成是 类Promise 对象。

``` javaScript
Object.prototype.then = function(){};
Array.prototype.then = function(){};

var v1 = { hello: "world" };
var v2 = [ "Hello", "World" ];
```
v1 和 v2 都被当成了 类Promise 对象。甚至可以说所有的对象和数组都被当成了 类Promise 对象。像这样在 prototype 上面定义 then 方法，不管你是有意还是无意，都会对我们的检测造成干扰。

是不是觉得很不可思议？（其实我一点不觉得）

要注意的是远在 ES6 出来之前，就有很多第三方框架使用了 类Promise 的方法，ES6 出来之后，其中有一个部分将原来的方法改写以避免冲突，有一些则不。

宗上所述，使用像上面那种方法检测 类Promise 是不可靠的。

## Promise Trust
经过上面几个栗子，我们大概了解了 Promise 的运作方式，但是如果仅仅是了解这些，就错失了 Promise 最重要的部分。

还记的我们在第二章时运用第三方工具使用 callback 编写异步代码可能会出现的几个问题吗？
- 调用 callback 太早
- 调用 callback 太晚，或者干脆没有调用
- 重复调用 callback
- 调用 callback 时传入了错误的参数
- 未知的错误发生

Promise 可以很好的解决这几个问题，看我们是怎么做的。

### Calling Too Early
造成这个问题的原因可以归根与在程序逻辑中有资源竞争，有时候我们需要异步加载两个资源，在两个资源都加载完成后再调用回调，这时如果不设置条件处理的话，可能在第一个资源加载的时候就启用了回调。

Promise 天生可以解决这个问题，因为 Promise 的代码都是异步代码，回调函数会放到下一个事件流执行，就像我们使用 setTimeout(fn,0) 一样。

### Calling Too Late
跟上面一样，因为 Promise 的回调函数都是异步执行，所以不会出现像同步代码一样链式回调阻塞代码，比如说下面这个栗子。
``` javaScript
p.then( function(){
	p.then( function(){
		console.log( "C" );
	} );
	console.log( "A" );
} );
p.then( function(){
	console.log( "B" );
} );
// A B C
```
输出的结果是 A B C ，C 不会打断 B 的执行。

### Promise Scheduling Quirks
有一点值得注意的是，当多个 Promise 共存时，他们的 callback 的执行顺序可能不时你预料的那样。

比如说有两个 Promise，p1 和 p2，都已经执行完成。如果按 p1.then(...) p2.then(...) 的方式设置回调，你是不是想像执行也应该是 p1 的回调函数先执行。其实不一定，看下面代码。
``` javaScript
var p3 = new Promise( function(resolve,reject){
	resolve( "B" );
} );

var p1 = new Promise( function(resolve,reject){
	resolve( p3 );
} );

var p2 = new Promise( function(resolve,reject){
	resolve( "A" );
} );

p1.then( function(v){
	console.log( v );
} );

p2.then( function(v){
	console.log( v );
} );

// A B  <-- not  B A  as you might expect
```
关于这方面，在下文会有详细解释，但是你应该也注意到了，在 p1 中执行的 resove() 方法时，并没有传入一个确定的值，取而代之传入的是另一个 Promise p3。p3
执行完成时才返回值 “B”。在这种情况下 p1 还需等待 p3 执行完成，才能调用回调函数，所以 p2 的回调函数会在 p1 之前调用。

在实际编码中，为了避免此类问题，最好不要将一个 Promise  当作参数传入另一个 Promise 中。

### Never Calling the Callback
这是一个非常普遍的情况。

首先需要说明的是，没有任何代码可以干扰当 Promise 状态改变时，执行它的回调函数，就算是在执行过程中出现程序错误，Promise 也可以执行状态变为 “rejection” 时的回调。

然而你可能会问，如果 Promise 一直执行下去，一直没有改变状态，那么 callback 不就一直不会调用了吗？要解决这个问题很简单，只需要借助 Promise.race 设定一个超时条件，
当 Promise 运行时间超过我们预设的值时，自动调用 “rejection” 即可。

``` javaScript
// a utility for timing out a Promise
function timeoutPromise(delay) {
	return new Promise( function(resolve,reject){
		setTimeout( function(){
			reject( "Timeout!" );
		}, delay );
	} );
}

// setup a timeout for `foo()`
Promise.race( [
	foo(),					// attempt `foo()`
	timeoutPromise( 3000 )	// give it 3 seconds
] )
.then(
	function(){
		// `foo(..)` fulfilled in time!
	},
	function(err){
		// either `foo()` rejected, or it just
		// didn't finish in time, so inspect
		// `err` to know which
	}
);
```
### Calling Too Few or Too Many Times

一般而言，callback 只需要执行一次，那么 “Too Few” 就代表 callback 一次也没有执行，这种情况上面我们已经讨论过了。

“Too many” 也很好理解， callback 执行了两次以上，这通常会导致错误（比如重复提交订单）。还记得上面说到过， Promise 状态一旦改变，不再可更改。这也就是说，Promise
的回调函数只会执行一次，如果你试图多次改变 Promise 的状态（resolve(...) 或 reject(...)），只有第一次改变状态时有效，后面的都会被忽略。

当然，如果你注册多个回调（比如 p.then(...);p.then(...)），Promise 在状态改变时也会执行你所有绑定的回调，它只是保证不会重复的执行你绑定的回调函数。

### Failing to Pass Along Any Parameters/Environment
在 Promise 内部执行 resolve 或者 reject 方法时，只可以带一个参数，供相应的回调函数使用，剩下的参数将会被忽略。（Promise 就是这么定义的我也没办法），不过作者给出了两个解决方案。

第一你可以将所有的参数包裹在一个变量或者数组里面，这很好理解。

其二就是闭包机制同样对 Promise 适用，在你可以使用闭包来解决此类问题。

### Swallowing Any Errors/Exceptions
在文章上面也有提到过，如果你在 Promise 中调用了 reject(...) ，Promise 的状态将会变为 “rejection”，相应的回调函数也会调用，而在使用 reject(...) 时传入的参数也会传如回调函数中。

并且如果 Promise 在执行时如果遇到错误（比如 TypeError 或者 ReferenceError），也会强制将 Promise 转为 “rejection” 状态，例如：
``` javaScript
var p = new Promise( function(resolve,reject){
	foo.bar();	// `foo` is not defined, so error!
	resolve( 42 );	// never gets here :(
} );

p.then(
	function fulfilled(){
		// never gets here :(
	},
	function rejected(err){
		// `err` will be a `TypeError` exception object
		// from the `foo.bar()` line.
	}
);
```
foo.bar() 导致了 Promise 状态变为 “rejection”。

但是如果 Promise 本体执行成功，在回调函数中出现的错误要怎么办呢？
``` javaScript
var p = new Promise( function(resolve,reject){
	resolve( 42 );
} );

p.then(
	function fulfilled(msg){
		foo.bar();
		console.log( msg );	// never gets here :(
	},
	function rejected(err){
		// never gets here either :(
	}
);
```
在 then() 中注册的回调函数中出现错误，我们就捕获不到了吗？答案当然时否定的，还记得上面说到过 then(...) 方法也会返回一个新的 Promise 吗？

你可能也想到了，在 then() 方法中出现的错误会使返回的 Promise 状态变为 “rejection”，我们可以在新的 Promise 上面注册回调来捕获错误。

### Trustable Promise?
关于 Promise 还有一点需要了解。

上文一直拿 Promise 与 callback 进行对比，但并不是说使用 Promise 就不要使用 callback 了，而是说将 callback 放入 Promise 中，而不是放入 foo 函数中。

但是为什么我们说使用 callback 比单独使用 callback 更可靠？我们怎么判断通过某个方法返回过来的值是 Promise？

解决这个问题我们需要使用到 Promise 的原生方法 Promise.resolve()。

因为如果使用 Promise.resolve() 的话，即使你传递一个既不是 Promise 也不是 tenable（类Promise）的直接值，你也会得到新的且状态为 “fulfilled”
的 Promise。换句话说，下面两个 Promise 的效果是一样的：
``` javaScript
var p1 = new Promise( function(resolve,reject){
	resolve( 42 );
} );

var p2 = Promise.resolve( 42 );
```
如果你传递一个 Promise 给 Promise.resolve(),那么 Promise.resove() 会将传递的这个 Promise 原封不动的返回:
``` javaScript
var p1 = Promise.resolve( 42 );

var p2 = Promise.resolve( p1 );

p1 === p2; // true
```
又或者，你仅仅是将一个 thenable 对象传递给 Promise.resolve()。它将会调用 thenable 对象的 then 方法求值。

回顾一下下上面我们所讲到的 thenable 对象。
``` javaScript
var p = {
	then: function(cb) {
		cb( 42 );
	}
};

// this works OK, but only by good fortune
p
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// never gets here
	}
);
```
P 是一个 thenable 对象，但是不是真正的 Promise 对象。但是似乎还有点不全，或者我们可以这样写：
``` javaScript
var p = {
	then: function(cb,errcb) {
		cb( 42 );
		errcb( "evil laugh" );
	}
};

p
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// oops, shouldn't have run
		console.log( err ); // evil laugh
	}
);
```
P 对象或许是一个 thenable 对象，但是却没有 Promose 的表现，它会同时调用两个回调函数。

想象下我们将 P 对象传递给 Promise.resolve() 会发生什么？
``` javaScript
Promise.resolve( p )
.then(
	function fulfilled(val){
		console.log( val ); // 42
	},
	function rejected(err){
		// never gets here
	}
);
```
上面提到过如果将一个 thenable 对像传递给 Promise.resolve(),它将会调用这个 thenable 对象的 then 方法，并且返回一个 Promise。如果传递一个 Promise
Promise.resolve() 将会将这个 Primise 对象返回。

还记得上面的 foo() 方法吗？我们不确定 foo() 方法是不是能返回一个可靠的 thenable 对象，但是至少是一个 thenable 对象，所以我们可以使用 Promise.reslove()
方法将其转化成一个真正可信赖的 Primise 对象。

``` javaScript
// don't just do this:
foo( 42 )
.then( function(v){
	console.log( v );
} );

// instead, do this:
Promise.resolve( foo( 42 ) )
.then( function(v){
	console.log( v );
} );
```

### Trust Built
在前面的栗子中我们讨论了使用 Promise 编写一个健壮的可维护的程序是非常可靠的。尽管我们使用不可靠的 callback 构建程序已经将近二十年了。

又或许你也早就发现了 callback 的种种不足...

Promise 提高了使用 callback 的可靠性，也不会将代码控制权转移到第三方插件，借此我们可以编写更清晰的更可靠的异步代码。

## Chain Flow
对于 Promise 的链式调用在上文已提及多次，这代表着我们可以通过联接多个 Promise 来编写链式的异步操作。

关于 Promise 的链式调用，主要由下面两点提供：
- 每次你调用 then 方法，它都会返回另一个 Promise 供你调用。
- 不管你在 then 方法里面 return 什么值，都会将返回的 Promise 的状态设置为 “fulfilled”

解释下第一点，主要是它给我们提供的链式调用功能：
``` javaScript
var p = Promise.resolve( 21 );

var p2 = p.then( function(v){
	console.log( v );	// 21

	// fulfill `p2` with value `42`
	return v * 2;
} );

// chain off `p2`
p2.then( function(v){
	console.log( v );	// 42
} );
```
通过 return v*2，p2（由第一个 then 方法返回新 Promise）的状态被设定为 “fulfilled”，并且值 42 将会被 p2 的 then 方法接收，当然 p2.then() 也会返回另一个 Promise
，它能被 p3 接收（假如你想的话）。

或许你觉得 p2，p3 这些中间值不太好看，你也可以直接这么写：
``` javaScript
var p = Promise.resolve( 21 );

p
.then( function(v){
	console.log( v );	// 21

	// fulfill the chained promise with value `42`
	return v * 2;
} )
// here's the chained promise
.then( function(v){
	console.log( v );	// 42
} );
```
第一个 then 是链式调用的第一步，第二个 then 是链式调用的第二步，不管多少步，只要你愿意，都是可以的。

但是我们上面使用的都是直接值，第一步执行完毕后马上执行第二步，第二步执行完毕后马上执行第三步等等，如果我们像链式调用异步代码呢？

上面的问题的诀窍在于使用 Promise.resolve() 方法。

``` javaScript
var p = Promise.resolve( 21 );

p.then( function(v){
	console.log( v );	// 21

	// create a promise and return it
	return new Promise( function(resolve,reject){
		// fulfill with value `42`
		resolve( v * 2 );
	} );
} )
.then( function(v){
	console.log( v );	// 42
} );
```
即使我们将 42 放进一个 Promise 中再返回该 Promise，链式调用的第二步也能接收到这个值，同理，我们可以将异步代码写进 Promise 中再返回。
``` javaScript
var p = Promise.resolve( 21 );

p.then( function(v){
	console.log( v );	// 21

	// create a promise to return
	return new Promise( function(resolve,reject){
		// introduce asynchrony!
		setTimeout( function(){
			// fulfill with value `42`
			resolve( v * 2 );
		}, 100 );
	} );
} )
.then( function(v){
	// runs after the 100ms delay in the previous step
	console.log( v );	// 42
} );
```
按照上面的代码，我们可以在 Promise 中包含任意数量的异步代码，它都是有效的。

当然，在 resolve() 中传入的参数是可选的，如果不传，默认是 undefined。传不传参数都不会影响到链式调用。

为了进一步的说明，我们现在编写一个具有延迟功能的 Promise 工具。
``` javaScript
function delay(time) {
	return new Promise( function(resolve,reject){
		setTimeout( resolve, time );
	} );
}

delay( 100 ) // step 1
.then( function STEP2(){
	console.log( "step 2 (after 100ms)" );
	return delay( 200 );
} )
.then( function STEP3(){
	console.log( "step 3 (after another 200ms)" );
} )
.then( function STEP4(){
	console.log( "step 4 (next Job)" );
	return delay( 50 );
} )
.then( function STEP5(){
	console.log( "step 5 (after another 50ms)" );
} )
...
```
调用 delay(200) 将会生成一个 200ms 后才会改变状态的 Promise，就像我们在第一个 then 中调用 delay(200)，第二个 then 将会等待 200ms 才会执行。

注意：