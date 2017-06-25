# You Don't Know JS: Async & Performance
## 第二章 Callbacks
在第一章，我们了解了 javaScript 的异步语法，和 javaScript 的单线程事件流 event loop。我们还体验了在 javaScript 中的各种各样的并发问题，以及解决方式。

第一章中，我们普遍采用函数作为基本单元，每个函数就时每个功能，用函数去完成需求，其实函数能做的远远不止于此。

在本章中，函数又叫做 callback，不管是简单的还是复杂的 javaScrpit 代码，多多少少会有异步编程，而 callback
 作为异步编程不可或缺的基本单元，被普遍使用。

 callback 也有缺点，很多开发者希望使用 Promise 来编写异步编程的代码，以获得更好的体验。
 但是使用前最好充分理解什么是 Promise 以及它的工作模式。

 在本章中，我们会深入讨论这点，讨论为什么需要诸如 Promise 之类的新异步 API ，
 以及它们的优缺点。
 ## Continuations
 来一起看以下第一章中的简单栗子：
 ``` javaScript
 // A
ajax( "..", function(..){
	// C
} );
// B
 ```
 // A 以及 // B 处的代码优先执行，// C 处的代码作为 ajax 异步请求的 callbak 会在浏览器收到服务器回应后再执行。因为 callback 延迟执行的关系，其实也可以说 callback 延续了程序的执行。

 再看个简单的栗子：
 ``` javaScript
 // A
setTimeout( function(){
	// C
}, 1000 );
// B
 ```
 好了，思维在此停顿，请你用你自己的话描述上述代码的运行方式，大声说出来~ （这是原作者的要求，说是会加深你对他后面将要说的事情的理解）。

 很多读者估计是这样说的，“执行 A ，使用定时器定时 1000 毫秒，1000 毫秒后触发 C”。

 你看到上面的说法后可能会自我修正下说：“执行 A，设置一个定时器，定时 1000 毫秒，执行 B，等 1000 毫秒过后，执行C”。这里的话都是作者猜的 :)

 上面这两句话有什么不同呢？尽管第二句话可能更准确。

 其实上面两句话有个共同的不足是，这都是我们脑袋里想象出来的运行方式，它符合我们人类思考的逻辑。但是对于 JS 引擎来说，它的思考方式与我们不同，它更能理解与管理 callback 异步编程。
 
 正是因为有区别，是们不理解机器的运行方式，只用自己的方式思考，那么在以后代码的编写与维护中就会遇到问题。

 ## Sequential Brain

 很多人都幽默的说自己可以身兼数职，同时干几件事情。说说就罢，其实不太可能。

 各位读者，你同时思考两件事试试？我们的脑袋不是并发的思维模式，一次只能思考一件事情。

 就是这么个设定，我们一次只能干一件事情，即使你可能不承认。

 当然，以上观点不适用于潜意识，你在吃饭的时候潜意识控制你呼吸，这种情况你不能说你同时干了两件事情，吃饭和呼吸，因为呼吸不是我们有意识的行为控制。

 我们讨论的是你自己控制的行为，对我来说，我现在正在控制我的大脑翻译这篇文章，对于作者来说，他正在控制他的大脑编写写篇文章。不管是翻译还是编写文章，我们的大脑都没有
 做其他的事情。

 这里我想插一段话，完全的自我管理很重要！正是因为我们的大脑一次只能干一件事情，不管这件事情是翻译文章还是开小差。开小差的时候就不能翻译，翻译的时候就不能开小差。但是
 有时候我们会不自觉地开小差，这样就影响了翻译效率，浪费了时间，其实开小差这件事实在是不可取。我想真正的做一件事情的效率的提高，情绪投入很重要，比如我，枯燥的翻译实在是无聊，
 为了让翻译显得更有意思，我就有时会写一些不着边际的话，来使我感到快乐，希望各位读者理解。

 回到翻译，有时候我们表面上看起来像是做两件事，其实不是，大脑对两件事的快速转换。比如你边打字边电话聊天，当你聊天的时候，其实是大脑将打字这件事情放到了“后台”，当你电话关闭的时候，
 大脑快速的将打字带回前台，这样你又可以立刻进行打字这件事，表面上看你是边聊天，边打字，实际上不是。

 这有点像我们在第一章所提到的 js 的事件驱动机制。如果你觉得不像，请继续看一边第一章，如果还是觉得不像，请看作者原文。

 事实上，简单来说，就是那么回事。
 
 如果你想象我现在翻译的每个字，都代表一个个异步的事件（event），当我打字的时候，可能有很多另外的事件打扰我打字，比如说突如其来的手机短信。但是我从来不让这些事请打断我，短信就短信
