---
title: 09-Promise的链式调用
---

<ArticleTopAd></ArticleTopAd>

## 前言

实际开发中，我们经常需要先后请求多个接口：发送第一次网络请求后，等待请求结果；有结果后，然后发送第二次网络请求，等待请求结果；有结果后，然后发送第三次网络请求。以此类推。

比如说：在请求完接口 1 的数据`data1`之后，需要根据`data1`的数据，继续请求接口 2，获取`data2`；然后根据`data2`的数据，继续请求接口 3。换而言之，现在有三个网络请求，请求 2 必须依赖请求 1 的结果，请求 3 必须依赖请求 2 的结果。

如果按照往常的写法，会有三层回调，陷入“回调地狱”的麻烦。

这种场景其实就是接口的多层嵌套调用，在前端的异步编程开发中，经常遇到。有了 Promise 以及更高级的写法之后，我们可以把多层嵌套调用按照**线性**的方式进行书写，非常优雅。也就是说：Promise 等ES6的写法可以把原本的**多层嵌套写法**改进为**链式写法**。

我们来对比一下嵌套写法和链式调用的写法，你会发现后者的非常优雅。

## Promise 链式调用：封装多次网络请求

### ES5 中的传统嵌套写法

伪代码举例：

```js
// 封装 ajax 请求：传入请求地址、请求参数，以及回调函数 success 和 fail。
function requestAjax(url, params, success, fail) {
  var xhr = new xhrRequest();
  // 设置请求方法、请求地址。请求地址的格式一般是：'https://api.example.com/data?' + 'key1=value1&key2=value2'
  xhr.open('GET', url);
  // 设置请求头（如果需要）
  xhr.setRequestHeader('Content-Type', 'application/json');
  xhr.send();
  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
      success && success(xhr.responseText);
    } else {
      fail && fail(new Error('接口请求失败'));
    }
  };
}

// ES5的传统写法，执行 ajax 请求，层层嵌套
requestAjax(
  'https://api.qianguyihao.com/url_1', params_1,
  res1 => {
    console.log('第一个接口请求成功:' + JSON.stringify(res1));
    // ajax嵌套调用
    requestAjax('https://api.qianguyihao.com/url_2', params_2, res2 => {
      console.log('第二个接口请求成功:' + JSON.stringify(res2));
      // ajax嵌套调用
      requestAjax('https://api.qianguyihao.com/url_3', params_3, res3 => {
        console.log('第三个接口请求成功:' + JSON.stringify(res3));
      });
    });
  },
  (err1) => {
    console.log('qianguyihao 请求失败:' + JSON.stringify(err1));
  }
);
```

上面的代码层层嵌套，可读性很差，而且出现了我们常说的回调地狱问题。

### Promise 的嵌套写法

改用 ES6 的 Promise  之后，写法上会稍微改进一些。代码举例如下：

```js
// 【公共方法层】封装 ajax 请求的伪代码。传入请求地址、请求参数，以及回调函数 success 和 fail。
function requestAjax(url, params, success, fail) {
  var xhr = new xhrRequest();
  // 设置请求方法、请求地址。请求地址的格式一般是：'https://api.example.com/data?' + 'key1=value1&key2=value2'
  xhr.open('GET', url);
  // 设置请求头（如果需要）
  xhr.setRequestHeader('Content-Type', 'application/json');
  xhr.send();
  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4 && xhr.status === 200) {
      success && success(xhr.responseText);
    } else {
      fail && fail(new Error('接口请求失败'));
    }
  };
}

// 【model层】将接口请求封装为 Promise
function requestData1(params_1) {
  return new Promise((resolve, reject) => {
    requestAjax('https://api.qianguyihao.com/url_1', params_1, res => {
      // 这里的 res 是接口返回的数据。返回码 retCode 为 0 代表接口请求成功。
      if (res.retCode == 0) {
        // 接口请求成功时调用
        resolve('request success' + res);
      } else {
        // 接口请求异常时调用
        reject({ retCode: -1, msg: 'network error' });
      }
    });
  });
}


// requestData2、requestData3的写法与 requestData1类似。他们的请求地址、请求参数、接口返回结果不同，所以需要挨个单独封装 Promise。
function requestData2(params_2) {
  return new Promise((resolve, reject) => {
    requestAjax('https://api.qianguyihao.com/url_2', params_2, res => {
      if (res.retCode == 0) {
        resolve('request success' + res);
      } else {
        reject({ retCode: -1, msg: 'network error' });
      }
    });
  });
}

function requestData3(params_3) {
  return new Promise((resolve, reject) => {
    requestAjax('https://api.qianguyihao.com/url_3', params_3, res => {
      if (res.retCode == 0) {
        resolve('request success' + res);
      } else {
        reject({ retCode: -1, msg: 'network error' });
      }
    });
  });
}

// 【业务层】Promise 调接口的嵌套写法
// 发送第一次网络请求
requestData1(params_1).then(res1 => {
  console.log('第一个接口请求成功:' + JSON.stringify(res1));

  // 发送第二次网络请求
  requestData1(params_2).then(res2 => {
    console.log('第二个接口请求成功:' + JSON.stringify(res2));

    // 发送第三次网络请求
    requestData1(params_3).then(res3 => {
      console.log('第三个接口请求成功:' + JSON.stringify(res3));
    })
  })
})
```

