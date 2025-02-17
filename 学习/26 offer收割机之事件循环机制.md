### JS为什么是单线程的?

最初设计JS是用来在浏览器验证表单操控DOM元素的是一门脚本语言，如果js是多线程的那么两个线程同时对一个DOM元素进行了相互冲突的操作，那么浏览器的解析器是无法执行的。

### JS为什么需要异步?

如果JS中不存在异步，只能自上而下执行，如果上一行解析时间很长，那么下面的代码就会被阻塞。对于用户而言，阻塞就意味着"卡死"，这样就导致了很差的用户体验。比如在进行ajax请求的时候如果没有返回数据后面的代码就没办法执行。

### JS单线程又是如何实现异步的呢?

js中的异步以及多线程都可以理解成为一种“假象”，就拿h5的WebWorker来说，子线程有诸多限制，不能控制DOM元素、不能修改全局对象 等等，通常只用来做计算做数据处理。这些限制并没有违背我们之前的观点，所以说是“假象”。JS异步的执行机制其实就是事件循环(eventloop)，理解了eventloop机制，就理解了JS异步的执行机制。

### JS的事件循环(eventloop)是怎么运作的？

事件循环、eventloop、运行机制 这三个术语其实说的是同一个东西，在写这篇文章之前我一直以为事件循环简单的很，就是先执行同步操作，然后把异步操作排在事件队列里，等同步操作都运行完了（运行栈空闲），按顺序运行事件队列里的内容。但是远不止这么肤浅，我们接下来一步一步的深入来了解。

“先执行同步操作异步操作排在事件队列里”这样的理解其实也没有任何问题但如果深入的话会引出来很多其他概念，比如event table和event queue，我们来看运行过程：

1.  首先判断JS是同步还是异步，同步就进入主线程运行，异步就进入event table。
2.  异步任务在event table中注册事件，当满足触发条件后（触发条件可能是延时也可能是ajax回调），被推入event queue。
3.  同步任务进入主线程后一直执行，直到主线程空闲时，才会去event queue中查看是否有可执行的异步任务，如果有就推入主线程中。

```javascript
setTimeout(() => {
  console.log('2秒到了')
}, 2000)
```

我们用上面的第二条来分析一下这段脚本，setTimeout是异步操作首先进入event table，注册的事件就是他的回调，触发条件就是2秒之后，当满足条件回调被推入event queue，当主线程空闲时会去event queue里查看是否有可执行的任务。

```kotlin
console.log(1) // 同步任务进入主线程
setTimeout(fun(),0)   // 异步任务，被放入event table， 0秒之后被推入event queue里
console.log(3) // 同步任务进入主线程
```

1、3是同步任务马上会被执行，执行完成之后主线程空闲去event queue(事件队列)里查看是否有任务在等待执行，这就是为什么setTimeout的延迟时间是0毫秒却在最后执行的原因。

关于setTimeout有一点要注意延时的时间有时候并不是那么准确。

```scss
setTimeout(() => {
  console.log('2秒到了')
}, 2000)
sleep(9999999999)
```

分析运行过程：

1.  console进入Event Table并注册，计时开始。
2.  执行sleep函数，sleep方法虽然是同步任务但sleep方法进行了大量的逻辑运算，耗时超过了2秒。
3.  2秒到了，计时事件timeout完成，console进入Event Queue，但是sleep还没执行完，主线程还被占用，只能等着。
4.  sleep终于执行完了，console终于从Event Queue进入了主线程执行，这个时候已经远远超过了2秒。

其实延迟2秒只是表示2秒后，setTimeout里的函数被会推入event queue，而event queue(事件队列)里的任务，只有在主线程空闲时才会执行。上述的流程走完，我们知道setTimeout这个函数，是经过指定时间后，把要执行的任务(本例中为console)加入到Event Queue中，又因为是单线程任务要一个一个执行，如果前面的任务需要的时间太久，那么只能等着，导致真正的延迟时间远远大于2秒。 我们还经常遇到setTimeout(fn，0)这样的代码，它的含义是，指定某个任务在主线程最早的空闲时间执行，意思就是不用再等多少秒了，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行。但是即便主线程为空，0毫秒实际上也是达不到的。根据HTML的标准，最低是4毫秒。

