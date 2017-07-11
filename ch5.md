# JavaScript轻量级函数式编程
# 第 5 章：减少副作用

在第 2 章，我们讨论了一个函数除了它的返回值之外还有什么输出。现在你应该很熟悉用函数式编程的方法定义一个函数了，所以对于函数式编程的副作用你应该有所了解。

我们将检查各种各样不同的副作用并且要看看他们为什么会对我们的代码质量和可读性造成损害。

这一章的要点是：编写出没有副作用的程序是不可能的。当然，也不是不可能，你当然可以编写出没有副作用的程序。但是这样的话程序就不会做任何有用和明显的事情。如果你编写出来一个零副作用的程序，你就无法区分它和一个被删除的或者空程序的区别。

函数式编程者并没有消除所有的副作用。实际上，我们的目标是尽可能的限制他们。要做到这一点，我们首先需要完全理解函数式编程的副作用。

## 什么是副作用

因果关系：举一个我们人类对周围世界影响的最基本、最直观的例子，推一下放在桌子边沿上的一本书，书会掉落。不需要你拥有一个物理学的学位你也会知道，这是因为你刚刚推了书并且书掉落是因为地心引力，这是一个明确并直接的关系。

在编程中，我们也完全会处理因果关系。如果你调用了一个函数（起因），就会在屏幕上输出一条消息（结果）。

当我们在阅读程序的时候，能够清晰明确的识别每一个起因和每一个结果是非常重要的。在某种程度上，通读程序但不能看到因果的直接关系，程序的可读性就会降低。

思考一下：

```js
function foo(x) {
	return x * 2;
}

var y = foo( 3 );
```

在这段代码中，有很直接的因果关系，调用值为 `3` 的 foo 将具有返回值 `6` 的效果，调用函数 foo() 是起因，然后将其赋值给 `y` 是结果。这里没有歧义，传入参数为 3 将会返回 6，将函数结果赋值给变量 `y` 是结果。

但是现在：

```js
function foo(x) {
	y = x * 2;
}

var y;

foo( 3 );
```
这段代码有相同的输出，但是却有很大的差异，这里的因果是没有联系的。这个影响是间接的。这种方式设置 `y` 就是我们所说的副作用。

**注意：** 当函数引用外部变量时，这个变量就称为自由变量。并不是所有的自由变量引用都是不好的，但是我们要对它们非常小心。

假使给你一个引用来调用函数 `bar(..)`，你看不到代码，但是我告诉你这段代码并没有间接的副作用，只有一个显式的 `return` 值会怎么样？

```js
bar( 4 );			// 42
```

因为你知道 `bar(..)` 的内部结构不会有副作用，你可以像这样直接的调用 `bar(..)`。但是如果你不知道 `bar(..)` 没有副作用，为了理解调用这个函数的结果，你必须去阅读和分析它的逻辑。这对读者来说是额外的负担。

**有副作用的函数可读性更低，**因为它需要更多的阅读来理解程序。

但是程序往往比这个要复杂，思考一下：

```js
var x = 1;

foo();

console.log( x );

bar();

console.log( x );

baz();

console.log( x );
```
你能确定每次 `console.log(x)` 的值都是你想要的吗？

答案是否定的。如果你不确定函数 `foo()`、`bar()` 和 `baz()` 是否有副作用，你就不能保证每一步的 `x` 将会是什么，除非你检查每个步骤的实现，然后从第一行开始跟踪程序，跟踪所有状态的改变。

换句话说，`console.log(x)` 最后的结果是不能分析和预测的，除非你已经在心里将整个程序执行到这里了。

猜猜谁擅长运行你的程序？JS 引擎。猜猜谁不擅长运行你的程序？你代码的读者。然而，如果你选择在一个或多个函数调用中编写带有（潜在）副作用的代码，那么这意味着你已经使你的读者必须将你的程序完整地执行到某一行，以便他们理解这一行。

如果 `foo()`、`bar()`、和 `baz()` 都没有副作用的话，它们就不会影响到 `x`，这就意味着我们不需要在心里默默地执行它们并且跟踪 `x` 的变化。这在精力上负担更小，并且使得代码更加的可读。

### 潜在的原因

输出和状态的变化，是最常被引用的副作用的表现。但是另一个有损可读性的实践是一些被认为的侧因，思考一下：

```js
function foo(x) {
	return x + y;
}

var y = 3;

foo( 1 );			// 4
```

`y` 不会随着 `foo(..)` 改变，所以这和我们之前看到的副作用有所不同。但是现在，对函数 `foo(..)` 的调用实际上取决于 `y` 当前的状态。之后我们如果这样做：

```js
y = 5;

// ..

foo( 1 );			// 6
```

我们可能会感到惊讶两次调用 `foo(1)` 返回的结果不一样。

`foo(..)` 对可读性有一个间接的破坏性。如果没有对函数 `foo(..)` 进行仔细检查，使用者可能不会知道导致这个输出的原因。这**看起来**仅仅像是参数 `1` 的原因，但却不是这样的。

为了帮助可读性，所有决定 `foo(..)` 输出的原因应该被设置的直接并明显。函数的使用者将会直接看到原因和结果。

#### 使用固定的状态

避免副作用就意味着函数 `foo(..)` 不能引用自由变量了吗？

思考下这段代码：

```js
function foo(x) {
	return x + bar( x );
}

function bar(x) {
	return x * 2;
}

foo( 3 );			// 9
```

