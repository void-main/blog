---
layout: post
title: "函数式编程随想（一）——函数式编程的好处及柯里化的意义"
date: 2016-02-27 15:17:43 +0800
comments: true
categories: functional programming
---

这篇博客主要是为了记录我最近关于函数式编程的好处，以及柯里化的作用的一些思考。

首先来举个简单的例子，如果我们有一组数，比如`[1,2,3,4,5,6]`，我们想取其中的奇数，然后把这些奇数取平方，并返回这个数组，即，我们期望的结果是`[1,9,25]`，那么应该怎么写呢？

最简单的写法就是来一次循环：

``` javascript
list = [1, 2, 3, 4, 5, 6];
result = [];
for(idx = 0; idx < list.length; idx ++) {
    if (list[idx] % 2 == 1) {
        result = result.concat(list[idx] * list[idx]);
    }
}

console.log(result);
// [1, 9, 25]
```

真的，我劝你不要这么写。

> “为什么不？课本上都是这么写的啊！”

这么写确实正确，而且符合需求，但是看看我们做了多少额外的工作！我们不仅需要描述具体的业务逻辑(`list[idx] % 2 == 1`和`list[idx] * list[idx]`)，还要操心如何遍历每个元素(`for`循环)、如何取出当前要处理的值(维护`idx`变量，以及用`list[idx]`来取值)，以及如何保存结果。

对于这么一个简单的任务我们几乎在基础工作上用了一半以上的代码，代价太高了。

有没有简单的写法呢？有啊！

``` javascript
// JS code
list = [1, 2, 3, 4, 5, 6];
result = list.filter((x) => x % 2).map((x) => x * x);
console.log(result);
// [1, 9, 25]
```

仔细观察这种写法。首先，这种写法只描述了要做什么，分别是过滤能被2整除的数和对每个数取平方。第二点就是我们不关心具体的实现，而是通过**函数调用**的方式来解决问题。

这就是函数式编程，全文完。

---

> “别开玩笑了，函数调用的方式是什么鬼？”

注意看，这里我们没有描述如何遍历这个数组，我们只是调用了`filter`和`map`两个函数。所以我们没有关心如何一步步实现循环，记录结果，我们知道`filter`和`map`函数可以帮我们便利数组，同时对每个元素给我们执行一段代码的机会。

有趣的是这两个函数的参数。以`map`函数为例，这个函数的第一个参数是一个接受3个参数的函数。没错，这种函数就是所谓的**高阶函数**。

高阶函数可以把函数当作参数，或者可以返回函数作为返回值。

正因为有了这种能力，我们才能把逻辑从循环的具体实现中**抽象**出来，才能使像`filter`和`map`这样的函数给调用方执行代码的机会。

> “但是你说有三个参数？”

没错，filter函数在调用它的参数的时候会传递3个参数，但是在我们调用它的时候，却只接受了一个参数，剩下的两个参数被丢掉了。

> “调用时传递过多的参数会被丢掉，那么参数如果不够呢？”

在JS里面会自动传`undefined`作为参数，这一点稍后会用到。

现在让我们开始时的例子。现在，如果我们要给`[7,8,9,10,11,12,13]`这个数组也执行同样的操作，为了避免复制、粘贴代码，我们可以通过**函数**进行**抽象**。做法也很简单，把之前代码里面的`list`变量作为参数传入一个函数，然后两次调用这个函数。就像这样：

``` javascript
function filterAndMap(data) {
    return data.filter((x) => x % 2).map((x) => x * x);
}
console.log(filterAndMap([1, 2, 3, 4, 5, 6]));
console.log(filterAndMap([7,8,9,10,11,12,13]));
```

但是这里的业务太具体了，如果后面我们想对所有的偶数做取平方的操作，或者对所有的奇数取立方操作，那这个代码显然不能满足。

> “等等，不是可以把函数当作参数？”

对！所以我们可以改一下`filterAndMap`方法，把具体的逻辑函数当作参数。同时考虑到我们会多次调用`filterAndMap`方法，为了避免每次调用都定义一次逻辑函数，所以顺便也提前定义这两个函数，分别是判断奇偶以及取平方：