关于setInterval： 以setInterval(fn，ms)为例，setInterval是循环执行的，setInterval会每隔指定的时间将注册的函数置入Event Queue，不是每过ms秒会执行一次fn，而是每过ms秒，会有fn进入Event Queue。需要注意的一点是，一旦setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了。

上面的概念很基础也很容易理解但不幸的消息是上面讲的一切都不是绝对的正确，因为涉及到Promise、async/await、process.nextTick(node)所以要对任务有更精细的定义：

宏任务(macro-task)：包括整体代码script、setTimeout、setInterval、MessageChannel、postMessage、setImmediate。  
微任务(micro-task)：Promise、process.nextTick、MutationObsever。

在划分宏任务、微任务的时候并没有提到async/await因为async/await的本质就是Promise。

事件循环机制到底是怎么样的？ *不同类型的任务会进入对应的Event Queue，比如setTimeout和setInterval会进入相同(宏任务)的Event Queue。而Promise和process.nextTick会进入相同(微任务)的Event Queue。*

1.  「宏任务」、「微任务」都是队列，一段代码执行时，会先执行宏任务中的同步代码。
2.  进行第一轮事件循环的时候会把全部的js脚本当成一个宏任务来运行。
3.  如果执行中遇到setTimeout之类宏任务，那么就把这个setTimeout内部的函数推入「宏任务的队列」中，下一轮宏任务执行时调用。
4.  如果执行中遇到 promise.then() 之类的微任务，就会推入到「当前宏任务的微任务队列」中，在本轮宏任务的同步代码都执行完成后，依次执行所有的微任务。
5.  第一轮事件循环中当执行完全部的同步脚本以及微任务队列中的事件，这一轮事件循环就结束了，开始第二轮事件循环。
6.  第二轮事件循环同理先执行同步脚本，遇到其他宏任务代码块继续追加到「宏任务的队列」中，遇到微任务，就会推入到「当前宏任务的微任务队列」中，在本轮宏任务的同步代码执行都完成后，依次执行当前所有的微任务。
7.  开始第三轮，循环往复...

下面用代码来深入理解上面的机制：

```javascript
setTimeout(function() {
    console.log('4')
})

new Promise(function(resolve) {
    console.log('1') // 同步任务
    resolve()
}).then(function() {
    console.log('3')
})
console.log('2')
```

1.  这段代码作为宏任务，进入主线程。
2.  先遇到setTimeout，那么将其回调函数注册后分发到宏任务Event Queue。
3.  接下来遇到了Promise，new Promise立即执行，then函数分发到微任务Event Queue。
4.  遇到console.log()，立即执行。
5.  整体代码script作为第一个宏任务执行结束。查看当前有没有可执行的微任务，执行then的回调。

第一轮事件循环结束了，我们开始第二轮循环。

6.  从宏任务Event Queue开始。我们发现了宏任务Event Queue中setTimeout对应的回调函数，立即执行。 执行结果：`1 - 2 - 3 - 4`

```javascript
console.log('1')
setTimeout(function() {
    console.log('2')
    process.nextTick(function() {
        console.log('3')
    })
    new Promise(function(resolve) {
        console.log('4')
        resolve()
    }).then(function() {
        console.log('5')
    })
})

process.nextTick(function() {
    console.log('6')
})

new Promise(function(resolve) {
    console.log('7')
    resolve()
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9')
    process.nextTick(function() {
        console.log('10')
    })
    new Promise(function(resolve) {
        console.log('11')
        resolve()
    }).then(function() {
        console.log('12')
    })
})
```