，老子不看。要不然这篇又不知道等到何时何地才能写完了。但是事情实在太多的时候（比如说手机短信，你不看没关系，它提醒你两次），就让我有点痛苦了，这种痛苦 js 引擎也能感觉得到，它也会
遇到很多突然出现的 event。

## Doing Versus Planning

根据上面的结论，我们的大脑可以简单类比为跟 javaScript 的单线程机制一样，拥有 event loop 事件流。

虽然在处理问题的方式上还是跟 js 引擎有些许不同。

就像我现在正在翻译这篇文章一样，我的想法就是翻译翻译再翻译，把我的想法都写下来，不允许有打断。尽管我们把大脑类比为异步事件，实际执行起来还是同步执行。

我们的计划总是直线型的，做完 A 做 B，做完 B 做 C ，B 一定等到 A 做完之后才执行，C 一定等到 A 与 B 都做完后再执行。

当我们写代码的时候，也会定义一系列将要发生的事件，如果是一个好的开发者，对每个事件的细节也应该是了如指掌的。

写同步代码的时候，就像是列出了一个待做事情清单。
``` javaScript
// swap `x` and `y` (via temp variable `z`)
z = x;
x = y;
y = z;
```
上面三行语句都是同步执行， x = y 等待 z = x 执行完成，y = z 等待 x = y 执行完成。这三个语句相当于一个代码块，这里面并没有包含异步代码，如果包含了，事情就会变得有点复杂。

我们的大脑很适合编写同步代码，但是对于异步代码，我们的大脑会怎么计划？

请想象下面一段话
``` html
<h3>我需要去商店，但是在去商店的路上我确定我会接到我妈打来的电话，然后我们开始交谈。我在 GPS 上面搜寻商店的地址，机器搜索需要花费一点时间。所以我将收音机音量调小，以听清楚
我妈说的话。我意识到我忘记带夹克，外面非常冷。但是不要紧，我继续一边开车一边跟我妈交流。然后安全系统提醒我要系好安全带。我一边跟老妈解释我平常开车都是系安全带的，一边搜索地址，
终于，我到了商店<h3>
```
这样的思考方式你会不会觉得有点搞笑？实际上如果以函数化的方式思考的话，我们的大脑就是这么干的，这不是同时做了很多事，只是大脑在飞速切换正在做的事。

听起来很奇葩？这就是为什么我们很难彻底去理解异步代码的原因。我们的思想总是一步一步按计划进行，但是异步编程不让我们这么思考。

## Nested/Chained Callbacks
考虑以下代码：
``` javaScript
listen( "click", function handler(evt){
	setTimeout( function request(){
		ajax( "http://some.url.1", function response(text){
			if (text == "hello") {
				handler();
			}
			else if (text == "world") {
				request();
			}
		} );
	}, 500) ;
} );
```
现在有三个相互嵌套的函数，每个函数都是以异步的方式执行，这种类似的多层异步嵌套，我们一般称为 “callback hell”，有时候也称为 “pyramid of doom”。

上面的代码首先等待 “click” 事件，然后等待定时器的时间到，然后等待 ajax 请求获取数据，如果服务器返回的值是 “hello”，那么从头再执行一遍。

我们看代码的时候首先是：
``` javaScript
listen( "..", function handler(..){
	// ..
} );
```
然后：
``` javaScript
setTimeout( function request(..){
	// ..
}, 500) ;
```
再然后：
``` javaScript
ajax( "..", function response(..){
	// ..
} );
```
最后：
``` javaScript
if ( .. ) {
	// ..
}
else ..
```
但是像这样的直线推理会又一些问题。