``` javascript
function odd(x) { return x % 2; }
function sqr(x) { return x * x; }

function filterAndMap(data, callback1, callback2) {
    return data.filter(callback1).map(callback2);
}
console.log(filterAndMap([1, 2, 3, 4, 5, 6], odd, sqr));
console.log(filterAndMap([7,8,9,10,11,12,13], odd, sqr));
```

但是，这也太挫了。我怎么知道第二个参数传`odd`而不是`sqr`？另外，我必须每次在调用的时候传递逻辑代码，而不是一次创建好，在更新数据的时候还必须附加上额外的参数。

显然我们还有更好的做法。上面这种写法的主要问题是我们不能**提前配置**过滤和映射函数。至于具体原因，让我们再来观察下`filter`跟`map`这两个函数：

``` javascript
array.filter(callback)
array.map(callback)
```

这两个函数其实是`Array.prototype`定义的属性，所以我们的数组都可以调用。但是在调用的时候，前提是数组已经存在了，所以必须得先有数组，然后每个数组传一次参数。

不过这个是可以解决的，利用**延迟执行**。让我们先定义两个对应的函数，不过这一次，把`array`改成函数的参数。

``` javascript
function filter(callback, array) { return array.filter(callback); }
function map(callback, array) { return array.map(callback); }
```

可以看到，这两个函数跟原来版本的区别是，我们把原来的数组当作了参数。

**特别注意：**这里的参数顺序是有讲究的，推荐把回调函数放倒前面，把数据放倒后面，具体原因后面会讨论。

但是这没有解决问题，因为我们在调用的时候仍然需要提供这两个参数。别着急，如果我们让`filter`函数变成一个接受`callback`参数，同时返回一个接受`array`参数的函数，那情况就不一样了：

``` javascript
function filter(callback) {
    return function (array) {
        return array.filter(callback);
    }
}

function map(callback) {
    return function (array) {
        return array.map(callback);
    }
}
```

这里会涉及到一些关于自由变量、闭包的讨论。我们暂且不提，可以先记住一个原则，就是我们用callback参数调用`filter`后返回的那个函数的函数体中的`callback`就是之前传入的那个参数。

好，现在让我们看看怎么用：

``` javascript
var oddFilter = filter(odd);
var sqrMap = map(sqr);
```

这里的`oddFilter`和`sqrMap`的类型签名都是`[Array]->[Array]`，就是接受一个数组作为参数，同时返回一个数组，所以我们可以把这两个函数串联起来：

``` javascript
function filterAndMap(array) {
    return sqrMap(oddFilter(array));
}
console.log(filterAndMap([1, 2, 3, 4, 5, 6]));
console.log(filterAndMap([7,8,9,10,11,12,13]));
```

看着不错，一旦我们构建好了大杀器`filterAndMap`方法，后续就只需要关心数据的变化即可。

那我们对`filter`和`map`函数做了什么呢？简单的说，我们实现了可以**部分配置**的`filter`和`map`函数。我们可以提前配置好过滤和映射要做的操作，这样就可以在后续不断的更换数据，从而减少代码的复杂程度。同时，创建`oddFilter`和`sqrMap`这两个对象的过程叫做**部分调用(partial application)**。

其实将`filter`这种接受两个参数的函数变成多个没次只接受一个参数的函数的过程就叫柯里化(curry)。单独看柯里化可能觉得并没有什么用，但是如果你想把多个函数进行像刚刚做的这种串联，那就非常关键了。关于柯里化的实现和更多应用后面会讨论。

> “完美了么？”

还没有！这种做法至少还有两个问题，第一个问题，我们的filterAndMap方法，如果我们要串联更多的操作，必须每次都去改这个方法，而且随着嵌套层次的增多，调整会越来越复杂。另外一个问题是，为了实现这个效果，我们的`filter`和`map`代码长了好几倍，显然我们不想每次都手撸一遍。

遇到这种问题，我们就得再请**高阶函数**出马做**抽象**啦。