1.  整体script作为第一个宏任务进入主线程，遇到console.log(1)输出1。
2.  遇到setTimeout，其回调函数被分发到宏任务Event Queue中。我们暂且记为setTimeout1。
3.  遇到process.nextTick()，其回调函数被分发到微任务Event Queue中。我们记为process1。
4.  遇到Promise，new Promise直接执行，输出7。then被分发到微任务Event Queue中。我们记为then1。
5.  又遇到了setTimeout，其回调函数被分发到宏任务Event Queue中，我们记为setTimeout2。
6.  现在开始执行微任务，我们发现了process1和then1两个微任务，执行process1,输出6。执行then1，输出8。 第一轮事件循环正式结束，这一轮的结果是输出1，7，6，8。那么第二轮事件循环从setTimeout1宏任务开始：
7.  首先输出2。接下来遇到了process.nextTick()，同样将其分发到微任务Event Queue中，记为process2。
8.  new Promise立即执行输出4，then也分发到微任务Event Queue中，记为then2。
9.  现在开始执行微任务，我们发现有process2和then2两个微任务可以执行输出3，5。 第二轮事件循环结束，第二轮输出2，4，3，5。第三轮事件循环从setTimeout2宏任务开始：
10.  直接输出9，将process.nextTick()分发到微任务Event Queue中。记为process3。
11.  直接执行new Promise，输出11。将then分发到微任务Event Queue中，记为then3。
12.  执行两个微任务process3和then3。输出10。输出12。 第三轮事件循环结束，第三轮输出9，11，10，12。 整段代码，共进行了三次事件循环，完整的输出为1，7，6，8，2，4，3，5，9，11，10，12。 (请注意，node环境下的事件监听依赖libuv与前端环境不完全相同，输出顺序可能会有误差)

```javascript
new Promise(function (resolve) { 
    console.log('1')// 宏任务一
    resolve()
}).then(function () {
    console.log('3') // 宏任务一的微任务
})
setTimeout(function () { // 宏任务二
    console.log('4')
    setTimeout(function () { // 宏任务五
        console.log('7')
        new Promise(function (resolve) {
            console.log('8')
            resolve()
        }).then(function () {
            console.log('10')
            setTimeout(function () {  // 宏任务七
                console.log('12')
            })
        })
        console.log('9')
    })
})
setTimeout(function () { // 宏任务三
    console.log('5')
})
setTimeout(function () {  // 宏任务四
    console.log('6')
    setTimeout(function () { // 宏任务六
        console.log('11')
    })
})
console.log('2') // 宏任务一
```

1.  全部的代码作为第一个宏任务进入主线程执行。
2.  首先输出1，是同步代码。then回调作为微任务进入到宏任务一的微任务队列。
3.  下面最外层的三个setTimeout分别是宏任务二、宏任务三、宏任务四按序排入宏任务队列。
4.  输出2，现在宏任务一的同步代码都执行完成了接下来执行宏任务一的微任务输出3。

第一轮事件循环完成了

5.  现在执行宏任务二输出4，后面的setTimeout作为宏任务五排入宏任务队列。

第二轮事件循环完成了

6.  执行宏任务三输出5，执行宏任务四输出6，宏任务四里面的setTimeout作为宏任务六。
7.  执行宏任务五输出7，8。then回调作为宏任务五的微任务排入宏任务五的微任务队列。
8.  输出同步代码9，宏任务五的同步代码执行完了，现在执行宏任务五的微任务。
9.  输出10，后面的setTimeout作为宏任务七排入宏任务的队列。

宏任务五执行完成了，当前已经是第五轮事件循环了。

10.  执行宏任务六输出11，执行宏任务七输出12。

结果：1、2、3、4、5、6、7、8、9、10、11、12

**\-^-*，这个案例是有点恶心，目的是搞明白各宏任务之间执行的顺序以及宏任务和微任务的执行关系。*