上方代码非常经典。在真正的实战中，我们往往需要嵌套请求**多个不同的接口**，它们的接口请求地址、要处理的 resolve 和 reject 的时机、业务逻辑往往是不同的，所以需要分开封装不同的 Promise 实例。也就是说，如果要调三个不同的接口，建议单独封装三个不同的 Promise 实例：requestData1、requestData2、requestData3。

这三个 Promise 实例，最终都需要调用底层的公共方法 requestAjax()。每个公司都有这样的底层方法，里面的代码会做一些公共逻辑，比如：封装原生的 ajax请求，用户登录态的校验等等；如果没有这种公共方法，你就自己写一个，为组织做点贡献。

但是，细心的你可能会发现：上面的最后10行代码仍然不够优雅，因为 Promise 在调接口时出现了嵌套的情况，实际开发中如果真这么写的话，是比较挫的，阅读性非常差，我不建议这么写。要怎么改进呢？这就需要用到 Promise 的**链式调用**。

### Promise 的链式调用写法【重要】

针对多个不同接口的嵌套调用，采用 Promise 链式调用的写法如下：（将上方代码的最后10行，改进如下）

```js
requestData1(params_1).then(res1 => {
  console.log('第一个接口请求成功:' + JSON.stringify(res1));
  // 【关键代码】继续请求第二个接口。如果有需要，也可以把 res1 的数据传给 requestData2()的参数
  return requestData2(res1);
}).then(res2 => {
  console.log('第二个接口请求成功:' + JSON.stringify(res2));
  // 【关键代码】继续请求第三个接口。如果有需要，也可以把 res2 的数据传给 requestData3()的参数
  return requestData3(res2);
}).then(res3 => {
  console.log('第三个接口请求成功:' + JSON.stringify(res3));
}).catch(err => {
  console.log(err);
})
```

上面代码中，then 是可以链式调用的，一旦 return 一个新的 Promise 实例之后，后面的 then() 就可以作为这个新 Promise 在成功后的回调函数。这种**扁平化**的写法，更方便维护，可读性更好；并且可以更好的**管理**请求成功和失败的状态。

这段代码很经典，你一定要多看几遍，多默写几遍，倒背如流也不过分。如果你平时的异步编程代码能写到这个水平，说明你对 Promise 已经入门了，因为绝大多数人都是用的这个写法。

其实还有更高级、更有水平的写法，那就是用生成器、用 async ... await 来写Promise的链式调用。你把它掌握了，编程水平才能更上一层楼。我们稍后会讲。

## Promise 链式调用：封装 Node.js 的回调方法

代码结构与上面的类似，这里仅做代码举例，不再赘述。

### 传统写法

```js
fs.readFile(A, 'utf-8', function (err, data) {
    fs.readFile(B, 'utf-8', function (err, data) {
        fs.readFile(C, 'utf-8', function (err, data) {
          console.log('qianguyihao:' + data);
        });
    });
});
```

上方代码多层嵌套，存在回调地狱的问题。

### Promise 写法

```js
function read(url) {
    return new Promise((resolve, reject) => {
        fs.readFile(url, 'utf8', (err, data) => {
            if (err) reject(err);
            resolve(data);
        });
    });
}

read(A)
    .then((data) => {
        return read(B);
    })
    .then((data) => {
        return read(C);
    })
    .then((data) => {
        console.log('qianguyihao:' + data);
    })
    .catch((err) => {
        console.log(err);
    });
```



