# Promise

Promise 是一种 JavaScript 异步的实现方案。Promise 对象内部有异步操作的状态（成功或失败），以及返回值。

日常开发过程中，使用频率最高的异步方案也是 Promise 。因此为了更加深入的了解 Promise 原理，阅读 promise polyfill 库 [promise-polyfill](https://github.com/taylorhakes/promise-polyfill) 的源码是十分必要的。下面将对 promise-polyfill 源码做详细的解析。

> 由于 promise-polyfill 源码是基于 ES5 规范编写的，相对于 ES6+ 规范而言，代码不够简介、直接。为此，下面的源码解析，将用 ES6+ 代码来表示。

## Promise 构造函数

初始化对象数据，执行 executor 。

```js
class CustomPromise {
  constructor(executor) {
    if (typeof executor !== 'function') throw new TypeError('not a function');

    // 0 - pending
    // 1 - fulfilled/noObject & noFunction
    // 2 - rejected
    // 3 - fulfilled/CustomPromise
    this._state = 0;

    // 标记当前 promise 实例是否被处理过
    this._handled = false;

    // 记录当前 promise 的值
    this._value = undefined;

    // 存放 then 方法中注册的函数
    this._deferreds = [];

    doResolve(executor, this);
  }
}

/*
 * 执行 promise_executor
 *
 * 1. 给 executor 提供两个函数参数，让其选择性执行 resolve 或 reject
 * 2. 通过 done 保证 resolve 、 reject 只执行一次
 * 3. 如果 executor 函数执行过程中产生错误，则立即调用 reject
 */
const doResolve = (executor, self) => {
  let done = false;
  try {
    executor(
      (value) => {
        if (done) return;
        done = true;
        resolve(self, value);
      },
      (reason) => {
        if (done) return;
        done = true;
        reject(self, reason);
      },
    );
  } catch (ex) {
    if (done) return;
    done = true;
    reject(self, ex);
  }
};

/*
 * 执行 resolve ，进行状态转换
 *
 * 1. newValue !== self
 * 2. newValue instanceOf CustomPromise, self._state = 3, self_value = newValue
 * 3. typeof newValue.then === 'function', promise_executor -> newValue.then
 * 4. newValue is otherValue, self._state = 1, self_value = newValue
 * 5. if code error, reject(self, error)
 */
const resolve = (self, newValue) => {
  try {
    if (newValue === self) throw new TypeError('A promise cannot be resolved with itself.');

    if (newValue instanceof CustomPromise) {
      self._state = 3;
      self._value = newValue;
      finale(self);
    } else if (typeof newValue?.then === 'function') {
      doResolve(bind(then, newValue), self);
    } else {
      self._state = 1;
      self._value = newValue;
      finale(self);
    }
  } catch (e) {
    reject(self, e);
  }
};

/*
 * 执行 reject ，进行状态转换
 */
const reject = (self, newValue) => {
  self._state = 2;
  self._value = newValue;
  finale(self);
};

/*
 * resolve 或者 reject 之后的最终函数，主要处理 _deferreds
 *
 * 1. 状态为 rejected 且 deferreds为空，则打印错误信息
 * 2. 依次执行 deferred
 */
const finale = (self) => {
  // 状态为 rejected 且 deferreds为空，则打印错误信息
  if (self._state === 2 && self._deferreds.length === 0 && !self._handled) {
    CustomPromise._immediateFn(() => {
      CustomPromise._unhandledRejectionFn(self._value);
    });
  }

  // 依次执行 deferred
  self._deferreds.forEach((deferred) => handle(self, deferred));

  self._deferreds = null;
};

/*
 * 处理 deferrd
 *
 * 1. 如果 _value 为 3 ，则遍历 self
 * 2. 如果状态为 pending ，则 deferred 入队，停止执行剩余代码
 * 3. 根据 _state 为 callback 赋值 onFulfilled 或 onRejected
 *     3.1 如果 callback 不是函数，则 _value 顺延至 deferred.promise._value
 *     3.2 否则 callback 执行结果赋值给 deferred.promise._value
 */
const handle = (self, deferred) => {
  // 不断遍历self，直到self._state不是3，即self._value不是promise
  while (self._state === 3) self = self._value;

  // self._state 为 pending时，deferred 入队
  if (self._state === 0) {
    self._deferreds.push(deferred);
    return;
  }

  self._handled = true;

  CustomPromise._immediateFn(() => {
    const callback = self._state === 1 ? deferred.onFulfilled : deferred.onRejected;

    if (callback === null) {
      // callback 没有函数，则 self._value 顺延
      (self._state === 1 ? resolve : reject)(deferred.promise, self._value);
      return;
    }

    // callback 执行结果作为 fulfiled，除非执行过程中出错
    let ret;
    try {
      ret = callback(self._value);
    } catch (e) {
      reject(deferred.promise, e);
      return;
    }
    resolve(deferred.promise, ret);
  });
};
```

## then

将 then 函数中的 onFulfilled、onRejected 挂载到 deferred 对象上。并且根据 \_state ，决定立即执行或插入 \_deferreds 等待被执行。

```js
class CustomPromise {
  /*
   * 创建 Promise 对象
   * 按照 self._state 选择性执行 resolve 或 reject
   * 返回该 Promise 对象
   */
  then(onFulfilled, onRejected) {
    const prom = new this.constructor(() => {});

    handle(this, new Handler(onFulfilled, onRejected, prom));
    return prom;
  }
}

/*
 * 创建 deferred 对象
 *
 * 对象包含：onFulfilled, onRejected, promise
 */
class Handler {
  constructor(onFulfilled, onRejected, promise) {
    this.onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : null;
    this.onRejected = typeof onRejected === 'function' ? onRejected : null;
    this.promise = promise;
  }
}
```

## catch

catch 方法是 then(null, onRejected) 的语法糖。

```js
class CustomPromise {
  catch(onRejected) {
    return this.then(null, onRejected);
  }
}
```

## finally

不论 promise 对象何种状态，finally-callback 函数一定会被执行。

```js
class CustomPromise {
  /*
   * 不论成功或拒绝，finally 的 callback 函数一定会执行。
   * 且 _value 和 _state 会被顺延。
   */
  finally(callback) {
    const constructor = this.constructor;
    return this.then(
      (value) => {
        callback();
        return value;
      },
      (reason) => {
        callback();
        return constructor.reject(reason);
      },
    );
  }
}
```

## resolve

```js
class CustomPromise {
  /*
   * 静态 resolve 方法返回一个 CustomPromise 对象
   */
  static resolve(value) {
    if (value && typeof value === 'object' && value.constructor === CustomPromise) return value;

    return new CustomPromise((resolve) => {
      resolve(value);
    });
  }
}
```

## reject

```js
class CustomPromise {
  /*
   * 静态 reject 方法返回一个 CustomPromise 对象
   */
  static reject(value) {
    return new CustomPromise((resolve, reject) => {
      reject(value);
    });
  }
}
```

## all

传入参数为可迭代的对象。

如果对象元素状态都为 fulfilled ，则按照顺序返回每个元素的结果。
如果某个对象元素状态为 rejected ，则返回该元素的拒绝结果。

```js
class CustomPromise {
  static all(list) {
    return new CustomPromise((resolve, reject) => {
      const len = list.length;
      const result = Array(len).fill(undefined);
      let count = 0;

      const resolver = (i) => {
        return (value) => {
          result[i] = value;
          if (++count === len) resolve(result);
        };
      };

      for (let i = 0; i < len; i++) {
        CustomPromise.resolve(list[i]).then(resolver(i), reject);
      }
    });
  }
}
```

## race

传入参数为可迭代的对象。

如果某个对象元素状态为 fulfilled ，则返回该元素的结果。
如果某个对象元素状态为 rejected ，则返回该元素的拒绝结果。

```js
class CustomPromise {
  static all(list) {
    return new CustomPromise((resolve, reject) => {
      const len = list.length;
      for (let i = 0; i < len; i++) {
        CustomPromise.resolve(list[i]).then(resolve, reject);
      }
    });
  }
}
```

## allSettled

传入参数为可迭代的对象。

按照对象元素的顺序，返回每个元素的状态以及结果。

```js
class CustomPromise {
  static allSettled(list) {
    return new CustomPromise((resolve, reject) => {
      const len = list.length;
      const result = Array(len).fill(undefined);
      let count = 0;

      const resolver = (i) => {
        return (value) => {
          result[i] = { status: 'fulfilled', value };
          if (++count === len) resolve(result);
        };
      };

      const rejecter = (i) => {
        return (reason) => {
          result[i] = { status: 'rejected', reason };
          if (++count === len) resolve(result);
        };
      };

      for (let i = 0; i < len; i++) {
        CustomPromise.resolve(list[i]).then(resolver, rejecter);
      }
    });
  }
}
```

## 辅助函数

\_unhandledRejectionFn，处理未能及时 reject 的数据

```js
class CustomPromise {
  static _unhandledRejectionFn(err) {
    if (console) console.warn('Possible Unhandled Promise Rejection:', err);
  }
}
```

\_immediateFn，以微任务的执行顺序，执行函数 fn

```js
class CustomPromise {
  // 利用 MutationObserver 的微任务执行特性，执行函数 fn
  static _immediateFn(fn) {
    let hiddenDiv = document.createElement('div');
    new MutationObserver(() => {
      fn();
      hiddenDiv = null;
    }).observe(hiddenDiv, { attributes: true });
    hiddenDiv.setAttribute('i', '1');
  }
}
```

## 实际应用

### promiseAllThrottled

以节流的方式调用异步函数，默认并发数为 3 。

```ts
const promiseAllThrottled = (iterable: (() => Promise<unknown>)[], concurrency = 3) => {
  const promises: Promise<unknown>[] = [];

  const enqueue = (current = 0, queue: Promise<unknown>[] = []): Promise<void> => {
    // return if done
    if (current === iterable.length) return Promise.resolve();

    // take one promise from collection
    const promise = iterable[current];
    const activatedPromise = promise();

    // add promise to the final result array
    promises.push(activatedPromise);

    // add current activated promise to queue and remove it when done
    const autoRemovePromise = activatedPromise.then(() => {
      // remove promise from the queue when done
      const removeIndex = queue.indexOf(autoRemovePromise);
      if (removeIndex > -1) queue.splice(removeIndex, 1);
      return true;
    });

    // add promise to the queue
    queue.push(autoRemovePromise);

    // if queue length >= concurrency, wait for one promise to finish before adding more.
    const readyForMore = queue.length < concurrency ? Promise.resolve() : Promise.race(queue);

    return readyForMore.then(() => enqueue(current + 1, queue));
  };

  return enqueue().then(() => Promise.all(promises));
};

// test
const requests = Array(10)
  .fill('')
  .map(
    (_, i) => () =>
      new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve(`task #${i}`);
        }, 1000);
      }),
  );

promiseAllThrottled(requests, 3)
  .then((arg) => console.log(`finished ${new Date()} ${arg}`))
  .catch((error) => console.error('Oops something went wrong', error));
```

## 练习一下吧

[45 道 Promise 面试题](https://juejin.cn/post/6844904077537574919#heading-54)

## 总结

求木之长者，必固其根本；欲流之远者，必浚其泉源。短短的两百多行代码，解释了 promise 所有的特性，平时开发中遇到的 promise 问题瞬间豁然开朗。