首先，我们不能像这样按流程的一步一步的思考异步代码，因为在 js 的异步代码中，往往会有很多干扰代码，我们需要跳过这些干扰代码，去更好的理解程序中的异步代码流，听起来玄，做起来或许也
不容易，但是只要多加练习，铁杵磨成绣花针。

还有一些问题，请看下面代码：
``` javaScript
doA( function(){
	doB();

	doC( function(){
		doD();
	} )

	doE();
} );

doF();
```
初看一眼可能会有些困惑，实际上上面代码的运行流程是这样的：
``` javaScript
doA()
doF()
doB()
doC()
doE()
doD()
```
是不是跟你想的一样？不一样的话或许是上面示例中的函数名字让你有点混淆，我改一下：
``` javaScript
doA( function(){
	doC();

	doD( function(){
		doF();
	} )

	doE();
} );

doB();
```
现在每个函数的命名都按他们实际执行的顺序排列，你觉得好理解点了嘛？

或许改了函数名之后你觉得自然很多了，但是我还是有点问题。

如果 doA 或者 doD 是同步执行，那么整个流程就又变了，变成 A -> C -> D -> F -> E -> B ...

这就是异步代码很难把握它们的运行流程的原因，很多开发者都会遇到这样的问题。

现在我们改写下上面的栗子。
``` javaScript
listen( "click", handler );

function handler() {
	setTimeout( request, 500 );
}

function request(){
	ajax( "http://some.url.1", response );
}

function response(text){
	if (text == "hello") {
		handler();
	}
	else if (text == "world") {
		request();
	}
}
```
上面的代码格式更容易看出 “callback hell”，上面的函数一层嵌套一层，我们从上往下看很容易得出有嵌套问题。so easy，但是到了实际情况中，因为代码量比较大，一般很难像这样分清楚代码的层次。

嵌套代码有一点不好的就是，如果某一个步骤中断执行了，下面的方法都不会执行。

## Trust Issues
callbacks 的问题就是我们大脑的思维模式于 js 引擎运行异步代码的方式不匹配，请看下面代码。
``` javaScript
// A
ajax( "..", function(..){
	// C
} );
// B
```
// A 与 // B 是同步执行，// C 是延迟的异步执行。具体延迟多久是由 ajax 请求化的时间决定，这一般不会又什么问题。

但是有些情况下， ajax 函数体并不是你自己写的，而是借助一些第三方插件，比如 $.ajax(...)。这时候你可能并不了解这些第三方插件的具体运行机制，这样就使代码变得很难维护了。

## Tale of Five Callbacks
也许你现在很难意识到这个问题，现在我说个夸张点的栗子给你看。

想象你现在作为一个开发者，被要求写一个电子商务系统。其他的页面你都写好了，只有一个页面你没有写，就是用户点击购买的时候，你需要使用一个第三方插件来追踪购买细节。

可以想象到这是一个异步插件，在用户点击购买后，你需要写一个回调函数去扣除用户信用卡里面的钱，然后提示购买成功。

代码可能是这个样子的：
``` javaScript
analytics.trackPurchase( purchaseData, function(){
	chargeCreditCard();
	displayThankyouPage();
} );
```
万事皆备，程序成功运行，你很高兴。

时光荏苒，几个月之间没有人告诉你说有任何错误，你甚至可能都忘了你写了这段代码。一天，你吃着火锅唱着歌，突然接到老板的电话，出问题了。

当你到了办公室之后，你得知有一个用户买一台电视机付了五次的款，他十分郁闷。你老板命令你搞清楚为什么会发生这种事（如果发生，请不要让顾客知道）。问你，你程序上线之前有做测试吗？

代码你早忘了，但是还是得硬着头皮找问题。

当查看一些日志文件之后，你得出结论，”老子的代码绝对没问题，一切都是那个第三发插件的问题！“，因为一些不明的原因，那个插件对你写的回调函数触发了五次！

你联系第三方，他们也很惊讶，并且保证更新程序，诸如此类事情以后不再发生。ok，事情解决，报告老板。

老板听了你的报告后觉得完全相信第三方，这事不靠谱，他要求，在我们自己的代码中写一些条件来阻止类似的事情发生，不依第三方插件的修复。

