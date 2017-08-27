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

请看下面栗子。

``` javaScript
var x = 1;

function foo() {
	x++;
	bar();  // what about this line?
	console.log( "x:", x );
}

function bar() {
	x++;
}

foo();  // x: 3
```
在这个栗子中，我们直到 bar() 运行于 x++ 和 console.log(x) 之间，如果不是的话，那么 console 的结果就是 2 而不是 3 了。

接着想，如果 bar() 不运行于上述两者之间，但是又能在别的地方影响 x 的值，有办法吗？

在多线程语言中，bar() 的确有可能在别的地方打断 foo() 函数运行，但是 javaScript 并不是多线程语言。所以 bar() 不能打断 foo() 的运行，但是在 ES6 中，foo() 可以打断自己的运行。

请看下面代码。

``` javaScript
var x = 1;

function *foo() {
	x++;
	yield; // pause!
	console.log( "x:", x );
}

function bar() {
	x++;
}
```
**注意**：你可能在别的文档或代码中以 function* foo(){} 来声明一个 Generator 函数，本文使用的是 function *foo(){}，这两种方式的唯一区别就是 * 的位置不同，现在这两种方式都是有效的，我个人比较倾向于第二种，因为这种写法，当我引用 *foo() 时，你能直到我使用的时 Generator 函数，而不是普通函数。

现在，我们运行 *foo() 的时候，它会运行到 yield 就暂停执行，这就给了我们将 bar() 插入到 *foo() 运行完之前运行的机会了。

``` javaScript
// construct an iterator `it` to control the generator
var it = foo();

// start `foo()` here!
it.next();
x;  // 2
bar();
x;  // 3
it.next();  // x: 3
```
ok,目前为止，你应该对于上面的代码有很多不理解的地方，下文将会一一讲到，在此之前，我们先列出上面代码运行的流程。
- 1、it = foo() 操作符并不会执行 *foo 函数，它只是创建了一个 iterator 对象，这个对象能控制 *foo() 函数体的执行。
- 2、使用 it.next() 操作符开始运行 *foo 函数，但是只运行 *foo 函数体内的 x++ 语句。
- 3、yield 语句暂停了 *foo 的执行。
- 4、输出 x 的值为 2。
- 5、调用 bar 函数，它的运行使 x 的值加至 3。
- 6、输出 x 的值为 3。
- 7、运行 it.next()，*foo 函数恢复执行，运行至 console.log，输出 x 的当前值为 3。

显而易见吗，*foo 函数执行了，但是并不是一下子就执行完毕，而是中间有个暂停执行的步骤（由 yield 关键字控制）。

generator 是一种特殊的函数，他可以在运行中多次暂停执行，且可以永远暂停下去（只要你想的话），这是一个非常有用的功能，我们将会在下文中详细了解 Generator，它将成为一种非常有效的构建异步模块的手段。

### Input and Output
刚刚说到 generator 函数是一种特殊的函数类型，但毕竟也是函数，它同样的也接受参数（input）和返回值（output）。

``` javaScript
function *foo(x,y) {
	return x * y;
}

var it = foo( 6, 7 );

var res = it.next();

res.value;	// 42
```
给 foo 传入 6, 7，并且 foo 返回 42。

但是调用 foo 的方式和普通函数有所不同，当运行 foo( 6, 7 ) 的时候，实际函数并没有运行，当 it.next() 执行的时候，foo 函数才开始执行，运行到 yield 或函数末尾。

next() 语句返回一个对象，它包含 value 属性，反应当前状态的结果，函数当前状态由函数内部决定，比如 yield 会暂停函数的运行。

### Iteration Messaging
generator 不仅仅是接受参数返回值，它还可以通过 yield 和 next 完成更有用更复杂的操作。

考虑如下代码。

``` javaScript
function *foo(x) {
	var y = x * (yield);
	return y;
}

var it = foo( 6 );

// start `foo(..)`
it.next();

var res = it.next( 7 );

res.value;  // 42
```

首先，运行 var it = foo( 6 )，此时函数并不会运行，调用 it.next()之后，才开始 foo 的运行。

在 foo 函数内部，运行 var y = x * (yield) 语句的时候，遇到 yield 使 foo 函数暂停运行，它期待一个返回值给 yield 关键字，然后才能继续运行函数。

当我们运行 it.next( 7 ) 的时候，7 就作为 yield 的返回值，让 foo 继续运行。

现在 var y = x * (yield) 变成了 var y = 6 * 7，所以后面的语句 返回了 42。

不知道你注意没有，上面的代码中有一点点 “问题”，即 yield 与 next 方法数量上不匹配，上面的代码只有一个 yield 却有两个 next 调用。

或者说 yield 总是比 next 少一个，不是一个暂停一个执行吗？应该是相等才对，为什么会这样？

因为 第一个 next 方法总是作为启动 generator 函数，后面的第二个 next 才对应函数体内的第一个 yield，以此类推。

### Tale of Two Questions

考虑如下代码。

``` javaScript
var y = x * (yield);
return y;
```

上面代码中，yield 好像是在问：“这里应该插入什么值？”。

插入什么值由谁决定？

上文说到，第一个 next() 方法是作为 generator 的启动方法，运行第一个  next() 方法后，函数将会运行到 yield 这个点。显而易见，此时的 yield 的输出值，并不是又第一个 next() 方法决定。

所以说上面程序的第一个 yield 操作符的返回值由程序里面的第二个 next() 方法决定。

**第二个 next 对应第一个 yield**。

现在让我们换个角度想想，不从 generator 的角度，而从 iterator 的角度看。

要理清其中的关系，首先我们需要解释下