很明显，对于函数 `foo(..)` 和函数 `bar(..)`，唯一和直接的原因就是参数 `x`。但是 `bar(x)` 被称为什么呢？`bar` 仅仅只是一个标识符，在 JS 中，默认情况下，它甚至不是一个常量（不可重新分配的变量）。`foo(..)` 函数依赖于 `bar` 的值，`bar` 作为一个自由变量被第二个函数引用。

所以说这个函数还依赖于其他的原因吗？

我认为不。虽然**可以**用其他的函数来重写 `bar` 这个变量，但是在代码中我没有这样做，这也不是我的惯例或先例。无论出于什么意图和目的，我的函数都是常量（从不重新分配）。

思考一下：

```js
const PI = 3.141592;

function foo(x) {
	return x * PI;
}

foo( 3 );			// 9.424776000000001
```

**注意：** JavaScript 有内置的 `Math.PI` 属性，所以我们在本文中仅仅是用 `PI` 做一个方便的说明。在实践中，总是使用 `Math.PI` 而不是你自己定义的。

上面的代码怎么样呢？`PI` 是否会对函数 `foo(..)` 造成其他的影响？

两个观察结果将会合理的帮助我们回答这个问题：

1. 想一下是否每次调用 `foo(3)`，都将会返回 `9.424..`？**答案是肯定的。**如果每一次都给一个相同的输入（`x`），那么都将会返回相同的输出。

2. 你能用 `PI` 的当前值来代替每一个 `PI` 吗，并且程序能够和之前一样**正确的**的运行吗？**是的。**程序没有任何一部分依赖于 `PI` 值的改变，因为 `PI` 的类型是 `const`，它是不能再分配的，所以变量 `PI` 在这里只是为了便于阅读和维护。它的值可以在不改变程序行为的情况下内联。

我的结论是：这里的 `PI` 并不违反减少或避免副作用的精神。在之前的代码也没有调用 `bar(x)`。

在这两种情况下，`PI` 和 `bar` 都不是程序状态的一部分。它们是固定的，不可重新分配的（“常量”）的引用。如果他们在整个程序中都不改变，那么我们就不需要担心将他们作为变化的状态追踪他们。同样的，他们不会损害程序的可读性。而且它们也不能成为与变量以意想不到的方式发生变化相关的错误的根源。

**注意：**在我看来，使用 `const` 并不能说明 `PI` 不是副作用；使用 `var PI` 也会是同样的结果。`PI` 没有被重新分配是问题的关键，而不是使用 `const`。我们将在后面的章节讨论 `const`。

#### 随机性

你以前可能从来没有考虑过，但是随机性是不纯的。一个使用 `Math.random()` 的函数永远都不是纯的，因为你不能根据它的输入来保证和预测它的输出。所以任何生成唯一随机的 ID 等都需要依靠程序的其他原因。

在计算中，我们使用的是伪随机算法。事实证明，真正的随机是非常难的，所以我们只是用复杂的算法来模拟它，产生的值看起来是随机的。这些算法计算很长的一串数字，但秘密是，如果你知道起始点，实际上这个序列是可以预测的。这个起点被称之为种子。

一些语言允许你指定生成随机数的种子。如果你总是指定了相同的种子，那么你将始终从后续的“随机数”中得到相同的输出序列。这对于测试是非常有用的，但是在真正的应用中使用也是非常危险的。

在 JS 中，`Math.random()` 的随机性计算是基于间接输入，因为你不能明确种子。因此，我们必须将内建的随机数生成视为不纯的一方。

### I/O 效果

这可能不太明显，但是最常见（并且本质上不可避免）的副作用就是 I/O（输入/输出）。一个没有 I/O 的程序是完全没有意义的，因为它的工作不能以任何方式被观察到。一个有用的程序必须最少有一个输出，并且也需要输入。输入会产生输出。

用户事件（鼠标、键盘）是 JS 编程者在浏览器中使用的典型的输入，而输出的则是 DOM。如果你使用 Node.js 比较多，你更有可能接收到和输出到文件系统、网络系统和/或者 `stdin` / `stdout`（标准输入流/标准输出流）的输入和输出。

事实上，这些来源既可以是输入也可以是输出，是因也是果。以 DOM 为例，我们更新（产生副作用的结果）一个 DOM 元素为了给用户展示文字或图片信息，但是 DOM 的当前状态是对这些操作的隐式输入（产生副作用的原因）。

### 其他的错误

在程序运行期间侧因和副作用可能导致的错误是多种多样的。让我们来看一个场景来说明这些危害，希望它们能帮助我们认出在我们自己的程序中类似的错误。

思考一下：

```js
var users = {};
var userOrders = {};

function fetchUserData(userId) {
	ajax( "http://some.api/user/" + userId, function onUserData(userData){
		users[userId] = userData;
	} );
}

function fetchOrders(userId) {
	ajax( "http://some.api/orders/" + userId, function onOrders(orders){
		for (let i = 0; i < orders.length; i++) {
		        // 对每个用户的最新订单保持引用
			users[userId].latestOrder = orders[i];
			userOrders[orders[i].orderId] = orders[i];
		}
	} );
}

function deleteOrder(orderId) {
	var user = users[ userOrders[orderId].userId ];
	var isLatestOrder = (userOrders[orderId] == user.latestOrder);
	
	// 删除用户的最新订单？
	if (isLatestOrder) {
		hideLatestOrderDisplay();
	}

	ajax( "http://some.api/delete/order/" + orderId, function onDelete(success){
		if (success) {
		        // 删除用户的最新订单？
			if (isLatestOrder) {
				user.latestOrder = null;
			}

			userOrders[orderId] = null;
		}
		else if (isLatestOrder) {
			showLatestOrderDisplay();
		}
	} );
}
```

