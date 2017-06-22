# Functional-Light JavaScript
# 第5章：减少副作用

在第2章，我们讨论了一个函数除了它的返回值之外还有什么输出。现在你应该很熟悉用函数式编程的方法定义一个函数了，所以对于函数式编程的副作用你应该有所了解。
In Chapter 2, we discussed how a function can have outputs besides its `return` value. By now you should be very comfortable with the FP definition of a function, so the idea of such side outputs -- side effects! -- should smell.

我们将检查各种各样不同的副作用并且要看看他们为什么会对我们的代码质量和可读性造成损害。
We're going to examine the various different forms of side effects and see why they are harmful to our code's quality and readability.

这一章的要点是：编写出没有副作用的程序是不可能的。当然，也不是不可能，你当然可以编写出没有副作用的程序。但是这样的话程序就不会做任何有用和明显的事情。如果你编写出来一个零副作用的程序，你就无法区分它和一个被删除的或者空程序的区别。
But let me not bury the lede here. The punchline to this chapter: it's impossible to write a program with no side effects. Well, not impossible; you certainly can. But that program won't do anything useful or observable. If you wrote a program with zero side effects, you wouldn't be able to tell the difference between it and a deleted or empty program.

函数式编程者并没有消除所有的副作用。实际上，我们的目标是尽可能的限制他们。要做到这一点，我们首先需要完全理解函数式编程的副作用。
The FPer doesn't eliminate all side effects. Rather, the goal is to limit them as much as possible. To do that, we first need to fully understand them.

## 什么是副作用
## Effects On The Side, Please

因果关系：举一个我们人类对周围世界影响的最基本、最直观的例子，推一下放在桌子边沿上的一本书，书会掉落。不需要你拥有一个物理学的学位你也会知道，这是因为你刚刚推了书并且书掉落是因为地心引力，这是一个明确并直接的关系。
Cause and effect: one of the most fundamental, intuitive observations we humans can make about the world around us. Push a book off the edge of a table, it falls to the ground. You don't need a physics degree to know the cause was you pushing the book and the effect was gravity pulling it to the ground. There's a clear and direct relationship.

在编程中，我们也完全会处理因果关系。如果你调用了一个函数（起因），就会在屏幕上输出一条消息（结果）。
In programming, we also deal entirely in cause and effect. If you call a function (cause), it displays a message on the screen (effect).

当我们在阅读程序的时候，能够清晰明确的识别每一个起因和每一个结果是非常重要的。在某种程度上，通读程序但不能看到因果的直接关系，程序的可读性就会降低。
When reading a program, it's supremely important that the reader be able to clearly identify each cause and each effect. To any extent where a direct relationship between cause and effect cannot be seen readily upon a read-through of the program, that program's readability is degraded.

思考一下：
Consider:

```js
function foo(x) {
	return x * 2;
}

var y = foo( 3 );
```

在这段代码中，有很直接的因果关系，调用值为`3`的 foo 将具有返回值`6`的效果，调用函数`foo()`是起因，然后将其赋值给`y`是结果。这里没有歧义，传入参数为3将会返回6，将函数结果赋值给变量`y`是结果。
In this trivial program, it is immediately clear that calling foo (the cause) with value `3` will have the effect of returning the value `6` that is then assigned to `y` (the effect). There's no ambiguity here.

但是当这种情况：
But now:

```js
function foo(x) {
	y = x * 2;
}

var y;

foo( 3 );
```
这段代码有相同的输出，但是却有很大的差异，这里的因果是没有联系的。这个影响是间接的。这种方式设置`y`就是我们所说的副作用。
This program has the exact same outcome. But there's a very big difference. The cause and the effect are disjoint. The effect is indirect. The setting of `y` in this way is what we call a side effect.

**注意：** 当函数引用外部变量时，这个变量就称为自由变量。并不是所有的自由变量引用都是不好的，但是我们要对它们非常小心。
**Note:** When a function makes a reference to a variable outside itself, this is called a free variable. Not all free variable references will be bad, but we'll want to be very careful with them.

假使我给你一个引用来调用函数`bar(..)`，你看不到代码，但是我告诉你这段代码并没有间接的副作用，只有一个显式的返回值会怎么样？
What if I gave you a reference to call a function `bar(..)` that you cannot see the code for, but I told you that it had no such indirect side effects, only an explicit `return` value effect?

```js
bar( 4 );			// 42
```

因为你知道`bar(..)`的内部结构不会有副作用，你可以像这样直接的调用`bar(..)`。但是如果你不知道`bar(..)`没有副作用，为了理解调用这个函数的结果，你必须去阅读和分析它的逻辑。这对读者来说是额外的负担。
Because you know that the internals of `bar(..)` do not create any side effects, you can now reason about any `bar(..)` call like this one in a much more straightforward way. But if you didn't know that `bar(..)` had no side effects, to understand the outcome of calling it, you'd have to go read and dissect all of its logic. This is extra mental tax burden for the reader.

**有副作用的函数可读性更低，**因为它需要更多的阅读来理解程序。
**The readability of a side effecting function is less** because it requires more reading to understand the program.

但是程序往往比这个要复杂，思考一下：
But the problem goes deeper than that. Consider:

```js
var x = 1;

foo();

console.log( x );

bar();

console.log( x );

baz();

console.log( x );
```
你能确定每次`console.log(x)`的值都是你想要的吗？
How sure are you what values are going to be printed at each `console.log(x)`?

答案是否定的。如果你不确定函数`foo()`、`bar()`和`baz()`是否有副作用，你就不能保证每一步的`x`将会是什么，除非你检查每个步骤的实现，然后从第一行开始跟踪程序，跟踪所有状态的改变。
The correct answer is: not at all. If you're not sure whether `foo()`, `bar()`, and `baz()` are side-effecting or not, you cannot guarantee what `x` will be at each step unless you inspect the implementations of each, **and** then trace the program from line one forward, keeping track of all the changes in state as you go.

换句话说，`console.log(x)`最后的结果是不能分析和预测的，除非你已经在心里将整个程序执行到这里了。
In other words, the final `console.log(x)` is impossible to analyze or predict unless you've mentally executed the whole program up to that point.

猜猜谁擅长运行你的程序？JS引擎。猜猜谁不擅长运行你的程序？你代码的读者。然而，如果你选择在一个或多个函数调用中编写带有（潜在）副作用的代码，那么这意味着你已经使你的读者必须将你的程序完整地执行到某一行，以便他们理解这一行。
Guess who's good at running your program? The JS engine. Guess who's not as good at running your program? The reader of your code. And yet, your choice to write code with (potentially) side effects in one or more of those function calls means that you've burdened the reader with having to mentally execute your program in its entirety up to a certain line, for them to understand that line.

如果 `foo()`, `bar()`, 和 `baz()`都没有副作用的话，它们就不会影响到`x`，这就意味着我们不需要在心里默默地执行它们并且跟踪`x`的变化。这在精力上负担更小，并且使得代码更加的可读。
If `foo()`, `bar()`, and `baz()` were all free of side effects, they could not affect `x`, which means we do not need to execute them to mentally trace what happens with `x`. This is less mental tax, and makes the code more readable.

### 潜在的原因
### Hidden Causes

输出和状态的变化，是最常被引用的副作用的表现。但是另一个有损可读性的实践是一些被认为的侧因，思考一下：
Outputs, changes in state, are the most commonly cited manifestation of side effects. But another readability-harming practice is what some refer to as side causes. Consider:

```js
function foo(x) {
	return x + y;
}

var y = 3;

foo( 1 );			// 4
```

`y`不会随着`foo(..)`改变，所以这和我们之前看到的副作用有所不同。但是现在，对函数`foo(..)`的调用实际上取决于`y`当前的状态。之后我们如果这样做：
`y` is not changed by `foo(..)`, so it's not the same kind of side effect as we saw before. But now, the calling of `foo(..)` actually depends on the presence and current state of a `y`. If later, we do:

```js
y = 5;

// ..

foo( 1 );			// 6
```

我们可能会感到惊讶两次调用 `foo(1)`返回的结果不一样。
Might we be surprised that the call to `foo(1)` returned different results from call to call?

