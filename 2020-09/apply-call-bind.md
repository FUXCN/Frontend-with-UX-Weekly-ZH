# 深入了解this call-apply-bind用法

在 JavaScript 中，当我们定义了一个函数之后，我们只需要在调用这个函数的时候为这个函数绑定不同的this,我们就可以把这个函数挂载到任意的一个对象上。我们可以这样做的原因是因为函数的 `this` 是在它被调用的时候决定的而不是它被定义的时候决定的。
看到这是不是还是不太能够理解？下面我们先从显示绑定和隐形绑定的区别开始来阐述函数中的 `this` 。

## 隐形绑定

隐形绑定就是我们日常编写 JavaScript 代码时经常使用的 `.` 语法来调用一个函数，使用.语法之后，函数的 `this` 就被隐形的绑定为 `.` 左边的对象（即调用这个函数的对象），下面我们来看一个简单的例子

![](https://miro.medium.com/max/1400/1*d22VJqe5yahmT8vigyU0ng.png)

在上面的例子中，我们定义了一个 `greeting` 函数，并且将这个函数作为值传递给了 `cat` 和 `dog` 这两个对象中的属性 `sayHelo` ,这样做之后，我们在调用 `cat.sayHello()` 和 `dog.sayHello()` 便可以得到不同的结果，可其实我们调用的都是同一个函数 `greeting()` 。这便又回到了我们刚才讨论的问题，我们会在调用这个函数的时候通过调用这个函数的对象来决定 `this` 的指向。如果我们没有为一个函数隐性的绑定一个对象，那么我们的 `greeting` 函数中的 `this` 还是作用域全局范围内的，即它绑定的对象为 `window`。

我们已经了解了隐性绑定是如何工作的，现在我们来看一下显示绑定。

## 使用 .call() 方法

在 JavaScript 中，任何对象都可以使用 `.call()` 方法，这个方法可以让我们显示的来绑定一个 `this` 到调用这个函数的对象。在我们学习 `call()` 函数之前，我们先回顾一下我们之前的代码然后做一些小的更改。

![](https://miro.medium.com/max/1400/1*ROcIo16w-eQCrubNRKvfnQ.png)

在上面的例子中，我们并没有把 `greeting` 作为值传递到 `cat` 对象中， 所以我们没有办法使用 . 语法来隐性的为这个函数绑定 `this` 。如果我们一定要这么做， 那么我们将会得到一个 `TyprError`，告诉我们 `cat.greeting()` 不是一个函数， 同时 `greeting()` 中的 `this.name` 也将会是 `undefined`。

为了解决这个问题，我们使用 `.call()` 方法来调用 `greeting()` 函数，并将 `cat` 对象作为上下文的参数进行传递，这样 JavaScript 就会知道 `this.name` 的值是 `Mrs. Pickles`。

![](https://miro.medium.com/max/1400/1*fTq8pXXJEXVa8A4RM2t3Yg.png)

`call()` 函数同样可以传递多个参数，能让我们可以构造更多补充信息。 但一定要记住一点，`call()` 的第一个参数一定要是绑定的 `this` 的上下文，其他的参数就直接传定义函数时所指定的参数。

![](https://miro.medium.com/max/2000/1*9b5EKP_YNeZHo3zNPBATTA.png)

尽管 `.call()` 确实提供给了我们很多有用的功能，但是我们需要手写没一个参数对我们来说是非常低效率且耗时的，这就是我们为什么需要 `.apply()`。

## Apply over .call()

与 `.call()` 相同，`apply()` 的第一个参数同样是你需要绑定的 `this` 的上下文，其他的参数就是这个函数需要的参数。但不同的是，我们不需要一个一个传递我们需要的参数，我们通常会传递一个数组作为参数，并得到同样的效果。

![](https://miro.medium.com/max/2000/1*-qven7gq8SWPUyW-clCPZw.png)

这些看上去已经够用了，可如果我们的需求是我们不需要立即执行这个函数，如果我们的函数需要过一会在执行呢？

## 使用便捷的 .bind()

如何你想使用一个拥有 `.call()` 的所有功能，同时有可以灵活扩展的函数，答案就是 `bind()` .

> `bind()` 函数创建一个新的 **绑定函数** (bound function, BF)。绑定函数是一个 exotic function object (怪异函数对象， ESMAScript 2015 中的术语)，它包装了原函数对象。调用**绑定函数**通常会导致执行包装函数
> —— MDN web 文档 

当 `.call()` 函数被调用的时候，它会创建一个稍后执行的绑定函数，并且不会丢失你所传递的 `this` 上下文。举例来说，我们可以创建一个新的函数 `newFunction` ，并且我们只在 `activities` 这个数组的长度大于 2 的时候才调用这个新的函数。目前我们数组里有三个元素，如果我们移除掉一个元素，便会执行 `else` 中的语句。

![](https://miro.medium.com/max/2000/1*NbBiKUb0CwYtVPsVuQ2Cgg.png)

## 结论

了解 JavaScript 中的显示和隐形绑定将会帮助您避免在处理对象和 `this` 时常见的一些陷阱。 一些函数，比如上面提到的这三个，可以帮助您在编码的时候绑定到你想要的 `this` ， 为了获得更多的提示的话，请使用严格模式，该模式将会标记所有可能潜在性的 `this` 问题，并且帮助您正确的使用 `this` 。

> 原文链接： https://medium.com/the-innovation/what-is-this-using-call-apply-and-bind-in-javascript-ac8c920985f8