我敢打赌，一些读者显然会发现其中潜在的错误。如果回调 `onOrders(..)` 在 回调`onUserData(..)` 之前运行，它会给一个尚未设置的值（`users[userId]`  的 `userData` 对象）添加一个 `latestOrder` 属性

因此，这种依赖于因果关系的“错误”是在两种不同操作（是否异步）紊乱情况下发生的，我们期望以确定的顺序运行，但在某些情况下，可能会以不同的顺序运行。有一些策略可以确保操作的顺序，很明显，在这种情况下顺序是至关重要的。

这里还有另一个细小的错误，你发现了吗？

思考下这个调用顺序：

```js
fetchUserData( 123 );
onUserData(..);
fetchOrders( 123 );
onOrders(..);

// later

fetchOrders( 123 );
deleteOrder( 456 );
onOrders(..);
onDelete(..);
```

你发现每一对 `fetchOrders(..)` / `onOrders(..)` 和 `deleteOrder(..)` / `onDelete(..)` 都是交替出现了吗？这个潜在的排序会伴随着我们状态管理的侧因/副作用暴漏出一个奇怪的状态。

在设置 `isLatestOrder` 标志和使用它来决定是否应该清空 `users` 中的用户数据对象的 `latestOrder` 属性时，会有一个延迟（因为回调）。在此延迟期间，如果 `onOrders(..)` 销毁，它可以潜在地改变用户的 `latestOrder` 引用的顺序值。当 `onDelete(..)` 在销毁之后，它会假定它仍然需要重新引用 `latestOrder`。

错误：数据（状态）**可能**不同步。当进入 `onOrders(..)` 时，`latestOrder` 可能仍然指向一个较新的顺序，这样 `latestOrder` 就会被重置。

这种错误最糟糕的是你不能和其他错误一样得到程序崩溃的异常。我们只是有一个不正确的状态；我们的应用程序“默默的”崩溃。

`fetchUserData(..)` 和 `fetchOrders(..)` 的序列依赖是相当明显的，并且直截了当的处理。但是，在 `fetchOrders(..)` 和 `deleteOrder(..)` 之间存在潜在的序列依赖关系，就不太清楚了。这两个似乎更加独立。并且确保他们的顺序被保留是比较棘手的，因为你事先不知道（在 `fetchOrders(..)` 产生结果之前）是否必须要按照这样的顺序执行。

是的，一旦 `deleteOrder(..)` 销毁，你就能重新计算 `isLatestOrder` 标志。但是现在你有另一个问题：你的 UI 状态可能不同步。

如果你之前已经调用过 `hideLatestOrderDisplay()`，现在你需要调用 `showLatestOrderDisplay()`，但是如果一个新的 `latestOrder` 已经被设置好了，你将要跟踪至少三个状态：被删除的状态是否本来是“最新的”、是否是“最新”设置的，和这两个顺序有什么不同吗？这些都是可以解决的问题，但无论如何都是不明显的。

所有这些麻烦都是因为我们决定在一组共享的状态下构造出有副作用的代码。

函数式编程人员讨厌这类因果的错误，因为这有损我们的阅读、推理、验证和最终**相信**代码的能力。这就是为什么他们要如此严肃地对待避免副作用 / 效果的原因。

有很多避免/修复副作用的策略。我们将在本章后面和后面的章节中讨论。我要说一个确定的事情：**写出有副作用/效果的代码是很正常的，**所以我们需要谨慎和刻意的避免产生有副作用的代码。

## 一次就好

如果你必须要使用副作用来改变状态，那么一种对限制潜在问题有用的操作是幂等。如果你的值的更新是幂次的，那么数据将会适应你可能有不同副作用来源的多个此类更新的情况。

幂等的定义有点让人困惑；数学家和程序员使用幂等的含义稍有不同。然而，这两种观点对于函数式编程人员都是有用的。

首先，让我们给出一个计数器的例子，它既不是数学上的，也不是程序上的幂等：

```js
function updateCounter(obj) {
	if (obj.count < 10) {
		obj.count++;
		return true;
	}

	return false;
}
```

这个函数通过引用递增 `obj.count` 来该改变一个对象，所以对这个对象产生了副作用。当 `o.count` 小于 10 时，如果 `updateCounter(o)` 被多次调用，即程序状态每次都要更改。另外，`updateCounter(..)` 的输出是一个布尔值，这不适合返回到 `updateCounter(..)` 的后续调用。

### 数学中的幂等

从数学的角度来看，幂等指的是在第一次调用后，其输出永远不会改变的操作，如果你将该输出一次又一次地输入到操作中。换句话说，`foo(x)` 将产生与 `foo(foo(x))`、`foo(foo(foo(x)))` 等相同的输出。

一个典型的数学例子是 `Math.abs(..)`（取绝对值）。`Math.abs(-2)` 的结果是 `2`，和 `Math.abs(Math.abs(Math.abs(Math.abs(-2))))` 的结果相同。像`Math.min(..)`、`Math.max(..)`、`Math.round(..)`、`Math.floor(..)` 和 `Math.ceil(..)`这些工具函数都是幂等的。

我们可以用同样的特征来定义一些数学运算：

```js
function toPower0(x) {
	return Math.pow( x, 0 );
}

function snapUp3(x) {
	return x - (x % 3) + (x % 3 > 0 && 3);
}

toPower0( 3 ) == toPower0( toPower0( 3 ) );			// true

snapUp3( 3.14 ) == snapUp3( snapUp3( 3.14 ) );		// true
```