先来说第一个问题，我期待的解决方案是这样的：

``` javascript
var filterAndMap = pipe(oddFilter, sqrMap);
console.log(filterAndMap([1, 2, 3, 4, 5, 6]));
console.log(filterAndMap([7,8,9,10,11,12,13]));
```

这里引入了一个`pipe`函数，我们马上就要实现它。先看看这种写法的好处是什么。首先，它对调用者来说是扁平的，调用者只需要传入符合要求的函数（函数只接受一个参数，且返回值与参数的类型一致），就可以串联多次，而且要调整顺序也十分简单，只需要改变传入`pipe`函数的参数的顺序即可，无需处理那一层层的括号。

其实实现`pipe`函数也很简单，但是需要两点基础知识。

1. 在JavaScript中，每个函数的作用域中都有一个名为arguments的变量，这个变量是实际传入函数的参数对象。这个对象是所谓的*类数组对象*。这个意思是说这个对象有`length`属性，而且虽然是个对象，但是对象中的key是从0开始的下标。在ES5中可以用`Array.prototype.slice`函数转换成数组。这样我们就有了实际传入的参数数组。这个数组的元素与函数定义的参数的数量无关。
2. 对于给定的函数我们可以通过`call`和`apply`方法执行。这两者的第一个参数都是this指针，而区别是，`apply`方法的第二个参数是一个数组，而`call`方法从第二个参数开始是一个变长参数列表。

考虑`pipe`函数做的事情：`pipe`函数的参数是一组函数，这些函数接受一个参数，并返回一个特定的类型。我们只需要对每个参数使用`call`方法，传入上一个函数执行的结果即可。

``` javascript
function toArr(args) {
    return Array.prototype.slice.call(args, 0);
}

function pipe () {
    var args = toArr(arguments);
    return function () {
        var innerArgs = toArr(arguments);
        result = void 0; // void 0 => undefined
        args.forEach(function (fn, idx) {
            if (idx == 0) {
                result = fn.apply(this, innerArgs);
            } else {
                result = fn.call(this, result);
            }
        });

        return result;
    }
}
```

然后我们再来考虑第二个问题，我们不想也不能没次都手撸一次柯里化之后的`filter`函数。如果一个函数有10个参数，那手撸一次估计就废了。在JS中其实也很好实现，不过还是需要补充一个知识点：

1. 如果一个对象是函数类型，那么它的`length`方法返回的是函数定义的参数列表的长度。

也就是说，如果要实现`curry`函数，需要每次接受一部分参数（不一定是一个），只有在总参数的长度大于等于定义的参数长度时，才真正的调用函数，否则就返回一个保存了已经传入参数的函数。实现如下：

``` javascript
function curry(fn) {
    var definedArgsCount = fn.length;

    function helper(args) {
        return function () {
            var newArgs = toArr(arguments);
            var wholeArgs = args.concat(newArgs);

            if (wholeArgs.length >= definedArgsCount) {
                return fn.apply(this, wholeArgs);
            } else {
                return helper(wholeArgs);
            }
        }
    }

    return helper([]);
}
```

到这里关于函数式编程的好处、柯里化的意义的讨论我想说的就都说完了。

> “那为什么要把callback放前面？”

这里又个原则是把容易变化的往后放。加入了柯里化之后，我们可以实现函数的部分配置。那么就应该尽可能把可以配置的参数放倒前面。当然这也与JS的bind机制有关，具体我就不讨论了。而这么做的好处就是，我们可以像前面的例子这样，简单的把多个函数串联起来。

> “然而我用PHP！”

没关系，我简单实现了4个有用的函数，并且[开源](https://github.com/void-main/fn-php)啦。

> “性能啊！”

实际上我认为如果不进行像网络和数据库读写这种耗时操作，比起这种写法带来的好处，性能的损耗可以接受。如果真的出现了性能问题，再进行特定的优化也不晚。

##参考资料
1. [http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-intro/](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-intro/)
2. [https://hughfdjackson.com/javascript/why-curry-helps/](https://hughfdjackson.com/javascript/why-curry-helps/)
