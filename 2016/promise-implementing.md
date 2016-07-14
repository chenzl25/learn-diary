## Promise

### [Promise Implementing](https://www.promisejs.org/implementing/)

Promise就是一个状态机，它包含了3个状态，PENDING, FULFILLED 和 REJECTED：

- PENDING代表还没有返回
- FULFILLED则代表了返回成功
- REJECTED代表了返回失败

#### Promise类
```js
var PENDING = 0;
var FULFILLED = 1;
var REJECTED = 2;

function Promise() {
  // store state which can be PENDING, FULFILLED or REJECTED
  var state = PENDING;

  // store value once FULFILLED or REJECTED
  var value = null;

  // store sucess & failure handlers
  var handlers = [];

  function fulfill(result) {
    state = FULFILLED;
    value = result;
  }

  function reject(error) {
    state = REJECTED;
    value = error;
  }
}
```
上面的代码中：还有fulfill方法和reject来转换状态，同时把结果记录到value中


----

#### Promise的（thenable）递归状态转移

```js
//这里Promise可以接受fn了
function Promise(fn) {
  ...
  function resolve(result) {
    try {
      var then = getThen(result);
      if (then) {
        doResolve(then.bind(result), resolve, reject)
        return
      }
      fulfill(result);
    } catch (e) {
      reject(e);
    }
  }
  doResolve(fn, resolve, reject);
}
```
resolve方法则是我们熟悉的Promise.resolve方法，这个方法可以接受一个普通的result值，或者接受一个thenable的对象(上面是通过getThen方法获取的)
*ps:递归调用resolve方法直到result不为thenable对象为止，不为thenable对象后会调用fulfill方法来更新Promise的状态，或者出错后调用reject来更新Promise的状态*

```js
function getThen(value) {
  var t = typeof value;
  if (value && (t === 'object' || t === 'function')) {
    var then = value.then;
    if (typeof then === 'function') {
      return then;
    }
  }
  return null;
}
```

getThen方法则是判断传进来的value是否有then方法，有则返回then方法，没则返回null

```js
function doResolve(fn, onFulfilled, onRejected) {
  var done = false;
  try {
    fn(function (value) {
      if (done) return
      done = true
      onFulfilled(value)
    }, function (reason) {
      if (done) return
      done = true
      onRejected(reason)
    })
  } catch (ex) {
    if (done) return
    done = true
    onRejected(ex)
  }
}
```

doResolve方法中接受的fn就是Promise(其实，可then的对象就行)的then方法。
*ps:上面Promise的resolve方法在传入fn的时候会绑定当时的thenable对象*

doResolve方法还会接受用于调用fn函数的两个参数onFulfilled和onRejected，均为函数。后面就是fn函数的执行了。fn函数就像是一个Promise，如果我们的thenable对象调用了then方法的时候就会期望处理了自己的内部逻辑后(例如：ajax请求)，如果行了，就会调用onFulfilled函数，如果不行就调用onRejected。
*ps:done变量用来控制onFulfilled和onRejected只调用一次。即状态只会转移一次。*

-----

#### Promise.done(onFulfilled, onRejected)
好了上面我们完成了Promise状态机的内部实现，接下来我们要观察这个状态机，如果它到达相应的状态，我们就调用相应的函数

```js
function Promise(fn) {
  ...
  //fulfill和reject是改变Promise状态的最底层方法
  function fulfill(result) {
    state = FULFILLED;
    value = result;
    handlers.forEach(handle);
    handlers = null;
  }

  function reject(error) {
    state = REJECTED;
    value = error;
    handlers.forEach(handle);
    handlers = null;
  }
  ...
  function handle(handler) {
    if (state === PENDING) {
      handlers.push(handler);
    } else {
      if (state === FULFILLED &&
        typeof handler.onFulfilled === 'function') {
        handler.onFulfilled(value);
      }
      if (state === REJECTED &&
        typeof handler.onRejected === 'function') {
        handler.onRejected(value);
      }
    }
  }

  this.done = function (onFulfilled, onRejected) {
    // ensure we are always asynchronous
    setTimeout(function () {
      handle({
        onFulfilled: onFulfilled,
        onRejected: onRejected
      });
    }, 0);
  }
  ...
}
```

done方法，用来注册函数，当Promise状态达到FULFILLED或REJECTED的时候就会调用相应的函数。注册的时候会先判断现在Promise的状态，如果是PENDING的话就会保存handler，等待到状态改变的时候调用，如果是已经到达了最终状态就会马上调用
*ps：done会保证异步，同步也会使用setTimeout来保证异步*

-----

#### Promise.then(onFulfilled, onRejected)
```js
this.then = function (onFulfilled, onRejected) {
  var self = this;
  return new Promise(function (resolve, reject) {
    return self.done(function (result) {
      if (typeof onFulfilled === 'function') {
        try {
          return resolve(onFulfilled(result));
        } catch (ex) {
          return reject(ex);
        }
      } else {
        return resolve(result);
      }
    }, function (error) {
      if (typeof onRejected === 'function') {
        try {
          return resolve(onRejected(error));
        } catch (ex) {
          return reject(ex);
        }
      } else {
        return reject(error);
      }
    });
  });
}
```