## 用 async ... await 封装链式调用

前面讲的内容是用 then().then().then() 这种

### 










## 链式调用，如何处理 reject 失败状态

### 例 1：不处理 reject

```js
getPromise('a.json')
    .then(
        (res) => {
            console.log(res);
            return getPromise('b.json'); // 继续请求 b
        },
        (err) => {
            // a 请求失败
            console.log('a: err');
        }
    )
    .then((res) => {
        // b 请求成功
        console.log(res);
        return getPromise('c.json'); // 继续请求 c
    })
    .then((res) => {
        // c 请求成功
        console.log('c：success');
    });
```

上面的代码中，假设 a 请求失败，那么，后面的代码会怎么走呢？

打印结果：

```
a: err
undefined
c：success
```

我们可以看到，虽然 a 请求失败，但后续的请求依然会继续执行。

为何打印结果的第二行是 undefined？这是因为，当 a 请求走到 reject 之后，我们并没有做任何处理。这就导致，代码走到第二个 `then`的时候，**其实是在执行一个空的 promise**。

### 例 2：单独处理 reject

```js
getPromise('a.json')
    .then(
        (res) => {
            console.log(res);
            return getPromise('b.json'); // 继续请求 b
        },
        (err) => {
            // a 请求失败
            console.log('a: err');
            // 【重要】即使 a 请求失败，也依然继续执行 b请求
            return getPromise('b.json');
        }
    )
    .then((res) => {
        // b 请求成功
        console.log(res);
        return getPromise('c.json'); // 继续请求 c
    })
    .then((res) => {
        // c 请求成功
        console.log('c：success');
    });
```

跟例 1 相比，例 2 在 reject 中增加了一行`return getPromise('b.json')`，意味着，即使 a 请求失败，也要继续执行 b。

这段代码，我们是单独处理了 a 请求失败的情况。

### 统一处理 reject

针对 a、b、c 这三个请求，不管哪个请求失败，我都希望做统一处理。这种代码要怎么写呢?我们可以在最后面写一个 catch。

代码举例如下：

```js
getPromise('a.json')
    .then((res) => {
        console.log(res);
        return getPromise('b.json'); // 继续请求 b
    })
    .then((res) => {
        // b 请求成功
        console.log(res);
        return getPromise('c.json'); // 继续请求 c
    })
    .then((res) => {
        // c 请求成功
        console.log('c：success');
    })
    .catch((err) => {
        // 统一处理请求失败
        console.log(err);
    });
```

上面的代码中，由于是统一处理多个请求的异常，所以**只要有一个请求失败了，就会马上走到 catch**，剩下的请求就不会继续执行了。比如说：

-   a 请求失败：然后会走到 catch，不执行 b 和 c

-   a 请求成功，b 请求失败：然后会走到 catch，不执行 c。

## return 的返回值

return 后面的返回值，有两种情况：

-   情况 1：返回 Promise 实例对象。返回的该实例对象会调用下一个 then。

-   情况 2：返回普通值。返回的普通值会直接传递给下一个 then，通过 then 参数中函数的参数接收该值。

我们针对上面这两种情况，详细解释一下。

### 情况 1：返回 Promise 实例对象

举例如下：

```js
getPromise('a.json')
    .then((res) => {
        // a 请求成功。从 resolve 获取正常结果：接口请求成功后，打印a接口的返回结果
        console.log(res);
        // 这里的 return，返回的是 Promise 实例对象
        return new Promise((resolve, reject) => {
            resolve('qianguyihao');
        });
    })
    .then((res) => {
        console.log(res);
    });
```

### 情况 2：返回 普通值

```js
getPromise('a.json')
    .then((res) => {
        // a 请求成功。从 resolve 获取正常结果：接口请求成功后，打印a接口的返回结果
        console.log(res);
        // 返回普通值
        return 'qianguyihao';
    })
    /*
        既然上方代码并没有返回 promise，那么，这里的 then 是谁来调用呢？
        答案是：这里会产生一个新的 默认的 promise实例，来调用这里的then，确保可以继续进行链式操作。
    */
    .then((res2) => {
        // 这里的 res2 接收的是 普通值 'qianguyihao'
        console.log(res2);
    });
```

## 赞赏作者

创作不易，你的赞赏和认可，是我更新的最大动力：

![](https://img.smyhvae.com/20220401_1800.jpg)
