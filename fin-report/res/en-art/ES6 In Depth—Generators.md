我的深入理解 ES6 系列文章简要讨论了 JavaScript 语言第六版————ESMAScript 6(缩写为 ES6)的新特性。

今天写的文章让我很兴奋，因为我们将去讨论 ES6 最神奇的一些特性。

我说的神奇是什么意思？对于初学者，这些特性和以往的 JS 有很大的不同，你第一次接触会感到很难理解。在某种程度上，这些特性完全颠覆了语言的行为！

不仅仅是这些，这些强大的特性可以简化代码、让你看清可怕的“回调函数地狱”的界限。todo: 改进翻译

我的溢美之词是不是太多了？那我们开始吧。

# ES6 Generators
Generator 是什么？

先来个例子吧。

```
function* quips(name) {
  yield "hello " + name + "!";
  yield "i hope you are enjoying the blog posts";
  if (name.startsWith("X")) {
    yield "it's cool how your name starts with X, " + name;
  }
  yield "see you later!";
}
```

这是一个“[会说话的猫](http://people.mozilla.org/~jorendorff/demos/meow.html)”小例子的一些代码，这可能是今天互联网上最重要的一种应用。(你可以点击[这个链接](http://people.mozilla.org/~jorendorff/demos/meow.html)，玩一玩这只猫。当你困惑的时候，再回到这篇文章寻找解释。)

看起来像函数，是吗？这个叫做生成器函数(generator-function), 和普通的函数有很多共同点。但你很快就能看出差别：
- 普通的函数以关键字 <kbd>function</kbd> 开头，而生成器函数(generator-function)以 <kbd>function*</kbd> 开头。
- 在 generator-function 中，<kbd>yield</kbd> 是关键字，语法上就像 <kbd>return</kbd>。不同点是虽然一个函数只能返回一次，但 generator-function 可以 yield 很多次。<kbd>yield</kbd> 关键字暂停了 generator 的执行，之后还可以继续往下执行。

就是这样，这就是普通的函数和 generator-function 最大的不同。普通的函数不能暂停，而 generator-function 就可以。

# generators 是怎么执行的
当你调用 quips() 这个 generator-function 时候发生了什么？

```
> var iter = quips("jorendorff");
  [object Generator]
> iter.next()
  { value: "hello jorendorff!", done: false }
> iter.next()
  { value: "i hope you are enjoying the blog posts", done: false }
> iter.next()
  { value: "see you later!", done: false }
> iter.next()
  { value: undefined, done: true }
```

你可能对普通的函数比较熟悉，知道他们怎么执行的。当你调用他们的时候，他们就立即执行，一直到他们返回或抛出异常。

调用一个 generator 形式上看起来一样：<kbd>quips("jorendorrff")</kbd>。但当你调用 generator 的时候，他不会立即开始执行。相反，他返回一个 Generator 对象(上个例子中叫做 <kbd>iter</kbd>)。你可以把这个 Generator 对象看成是函数调用，在 generator-function 执行第一段代码之前就会停住。

之后你每次调用 Generator 对象的 <kbd>.next()</kbd> 方法，函数就会往下执行直到遇到下个 <kbd>yield</kbd>。

这就是为什么每次我们每次调用上面的 <kbd>iter.next()</kbd>, 我们就会得到一个不同的字符串值。这些值是由 <kbd>yield</kbd> 产生的。

最后一次 <kbd>iter.next()</kbd> 调用，终于执行到 generator-function 的最后，打印的对象的 <kbd>done</kbd> 字段由 <kbd>false</kbd> 变为 <kbd>true</kbd>, 而此时 <kbd>.value</kbd> 字段变为 <kbd>undefined</kbd>。

现在是时候回到我们的“说话的猫”这个演示页面。如果把 <kbd>yield</kbd> 放进 loop 会怎样？

在技术术语上，每次 generator yields, 他的栈结构（局部变量，参数，临时值，当前执行到的位置）会从栈中移出。但 Generator 对象保留着栈的引用，这样之后的 <kbd>.next()</kbd> 调用就可以重新继续执行。

值得注意的是 Generators 不是线程。有线程的语言，多块代码可以同时运行，这样会导致资源竞争，不确定性以及性能问题。Generators 和这个不同。generator 执行时，执行是有确定的顺序的，不会并发。不像系统进程，generator 只会在 <kbd>yield</kbd>
标记的地方暂停。

我们现在知道了 generators 是什么了。我们知道 generator 执行的时候，暂停自己，然后再继续执行。那么问题又来了，这种奇怪的特性有什么呢？

# Generators 是迭代器(iterators)
上周，我们看到迭代器不仅仅是是个内置的类。是可以扩展的。你可以创建自己的迭代器，只需实现两个方法：: <kbd>[Symbol.iterator]()</kbd> and <kbd>.next()</kbd>。

但去实现一个接口要花点时间。我们来看看一个 iterator 是如何实现的。先来看个例子，我们来写一个简单的 <kbd>range</kbd> iterator ，作用是从一个数字一直计数到另一个数字，类似于 <kbd>for(;;)</kbd> 循环。

```
// This should "ding" three times
for (var value of range(0, 3)) {
  alert("Ding! at floor #" + value);
}
```

实现的一种方法是使用 ES6 的 class。（如果对 class 语法不是很清楚，别担心，我会在后面的文章讨论）。

```
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }

  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    } else {
      return {done: true, value: undefined};
    }
  }
}

// Return a new iterator that counts up from 'start' to 'stop'.
function range(start, stop) {
  return new RangeIterator(start, stop);
}
```

这就是 iterator 的一种实现，类似于 Java 和 Swift 语言。这段代码有 bugs 吗? 不好说。看起来和 for(;;) 循环不像。iterator 的这种特性迫使我们拆开循环。

现在你可能对 iterators 感到有点不爽了是吧。iterator 挺好用，但不好实现。

在 JS 语言中引入一个新的令人费解的控制结构让 iterator 更容易实现，这种做法我们或许想不到。但既然我们有 generators, 我们可以使用它吗？试试看吧：

```
function* range(start, stop) {
  for (var i = start; i < stop; i++)
    yield i;
}
```

上面四行代码就是对上面的 23 行代码的一个简单的替代包括整个 RangeIterator class. 这可能是由于 generators 是 iterator 的缘故。所有的 generators 都有一个内置的 <kbd>.next()</kbd> 和 <kbd>[Symbol.iterator]</kbd> 。

不用 generators 去实现 iterators 就像被迫全部用被动语态写一篇长的 email。RangeIterator 这个类的代码又长又奇怪，因为他必须描述一个循环而且还不能使用循环语法。Generators 就是很好的一个替代品。

我们还可以怎样用 generator 的特性去实现 iterator ?

- 让任意的对象可迭代(iterable). 你就写一个 genertor-function 遍历 this 对象，每次用 yield 产生对应的值。再把 generator-function 安装为 [Symbol.iterator] 这个对象的方法。
- 简化构建数组的函数。假设你有个函数，每次被调用时返回一个数组, 就像下面这个：

```
// Divide the one-dimensional array 'icons'
// into arrays of length 'rowLength'.
function splitIntoRows(icons, rowLength) {
  var rows = [];
  for (var i = 0; i < icons.length; i += rowLength) {
    rows.push(icons.slice(i, i + rowLength));
  }
  return rows;
}
```

 Generators 可以让这种代码更简洁：

 ```
 function* splitIntoRows(icons, rowLength) {
   for (var i = 0; i < icons.length; i += rowLength) {
     yield icons.slice(i, i + rowLength);
   }
 }
 ```

这种方式与上面的方式的唯一不同就是，这种方法不会一次性计算所有的结果后返回一个数组，而是返回一个 iterator, 根据需要一个一个去计算。

- 返回不寻常(数组)大小的结果。你不能创建一个无穷的数组。但你可以返回一个 generator 去产生一个无穷的序列，根据你的需要去产生任意多个数据。
- 重构复杂的循环。你曾写过又庞大有丑陋的函数吗？你想不想把他分成两个简单的部分？Generators 就是为数不多的工具之一，能帮你重构代码。当你去写一个复杂的循环的时候，你可以把产生数据的部分单独用 generator 去重构。然后把你的循环改成 <kbd>for (var data of myNewGenerator(args)) </kbd>.
- 作为处理 iterables 的工具。ES 6 没有专门提供一些库，为用来对任意可迭代数据集进行过滤、映射、或 hack，但 generators 仅用几行代码就能实现。

假设你需要一个类似于 <kbd>Array.prototype.filter</kbd> 的方法，用来对一些 DOM 节点进行操作，而不是数组。很简单，就像下面这样：

```
function* filter(test, iterable) {
  for (var item of iterable) {
    if (test(item))
      yield item;
  }
}
```

generators 是不是很有用？当然，这是一种很简单的方法，能够实现自定义的迭代器，而迭代器是 ES6 中的新标准。

但 generator 不仅仅能实现上面这些功能，而且这还不是 generator 能做的最重要的事。

# generators 和异步代码
我曾经写过这样的代码：
```
          };
        })
      });
    });
  });
});
```

你曾经也可能写过类似的代码。异步 API 返回一个回调函数，这就意味着你每次都要写一个额外的匿名函数。如果你要用代码做三件事，你不是去三行代码，而是像上面那样层层缩进。

下面是我写的另外一些代码：
```
}).on('close', function () {
  done(undefined, undefined);
}).on('error', function (error) {
  done(error);
});
```

异步 API 通常都有错误处理机制，但没有异常处理机制。不同的 API 有不同的惯例。多数情况下，错误就被忽略了。甚至成功的结束了，但还是会被忽略。

这些问题就是异步编程的代价。我们不得不接受这些比同步代码难看多了的异步代码。

Genertors 给我们了一些希望。[Q.sync()](https://github.com/kriskowal/q/tree/v1/examples/async-generators) 是一个例子，用 generator 和 promise 去让代码写起了更像同步的代码。

```
// Synchronous code to make some noise.
function makeNoise() {
  shake();
  rattle();
  roll();
}

// Asynchronous code to make some noise.
// Returns a Promise object that becomes resolved
// when we're done making noise.
function makeNoise_async() {
  return Q.async(function* () {
    yield shake_async();
    yield rattle_async();
    yield roll_async();
  });
}
```

主要的不同就是异步的写法必须在每个调用异步函数的地方加上 <kbd>yield</kbd> 。

所以 generators 让我们以更习惯的方式去进行异步编程。这项工作会继续进行下去，因为 ES7 正在已经在为这个努力了，灵感来自 C#.

# 我什么时候可以使用 generator
在服务器端，如果你采用了 io.js 框架，你可以使用 ES6 的 generator 。或者如果你使用 nodejs 的话，使用时在命令行加上 --harmony 选项。

浏览器中，目前只有 Firefox 27+ 和 Chrome 39+ 支持 ES6 的 generator。其他版本的浏览器你可能需要 Babel 或 Traceur 把你的 ES6 代码转换成 ES5。

Generators 是由 Brendan Eich 第一p次引入的；他的设计灵感来自于 Python generators , 而 Python generators 是受 Icon 启发的。标准化的道路是曲折的，语法和行为都在变化。ES6 的 generator 是由 Andy Wingo 集成到 Firefox 和 Chrome 中的，Bloomberg 赞助了他。