数学上的幂等**不**仅限于数学运算。我们还可以用 JavaScript 的原始类型来说明幂等的另一种形式：

```js
var x = 42, y = "hello";

String( x ) === String( String( x ) );				// true

Boolean( y ) === Boolean( Boolean( y ) );			// true
```

在本文的前面，我们探究了一种常见的函数式编程工具，它可以实现这种形式的幂等：

```js
identity( 3 ) === identity( identity( 3 ) );	// true
```

某些字符串操作自然也是幂等的，例如：

```js
function upper(x) {
	return x.toUpperCase();
}

function lower(x) {
	return x.toLowerCase();
}

var str = "Hello World";

upper( str ) == upper( upper( str ) );				// true

lower( str ) == lower( lower( str ) );				// true
```

我们甚至可以以一种幂等方式设计更复杂的字符串格式操作，比如：

```js
function currency(val) {
	var num = parseFloat(
		String( val ).replace( /[^\d.-]+/g, "" )
	);
	var sign = (num < 0) ? "-" : "";
	return `${sign}$${Math.abs( num ).toFixed( 2 )}`;
}

currency( -3.1 );									// "-$3.10"

currency( -3.1 ) == currency( currency( -3.1 ) );	// true
```

`currency(..)` 举例说明了一个重要的技巧：在某些情况下，开发人员可以采取额外的步骤来规范化输入/输出操作，以确保操作是幂等的来避免意外的发生。

在任何可能的情况下通过幂等的操作限制副作用要比不做限制的更新要好得多。

### 编程中的幂等

幂等的面向程序的定义也是类似的，但不太正式。编程中的幂等仅仅是 `f(x);` 的结果与 `f(x); f(x)` 相同而不是要求 `f(x) === f(f(x))`。换句话说，之后每一次调用 `f(x)` 的结果和第一次调用 `f(x)` 的结果没有任何改变。

这种观点更符合我们对副作用的观察。因为这更像是一个 `f(..)` 创建了一个幂等的副作用而不是必须要返回一个幂等的输出值。

这种幂等性的方式经常被用于 HTTP 操作（动词），例如 GET 或 PUT。如果 HTTP REST API 正确地遵循了幂等的规范指导，那么 PUT 被定义为一个更新操作，它可以完全替换资源。同样的，客户端可以一次或多次发送 PUT 请求（使用相同的数据），而服务器无论如何都将具有相同的结果状态。

让我们用更具体的编程方法来考虑这个问题，来检查一下使用幂等和没有使用幂等是否产生副作用：

```js
// 幂等的：
obj.count = 2;
a[a.length - 1] = 42;
person.name = upper( person.name );

// 非幂等的：
obj.count++;
a[a.length] = 42;
person.lastUpdated = Date.now();
```

记住：这里的幂等性的概念是每一个幂等运算（比如 `obj.count = 2`）可以重复多次，而不是在第一次更新后改变程序操作。非幂等操作每次都改变状态。

那么更新 DOM 呢？

```js
var hist = document.getElementById( "orderHistory" );

// 幂等的：
hist.innerHTML = order.historyText;

// 非幂等的：
var update = document.createTextNode( order.latestUpdate );
hist.appendChild( update );
```

这里的关键区别在于，幂等的更新替换了 DOM 元素的内容。DOM 元素的当前状态是独立的，因为它是无条件覆盖的。非幂等的操作将内容添加到元素中；隐式地，DOM 元素的当前状态是计算下一个状态的一部分。

我们将不会一直用幂等的方式去定义你的数据，但如果你能做到，这肯定会减少你的副作用在你最意想不到的时候突然出现的可能性。

## 纯粹的快乐

没有副作用的函数称为纯函数。在编程的意义上，纯函数是一种幂等函数，因为它不可能有任何副作用。思考一下：

```js
function add(x,y) {
	return x + y;
}
```

所有输入（`x` 和 `y`）和输出（`return ..`）都是直接的，没有引用自由变量。调用 `add(3,4)` 多次和调用一次是没有区别的。`add(..)` 是纯粹的编程风格的幂等。

然而，并不是所有的纯函数都是数学概念上的幂等，因为它们返回的值不一定适合作为再次调用它们时的输入。思考一下：

```js
function calculateAverage(list) {
	var sum = 0;
	for (let i = 0; i < list.length; i++) {
		sum += list[i];
	}
	return sum / list.length;
}

calculateAverage( [1,2,4,7,11,16,22] );			// 9
```

输出的 `9` 并不是一个数组，所以你不能在 `calculateAverage(calculateAverage(..))` 中将其传入。

正如我们前面所讨论的，一个纯函数**可以**引用自由变量，只要这些自由变量不是侧因。

例如：

```js
const PI = 3.141592;

function circleArea(radius) {
	return PI * radius * radius;
}

function cylinderVolume(radius,height) {
	return height * circleArea( radius );
}
```

`circleArea(..)` 中引用了自由变量 `PI`，但是这是一个常量所以不是一个侧因。`cylinderVolume(..)` 引用了自由变量 `circleArea`，这也不是一个侧因，因为这个程序把它当作一个常量引用它的函数值。这两个函数都是纯的。

另一个例子，一个函数仍然可以是纯的，但引用的自由变量是闭包:

```js
function unary(fn) {
	return function onlyOneArg(arg){
		return fn( arg );
	};
}
```

`unary(..)` 本身显然是纯函数 —— 它唯一的输入是 `fn`，并且它唯一的输出是返回的函数，但是闭合了自由变量 `fn` 的内部函数 `onlyOneArg(..)` 是不是纯的呢？