`foo(..)`对可读性有一个间接的破坏性。如果没有对函数`foo(..)`进行仔细检查 ，使用者可能不会知道导致这个输出的原因。这**看起来**仅仅像是参数`1`的原因，但却不是这样的。
`foo(..)` has an indirection of cause that is harmful to readability. The reader cannot see, without inspecting `foo(..)`'s implementation carefully, what causes are contributing to the output effect. It *looks* like the argument `1` is the only cause, but it turns out it's not.

为了帮助可读性，所有决定`foo(..)`输出的原因应该被设置的直接并明显。函数的使用者将会直接看到原因和结果。
To aid readability, all of the causes that will contribute to determining the effect output of `foo(..)` should be made as direct and obvious inputs to `foo(..)`. The reader of the code will clearly see the cause(s) and effect.

#### 使用固定的状态
#### Fixed State

避免副作用就意味着函数`foo(..)`不能引用自由变量了吗？
Does avoiding side causes mean the `foo(..)` function cannot reference any free variables?

考虑下这段代码
Consider this code:

```js
function foo(x) {
	return x + bar( x );
}

function bar(x) {
	return x * 2;
}

foo( 3 );			// 9
```

很明显，对于函数`foo(..)`和函数`bar(..)`，唯一和直接的原因就是参数`x`。但是`bar(x)`被称为什么呢？`bar`仅仅只是一个标识符，在JS中，默认情况下，它甚至不是一个常量（不可重新分配的变量）。`foo(..)`函数依赖于`bar`的值，`bar`作为一个自由变量被第二个函数引用。
It's clear that for both `foo(..)` and `bar(..)`, the only direct cause is the `x` parameter. But what about the `bar(x)` call? `bar` is just an identifier, and in JS it's not even a constant (non-reassignable variable) by default. The `foo(..)` function is relying on the value of `bar` -- a variable that references the second function -- as a free variable.

所以说这个函数还依赖于其他的原因吗？
So is this program relying on a side cause?

我认为不。虽然*可以*用其他的函数来重写`bar`这个变量，但是在代码中我没有这样做，这也不是我的惯例或先例。无论出于什么意图和目的，我的函数都是常量（从不重新分配）。
I say no. Even though it is *possible* to overwrite the `bar` variable's value with some other function, I am not doing so in this code, nor is it a common practice of mine or precedent to do so. For all intents and purposes, my functions are constants (never reassigned).

考虑一下：
Consider:

```js
const PI = 3.141592;

function foo(x) {
	return x * PI;
}

foo( 3 );			// 9.424776000000001
```

**注意：**JavaScript有内置的`Math.PI`属性，所以我们在本文中仅仅是用`PI`做一个方便的说明。在实践中，总是使用`Math.PI`而不是你自己定义的。
**Note:** JavaScript has `Math.PI` built-in, so we're only using the `PI` example in this text as a convenient illustration. In practice, always use `Math.PI` instead of defining your own!

上面的代码怎么样呢？`PI`是否会对函数`foo(..)`造成其他的影响？
How about the above code snippet? Is `PI` a side cause of `foo(..)`?

两个观察结果将会合理的帮助我们回答这个问题：
Two observations will help us answer that question in a reasonable way:

1. 想一下是否每次调用`foo(3)`，都将会返回`9.424..`？**答案是肯定的。**如果每一次都给一个相同的输入（`x`），那么都将会返回相同的输出。
1. Think about every call you might ever make to `foo(3)`. Will it always return that `9.424..` value? **Yes.** Every single time. If you give it the same input (`x`), it will always return the same output.

2. 你能用`PI`的当前值来代替每一个`PI`吗，并且程序能够和之前一样**正确的**的运行吗？**是的。**程序没有任何一部分依赖于`PI`值的改变，因为`PI`的类型是`const`，它是不能再分配的，所以变量`PI`在这里只是为了便于阅读和维护。它的值可以在不改变程序行为的情况下内联。
2. Could you replace every usage of `PI` with its immediate value, and could the program run **exactly** the same as it did before? **Yes.** There's no part of this program that relies on being able to change the value of `PI` -- indeed since it's a `const`, it cannot be reassigned -- so the `PI` variable here is only for readability/maintenance sake. Its value can be inlined without any change in program behavior.

我的结论是：这里的`PI`并不违反减少或避免副作用（或原因）的精神。在之前的代码也没有调用`bar(x)`。
My conclusion: `PI` here is not a violation of the spirit of minimizing/avoiding side effects (or causes). Nor is the `bar(x)` call in the previous snippet.

在这两种情况下，`PI`和`bar`都不是程序状态的一部分。它们是固定的，不可重新分配的（“常量”）的引用。如果他们在整个程序中都不改变，那么我们就不需要担心将他们作为变化的状态追踪他们。同样的，他们不会损害程序的可读性。而且它们也不能成为与变量以意想不到的方式发生变化相关的错误的根源。
In both cases, `PI` and `bar` are not part of the state of the program. They're fixed, non-reassignable ("constant") references. If they don't change throughout the program, we don't have to worry about tracking them as changing state. As such, they don't harm our readability. And they cannot be the source of bugs related to variables changing in unexpected ways.

**注意：**在我看来，使用`const`并不能说明将`PI`没有产生副作用的原因；使用`var PI`也会是同样的结果。`PI`没有被重新分配是问题的关键，而不是有没有使用`const`。我们将在后面的章节讨论`const`。
**Note:** The use of `const` above does not, in my opinion, make the case that `PI` is absolved as a side cause; `var PI` would lead to the same conclusion. The lack of reassigning `PI` is what matters, not the inability to do so. We'll discuss `const` in a later chapter.

#### 随机性
#### Randomness

你以前可能从来没有考虑过，但是随机性是不纯的。一个使用`Math.random()`的函数永远都不是纯的，因为你不能根据它的输入来保证和预测它的输出。所以任何生成唯一随机的ID等都需要依靠程序的其他原因。
You may never have considered it before, but randomness is impure. A function that uses `Math.random()` can never be pure, because you cannot ensure/predict its output based on its input. So any code that generates unique random IDs/etc will by definition be considered reliant on your program's side causes.

在计算中，我们使用的是伪随机算法。事实证明，真正的随机是非常难的，所以我们只是用复杂的算法来模拟它，产生的值看起来是随机的。这些算法计算很长的一串数字，但秘密是，如果你知道起始点，实际上这个序列是可以预测的。这个起点被称之为种子。
In computing, we use what's called pseudo-random algorithms for generation. Turns out true randomness is pretty hard, so we just kinda fake it with complex algorithms that produce values that seem observably random. These algorithms calculate long streams of numbers, but the secret is, the sequence is actually predictable if you know the starting point. This starting point is referred to as a seed.

一些语言允许你指定生成随机数的种子。如果你总是指定了相同的种子，那么你将始终从后续的“随机数”中得到相同的输出序列。这对于测试是非常有用的，但是在真正的应用中使用也是非常危险的。
Some languages let you specify the seed value for the random number generation. If you always specify the same seed, you'll always get the same sequence of outputs from subsequent "random number" generations. This is incredibly useful for testing purposes, for example, but incredibly dangerous for real world application usage.

在JS中，`Math.random()`的随机性计算是基于间接输入，因为你不能明确种子。因此，我们必须将内建的随机数生成视为不纯的一方。
In JS, the randomness of `Math.random()` calculated is based on an indirect input, because you cannot specify the seed. As such, we have to treat built-in random number generation as an impure side cause.

### I/O效果
### I/O Effects

这可能不太明显，但是最常见（并且本质上不可避免）的副作用就是I/O（输入/输出）。一个没有I/O的程序是完全没有意义的，因为它的工作不能以任何方式被观察到。一个有用的程序必须最少有一个输出，并且也需要输入。输入会产生输出。
It may not have been terribly obvious yet, but the most common (and essentially unavoidable) form of side cause/effect is I/O (input/output). A program with no I/O is totally pointless, because its work cannot be observed in any way. Useful programs must at a minimum have output, and many also need input. Input is a side cause and output is a side effect.

用户事件（鼠标、键盘）是JS编程者在浏览器中使用的典型的输入，而输出的则是DOM。如果你使用Node.js更多，你更有可能接收到来自文件系统、网络系统和/或`stdin`/`stdout`（标准输入流/标准输出流）的输入和输出.
The typical input for the browser JS programmer is user events (mouse, keyboard) and for output is the DOM. If you work more in Node.js, you may more likely receive input from, and send output to, the file system, network connections, and/or the `stdin`/`stdout` streams.