**初步总结：宏任务是一个队列按先入先执行的原则，微任务也是一个队列也是先入先执行。 但是每个宏任务都对应会有一个微任务队列，宏任务在执行过程中会先执行同步代码再执行微任务队列。**

上面的案例只是用setTimeout和Promise模拟了一些场景来帮助理解，并没有用到async/await下面我们从什么是async/await开始讲起。

### async/await是什么？

我们创建了 promise 但不能同步等待它执行完成。我们只能通过 then 传一个回调函数这样很容易再次陷入 promise 的回调地狱。实际上，async/await 在底层转换成了 promise 和 then 回调函数。也就是说，这是 promise 的语法糖。每次我们使用 await, 解释器都创建一个 promise 对象，然后把剩下的 async 函数中的操作放到 then 回调函数中。async/await 的实现，离不开 Promise。从字面意思来理解，async 是“异步”的简写，而 await 是 async wait 的简写可以认为是等待异步方法执行完成。

### async/await用来干什么？

用来优化 promise 的回调问题，被称作是异步的终极解决方案。

### async/await内部做了什么？

async 函数会返回一个 Promise 对象，如果在函数中 return 一个直接量（普通变量），async 会把这个直接量通过 Promise.resolve() 封装成 Promise 对象。如果你返回了promise那就以你返回的promise为准。 await 是在等待，等待运行的结果也就是返回值。await后面通常是一个异步操作（promise），但是这不代表 await 后面只能跟异步操作 await 后面实际是可以接普通函数调用或者直接量的。

### await的等待机制？

如果 await 后面跟的不是一个 Promise，那 await 后面表达式的运算结果就是它等到的东西；如果 await 后面跟的是一个 Promise 对象，await 它会“阻塞”后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值作为 await 表达式的运算结果。但是此“阻塞”非彼“阻塞”这就是 await 必须用在 async 函数中的原因。async 函数调用不会造成“阻塞”，它内部所有的“阻塞”都被封装在一个 Promise 对象中异步执行。（这里的阻塞理解成异步等待更合理）

### async/await在使用过程中有什么规定？

每个 async 方法都返回一个 promise 对象。await 只能出现在 async 函数中。

### async/await 在什么场景使用？

单一的 Promise 链并不能发现 async/await 的优势，但是如果需要处理由多个 Promise 组成的 then 链的时候，优势就能体现出来了（Promise 通过 then 链来解决多层回调的问题，现在又用 async/await 来进一步优化它）。

### async/await如何使用？

假设一个业务，分多个步骤完成，每个步骤都是异步的且依赖于上一个步骤的结果。

```javascript
function myPromise(n) {
    return new Promise(resolve => {
        console.log(n)
        setTimeout(() => resolve(n+1), n)
    })
}
function step1(n) {
    return myPromise(n)
}
function step2(n) {
    return myPromise(n)
}
function step3(n) {
    return myPromise(n)
}

如果用 Promise 实现
step1(1000)
.then(a => step2(a))
.then(b => step3(b))
.then(result => {
    console.log(result)
})

如果用 async/await 来实现呢
async function myResult() {
    const a = await step1(1000)
    const b = await step2(a)
    const result = await step3(b)
    return result
}
myResult().then(result => {
    console.log(result)
}).catch(err => {
    // 如果myResult内部有语法错误会触发catch方法
})
```

看的出来async/await的写法更加优雅一些要比Promise的链式调用更加直观也易于维护。

我们来看在任务队列中async/await的运行机制，先给出大概方向再通过案例来证明：

1.  async定义的是一个Promise函数和普通函数一样只要不调用就不会进入事件队列。
2.  async内部如果没有主动return Promise，那么async会把函数的返回值用Promise包装。
3.  await关键字必须出现在async函数中，await后面不是必须要跟一个异步操作，也可以是一个普通表达式。
4.  遇到await关键字，await右边的语句会被立即执行然后await下面的代码进入等待状态，等待await得到结果。 await后面如果不是 promise 对象, await会阻塞后面的代码，先执行async外面的同步代码，同步代码执行完，再回到async内部，把这个非promise的东西，作为 await表达式的结果。 await后面如果是 promise 对象，await 也会暂停async后面的代码，先执行async外面的同步代码，等着 Promise 对象 fulfilled，然后把 resolve 的参数作为 await 表达式的运算结果。