它仍然是纯的，因为 `fn` 永远不变。事实上，我们对这一事实有充分的自信，因为从词法上讲，这几行是唯一可能重新分配 `fn` 的代码。

**注意：**`fn` 是一个函数对象的引用，它默认是一个可变的值。在程序的其他地方**可能**为这个函数对象添加一个属性，这在技术上“改变”这个值（改变，而不是重新分配）。然而，因为我们除了调用 `fn`，不依赖 `fn` 以外的任何事情，并且不可能影响函数值的可调用性，因此 `fn` 在最后的结果中仍然是有效的不变的；它不可能是一个侧因。

表达一个函数的纯度的另一种常用方法是：**给定相同的输入（一个或多个），它总是产生相同的输出。**如果你把 `3` 传给 `circleArea(..)` 它总是输出相同的结果（`28.274328`）。

如果一个函数每次在给予相同的输入时，**可能**产生不同的输出，那么它是不纯的。即使这样的函数总是返回相同的值，如果它产生间接输出副作用，那么程序状态每次被调用时都会被改变；这是不纯的。

不纯的函数是不受欢迎的，因为它们使得所有的调用都变得更加难以理解。纯的函数的调用是完全可预测的。当有人阅读代码时，看到多个 `circleArea(3)` 调用，他们不需要花费额外的精力来计算**每次**的输出结果。

### 相对的纯粹

当我们讨论一个函数是纯的时，我们必须非常小心。JavaScript 的动态值特性使其很容易产生不明显的副作用。

思考一下：

```js
function rememberNumbers(nums) {
	return function caller(fn){
		return fn( nums );
	};
}

var list = [1,2,3,4,5];

var simpleList = rememberNumbers( list );
```
`simpleList(..)` 看起来是一个纯函数，因为它只涉及内部的 `caller(..)` 函数，它仅仅是闭合了自由变量 `nums`。然而，有很多方法证明 `simpleList(..)` 是不纯的。

首先，我们对纯度的断言是基于数组的值（通过 `list` 和 `nums` 引用）一直不改变：

```js
function median(nums) {
	return (nums[0] + nums[nums.length - 1]) / 2;
}

simpleList( median );		// 3

// ..

list.push( 6 );

// ..

simpleList( median );		// 3.5
```

当我们改变数组时，`simpleList(..)` 的调用改变它的输出。所以，`simpleList(..)` 是纯的还是不纯的呢？这就取决于你的视角。对于给定的一组假设来说，它是纯函数。在任何没有 `list.push(6)` 的情况下中是纯的。

我们可以通过改变 `rememberNumbers(..)` 的定义来修改这种不纯。一种方法是复制 `nums` 数组：

```js
function rememberNumbers(nums) {
        // 复制一个数组
	nums = nums.slice();

	return function caller(fn){
		return fn( nums );
	};
}
```

但这可能会隐含一个更棘手的副作用：

```js
var list = [1,2,3,4,5];

// 把 `list[0]` 作为一个有副作用的接收者
Object.defineProperty(
	list,
	0,
	{
		get: function(){
			console.log( "[0] was accessed!" );
			return 1;
		}
	}
);

var simpleList = rememberNumbers( list );

// [0] 已经被使用！
```

一个更粗鲁的选择是更改 `rememberNumbers(..)` 的参数。首先，不要接收数组，而是把数字作为单独的参数：

```js
function rememberNumbers(...nums) {
	return function caller(fn){
		return fn( nums );
	};
}

var simpleList = rememberNumbers( ...list );

// [0] 已经被使用！
```

这两个 `...` 的作用是将列表复制到 `nums` 中，而不是通过引用来传递。

**注意：**控制台消息的副作用不是来自于 `rememberNumbers(..)`，而是 `...list` 的扩展中。因此，在这种情况下，`rememberNumbers(..)` 和 `simpleList(..)` 是纯的。

但是如果这种突变更难被发现呢？纯函数和不纯的函数的合成总是产生不纯的函数。如果我们将一个不纯的函数传递到另一个纯函数 `simpleList(..)` 中，那么这个函数就是不纯的：

```js
// 是的，一个愚蠢的人为的例子 :)
function firstValue(nums) {
	return nums[0];
}

function lastValue(nums) {
	return firstValue( nums.reverse() );
}

simpleList( lastValue );	// 5

list;						// [1,2,3,4,5] -- OK!

simpleList( lastValue );	// 1
```

**注意：**不管 `reverse()` 看起来多安全（就像 JS 中的其他数组方法一样），它返回一个反向数组，实际上它对数组进行了修改，而不是创建一个新的数组。

我们需要对 `rememberNumbers(..)` 下一个更粗鲁的定义来防止 `fn(..)` 改变它的闭合的 `nums` 变量的引用。

```js
function rememberNumbers(...nums) {
	return function caller(fn){
	        // 提交一个副本！
		return fn( nums.slice() );
	};
}
```