你觉得有道理，稍微思考一阵之后你写下下面的代码：
``` javaScript
var tracked = false;

analytics.trackPurchase( purchaseData, function(){
	if (!tracked) {
		tracked = true;
		chargeCreditCard();
		displayThankyouPage();
	}
} );
```
好了，问题解决了，callback 只会调用一遍，但是，总是有但是，有人提出问题，如果第三方插件不调用五次 callback，它一次都不调用，怎么办？

的确，是会出问题，你决定定下心来好好想一想可能会出现的问题。
- 调用 callback 太早
- 调用 callback 太晚，或者干脆不调用
- 调用 callback 太多次，比如说五次
- 调用 callback 的时候没有传必要参数，或者传错了
- 还有一些不可预知的问题可能发生
- ... ...

好了，你意识到这些问题修复起来会相当麻烦，你碰到了 ”callback hell“

## Not Just Others' Code
怎么样，是不是意识到了问题所在？希望你在工作中不会碰到这样类似的问题。

第三方插件真的不应该被相信吗？

事实上，为了减少或者避免不可知问题的发生，我们经常写一些条件代码。看下面栗子：

``` javaScript
function addNumbers(x,y) {
	// + is overloaded with coercion to also be
	// string concatenation, so this operation
	// isn't strictly safe depending on what's
	// passed in.
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// "2121"
```
改善后：
``` javaScript
function addNumbers(x,y) {
	// ensure numerical input
	if (typeof x != "number" || typeof y != "number") {
		throw Error( "Bad parameters" );
	}

	// if we get here, + will safely do numeric addition
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// Error: "Bad parameters"
```
上面代码安全，但不友好，我们可以这样写：
``` javaScript
function addNumbers(x,y) {
	// ensure numerical input
	x = Number( x );
	y = Number( y );

	// + will safely do numeric addition
	return x + y;
}

addNumbers( 21, 21 );	// 42
addNumbers( 21, "21" );	// 42
```
这样处理之后就显得更加友好，相信用户输入，但是也检测用户输入。

对于异步编程中，我们也要有这样的思维。这样以来，在异步编程中，我们要自己定义规则，最终规则一大推，每个 callback 都要按我们规定的规则改写。

其实 callback 的问题在于，控制权的交换导致出现问题。

如果你的代码中有 callback ，尤其是使用了第三方插件的情况下，请谨慎的检查下会不会出现上述问题。

## Trying to Save Callbacks
这里有几个 callback 的写法，也可能会出现问题。

例如，下面的栗子，ajax 接受两个回调函数，一个请求失败时调用，一个请求成功时调用。
``` javaScript
function success(data) {
	console.log( data );
}

function failure(err) {
	console.error( err );
}

ajax( "http://some.url.1", success, failure );
```
在这个设计中，failure 是可选的，如果不传错误的回调函数，也没关系，只是不会报出相关错误给用户。

与之相关的，还有一种设计是，第一个参数是错误回调，第二个才是成功回调。
``` javaScript
function response(err,data) {
	// error?
	if (err) {
		console.error( err );
	}
	// otherwise, assume success
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", response );
```
针对上面API的设计思路，有几点你需要注意的是。

首先，上面的写法还是没有解决重复调用的问题。

第二，只写一个回调函数使你的代码很难做到重用，如果 ajax 请求过多，你需要给每个 ajax 请求写一个 callback。

针对上面问题，你可以这样写：
``` javaScript
function timeoutify(fn,delay) {
	var intv = setTimeout( function(){
			intv = null;
			fn( new Error( "Timeout!" ) );
		}, delay )
	;

	return function() {
		// timeout hasn't happened yet?
		if (intv) {
			clearTimeout( intv );
			fn.apply( this, [ null ].concat( [].slice.call( arguments ) ) );
		}
	};
}

// using "error-first style" callback design
function foo(err,data) {
	if (err) {
		console.error( err );
	}
	else {
		console.log( data );
	}
}

ajax( "http://some.url.1", timeoutify( foo, 500 ) );
```
记得上面提出的几大问题吗？还有一个问题就是过早的调用 callback，考虑下面代码：
``` javaScript
function result(data) {
	console.log( a );
}

var a = 0;

ajax( "..pre-cached-url..", result );
a++;
```
运行结果会输出什么,0 还是 1？

## Review
callback 是 js 异步编程的基本单元，