then方法则用了done方法来做跳板实现。
then方法会返回一个新的Promise实例，其中新的Promise实例接受的函数fn会立刻被递归调用(直至不在resolve到thenable对象)，在fn函数里面就调用了self.done方法(这里的self绑定了调用then的Promise对象，意思是当调用then的Promise已经有状态的时候，我们就可以调用then中的函数了)。
之后就到了done里面的回调函数执行，这里会对done中获得调用then的Promise对象的结果传入到then方法中注册的函数中（注册在then中的函数(onFulfilled, onRejected)，then中注册函数的返回值会被then方法新建的Promise用来判断是否能完成自己的状态转移
*ps: 看到了定义的`this.done`和`this.then`方法就知道这需要new出Promise才能使用*

----
### **conclusion**
这里总结下调用的总过程。
首先所有一切到围绕着Promise这个function来运作
在这个实现中要使用Promise的话就必须先`new Promise(fn)`
一旦我们提供了`fn`（*不提供就是reject*）用来`new`，就会开始调用`doResolve(fn, resolve, reject);`方法，而这里的`reslove`又会调用`doResolve`方法。这样就会递归地调用下去，直到`resolve`函数里面调用`getThen`获取不到thenable对象。
在`doResolve`函数中会调用传递进来的fn函数。fn函数里包含了调用者自身函数的逻辑,例如

```js
var p = new Promise(function(resolve, reject) {
  ajax(function(err, data) {
    if(err) reject(err);
    else resolve(data);
  })
})
```

当调用者完成了自己的逻辑后就会根据情况调用`resolve`，或者`reject`：

- 这个时候如果`resolve`的对象不是thenable对象，或者`reject`了就可以让当前的Promise实例进行状态转移了。
- 如果`resolve`的对象是thenable对象，则需要用`getThen`函数来获取`then`方法，使用doResolve来完成thenable对象的提供的逻辑，如果thenable对象逻辑里面的`resolve`方法再次接受一个thenable对象就会一直地递归调用，直到不再thenable为止。当最后的thenable完成后，resolve一个非thenable的对象就可以对Promise进行状态转移了。当然上述过程中有一处出现了错误，或者`reject`函数被调用了也会使得Promise实例的状态进行转移。

好了，完成了Promise内部的状体转移方法后，我们还需要hook，也就是当Promise转移到`FULFILLED`或者`REJECTED`的时候，可以自动调用我们注册的对应的函数。这就引来了`done`方法

`p.done(onFulfilled, onRejected)`当p转移到最终状态时就会调用这些函数。这里值得一提的是done方法用了`setTimeout(f,0)`来保证异步执行，这样做的理由是因为，如果当前的Promise已经是到达了最终状体的时候，如果没有异步调用就会导致done函数马上执行，这就会造成不确定性。我们编写程序时，不知道done函数的运行时刻，后续的步骤就会由此而混乱了。所以统一异步执行。
当然如果Promise状态还是`PENDING`的时候就要把done传入的函数进行保存，等到状态转移的时候再调用

最后，就是我们的`then`函数的实现了。
先思考下。我们之前所做的一切都是单个Promise，单个状态机在转移。
虽然我们可以借助thenable对象加入自己的逻辑，一直这样resolve thenable对象递归地完成我们的异步调用。虽然callback hell的深度嵌套没了，但是这样的代码因为不断地跳转难以阅读。
*PS：（通常分开回调函数也能做到，当我这段没讲。。。）*

如果我们引入多个Promise，即多个状态机，当前一个状态机到达`FULFILLED`后，把对应状态的value往后面传递，根据value链式串行激活后面的状态机。如果其中一个状态机到达了`REJECTED`就会停止在`REJECTED`这个状态。
*这就是then的思想。*

其中我们需要在then上做的工作就是，利用我们之前已有的基础，把状态机的串接实现，封装一个好的接口出来给用户使用。
`then`函数里会先把`then`函数的`this`保存一份到`self`为什么要保存？(因为new Promise里面只有通过done和then来访问这个Promise对象，其他都被闭包封闭掉了，而后面会new一个新的Promise来串接，如果不保存self就不能访问回前一个Promise了)，new一个新的Promise用来返回以便继续串接下去，这个Promise的`fn`函数就用来串接的，它会在前一个Promise中调用`done`函数，如果done函数返回了值value（`FULFILLED`状态下），error（`REJECTED`状态下）。就会把这些值传到我们的注册在then方法的函数中(*也是我们熟悉的then（value => xxx,err=>xxx）*)的用法，而我们传到`then`中的方法

```js
.then(
  value => doSomething(value),
  err => doAnotherThing(err)
)
```

`doSomething(value)`和`doAnotherThing(err)`的返回值会被放到我们新创建的Promise的value中。这样的话我们就封装好了Promise的then的串接。
*当然`doSomething(value)`可以返回一个thenable的对象。*

有了then以后，我们写的代码异步就变得很清晰了。其中的要义就是Promise的then为我们做了一层状态机的串接，再结合我们的异步逻辑代码，一步一步地执行下去。这就是以同步的形式来做异步调用。

有了状体机的思想，我们就可以继续去专研`all`，`race`等Promise的方法了。