所以 `simpleList(..)` 是可靠的纯函数吗！？**不。**:(

我们只防范我们可以控制的副作用（通过引用改变）。我们传递的任何带有副作用的函数，都将会污染 `simpleList(..)` 的纯度:

```js
simpleList( function impureIO(nums){
	console.log( nums.length );
} );
```

事实上，没有办法定义 `rememberNumbers(..)` 去产生一个完美纯粹的 `simpleList(..)` 函数。

纯度是和自信是有关的。但我们不得不承认，在很多情况下，**我们所感受到的自信实际上是与我们程序的上下文**和我们对程序了解有关的。在实践中（在 JavaScript 中），函数纯度的问题不是纯粹的纯粹性，而是关于其纯度的一系列信心。

越纯洁越好。制作纯函数时越努力，当您阅读使用它的代码时，你的自信就会越高，这将使代码的一部分更加可读。

## 有或者无

到目前为止，我们已经将函数纯度定义为一个没有副作用的函数，并且作为这样一个函数，给定相同的输入，总是产生相同的输出。这只是看待相同特征的两种不同方式。

但是，第三种看待函数纯性的方法，也许是广为接受的定义，即纯函数具有引用透明性。

引用透明性是指一个函数调用可以被它的输出值所代替，并且整个程序的行为不会改变。换句话说，不可能从程序的执行中分辨出函数调用是被执行的，还是它的返回值是在函数调用的位置上内联的。

从引用透明的角度来看，这两个程序都有完全相同的行为因为它们都是用纯粹的函数构建的：

```js
function calculateAverage(list) {
	var sum = 0;
	for (let i = 0; i < list.length; i++) {
		sum += list[i];
	}
	return sum / list.length;
}

var nums = [1,2,4,7,11,16,22];

var avg = calculateAverage( nums );

console.log( "The average is:", avg );		// The average is: 9
```

```js
function calculateAverage(list) {
	var sum = 0;
	for (let i = 0; i < list.length; i++) {
		sum += list[i];
	}
	return sum / list.length;
}

var nums = [1,2,4,7,11,16,22];

var avg = 9;

console.log( "The average is:", avg );		// The average is: 9
```

这两个片段之间的唯一区别在于，在后者中，我们跳过了调用 `calculateAverage(nums)` 并内联。因为程序的其他部分的行为是相同的，`calculateAverage(..)` 是引用透明的，因此是一个纯粹的函数。

### 思考上的透明

一个引用透明的纯函数**可能**会被它的输出替代，这并不意味着它**应该**被替换。远非如此。

我们用在程序中使用函数而不是使用预先计算好的常量的原因不仅仅是应对变化的数据，也是和可读性和适当的抽象等有关。调用函数去计算一列数字的平均值让这部分程序比只是使用确定的值更具有可读性。它向读者讲述了 `avg` 从何而来，它意味着什么，等等。

我们真正建议使用引用透明是当你阅读程序，一旦你已经在内心计算出纯函数调用输出的是什么的时候，当你看到它的代码的时候不需要再去思考确切的函数调用是做什么，特别是如果它出现很多次。

这个结果有一点像你在心里面定义一个 `const`，当你阅读的时候，你可以直接跳过并且不需要花更多的精力去计算。

希望纯函数的这种特性的重要性更加的显而易见。我们正在努力使我们的程序更容易读懂。我们能做的一种方法是给读者较少的工作，通过提供帮助来跳过不必要的东西，这样他们就可以把注意力集中在重要的事情上。

读者不需要重新计算一些不会改变（也不需要改变）的结果。如果用引用透明定义一个纯函数，读者就不必这样做了。

### 不够透明？

那么如果一个有副作用的函数，并且这个副作用在程序的其他地方没有被观察到或者依赖会怎么样？这个功能还具有引用透明性吗？

这里有一个例子：

```js
function calculateAverage(list) {
	sum = 0;
	for (let i = 0; i < list.length; i++) {
		sum += list[i];
	}
	return sum / list.length;
}

var sum, nums = [1,2,4,7,11,16,22];

var avg = calculateAverage( nums );
```
你发现了吗?

`sum` 是一个 `calculateAverage(..)` 使用的外部自由变量。但是，每次我们使用相同的列表调用 `calculateAverage(..)`，我们将得到 `9` 作为输出。并且这个程序无法和使用参数 `9` 调用 `calculateAverage（nums）` 在行为上区分开来。程序的其他部分和 `sum` 变量有关，所以这是一个不可观察的副作用。
这是一个像这棵树一样不能观察到的副作用吗？

> 假如一棵树在森林里倒下而没有人在附近听见，它有没有发出声音？

通过引用透明的狭义的定义，我想你一定会说 `calculateAverage(..)` 仍然是一个纯函数。但是，因为在我们的学习中不仅仅是学习学术，而且与实用主义相平衡，我认为这个结论需要更多的观点。让我们探索一下。

#### 性能影响

你经常会发现这些不易观察的副作用被用于性能优化的操作。例如：

```js
var cache = [];

function specialNumber(n) {
        // 如果我们已经计算过这个特殊的数，
	// 跳过这个操作，然后从缓存中返回
	// if we've already calculated this special number,
	// skip the work and just return it from the cache
	if (cache[n] !== undefined) {
		return cache[n];
	}

	var x = 1, y = 1;

	for (let i = 1; i <= n; i++) {
		x += i % 2;
		y += i % 3;
	}

	cache[n] = (x * y) / (n + 1);

	return cache[n];
}

specialNumber( 6 );				// 4
specialNumber( 42 );			// 22
specialNumber( 1E6 );			// 500001
specialNumber( 987654321 );		// 493827162
```

这个愚蠢的 `specialNumber(..)` 算法是确定性的，并且，纯函数从定义来说，它总是为相同的输入提供相同的输出。从引用透明的角度来看 —— 用 `22` 替换对 `specialNumber(42)` 的任何调用，程序的最终结果是相同的。

但是，这个函数必须做一些工作来计算一些较大的数字，特别是输入像 `987654321` 这样的数字。如果我们需要在我们的程序中多次获得特定的特殊号码，那么结果的缓存意味着后续的调用效率会更高。

**注意：**思考一个有趣的事情：CPU 在执行任何给定操作时产生的热量，即使是最纯粹的函数 / 程序，也是不可避免的副作用吗？那么 CPU 的时间延迟，因为它花时间在一个纯操作上，然后再执行另一个操作，是否也算作副作用？

不要这么快的做出假设，你仅仅运行 `specialNumber（987654321）` 计算一次，并手动将该结果粘贴到一些变量 / 常量中。程序通常是高度模块化的并且全局可访问的作用域并不是通常你想要在这些独立部分之间分享状态的方式。让`specialNumber（..）` 使用自己的缓存（即使它恰好是使用一个全局变量来实现这一点）是对状态共享更好的抽象。

关键是，如果 `specialNumber(..)` 只是程序访问和更新 `cache` 副作用的唯一部分，那么引用透明的观点显然可以适用，这可以被看作是可以接受的实际的“欺骗”的纯函数思想。

但是真的应该这样吗？

典型的，这种性能优化方面的副作用是通过隐藏缓存结果产生的，因此它们**不能**被程序的任何其他部分所观察到。这个过程被称为记忆化。我一直称这个词是 “记忆化”；我不知道这个想法是从哪里来的，但它确实有助于我更好地理解这个概念。

思考一下：

```js
var specialNumber = (function memoization(){
	var cache = [];

	return function specialNumber(n){
	        // 如果我们已经计算过这个特殊的数，
	        // 跳过这个操作，然后从缓存中返回
		if (cache[n] !== undefined) {
			return cache[n];
		}

		var x = 1, y = 1;

		for (let i = 1; i <= n; i++) {
			x += i % 2;
			y += i % 3;
		}

		cache[n] = (x * y) / (n + 1);

		return cache[n];
	};
})();
```

我们已经遏制 `memoization()` 内部 `specialNumber(..)` IIFE 范围内的 `cache` 的副作用，所以现在我们确定程序任何的部分都**不能**观察到它们，而不仅仅是**不**观察它们。

最后一句话似乎是一个的微妙观点，但实际上我认为这可能是**整章中最重要的一点**。再读一遍。

回到这个哲学理论：

> 假如一棵树在森林里倒下而没有人在附近听见，它有没有发出声音？

通过这个暗喻，我所得到的是：无论是否产生声音，如果我们从不创造一个当树落下时周围没有人的情景会更好一些。当树落下时，我们总是会听到声音。

减少副作用的目的并不是他们在程序中不能被观察到，而是设计一个程序，让副作用尽可能的少，因为这使代码更容易理解。一个没有观察到的**发生**的副作用的程序在这个目标上并不像一个**不能**观察它们的程序那么有效。

如果副作用可能发生，作者和读者必须尽量应对它们。使它们不发生，作者和读者都要对任何可能或不可能发生的事情更有自信。

## 纯化

如果你有不纯的函数，且你无法将其重构为纯函数，此时你能做些什么？

您需要确定该函数有什么样的副作用。副作用来自不同的地方，可能是由于词法自由变量、引用变化，甚至是 `this` 的绑定。我们将研究解决这些情况的方法。

### 封闭的影响

如果副作用的本质是使用词法自由变量，并且您可以选择修改周围的代码，那么您可以使用作用域来封装它们。

回忆一下：

```js
var users = {};

function fetchUserData(userId) {
	ajax( "http://some.api/user/" + userId, function onUserData(userData){
		users[userId] = userData;
	} );
}
```

纯化此代码的一个方法是在变量和不纯的函数周围创建一个容器。本质上，容器必须接收所有的输入。

```js
function safer_fetchUserData(userId,users) {
        // 简单的、原生的 ES6 + 浅拷贝，也可以
	// 用不同的库或框架
	users = Object.assign( {}, users );

	fetchUserData( userId );

        // 返回拷贝过的状态 
	return users;


	// ***********************

        // 原始的没被改变的纯函数：
	function fetchUserData(userId) {
		ajax( "http://some.api/user/" + userId, function onUserData(userData){
			users[userId] = userData;
		} );
	}
}
```
`userId` 和 `users` 都是原始的的 `fetchUserData` 的输入，`users` 也是输出。`safer_fetchUserData(..)` 取出他们的输入，并返回 `users`。为了确保在 `users` 被改变时我们不会在外部创建副作用，我们制作一个 `users` 的本地副本。

这种技术的有效性有限，主要是因为如果你不能将函数本身改为纯的，你也几乎不可能修改其周围的代码。然而，如果可能，探索它是有帮助的，因为它是所有修复方法中最简单的。

无论这是否是重构纯函数的一个实际方法，最重要的是函数的纯度仅仅需要深入到皮肤。也就是说，**函数的纯度是从外部判断的**，不管内部是什么。只要一个函数的使用表现为纯的，它就是纯的。在纯函数的内部，由于各种原因，包括最常见的性能方面，可以适度的使用不纯的技术。正如他们所说的“世界是一只驮着一只一直驮下去的乌龟群”。

不过要小心。程序的任何部分都是不纯的，即使它仅仅是用纯函数包裹的，也是代码错误和困惑读者的潜在的根源。总体目标是尽可能减少副作用，而不仅仅是隐藏它们。

### 覆盖效果

很多时候，你无法在容器函数的内部为了封装词法自由变量来修改代码。例如，不纯的函数可能位于一个你无法控制的第三方库文件中，其中包括：

```js
var nums = [];
var smallCount = 0;
var largeCount = 0;

function generateMoreRandoms(count) {
	for (let i = 0; i < count; i++) {
		let num = Math.random();

		if (num >= 0.5) {
			largeCount++;
		}
		else {
			smallCount++;
		}

		nums.push( num );
	}
}
```

蛮力的策略是，在我们程序的其余部分使用此通用程序时**隔离**副作用的方法时创建一个接口函数，执行以下步骤：

1. 捕获受影响的当前状态
2. 设置初始输入状态
3. 运行不纯的函数
4. 捕获副作用状态
5. 恢复原来的状态
6. 返回捕获的副作用状态

```js
function safer_generateMoreRandoms(count,initial) {
        // (1) 保存原始状态
	var orig = {
		nums,
		smallCount,
		largeCount
	};

        // (2) 设置初始副作用状态
	nums = initial.nums.slice();
	smallCount = initial.smallCount;
	largeCount = initial.largeCount;

        // (3) 当心杂质！
	generateMoreRandoms( count );

        // (4) 捕获副作用状态
	var sides = {
		nums,
		smallCount,
		largeCount
	};

        // (5) 重新存储原始状态
	nums = orig.nums;
	smallCount = orig.smallCount;
	largeCount = orig.largeCount;

        // (6) 作为输出直接暴露副作用状态
	return sides;
}
```

并且使用 `safer_generateMoreRandoms(..)`：

```js
var initialStates = {
	nums: [0.3, 0.4, 0.5],
	smallCount: 2,
	largeCount: 1
};

safer_generateMoreRandoms( 5, initialStates );
// { nums: [0.3,0.4,0.5,0.8510024448959794,0.04206799238...

nums;			// []
smallCount;		// 0
largeCount;		// 0
```

这需要大量的手动操作来避免一些副作用；如果我们一开始就没有它们，那就容易多了。但如果我们别无选择，那么这种额外的努力是值得的，以避免我们的项目出现意外。

**注意：**这种技术只有在处理同步代码时才有用。异步代码不能可靠地使用这种方法被管理，因为如果程序的其他部分在期间也在访问 / 修改状态变量，它就无法防止意外。

### 回避影响

当要处理的副作用的本质是直接输入值（对象、数组等）的突变时，我们可以再次创建一个接口函数来替代原始的不纯的函数去交互。

考虑一下：

```js
function handleInactiveUsers(userList,dateCutoff) {
	for (let i = 0; i < userList.length; i++) {
		if (userList[i].lastLogin == null) {
			// remove the user from the list
			userList.splice( i, 1 );
			i--;
		}
		else if (userList[i].lastLogin < dateCutoff) {
			userList[i].inactive = true;
		}
	}
}
```

`userList` 数组本身，加上其中的对象，都发生了改变。防御这些副作用的一种策略是先做一个深拷贝（不是浅拷贝）：

```js
function safer_handleInactiveUsers(userList,dateCutoff) {
        // 拷贝列表和其中 `user` 的对象
	let copiedUserList = userList.map( function mapper(user){
	        // 拷贝 `user` 对象
		// copy a `user` object
		return Object.assign( {}, user );
	} );

        // 使用拷贝过的对象调用最初的函数
	handleInactiveUsers( copiedUserList, dateCutoff );
         
	// 将突变的 list 作为直接的输出暴露出来
	return copiedUserList;
}
```

这个技术的成功将取决于你所做的**复制**的深度。使用 `userList.slice()` 在这里不起作用，因为这只会创建一个 `userList` 数组本身的浅拷贝。数组的每个元素都是一个需要复制的对象，所以我们需要格外小心。当然，如果这些对象在它们之内有对象（可能会这样），则复制需要更加完善。

#### 再看一下 `this`

另一个参数变化的副作用是和 `this` 有关的，我们应该意识到 `this` 是函数隐式的输入。查看第 2 章中的 “什么是This”获取更多的信息，为什么 `this` 关键字对函数式编程者是不确定的。

思考一下：

```js
var ids = {
	prefix: "_",
	generate() {
		return this.prefix + Math.random();
	}
};
```

我们的策略类似于上一节的讨论：创建一个接口函数，强制 `generate()` 函数使用可预测的 `this` 上下文：

```js
function safer_generate(context) {
	return ids.generate.call( context );
}

// *********************

safer_generate( { prefix: "foo" } );
// "foo0.8988802158307285"
```
这些策略绝对不是愚蠢的；对副作用的最安全的保护是不要产生它们。但是，如果您想提高程序的可读性和你对程序的自信，无论在什么情况下尽可能减少副作用 / 效果是巨大的进步。

本质上，我们并没有真正消除副作用，而是克制和限制它们，以便我们的代码更加的可验证和可靠。如果我们后来遇到程序错误，我们就知道代码仍然产生副作用的部分最有可能是罪魁祸首。

## 总结

副作用对代码的可读性和质量都有害，因为它们使您的代码难以理解。副作用也是程序中最常见的错误**原因**之一，因为很难应对他们。幂等是通过本质上创建仅有一次的操作来限制副作用的一种策略。

避免副作用的最优方法是使用纯函数。纯函数给定相同输入时总返回相同输出，并且没有副作用。引用透明更近一步的状态是 —— 更多的是一种脑力运动而不是文字行为 —— 纯函数的调用是可以用它的输出来代替，并且程序的行为不会被改变。

将一个不纯的函数重构为纯函数是首选。但是，如果无法重构，尝试封装副作用，或者创建一个纯粹的接口来解决问题。

没有程序可以完全没有副作用。但是在实际情况中的很多地方更喜欢纯函数。尽可能地收集纯函数的副作用，这样当错误发生时更加容易识别和审查出最像罪魁祸首的错误。