##### 例1

```javascript
async function async1() {
  console.log('1')
  await new Promise((resolve) => {
    console.log('2')
    resolve()
  }).then(() => {
    console.log('3')
  })
  console.log('4')
}
async1()
```

1.  new Promise 的函数体是同步脚本所以先执行的是1、2。
2.  3和4都是微任务，这里因为有await，4要等Promise.then()之后才会执行。

console.log('4')已经被放在await语法糖生成的Promise.then里了，而await的等待必须要等后面Promise.then之后才会结束。

1 -> 2 -> 3 -> 4

##### 例2

```javascript
setTimeout(function () {
  console.log('6')
}, 0)
console.log('1')
async function async1() {
  console.log('2')
  await async2()
  console.log('5')
}
async function async2() {
  console.log('3')
}
async1()
console.log('4')
```

1.  6是宏任务在下一轮事件循环执行
2.  先同步输出1，然后调用了async1()，输出2。
3.  await async2() 会先运行async2()，5进入等待状态。
4.  输出3，这个时候先执行async函数外的同步代码输出4。
5.  最后await拿到等待的结果继续往下执行输出5。
6.  进入第二轮事件循环输出6。

1 -> 2 -> 3 -> 4 -> 5 -> 6

##### 例3

```javascript
console.log('1')
async function async1() {
  console.log('2')
  await 'await的结果'
  console.log('5')
}

async1()
console.log('3')

new Promise(function (resolve) {
  console.log('4')
  resolve()
}).then(function () {
  console.log('6')
})
```

1.  首先输出1，然后进入async1()函数，输出2。
2.  await后面虽然是一个直接量，但是还是会先执行async函数外的同步代码。
3.  输出3，进入Promise输出4，then回调进入微任务队列。
4.  现在同步代码执行完了，回到async函数继续执行输出5。
5.  最后运行微任务输出6。

1 -> 2 -> 3 -> 4 -> 5 -> 6

##### 例4

```javascript
async function async1() {
  console.log('2')
  await async2()
  console.log('7')
}

async function async2() {
  console.log('3')
}

setTimeout(function () {
  console.log('8')
}, 0)

console.log('1')
async1()

new Promise(function (resolve) {
  console.log('4')
  resolve()
}).then(function () {
  console.log('6')
})
console.log('5')
```

1.  首先输出同步代码1，然后进入async1方法输出2。
2.  因为遇到await所以先进入async2方法，后面的7被放入微任务队列。
3.  在async2中输出3，现在跳出async函数先执行外面的同步代码。
4.  输出4，5。then回调6进入微任务队列。
5.  现在宏任务执行完了，微任务先入先执行输出7、6。
6.  第二轮宏任务输出8。

1 -> 2 -> 3 -> 4 -> 5 -> 7 -> 6 -> 8

##### 例5

```javascript
setTimeout(function () {
  console.log('9')
}, 0)
console.log('1')
async function async1() {
  console.log('2')
  await async2()
  console.log('8')
}
async function async2() {
  return new Promise(function (resolve) {
    console.log('3')
    resolve()
  }).then(function () {
    console.log('6')
  })
}
async1()

new Promise(function (resolve) {
  console.log('4')
  resolve()
}).then(function () {
  console.log('7')
})
console.log('5')
```

1.  先输出1，2，3。3后面的then进入微任务队列。
2.  执行外面的同步代码，输出4，5。4后面的then进入微任务队列。
3.  接下来执行微任务，因为3后面的then先进入，所以按序输出6，7。
4.  下面回到async1函数，await关键字等到了结果继续往下执行。
5.  输出8，进行下一轮事件循环也就是宏任务二，输出9。