事实上，这些来源既可以是输入也可以是输出，是因也是果。以DOM为例，我们更新（产生副作用的结果）一个DOM元素为了给用户展示文字或图片信息，但是DOM的当前状态是对这些操作的隐式输入（产生副作用的原因）
As a matter of fact, these sources can be both input and output, both cause and effect. Take the DOM, for example. We update (side effect) a DOM element to show text or an image to the user, but the current state of the DOM is an implicit input (side cause) to those operations as well.

### 其他的错误
### Side Bugs

在程序运行期间侧因和副作用可能导致的错误是多种多样的。让我们来看一个场景来说明这些危害，希望它们能帮助我们认识到在我们自己的程序中类似的错误。
The scenarios where side causes and side effects can lead to bugs are as varied as the programs in existence. But let's examine a scenario to illustrate these hazards, in hopes that they help us recognize similar mistakes in our own programs.

思考一下
Consider:

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
		        //为每个用户保留一个最新的订单
			// keep a reference to latest order for each user
			users[userId].latestOrder = orders[i];
			userOrders[orders[i].orderId] = orders[i];
		}
	} );
}

function deleteOrder(orderId) {
	var user = users[ userOrders[orderId].userId ];
	var isLatestOrder = (userOrders[orderId] == user.latestOrder);

        //删除用户的最新订单？
	// deleting the latest order for a user?
	if (isLatestOrder) {
		hideLatestOrderDisplay();
	}

	ajax( "http://some.api/delete/order/" + orderId, function onDelete(success){
		if (success) {
			// deleted the latest order for a user?
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

我敢打赌，一些读者会显然的发现其中潜在的错误。如果回调`onOrders(..)`在`onUserData(..)`回调之前运行，它会给一个尚未设置的值（`users[userId]`的`userData`对象）添加一个`latestOrder`属性
I bet for some of you readers one of the potential bugs here is fairly obvious. If the callback `onOrders(..)` runs before the `onUserData(..)` callback, it will attempt to add a `latestOrder` property to a value (the `userData` object at `users[userId]`) that's not yet been set.

因此，这种依赖于因果关系的“错误”是在两种不同操作（是否异步）紊乱情况下发生的，我们期望以确定的顺序运行，但在某些情况下，可能会以不同的顺序运行。有一些策略可以确保操作的顺序，很明显，在这种情况下顺序是至关重要的。
So one form of "bug" that can occur with logic that relies on side causes/effects is the race condition of two different operations (async or not!) that we expect to run in a certain order but under some cases may run in a different order. There are strategies for ensuring the order of operations, and it's fairly obvious that order is critical in that case.

这里还有另一个细小的错误，你发现了吗？
Another more subtle bug can bite us here. Did you spot it?

考虑下这个调用顺序：
Consider this order of calls:

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

你发现每一对`fetchOrders(..)`/`onOrders(..)`和`deleteOrder(..)`/`onDelete(..)`都是交替出现了吗？这个潜在的排序会伴随着我们状态管理的侧因/副作用暴漏出一个奇怪的状态。
Do you see the interleaving of `fetchOrders(..)` / `onOrders(..)` with the `deleteOrder(..)` / `onDelete(..)` pair? That potential sequencing exposes a weird condition with our side causes/effects of state management.

在设置`isLatestOrder`标志和使用它来决定是否应该清空`users`中的用户数据对象的`latestOrder`属性时，会有一个延迟（因为回调）。在此延迟期间，如果`onOrders(..)`销毁，它可以潜在地改变用户的`latestOrder`引用的顺序值。当`onDelete(..)`在销毁之后，它会假定它仍然需要重新引用`latestOrder`。
There's a delay in time (because of the callback) between when we set the `isLatestOrder` flag and when we use it to decide if we should empty the `latestOrder` property of the user data object in `users`. During that delay, if `onOrders(..)` callback fires, it can potentially change which order value that user's `latestOrder` references. When `onDelete(..)` then fires, it will assume it still needs to unset the `latestOrder` reference.

错误：数据（状态）*可能*不同步。当进入`onOrders(..)`时，`latestOrder`可能仍然指向一个较新的顺序，这样`latestOrder`就会被重置。
The bug: the data (state) *might* now be out of sync. `latestOrder` will be unset, when potentially it should have stayed pointing at a newer order that came in to `onOrders(..)`.

这种错误最糟糕的是你不能和其他错误一样得到程序崩溃的异常。我们只是有一个不正确的状态；我们的应用程序“默默的”崩溃。
The worst part of this kind of bug is that you don't get a program-crashing exception like we did with the other bug. We just simply have state that is incorrect; our application's behavior is "silently" broken.

`fetchUserData(..)`和`fetchOrders(..)`的序列依赖是相当明显的，并且直截了当的处理。但是，在`fetchOrders(..)`和`deleteOrder(..)`之间存在潜在的序列依赖关系，就不太清楚了。这两个似乎更加独立。并且确保他们的顺序被保留是比较棘手的，因为你事先不知道（在`fetchOrders(..)`产生结果之前）是否必须要按照这样的顺序执行。
The sequencing dependency between `fetchUserData(..)` and `fetchOrders(..)` is fairly obvious, and straightforwardly addressed. But it's far less clear that there's a potential sequencing dependency between `fetchOrders(..)` and `deleteOrder(..)`. These two seem to be more independent. And ensuring that their order is preserved is more tricky, because you don't know in advance (before the results from `fetchOrders(..)`) whether that sequencing really must be enforced.

是的，一旦`deleteOrder(..)`销毁，你就能重新计算`isLatestOrder`标志。但是现在你有另一个问题：你的UI状态可能不同步。
Yes, you can recompute the `isLatestOrder` flag once `deleteOrder(..)` fires. But now you have a different problem: your UI state can be out of sync.

如果你之前已经调用过`hideLatestOrderDisplay()`，现在你需要调用`showLatestOrderDisplay()`，但是如果一个新的`latestOrder`已经被设置好了，你将要跟踪至少三个状态：被删除的状态是否本来是“最新的”、是否是“最新”设置的，和这两个顺序有什么不同吗？这些都是可以解决的问题，但无论如何都是不明显的。
If you had called the `hideLatestOrderDisplay()` previously, you'll now need to call `showLatestOrderDisplay()`, but only if a new `latestOrder` has in fact been set. So you'll need to track at least three states: was the deleted order the "latest" originally, and is the "latest" set, and are those two orders different? These are solvable problems, of course. But they're not obvious by any means.

所有这些麻烦都是因为我们决定在一组共享的状态下构造出有副作用/效果的代码。
All of these hassles are because we decided to structure our code with side causes/effects on a shared set of state.

函数式编程人员讨厌这类因果的错误因为这有损我们的阅读、推理、验证和最终**相信**代码的能力。这就是为什么他们要如此严肃地对待避免副作用/效果的原因。
Functional programmers detest these sorts of side cause/effect bugs because of how much it hurts our ability read, reason about, validate, and ultimately **trust** the code. That's why they take the principle to avoid side causes/effects so seriously.

有很多避免/修复副作用的策略。我们将在本章后面和后面的章节中讨论。我要说一个确定的事情：**写出有副作用/效果的代码是很正常的，**所以我们需要谨慎和刻意的避免产生有副作用的代码。
There are multiple different strategies for avoiding/fixing side causes/effects. We'll talk about some later in this chapter, and others in later chapters. I'll say one thing for certain: **writing with side causes/effects is often of our normal default** so avoiding them is going to require careful and intentional effort.

## 一次就好
## Once Is Enough, Thanks

如果你必须要使用副作用来改变状态，那么一种对限制潜在问题有用的操作是幂等。如果你的值的更新是幂次的，那么数据将会适应你可能有不同副作用来源的多个此类更新的情况。
If you must make side effect changes to state, one class of operations that's useful for limiting the potential trouble is idempotence. If your update of a value is idempotent, then data will be resilient to the case where you might have multiple such updates from different side effect sources.

幂等的定义有点让人困惑；数学家和程序员使用幂等的含义稍有不同。然而，这两种观点对于函数式程序员都是有用的。
The definition of idempotence is a little confusing; mathematicians use a slightly different meaning than programmers typically do. However, both perspectives are useful for the functional programmer.

首先，让我们给出一个计数器的例子，它既不是数学上的，也不是程序上的幂等:
First, let's give a counter example that is neither mathematically nor programmingly idempotent:

```js
function updateCounter(obj) {
	if (obj.count < 10) {
		obj.count++;
		return true;
	}

	return false;
}
```

这个函数通过引用递增`obj.count`来该改变一个对象，所以对这个对象产生了副作用。如果`updateCounter(o)`被多次调用。当 `o.count`小于10，即程序状态每次都要更改。另外，`updateCounter(..)`的输出是一个布尔值，这不适合返回到`updateCounter(..)`的后续调用。
This function mutates an object via reference by incrementing `obj.count`, so it produces a side effect on that object. If `updateCounter(o)` is called multiple times -- while `o.count` is less than `10`, that is -- the program state changes each time. Also, the output of `updateCounter(..)` is a boolean, which is not suitable to feed back into a subsequent call of `updateCounter(..)`.

### 数学中的幂等
### Mathematic Idempotence

从数学的角度来看，幂等指的是在第一次调用后，其输出永远不会改变的操作，如果你将该输出一次又一次地输入到操作中。换句话说，`foo(x)`将产生与`foo(foo(x))`、`foo(foo(foo(x)))`等相同的输出。
From the mathematical point of view, idempotence means an operation whose output won't ever change after the first call, if you feed that output back into the operation over and over again. In other words, `foo(x)` would produce the same output as `foo(foo(x))`, `foo(foo(foo(x)))`, etc.

一个典型的数学例子是`Math.abs(..)`（取绝对值）。`Math.abs(-2)`的结果是`2`，和`Math.abs(Math.abs(Math.abs(Math.abs(-2))))`的结果相同。像`Math.min(..)`、 `Math.max(..)`、 `Math.round(..)`、 `Math.floor(..)` 和 `Math.ceil(..)`这些工具函数都是幂等的。
A typical mathematic example is `Math.abs(..)` (absolute value). `Math.abs(-2)` is `2`, which is the same result as `Math.abs(Math.abs(Math.abs(Math.abs(-2))))`. Utilities like `Math.min(..)`, `Math.max(..)`, `Math.round(..)`, `Math.floor(..)` and `Math.ceil(..)` are also idempotent.

我们可以用同样的特征来定义一些数学运算：
Some custom mathematical operations we could define with this same characteristic:

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

数学风格的幂等**不**仅限于数学运算。我们还可以用JavaScript的原始类型来说明幂等的另一种形式：
Mathematical-style idempotence is **not** restricted to mathematic operations. Another place we can illustrate this form of idempotence is with JavaScript primitive type coercions:

```js
var x = 42, y = "hello";

String( x ) === String( String( x ) );				// true

Boolean( y ) === Boolean( Boolean( y ) );			// true
```

在本文的前面，我们探究了一种常见的函数式编程工具，它可以实现这种形式的幂等:
Earlier in the text, we explored a common FP tool that fulfills this form of idempotence:

```js
identity( 3 ) === identity( identity( 3 ) );	// true
```

某些字符串操作自然也是幂等的，例如:
Certain string operations are also naturally idempotent, such as:

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

我们甚至可以以一种幂等方式设计更复杂的字符串格式操作，比如:
We can even design more sophisticated string formatting operations in an idempotent way, such as:

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

`currency(..)`举例说明了一个重要的技巧：在某些情况下，开发人员可以采取额外的步骤来规范化输入/输出操作，以确保操作是幂等的来避免意外的发生。
`currency(..)` illustrates an important technique: in some cases the developer can take extra steps to normalize an input/output operation to ensure the operation is idempotent where it normally wouldn't be.

在任何可能的情况下通过幂等的操作限制副作用要比不做限制的更新要好得多。
Wherever possible, restricting side effects to idempotent operations is much better than unrestricted updates.

### 编程中的幂等
### Programming Idempotence

幂等的面向程序的定义也是类似的，但不太正式。编程中的幂等仅仅是`f(x);`的结果与`f(x); f(x)`相同而不是要求`f(x) === f(f(x))`。换句话说，之后每一次调用`f(x)`的结果和第一次调用`f(x)`的结果没有任何改变。
The programming-oriented definition for idempotence is similar, but less formal. Instead of requiring `f(x) === f(f(x))`, this view of idempotence is just that `f(x);` results in the same program behavior as `f(x); f(x);`. In other words, the result of calling `f(x)` subsequent times after the first call doesn't change anything.

这种观点更符合我们对副作用的观察。因为这更像是一个`f(..)`创建了一个幂等的副作用而不是必须要返回一个幂等的输出值。
That perspective fits more with our observations about side effects, because it's more likely that such an `f(..)` operation creates an idempotent side effect rather than necessarily returning an idempotent output value.

这种幂等性的方式经常被用于HTTP操作（动词），例如GET或PUT。如果HTTP REST API正确地遵循了幂等的规范指导，那么PUT被定义为一个更新操作，它可以完全替换资源。同样的，客户端可以一次或多次发送PUT请求（使用相同的数据），而服务器无论如何都将具有相同的结果状态。
This idempotence-style is often cited for HTTP operations (verbs) such as GET or PUT. If an HTTP REST API is properly following the specification guidance for idempotence, PUT is defined as an update operation that fully replaces a resource. As such, a client could either send a PUT request once or multiple times (with the same data), and the server would have the same resultant state regardless.

让我们用更具体的编程方法来考虑这个问题，来检查一下使用幂等和没有使用幂等是否产生副作用:
Thinking about this in more concrete terms with programming, let's examine some side effect operations for their idempotence (or not):

```js
// idempotent:
obj.count = 2;
a[a.length - 1] = 42;
person.name = upper( person.name );

// non-idempotent:
obj.count++;
a[a.length] = 42;
person.lastUpdated = Date.now();
```

记住：这里的幂等性的概念是每一个幂等运算（比如`obj.count = 2`）可以重复多次，而不是在第一次更新后改变程序操作。非幂等操作每次都改变状态。
Remember: the notion of idempotence here is that each idempotent operation (like `obj.count = 2`) could be repeated multiple times and not change the program operation beyond the first update. The non-idempotent operations change the state each time.

那么更新DOM呢？
What about DOM updates?

```js
var hist = document.getElementById( "orderHistory" );

// idempotent:
hist.innerHTML = order.historyText;

// non-idempotent:
var update = document.createTextNode( order.latestUpdate );
hist.appendChild( update );
```

这里的关键区别在于，幂等的更新替换了DOM元素的内容。DOM元素的当前状态是独立的，因为它是无条件覆盖的。非幂的操作将内容添加到元素中;隐式地，DOM元素的当前状态是计算下一个状态的一部分。
The key difference illustrated here is that the idempotent update replaces the DOM element's content. The current state of the DOM element is irrelevant, because it's unconditionally overwritten. The non-idempotent operation adds content to the element; implicitly, the current state of the DOM element is part of computing the next state.

我们将不会一直用幂等的方式去定义你的数据，但如果你能做到，这肯定会减少你的副作用在你最意想不到的时候突然出现的可能性。
It won't always be possible to define your operations on data in an idempotent way, but if you can, it will definitely help reduce the chances that your side effects will crop up to break your expectations when you least expect it.

## 纯粹的快乐
## Pure Bliss

没有副作用的函数称为纯函数。在编程的意义上，纯函数是一种等幂函数，因为它不可能有任何副作用。考虑一下
A function with no side causes/effects is called a pure function. A pure function is idempotent in the programming sense, since it cannot have any side effects. Consider:

```js
function add(x,y) {
	return x + y;
}
```

所有输入(`x`和`y`)和输出(`return ..`)都是直接的，没有自由变量引用。调用`add(3,4)`多次和调用一次是没有区别的。`add(..)`是纯粹的编程风格的幂等。
All the inputs (`x` and `y`) and outputs (`return ..`) are direct; there are no free variable references. Calling `add(3,4)` multiple times would be indistinguishable from only calling it once. `add(..)` is pure and programming-style idempotent.

然而，并不是所有的纯函数都是数学概念上的幂等，因为它们不需要返回一个值，这个值适合作为输入来返回。思考一下:
However, not all pure functions are idempotent in the mathematical sense, because they don't have to return a value that would be suitable for feeding back in as their own input. Consider:

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

输出的`9`并不是一个数组，所以你不能在`calculateAverage(calculateAverage( .. ))`中将其传入。
The output `9` is not an array, so you cannot pass it back in: `calculateAverage(calculateAverage( .. ))`.

正如我们前面所讨论的，一个纯函数*可以*引用自由变量，只要这些自由变量不是侧因。
As we discussed earlier, a pure function *can* reference free variables, as long as those free variables aren't side causes.

例子：
Some examples:

```js
const PI = 3.141592;

function circleArea(radius) {
	return PI * radius * radius;
}

function cylinderVolume(radius,height) {
	return height * circleArea( radius );
}
```

`circleArea(..)`中引用了自由变量`PI`，但是这是一个常量所以不是一个侧因。`cylinderVolume(..)`引用了自由变量`circleArea`，这也不是一个侧因，因为这个程序把它当作一个常量引用它的函数值。这两个函数都是纯的。
`circleArea(..)` references the free variable `PI`, but it's a constant so it's not a side cause. `cylinderVolume(..)` references the free variable `circleArea`, which is also not a side cause because this program treats it as, in effect, a constant reference to its function value. Both these functions are pure.

另一个例子，一个函数仍然可以是纯的，但引用的自由变量是闭包:
Another example where a function can still be pure but reference free variables is with closure:

```js
function unary(fn) {
	return function onlyOneArg(arg){
		return fn( arg );
	};
}
```

`unary(..)`本身显然是纯函数--它唯一的输入是`fn`，并且它唯一的输出是返回的函数--但是闭合了自由变量`fn`的内部函数`onlyOneArg(..)`是不是纯的呢?
`unary(..)` itself is clearly pure -- its only input is `fn` and its only output is the `return`ed function -- but what about the inner function `onlyOneArg(..)`, which closes over the free variable `fn`?

它仍然是纯的，因为`fn`永远不变。事实上，我们对这一事实有充分的信心，因为从词汇上讲，这几行是唯一可能重新分配`fn`的。
It's still pure because `fn` never changes. In fact, we have full confidence in that fact because lexically speaking, those few lines are the only ones that could possibly reassign `fn`.

**注意：**`fn`是一个函数对象的引用，它默认是一个可变的值。在程序的其他地方，例如在这个函数对象中添加一个属性，它在技术上“改变”值（突变，而不是重新分配）。然而，因为我们除了调用`fn`以外不依赖`fn`的任何事情，并且不可能影响函数值的可调用性，因此`fn`在最后的结果中仍然是有效的不变的;它不可能是一个侧因。
**Note:** `fn` is a reference to a function object, which is by default a mutable value. Somewhere else in the program *could* for example add a property to this function object, which technically "changes" the value (mutation, not reassignment). However, since we're not relying on anything about `fn` other than our ability to call it, and it's not possible to affect the callability of a function value, `fn` is still effectively unchanging for our reasoning purposes; it cannot be a side cause.

表达一个函数的纯度的另一种常用方法是：**给定相同的输入（一个或多个），它总是产生相同的输出。**如果你把`3`传给`circleArea(..)`它总是输出相同的结果(`28.274328`)。
Another common way to articulate a function's purity is: **given the same input(s), it always produces the same output.** If you pass `3` to `circleArea(..)`, it will always output the same result (`28.274328`).

如果一个函数每次在给予相同的输入时，**可能**产生不同的输出，那么它是不纯的。即使这样的函数总是返回相同的值，如果它产生间接输出副作用，那么程序状态每次被调用时都会被改变；这是不纯的。
If a function *can* produce a different output each time it's given the same inputs, it is impure. Even if such a function always `return`s the same value, if it produces an indirect output side effect, the program state is changed each time it's called; this is impure.

不纯的函数是不受欢迎的，因为它们使得所有的调用都变得更加难以理解。纯的函数的调用是完全可预测的。当有人阅读代码时，看到多个`circleArea(3)`调用，他们不需要花费额外的精力来计算**每次**的输出结果。
Impure functions are undesirable because they make all of their calls harder to reason about. A pure function's call is perfectly predictable. When someone reading the code sees multiple `circleArea(3)` calls, they won't have to spend any extra effort to figure out what its output will be *each time*.

### 相对的纯粹
### Purely Relative

当我们讨论一个函数是纯的时，我们必须非常小心。JavaScript的动态值特性使其很容易产生不明显的副作用。
We have to be very careful when talking about a function being pure. JavaScript's dynamic value nature makes it all too easy to have non-obvious side causes/effects.

思考一下：
Consider:

```js
function rememberNumbers(nums) {
	return function caller(fn){
		return fn( nums );
	};
}

var list = [1,2,3,4,5];

var simpleList = rememberNumbers( list );
```
`simpleList(..)`看起来是一个纯函数，因为它只涉及内部的`caller(..)`函数，它仅仅是闭合了自由变量`nums`。然而，有很多方法证明`simpleList(..)`是不纯的。
`simpleList(..)` looks like a pure function, as it's a reference to the inner function `caller(..)`, which just closes over the free variable `nums`. However, there's multiple ways that `simpleList(..)` can actually turn out to be impure.

首先，我们对纯度的断言是基于数组的值（通过`list`和`nums`引用）一直不改变:
First, our assertion of purity is based on the array value (referenced both by `list` and `nums`) never changing:

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

当我们改变数组时，`simpleList(..)`的调用改变它的输出。所以，`simpleList(..)`是纯呢还是不纯呢？这就取决于你的视角。对于给定的一组假设来说，它是纯函数。在任何没有`list.push(6)`的情况下中是纯的。
When we mutate the array, the `simpleList(..)` call changes its output. So, is `simpleList(..)` pure or impure? Depends on your perspective. It's pure for a given set of assumptions. It could be pure in any program that didn't have the `list.push(6)` mutation.

我们可以通过改变`rememberNumbers(..)`的定义来修改这种不纯。一种方法是复制`nums`数组:
We could guard against this kind of impurity by altering the definition of `rememberNumbers(..)`. One approach is to duplicate the `nums` array:

```js
function rememberNumbers(nums) {
	// make a copy of the array
	nums = nums.slice();

	return function caller(fn){
		return fn( nums );
	};
}
```

但一个更加棘手的副作用可能潜伏着:
But an even trickier hidden side effect could be lurking:

```js
var list = [1,2,3,4,5];

// make `list[0]` be a getter with a side effect
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
// [0] was accessed!
```

一个更粗鲁的选择是更改`rememberNumbers(..)`的参数。首先，不要接收数组，而是把数字作为单独的参数:
A perhaps more robust option is to change the signature of `rememberNumbers(..)` to not receive an array in the first place, but rather the numbers as individual arguments:

```js
function rememberNumbers(...nums) {
	return function caller(fn){
		return fn( nums );
	};
}

var simpleList = rememberNumbers( ...list );
// [0] was accessed!
```

这两个`...`的作用是将列表复制到`nums`中，而不是通过引用来传递。
The two `...`s have the effect of copying `list` into `nums` instead of passing it by reference.

**注意：**控制台消息的副作用不是来自于`rememberNumbers(..)`，而是从`...list`的扩展中。因此，在这种情况下，`rememberNumbers(..)`和`simpleList(..)`是纯的。
**Note:** The console message side effect here comes not from `rememberNumbers(..)` but from the `...list` spreading. So in this case, both `rememberNumbers(..)` and `simpleList(..)` are pure.

但是如果这种突变更难被发现呢？纯函数和不纯函数的合成总是产生不纯的函数。如果我们将一个不纯的函数传递到另一个纯`simpleList(..)`中，那么就是不纯的：
But what if the mutation is even harder to spot? Composition of a pure function with an impure function **always** produces an impure function. If we pass an impure function into the otherwise pure `simpleList(..)`, it's now impure:

```js
// yes, a silly contrived example :)
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

**注意：**不管`reverse()`看起来多安全（就像JS中的其他数组方法一样），它返回一个反向数组，实际上它对数组进行了修改，而不是创建一个新的数组。
**Note:** Despite `reverse()` looking safe (like other array methods in JS) in that it returns a reversed array, it actually mutates the array rather than creating a new one.

我们需要对`rememberNumbers(..)`有的一个更粗鲁的定义来防止`fn(..)`改变它的关闭的`nums`变量的引用。
We need a more robust definition of `rememberNumbers(..)` to guard against the `fn(..)` mutating its closed over `nums` via reference:

```js
function rememberNumbers(...nums) {
	return function caller(fn){
		// send in a copy!
		return fn( nums.slice() );
	};
}
```

所以`simpleList(..)`是可靠的纯的呢！？**不。**:(
So is `simpleList(..)` reliably pure yet!? **Nope.** :(

我们只防范我们可以控制的副作用（通过引用变异）。我们传递的任何带有副作用的函数，都将会污染`simpleList(..)`的纯度:
We're only guarding against side effects we can control (mutating by reference). Any function we pass that has other side effects will have polluted the purity of `simpleList(..)`:

```js
simpleList( function impureIO(nums){
	console.log( nums.length );
} );
```

事实上，没有办法定义`rememberNumbers(..)`去产生一个完美纯粹的 `simpleList(..)`的函数。
In fact, there's no way to define `rememberNumbers(..)` to make a perfectly-pure `simpleList(..)` function.

纯度是和信心有关的。但我们不得不承认，在很多情况下，我们所感受到的自信实际上是与我们的背景有关。在实践中（在JavaScript中），函数纯度的问题不是纯粹的纯粹性，而是关于其纯度的一系列信心。
Purity is about confidence. But we have to admit that in many cases, **any confidence we feel is actually relative to the context** of our program and what we know about it. In practice (in JavaScript) the question of function purity is not about being absolutely pure or not, but about a range of confidence in its purity.

越纯洁越好。制作纯函数时越努力，当您阅读使用它的代码时，您的信心就会越高，这将使代码的一部分更加可读。
The more pure, the better. The more effort you put into making a function pure(r), the higher your confidence will be when you read code that uses it, and that will make that part of the code more readable.

## 有或者无
## There Or Not

到目前为止，我们已经将函数纯度定义为一个没有副作用的函数，并且作为一个函数，给定相同的输入，总是产生相同的输出。这只是看待相同特征的两种不同方式。
So far, we've defined function purity both as a function without side causes/effects and as a function that, given the same input(s), always produces the same output. These are just two different ways of looking at the same characteristics.

但是，第三种看待函数纯性的方法，也许是广为接受的定义，即纯函数具有引用透明性。
But a third way of looking at function purity, and perhaps the most widely accepted definition, is that a pure function has referential transparency.

引用透明性是指一个函数调用可以被它的输出值所取代，并且整个程序的行为不会改变。换句话说，不可能从程序的执行中分辨出函数调用是被执行的，还是它的返回值是在函数调用的位置上内联的。
Referential transparency is the assertion that a function call could be replaced by its output value, and the overall program behavior wouldn't change. In other words, it would be impossible to tell from the program's execution whether the function call was made or its return value was inlined in place of the function call.

从引用透明的角度来看，这两个程序都有完全相同的行为因为它们都是用纯粹的函数构建的:
From the perspective of referential transparency, both of these programs have identical behavior as they are built with pure functions:

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

这两个片段之间的唯一区别在于，在后者中，我们跳过了调用`calculateAverage(nums)`并内联。因为程序的其他部分的行为是相同的，`calculateAverage(..)`是引用透明的，因此是一个纯粹的功能
The only difference between these two snippets is that in the latter one, we skipped the `calculateAverage(nums)` call and just inlined its ouput (`9`). Since the rest of the program behaves identically, `calculateAverage(..)` has referential transparency, and is thus a pure function.

### 精神上的透明
### Mentally Transparent

一个引用透明的纯函数*可能*会被它的输出替代，这并不意味着它*就会被正确的*被替换。远非如此。
The notion that a referentially transparent pure function *can be* replaced with its output does not mean that it *should literally be* replaced. Far from it.

我们用在程序中使用函数而不是使用预先计算好的常量的原因不仅仅是应对变化的数据，也是和可读性和适当的抽象等有关。调用函数去计算一列数字的平均值让这部分程序比只是使用确定的值更具有可读性。它向读者讲述了`avg`从何而来，它意味着什么，等等。
The reasons we build functions into our programs instead of using pre-computed magic constants are not just about responding to changing data, but also about readability with proper abstractions, etc. The function call to calculate the average of that list of numbers makes that part of the program more readable than the line that just assigns the value explicitly. It tells the story to the reader of where `avg` comes from, what it means, etc.

我们真正建议使用引用透明是当你阅读程序，一旦你已经在内心计算出纯函数调用输出的是什么的时候,当你看到它的代码的时候不需要再去思考确切的函数调用是做什么，特别是如果它出现很多次。
What we're really suggesting with referential transparency is that as you're reading a program, once you've mentally computed what a pure function call's output is, you no longer need to think about what that exact function call is doing when you see it in code, especially if it appears multiple times.

这个结果有一点像你在心里面定义一个`const`，当你阅读的时候，你可以直接跳过并且不需要花更多的精力去计算炼。
That result becomes kinda like a mental `const` declaration, which as you're reading you can transparently swap in and not spend any more mental energy working out.

希望纯函数的这种特性的重要性是显而易见的。我们正在努力使我们的程序更容易读懂。我们能做的一种方法是给读者较少的工作，通过提供帮助来跳过不必要的东西，这样他们就可以把注意力集中在重要的事情上。
Hopefully the importance of this characteristic of a pure function is obvious. We're trying to make our programs more readable. One way we can do that is give the reader less work, by providing assistance to skip over the unnecessary stuff so they can focus on the important stuff.

读者不需要重新计算一些不会改变的结果（也不需要改变）。如果用引用透明定义一个纯函数，读者就不必这样做了。
The reader shouldn't need to keep re-computing some outcome that isn't going to change (and doesn't need to). If you define a pure function with referential transparency, the reader won't have to.

### 不够透明？
### Not So Transparent?

那么如果一个有副作用的函数，并且这个副作用在程序的其他地方没有被观察到或者依赖会怎么样？这个功能还有引用透明吗？
What about a function that has a side effect, but this side effect isn't ever observed or relied upon anywhere else in the program? Does that function still have referential transparency?

这里有一个例子：
Here's one:

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
Did you spot it?

`sum`是一个`calculateAverage(..)`使用的外部自由变量。但是，每次我们使用相同的列表调用`calculateAverage(..)`，我们将得到`9`作为输出。并且这个程序无法和使用参数`9`调用`calculateAverage（nums）`在行为上区分开来。程序的其他部分和`sum`变量有关，所以这是一个不可观察的副作用。
`sum` is an outer free variable that `calculateAverage(..)` uses to do its work. But, every time we call `calculateAverage(..)` with the same list, we're going to get `9` as the output. And this program couldn't be distinguished in terms of behavior from a program that replaced the `calculateAverage(nums)` call with the value `9`. No other part of the program cares about the `sum` variable, so it's an unobserved side effect.

这是一个像这棵树一样不能观察到的副作用吗？
Is a side cause/effect that's unobserved like this tree?

> 如果一棵树落在森林里，但附近没有人听到它，它仍然会发出声音吗？
> If a tree falls in the forest, but no one is around to hear it, does it still make a sound?

通过引用透明的狭义的定义，我想你一定会说`calculateAverage(..)`仍然是一个纯函数。但是，由于在我们的学习中不仅仅是学习学术，而且与实用主义相平衡，我认为这个结论需要更多的观点。让我们探索一下
By the narrowest definition of referential transparency, I think you'd have to say `calculateAverage(..)` is still a pure function. But as we're trying to not just be academic in our study, but balanced with pragmatism, I think this conclusion needs more perspective. Let's explore.

#### 性能效果
#### Performance Effects

通常情况下，您会发现这些副作用--不可观察的用于优化操作的性能。例如：
Often times, you'll find these kind of side-effects-that-go-unobserved being used to optimize the performance of an operation. For example:

```js
var cache = [];

function specialNumber(n) {
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

这个愚蠢的`specialNumber（..）`算法是确定性的，并且，从定义来说，它总是为相同的输入提供相同的输出。从引用透明的角度来看，它也是纯的--用`22`替换对`specialNumber(42)` 的任何调用，程序的最终结果是相同的。
This silly `specialNumber(..)` algorithm is deterministic and thus pure from the definition that it always gives the same output for the same input. It's also pure from the referential transparency perspective -- replace any call to `specialNumber(42)` with `22` and the end result of the program is the same.

但是，这个函数必须做一些工作来计算一些较大的数字，特别是输入像`987654321`这样的数字。如果我们需要在我们的程序中多次获得特定的特殊号码，那么结果的缓存意味着后续的调用效率会更高。
However, the function has to do quite a bit of work to calculate some of the bigger numbers, especially the `987654321` input. If we needed to get that particular special number multiple times throughout our program, the `cache`ing of the result means that subsequent calls are far more efficient.

**注意：**思考一个有趣的事情：CPU在执行任何给定操作时产生的热量，即使是最纯粹的函数/程序，也是不可避免的副作用吗？那么CPU时间延迟，因为它花时间在一个纯操作上，然后再执行另一个操作？
**Note:** An interesting thing to ponder: is the heat produced by the CPU while performing any given operation an unavoidable side effect of even the most pure functions/programs? What about just the CPU time delay as it spends time on a pure operation before it can do another one?

假设你运行`specialNumber（987654321）`一次，并手动将该结果粘贴到一些变量/常量中。程序通常是高度模块化的并且全局可访问的范围并不是通常你想要在这些独立部分之间分享状态的方式。`specialNumber（..）`有自己的缓存（尽管恰好是使用一个全局变量来实现这一点），是更好的抽象的状态共享。
Don't be so quick to assume that you could just run the `specialNumber(987654321)` calculation once and manually stick that result in some variable / constant. Programs are often highly modularized and globally accessible scopes are not usually the way you want to go around sharing state between those independent pieces. Having `specialNumber(..)` do its own caching (even though it happens to be using a global variable to do so!) is a more preferable abstraction of that state sharing.

关键是，如果`specialNumber（..）`只是程序访问和更新`cache`副作用的唯一部分，那么引用透明的观点可以显然适用，这可以被看作是可以接受的实际的“欺骗”的纯函数思想。
The point is that if `specialNumber(..)` is the only part of the program that accesses and updates the `cache` side cause/effect, the referential transparency perspective observably holds true, and this might be seen as an acceptable pragmatic "cheat" of the pure function ideal.

但是真的应该这样吗？
But should it?

典型的，这种性能优化方面的副作用是通过隐藏缓存结果来实现的，因此它们*不能*被程序的任何其他部分所观察到。这个过程被称为"memorization"。 我一直认为这个词是"memorization"；我不知道这个想法是从哪里来的，但它肯定有助于我更好地理解这个概念。
Typically, this sort of performance optimization side effecting is done by hiding the caching of results so they *cannot* be observed by any other part of the program. This process is referred to as memoization. I always think of that word as "memorization"; I have no idea if that's even remotely where it comes from, but it certainly helps me understand the concept better.

思考一下：
Consider:

```js
var specialNumber = (function memoization(){
	var cache = [];

	return function specialNumber(n){
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
	};
})();
```

我们在`specialNumber(..)`IIFE的范围内包含`specialNumber(..)`的`cache`的副作用，所以现在我们确定程序的其他部分*能*观察到它们，不仅仅是它们*不*观察它们。
We've contained the `cache` side causes/effects of `specialNumber(..)` inside the scope of the `memoization()` IIFE, so now we're sure that no other parts of the program *can* observe them, not just that they *don't* observe them.

最后一句话似乎是一个的微妙观点，但实际上我认为这可能是**整章中最重要的一点**。再读一遍。
That last sentence may seem like a subtle point, but actually I think it might be **the most important point of the entire chapter**. Read it again.

回到这个哲学理论：
Back to this philosophical musing:

如果一棵树落在森林里，但没有人在听到它，是否依然发出声音？
> If a tree falls in the forest, but no one is around to hear it, does it still make a sound?

通过这个暗喻，我所得到的是：无论是否产生声音，如果我们从不创建一个我们不在树的附近树倒下时我们仍然可以听声音的方案。
Going with the metaphor, what I'm getting at is: whether the sound is made or not, it would be better if we never create a scenario where the tree can fall without us being around; we'll always hear the sound when a tree falls.

减少副作用的目的并不是他们在程序中不能被观察到，而是设计一个程序，让副作用尽可能的少，因为这使代码更容易理解。一个没有观察到的*发生*的副作用的程序在这个目标上并不像一个*不能*观察它们的程序那么有效。
The purpose of reducing side causes/effects is not per se to have a program where they aren't observed, but to design a program where fewer of them are possible, because this makes the code easier to reason about. A program with side causes/effects that *happen* to not be observed is not nearly as effective in this goal as a program that *cannot* observe them.

如果副作用可能发生，作者和读者必须尽量对付它们。使它们不会发生，作家和读者都会对任何部可能或不可能发生的事情都有更多的信心。
If side causes/effect can happen, the writer and reader must mentally juggle them. Make it so they can't happen, and both writer and reader will find more confidence over what can and cannot happen in any part.

## 精简
## Purifying

如果你有不纯的功能，你不能重构为纯的，你能做些什么？
What can you do if you have an impure function that you cannot refactor to be pure?

您需要确定该函数具有什么样的副作用。副作用的不同之处来自各方面，可能是由于词法自由变量、引用变化，甚至是`this`的绑定。 我们将研究解决这些情况的方法。
You need to figure what kind of side causes/effects the function has. It may be that the side causes/effects come variously from lexical free variables, mutations-by-reference, or even `this` binding. We'll look at approaches that address each of these scenarios.

### 封闭的影响
### Containing Effects

如果副作用的本质是使用词法自由变量，并且您可以选择修改周围的代码，那么您可以使用范围来封装它们。
If the nature of the concerned side causes/effects is with lexical free variables, and you have the option to modify the surrounding code, you can encapsulate them using scope.

Recall:

```js
var users = {};

function fetchUserData(userId) {
	ajax( "http://some.api/user/" + userId, function onUserData(userData){
		users[userId] = userData;
	} );
}
```

纯化此代码的一个选项是在变量和不纯的函数周围创建一个容器。本质上，容器必须接收所有的输入。
One option for purifying this code is to create a wrapper around both the variable and the impure function. Essentially, the wrapper has to receive as input "the entire universe" of state it can operate on.

```js
function safer_fetchUserData(userId,users) {
	// simple, naive ES6+ shallow object copy, could also
	// be done w/ various libs or frameworks
	users = Object.assign( {}, users );

	fetchUserData( userId );

	// return the copied state
	return users;


	// ***********************

	// original untouched impure function:
	function fetchUserData(userId) {
		ajax( "http://some.api/user/" + userId, function onUserData(userData){
			users[userId] = userData;
		} );
	}
}
```
`userId`和`users`都是原始的的`fetchUserData`的输入，`users`也被输出。 `safer_fetchUserData(..)`取出他们的输入，并返回`users`。为了确保在`users`被突变时我们不会在外部创建副作用，我们制作一个`users`的本地副本。
Both `userId` and `users` are input for the original `fetchUserData`, and `users` is also output. The `safer_fetchUserData(..)` takes both of these inputs, and returns `users`. To make sure we're not creating a side effect on the outside when `users` is mutated, we make a local copy of `users`.

这种技术的实用性有限，主要是因为如果你不能将函数本改为纯的，那么你也不可能修改其周围的代码。 然而，如果可能，探索它是有帮助的，因为它是我们最简单的修复。
This technique has limited usefulness mostly because if you cannot modify a function itself to be pure, you're not that likely to be able to modify its surrounding code either. However, it's helpful to explore it if possible, as it's the simplest of our fixes.

无论这是否是重构纯功能的实用技术，更重要的就是功能纯度只需要深入皮肤。 也就是说，**函数的纯度是从外部判断的**，不管内部是什么。只要一个函数的使用表现为纯的，它就是纯的。在纯函数的内部，可以适度的使用不纯的技术，由于各种原因，包括最常见的表现，为了性能。 正如他们所说的“海龟垂钓”。
Regardless of whether this will be a practical technique for refactoring to pure functions, the more important take-away is that function purity only need be skin deep. That is, the **purity of a function is judged from the outside**, regardless of what goes on inside. As long as a function's usage behaves pure, it is pure. Inside a pure function, impure techniques can be used -- in moderation! -- for a variety of reasons, including most commonly, for performance. It's not necessarily, as they say, "turtles all the way down".

不过要小心，程序的任何部分都是不纯的，即使它被包装并且只能通过纯函数使用，也是代码错误和困惑读者的潜在的根源。总体目标是尽可能减少副作用，而不仅仅是隐藏它们。
Be very careful, though. Any part of the program that's impure, even if it's wrapped with and only ever used via a pure function, is a potential source of bugs and confusion for readers of the code. The overall goal is to reduce side effects wherever possible, not just hide them.

### 掩盖效应
### Covering Up Effects

很多时候，你无法在容器函数的内部为了封装词法自由变量来修改代码。例如，不纯的函数可能位于一个你无法控制的第三方库文件中，其中包括:
Many times you will be unable to modify the code to encapsulate the lexical free variables inside the scope of a wrapper function. For example, the impure function may be in a third-party library file that you do not control, containing something like:

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


蛮力的策略是，在我们程序的其余部分使用此实用程序时*隔离*副作用的方法时创建一个接口函数，执行以下步骤：
The brute-force strategy to *quarantine* the side causes/effects when using this utility in the rest of our program is to create an interface function that performs the following steps:

1. 捕获受影响的当前状态
2. 设置初始输入状态
3. 运行不纯的函数
4. 捕获副作用状态
5. 恢复原来的状态
6. 返回捕获的副作用状态

1. capture the to-be-affected current states
2. set initial input states
3. run the impure function
4. capture the side effect states
5. restore the original states
6. return the captured side effect states

```js
function safer_generateMoreRandoms(count,initial) {
	// (1) save original state
	var orig = {
		nums,
		smallCount,
		largeCount
	};

	// (2) setup initial pre-side effects state
	nums = initial.nums.slice();
	smallCount = initial.smallCount;
	largeCount = initial.largeCount;

	// (3) beware impurity!
	generateMoreRandoms( count );

	// (4) capture side effect state
	var sides = {
		nums,
		smallCount,
		largeCount
	};

	// (5) restore original state
	nums = orig.nums;
	smallCount = orig.smallCount;
	largeCount = orig.largeCount;

	// (6) expose side effect state directly as output
	return sides;
}
```
并且使用`safer_generateMoreRandoms(..)`
And to use `safer_generateMoreRandoms(..)`:

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
That's a lot of manual work to avoid a few side causes/effects; it'd be a lot easier if we just didn't have them in the first place. But if we have no choice, this extra effort is well worth it to avoid surprises in our programs.

**注意：**这种技术只有在处理同步代码时才有用。异步代码不能可靠地使用这种方法实现，因为如果程序的其他部分在临时访问/修改状态变量，它就无法防止意外。
**Note:** This technique really only works when you're dealing with synchronous code. Asynchronous code can't reliably be managed with this approach because it can't prevent surprises if other parts of the program access/modify the state variables in the interim.

### 逃避影响
### Evading Effects

当要处理的副作用的本质是直接输入值(对象、数组等)的突变时，我们可以再次创建一个接口函数来与原始的不纯函数交互。
When the nature of the side effect to be dealt with is a mutation of a direct input value (object, array, etc) via reference, we can again create an interface function to interact with instead of the original impure function.

考虑一下：
Consider:

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

`userList`数组本身，加上其中的对象，都发生了突变。保护这些副作用的一种策略是先做一个深拷贝（而不是浅拷贝）:
Both the `userList` array itself, plus the objects in it, are mutated. One strategy to protect against these side effects is to do a deep (well, just not shallow) copy first:

```js
function safer_handleInactiveUsers(userList,dateCutoff) {
	// make a copy of both the list and its user objects
	let copiedUserList = userList.map( function mapper(user){
		// copy a `user` object
		return Object.assign( {}, user );
	} );

	// call the original function with the copy
	handleInactiveUsers( copiedUserList, dateCutoff );

	// expose the mutated list as a direct output
	return copiedUserList;
}
```

这个技术的成功将取决于你所做的*复制*的彻底性。使用`userList.slice()`在这里不起作用，因为这只会创建一个`userList`数组本身的浅拷贝。数组的每个元素都是一个需要复制的对象，所以我们需要特别小心。当然，如果这些对象在它们之内有对象（可能会这样），则复制需要更加强大。
The success of this technique will be dependent on the thoroughness of the *copy* you make of the value. Using `userList.slice()` would not work here, since that only creates a shallow copy of the `userList` array itself. Each element of the array is an object that needs to be copied, so we need to take extra care. Of course, if those objects have objects inside them (they might!), the copying needs to be even more robust.

#### 再看一下`this`
#### `this` Revisited

另一个参数变化的副作用是和`this`有关的，我们应该意识到`this`是函数隐式的输入。为什么`this`关键字对函数式编程者有更多信息，请参阅第2章中的"What's This"。
Another variation of the via-reference side cause/effect is with `this`-aware functions having `this` as an implicit input. See "What's This" in Chapter 2 for more info on why the `this` keyword is problematic for FPers.

思考一下：
Consider:

```js
var ids = {
	prefix: "_",
	generate() {
		return this.prefix + Math.random();
	}
};
```
我们的策略类似于上一节的讨论：创建一个接口函数，强制`generate（）`函数使用可预测的`this`上下文：
Our strategy is similar to the previous section's discussion: create an interface function that forces the `generate()` function to use a predictable `this` context:

```js
function safer_generate(context) {
	return ids.generate.call( context );
}

// *********************

safer_generate( { prefix: "foo" } );
// "foo0.8988802158307285"
```
这些策略绝对不是愚蠢的；对副作用的最安全的保护是不要执行它们。但是，如果您想提高程序的可读性和你对程序的信心，尽可能减少副作用/效果是巨大的进步。
These strategies are in no way fool-proof; the safest protection against side causes/effects is to not do them. But if you're trying to improve the readability and confidence level of your program, reducing the side causes/effects wherever possible is a huge step forward.


基本上，我们并没有真正消除副作用，而是包含和限制它们，以便我们的代码更加的可验证和可靠。如果我们后来遇到程序错误，我们知道我们的代码部分仍然使用副作用是最有可能的罪魁祸首。
Essentially, we're not really eliminating side causes/effects, but rather containing and limiting them, so that more of our code is verifiable and reliable. If we later run into program bugs, we know that the parts of our code still using side causes/effects are the most likely culprits.

## 总结
## Summary

副作用对代码的可读性和质量都有害，因为它们使您的代码难以理解。副作用也是程序中最常见的错误*原因*之一，因为很难欺骗他们。幂等一种策略，通过基本上创建一次性操作来限制副作用。
Side effects are harmful to code readability and quality because they make your code much harder to understand. Side effects are also one of the most common *causes* of bugs in programs, because juggling them is hard. Idempotence is a strategy for restricting side effects by essentially creating one-time-only operations.

纯功能是最好避免副作用。纯函数是给相同输入时总是返回相同输出，并且没有副作用或其他原因。参考透明度进一步指出--更多的是作为一种精神操练而不是字面行为 --纯函数的调用是可以用它的输出来代替，并且程序的行为不会被改变。
Pure functions are how we best avoid side effects. A pure function is one that always returns the same output given the same input, and has no side causes or side effects. Referential transparency further states that -- more as a mental exercise than a literal action -- a pure function's call could be replaced with its output and the program would not have altered behavior.

将一个不纯的函数重构为纯函数是首选。但是，如果不可能，尝试封装副作用，或者创建一个纯粹的接口来反对他们。
Refactoring an impure function to be pure is the preferred option. But if that's not possible, try encapsulating the side causes/effects, or creating a pure interface against them.

没有程序可以完全没有副作用。但是在尽可能多的地方更喜欢纯粹的功能。 尽可能地收集不纯的功能副作用，以便在出现错误时最容易识别和审核错误的凶手。
No program can be entirely free of side effects. But prefer pure functions in as many places as that's practical. Collect impure functions side effects together as much as possible, so that it's easier to identify and audit the most likely culprits of bugs when they arise.
