### 1 实现一个timeout函数,fn在seconds秒内执行则成功（直接使用setTimeout）

```js
function fn1() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(1);
      resolve()
    }, 1000);
  })
}

//实现一个timeout函数,fn seconds秒内执行则成功
function timeout(fn, seconds) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // 如果在秒数到达还未执行，则返回失败
      reject()
    }, seconds);

    fn().then(() => {
      // 如果fn返回了成功，则立即将当前的promise状态变为成功
      resolve()
    })
  })
}

// fn返回一个promise
// seconds：时间，单位秒
// 假设fn1 2秒执行
timeout(fn1, 2000).then(() => {
  console.log("成功")
}).catch(() => {
  console.log("失败")
})


```

### 1 实现一个timeout函数,fn在seconds秒内执行则成功（使用setTimeout和Promise.race）

```js
function fn1() {//模仿一个异步函数，此函数一秒以后执行完，返回成功的promise
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log(1)
      resolve(777)
    }, 1000)
  })
}

function timerCtrl(time) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve("超时") //如果在秒数要求到达还未执行完
    }, time)
  })
}

function beforeTime(fn1, time) {//传入要监听的函数和秒数要求
  return new Promise((resolve, reject) => {
    Promise.race([timerCtrl(time), fn1()]).then((result) => {
      if (result == '超时') reject("超时")
      else resolve(result)
    })
  })
}

beforeTime(fn1, 2000).then((res) => {
  console.log("成功", res)
}, (err) => {
  console.log("失败", err)
})
```

### 2 实现一个函数，三秒后执行指定函数

```js
function myFun() {
  console.log("执行了");
}


async function timeSleep(delay) {
  return await new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("执行完毕")
    }, delay);
  })
}

timeSleep(3000).then(res => {
  myFun()
}).catch(err => {
  console.log(err);
})
```

### 3 比较fn1和fn2哪个先执行完

```js
function fn1() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("fn1先完成")
    }, 1000);
  })
}

function fn2() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("fn2先完成")
    }, 2000);
  })
}

Promise.race([fn1(), fn2()]).then(res => {
  console.log(res);
}).catch(err => {
  console.log(err);
})
```

### 4 改写Promise.all，给其中每个请求都加上超时时间，并给所有请求加上超时总时间

```js
//改写Promise.all：1. 给其中的每个请求都加上超时时间,并给所有请求加上超时总时间
function promiseAll(list) {
  let len = list.length
  let resArr = new Array.fill(0)
  let count = 0

  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('请求总时间超时')
    }, 10000)
    for (let i = 0; i < len; i++) {
      setTimeout(() => {
        reject('单个请求超时')
      }, 5000)
      list[i].then((result) => {
        resArr[i] = result
        count++
        if (count >= len) {
          resolve(resArr)
        }
      }, (err) => {
        reject(err)
      })
    }
  })
}
```

### 5 有20个请求，需要限制执行，并发请求只有4个

```js
async function asyncPool(poolLimit, array) {
  const ret = []; // 用于存放所有的promise实例
  const executing = []; // 用于存放目前正在执行的promise
  for (const item of array) {
    const p = Promise.resolve(item);
    ret.push(p);
    if (poolLimit <= array.length) {
      // then回调中，当这个promise状态变为fulfilled后，将其从正在执行的promise列表executing中删除
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e);
      if (executing.length >= poolLimit) {
        // 一旦正在执行的promise列表数量等于限制数，就使用Promise.race等待某一个promise状态发生变更，
        // 状态变更后，就会执行上面then的回调，将该promise从executing中删除，
        // 然后再进入到下一次for循环，生成新的promise进行补充
        await Promise.race(executing);
      }
    }
  }
  return Promise.all(ret);
}
```

### 6 用promise实现最大并发请求数

```js
function multiRequest(urls, maxNum) {
  const len = urls.length; // 请求总数量
  const res = new Array(len).fill(0); // 请求结果数组
  let sendCount = 0; // 已发送的请求数量
  let finishCount = 0; // 已完成的请求数量
  return new Promise((resolve, reject) => {
    // 首先发送 maxNum 个请求，注意：请求数可能小于 maxNum，所以也要满足条件2
    // 同步的 创建maxNum个next并行请求 然后才去执行异步的fetch 所以一上来就有5个next并行执行
    while (sendCount < maxNum && sendCount < len) {
      next();
    }
    function next() {
      let current = sendCount++; // 当前发送的请求数量，后加一 保存当前请求url的位置
      // 递归出口
      if (finishCount >= len) {
        // 如果所有请求完成，则解决掉 Promise，终止递归
        resolve(res);
        return;
      }
      fetch(urls[current]).then(result => {//利用fetch发请求
        resultArr[current] = result
      }).catch(err => {
        resultArr[current] = err
      }).finally(() => {
        finishCount++
        if (sendCount < len) { //如果未发送完，继续发送
          _next()
        }
      })
    }
  });
}
```

### 7 手写resolve、reject、race、all方法

```js
Promise.resolve = function (res) {
  return new Promise((resolve, reject) => {
    if (res instanceof Promise) {
      res.then(data => {
        resolve(data)
      }).catch(err => {
        reject(err)
      })
    } else {
      resolve(res)
    }
  })
}

Promise.reject = function (res) {
  return new Promise((resolve, reject) => {
    reject(res)
  })
}

Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let arr = []
    let count = 0

    for (let i = 0; i < promises.length; i++) {
      promises[i].then(res => {
        count += 1
        arr[i] = res
        if (count === promises.length) {
          resolve(arr)
        }
      }).catch(err => {
        reject(err)
      })
    }
  })
}

Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(res => {
        resolve(res)
      }).catch(err => {
        reject(err)
      })
    }
  })
}
```

### 8 每隔一秒打印一次

```js
// 需要实现的函数 
function repeat(func, times, wait) {
  return async function (...args) {
    for (let i = 0; i < times; i++) {
      await new Promise((resolve, reject) => {
        setTimeout(() => {
          func.apply(this, args)
          resolve()
        }, wait);
      })
    }
  }
}

// 使下面调用代码能正常工作 
const repeatFunc = repeat(console.log, 4, 1000);
repeatFunc("hellworld");//会输出4次 helloworld, 每次间隔1秒
```

### 9 每隔一秒依次输出0,1,2,3,4,5

```js
// 模拟其他语言中的 sleep，实际上可以是任何异步操作
const sleep = (timeountMS) => new Promise((resolve) => {
  setTimeout(resolve, timeountMS);
});


async function main() {
  for (let i = 0; i < 6; i++) {
    await sleep(1000)
    console.log(i);
  }
}

main()
```

### 10 每隔n秒打印n

```js
// 给了一个sleep函数， 实现功能，隔1s打印1，再隔2s打印2，隔3秒打印3
function sleep(timeout) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, timeout)
  })
}


async function main() {
  // 写代码
  for (let i = 1; ; i++) {
    if (i % 3 === 0) {
      await sleep(3000)
      console.log(3);
    } else {
      await sleep(i % 3 * 1000)
      console.log(i % 3);
    }

    console.log(new Date);
  }
}

main()
```