1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9

##### 例6

```javascript
async function async1() {
  console.log('2')
  const data = await async2()
  console.log(data)
  console.log('8')
}

async function async2() {
  return new Promise(function (resolve) {
    console.log('3')
    resolve('await的结果')
  }).then(function (data) {
    console.log('6')
    return data
  })
}
console.log('1')

setTimeout(function () {
  console.log('9')
}, 0)

async1()

new Promise(function (resolve) {
  console.log('4')
  resolve()
}).then(function () {
  console.log('7')
})
console.log('5')
```

1.  函数async1和async2只是定义先不去管他，首先输出1。
2.  setTimeout作为宏任务进入宏任务队列等待下一轮事件循环。
3.  进入async1()函数输出2，await下面的代码进入等待状态。
4.  进入async2()输出3，then回调进入微任务队列。
5.  现在执行外面的同步代码，输出4，5，then回调进入微任务队列。
6.  按序执行微任务，输出6，7。现在回到async1函数。
7.  输出data，也就是await关键字等到的内容，接着输出8。
8.  进行下一轮时间循环输出9。 执行结果：`1 - 2 - 3 - 4 - 5 - 6 - 7 - await的结果 - 8 - 9`

##### 例7

```javascript
setTimeout(function () {
  console.log('8')
}, 0)

async function async1() {
  console.log('1')
  const data = await async2()
  console.log('6')
  return data
}

async function async2() {
  return new Promise(resolve => {
    console.log('2')
    resolve('async2的结果')
  }).then(data => {
    console.log('4')
    return data
  })
}

async1().then(data => {
  console.log('7')
  console.log(data)
})

new Promise(function (resolve) {
  console.log('3')
  resolve()
}).then(function () {
  console.log('5')
})
```

1.  setTimeout作为宏任务进入宏任务队列等待下一轮事件循环。
2.  先执行async1函数，输出1，6进入等待状态，现在执行async2。
3.  输出2，then回调进入微任务队列。
4.  接下来执行外面的同步代码输出3，then回调进入微任务队列。
5.  按序执行微任务，输出4，5。下面回到async1函数。
6.  输出了4之后执行了return data，await拿到了内容。
7.  继续执行输出6，执行了后面的 return data 才触发了async1()的then回调输出7以及data。
8.  进行第二轮事件循环输出8。 执行结果：`1 - 2 - 3 -4 - 5 - 6 - 7 - async2的结果 - 8`

##### 例8

```javascript
async function test() {
  console.log('test start');
  await undefined;
  console.log('await 1');
  await new Promise(r => {
    console.log('promise in async');
    r();
  });
  console.log('await 2');
}

test();
new Promise((r) => {
  console.log('promise');
  r();
}).then(() => {
  console.log(1)
}).then(() => {
  console.log(2)
}).then(() => {
  console.log(3)
}).then(() => {
  console.log(4)
});
```

1.  从test()开始，先输出 'test start'，await 后面并不是Promise，不需要等then，这时候后面的 `console.log('await 1')` `await new Promise` `console.log('await 2')` 会立即进入微任务队列。
2.  然后 `console.log('promise')` 运行，`console.log(1)` 进入微任务队列。
3.  现在开始运行微任务，依据先入先出，`console.log('await 1')`、`console.log('promise in async')` 被运行。
4.  `console.log('promise in async')` 所在的Promise前面加了await，所以 `console.log('await 2')` 作为微任务进入微任务队列。
5.  刚才第二位进入微任务队列的 `console.log(1)` 运行，`console.log(2)` 进入微任务队列，在 `console.log(2)` 运行之前要先运行 `console.log('await 2')` 因为await2入队列早，后面的依次 2、3、4。

`test start` -> `promise` -> `await 1` -> `promise in async` -> `1` -> `await 2` -> `2` -> `3` -> `4`

