# 一诺千金,Promise 不会让你失望

## > 概念:

1. Promise是异步处理工具, 我们在前端开发过程中, 不可避免的需要用到Promise
2. 在Promise标准出来之前 , 通常我们使用ajax来请求后台数据,也就是js原生技术(XMLHttpRequest对象),但是传统的ajax请求会出现回调地狱的问题, 也就是请求嵌套请求, 层层嵌套下来, 会给我们的代码带来阅读上和定位bug上的难度, 也不符合我们人类线性思维 , 也就是从上到下执行
3. Promise的用法其实不难, 但是要想知道Promise具体是如何工作, 还需要剖析源码 , 现在我们就来以线性的思维一步一步理解Promise

[这篇文章非常详细, 我是基于这篇文章引发得思考,以便于彻底理解]: https://juejin.im/post/5e3b9ae26fb9a07ca714a5cc#heading-13



##  > 先来一段标准的Promise代码

```javascript
const P1 = new Promise((resolve,reject) => {
    setTimeOut(() => {
        resolve(1)
    },1000)
})
P1.then(res => {
    console.log(res)
})
// 输出: 1 
```



- 我们来看这段代码 , 首先: 我们定义了一个常量P1, 这个常量被初始化为一个Promise对象, 那也就是P1现在就是Promise的实例, 拥有Promise对象的方法和属性, 然后new Promise的时候, 会立即执行以下这段代码 

  ```javascript
  (resolve,reject) => {
      setTimeOut(() => {
          resolve(1)
      },1000)
  }
  // 这段代码就是一个整体的函数, 箭头函数也是函数, 这段代码就是要传入Promise构造函数的executor函数(英文翻译为:'执行者')
  // 因为Promise对象中的构造函数接受一个executor函数,当Promise对象被new的时候, 会立即执行构造函数
  // 而构造函数执行的时候, 又会立即执行executor函数,执行executor函数的时候, 会将resolve和reject两个函数作为参数传递给executor函数
  
  // /****************结合上下文来理解以上注释, 知识是要反复来阅读,反复来理解才行************/
  ```

- 执行完这段代码 , 发生了下列事件: 

  1. Promise内部会实现 resolve  reject这两个的函数的具体行为 , 然后会将这两个函数传入executor函数 , 然后executor函数立即执行,  那么我们书写的上面的代码片段会立即执行 , 程序执行到setTimeOut , 发现这是一个延迟操作,也称异步操作 我们知道JavaScript采用了事件轮询机制, 这种异步操作时会被Javascript主线程移交给事件轮询机制 , 然后程序向下继续执行同步代码

  2. 涉及Javascript的事件轮询机制, 可以访问:

     [什么是 Event Loop？]: https://www.ruanyifeng.com/blog/2013/10/event_loop.html

- 程序继续向下执行, 执行到 以下代码

  ```javascript
  P1.then(res => {
      console.log(res)
  })
  ```

- 这段代码又发生了什么呢, 我们坐下来好好谈谈:

  1. 首先,我们应该知道 then 方法就定义在Promise的对象中, 不然Promise实例为什么能调出来 then 方法呢,

  2. Promise源码中的then函数实现,是这样的:  then函数接收两个函数作为参数 , 第一个参数是异步成功时, resolve所执行的函数 , 第二个参数是异步执行失败时, reject所执行的函数 , 那么现在我们传入了一个函数,也就是给then函数传了一个参数, 这个参数就默认被设置为then函数的第一个参数

  3.  这个箭头函数作为参数传递给then函数

     ```javascript
        res => {
          console.log(res)
        }
     ```

#### 直到现在, 或许你还不知道Promise的源码实现是怎么样的, 那么我当然会带你来进入源码 , 来探究Promise的奥秘

> 首先 : 我们来模拟一下Promise对象
>
> ```javascript
> // 我们来写一个Promise的类
> class MyPromise{
> 	constructor(executor){
>         this.resolveQueue = []	//收集成功状态的函数集合 采用数组队列形式, 可以依次取出任务
>         this.rejectQueue = []	 // 收集拒绝状态的函数集合
>     }
>     const _resolve = (value) => { 				// 定义要传给executor函数的resolve函数
>         while(this.resolveQueue.length){							
>             const FN = this.resolveQueue.shift() // shift可以取出并返回数组第一个元素, 并在数组中删除此元素 , 会改变数组的长度
>             FN(value) 		// 执行取出的函数
>         }
>     }
>     // 从失败的回调集合中, 依次取出函数并执行
>     const _reject = (value) => {
>         while(this.rejectQueue.length){
>             const FN = this.rejectQueue.shift()
>             FN(value)
>         }
>      // 当new Promise的时候,会立即执行这个函数, 这就是上文说的我们传入的函数
>      executor(_resolve,_reject)
>     }
>     
>     // then方法, 接收两个函数参数, 第一个是执行成功的函数回调, 第二个是执行失败的函数回调
>     then(resolveFn,rejectFn){
>         this.resolveQueue.push(resolveFn)
>         this.rejectQueue.push(rejectFn)
>     }
> }
> // new 实例 并运行
> const P1 = new MyPromise((resolve,reject) => {
>     setTimeOut(() => {
>         resolve(1)
>     },1000)
> })
> P1.then(res => {
>     console.log(res)
> })
> // 输出:1
> ```

- 至此, Promise的简单实现就已经完成 ,  现在我们就可以大概知道Promise内部是怎么实现的了
- 在Chrome浏览器借助打断点的方法, 我们可以知道其运行过程如下:
  1. 首先 , P1是一个Promise实例, 然后程序执行executor函数, 并把setTimeOut交给事件轮询机制 , 
  2. 然后程序进入then函数, 我们给then函数传入了第一个参数(是一个函数哦),  然后then函数把第一个参数添加到成功执行的函数集合中,现在, 同步代码已经执行完毕, 
  3. 事件轮询机制开始执行 , 1秒后执行 `resolve(1)`这段代码 , 这个函数已经在Promise对象中定义, 程序开始执行这段代码 , 
  4. 我们书写的时候, 给resolve传入了一个参数 , 所以执行对象中的resolve的时候, value值就为1, 然后程序进去while循环, 先取出第一个成功状态的函数集合 , 然后执行, 这段代码就是我们传入的 res => {console.log(res)} , 所以程序会输出1.
  5. 怎么样, 是不是很简单呢



#### 读到现在, 你不妨也在Chrome浏览器下运行一番代码, 会有很清晰的理解

------

# 我们实现了Promise的冰山一角, 还有一段路要走,不过,不远了, 请坚持,别放弃

- 我们实现了最简单版本的Promise, 还不够, 因为Promise的实现遵循Promise/A+规范, 该规范核心的两条是:

  1.  Promise本质是一个状态机, 且状态只能为以下三种, `Pending(等待)` , `Fulfilled(执行)` , `rejected(拒绝)` . 状态的变更时单向的 , 且不可逆 , 只能由`Pending` => `Fulfilled` || `Pending` => `rejected`. 且状态一经变更 就会固定此状态
  2. then方法接收两个可选参数 , 分别对应状态改变时触发的回调, then方法返回一个Promise , then方法可以被同一个Promise调用多次

- 那么 , 我们来补充一下A+规范

  ```javascript
  const PENDING = 'PENDING'		//等待状态
  const FULFILLED = 'FULFILLED'	//执行状态
  const REJECTED = 'REJECTED'		//拒绝状态
  // 我们来写一个Promise的类
  class MyPromise{
  	constructor(executor){
          this.status = PENDING	//默认状态设置为等待状态
          this.resolveQueue = []	//收集成功状态的函数集合 采用数组队列形式, 可以依次取出任务
          this.rejectQueue = []	 // 收集拒绝状态的函数集合
      }
      // 定义要传给executor函数的resolve函数
      const _resolve = (value) => { 			
          if(this.status !== 'PENDING') return 	//如果状态不是等待状态,那么说明状态已经被变更,就return 出去
          this.status = FULFILLED	//将状态变为执行状态
          // 这里使用一个队列来存储回调, 是为了实现规范要求的 then方法可以被同一个Promise调用多次 
          // 如果不使用队列,而是使用一个变量来存储回调, 那么即使多次调用then方法也只会执行一次,因为每次调用then方法会返回一个Promise对象,变量的指向会被每次调用then覆盖掉
          while(this.resolveQueue.length){							
              const FN = this.resolveQueue.shift() // shift可以取出并返回数组第一个元素, 并在数组中删除此元素 , 会改变数组的长度
              FN(value) 		// 执行取出的函数
          }
      }
      // 从失败的回调集合中, 依次取出函数并执行
      const _reject = (value) => {
          if(this.status !== 'PENDING') return
          this.status = REJECTED
          // 这里使用一个队列来存储回调, 是为了实现规范要求的 then方法可以被同一个Promise调用多次 
          // 如果不使用队列,而是使用一个变量来存储回调, 那么即使多次调用then方法也只会执行一次,因为每次调用then方法会返回一个Promise对象,变量的指向会被每次调用then覆盖掉
          while(this.rejectQueue.length){
              const FN = this.rejectQueue.shift()
              FN(value)
          }
       // 当new Promise的时候,会立即执行这个函数, 这就是上文说的我们传入的函数
       executor(_resolve,_reject)
      }
      
      // then方法, 接收两个函数参数, 第一个是执行成功的函数回调, 第二个是执行失败的函数回调
      then(resolveFn,rejectFn){
          this.resolveQueue.push(resolveFn)
          this.rejectQueue.push(rejectFn)
      }
  }
  // new 实例 并运行
  const P1 = new MyPromise((resolve,reject) => {
      setTimeOut(() => {
          resolve(1)
      },1000)
  })
  P1.then(res => {
      console.log(res)
  })
  // 输出:1
  ```

  

  # then的链式调用(个人认为是Promise的核心,以及重点和难点)

  > 我们来看一段then链式调用的代码(标准Promise的代码)
  >
  > ```javascript
  > const P1 = new Promise((resolve,reject) => {
  > 	setTimeout(() => {
  > 		resolve(1)
  > 	},1000)
  > })
  > P1.then(res => {
  >     console.log(res)
  >     return new Promise((resolve,reject) => {
  >        setTimeout(() => {
  >             resolve(2)
  >        },1000)
  >     })
  > }).then(res => {
  >     console.log(res)
  >     resolve(3)
  > })
  > ```
  >
  > ```
  > 代码输出:
  > 	1	
  > 	2	
  > 	3
  > ```

  ##### 我们来分析一番代码

  - then方法需要返回一个Promise对象, 也可以返回一个数值
  - `.then()`方法返回一个Promise, 以确保下次可以继续调用then方法
  - 即使中间返回了一个Promise , 程序的执行顺序也是1-2-3,
  - 我们要等待当前Promise状态变更后, 再执行下一个then收集的回调, 这就要求我们要对then 的返回值进行讨论

  > 我们来重构有关then的代码(我们自己的代码)
  >
  > ```javascript
  > then(resolveFn,rejectFn){
  >     // then() 方法返回一个Promise
  >     return new MyPromise((resolve,reject) => {
  >        // 将then收集的回调函数,重新包装一下,再push进resolve队列,这是为了能获取到回调的返回值
  >         const fulfilledFn = value => {
  >            try{
  >              // 执行第一个Promise的成功回调,并获取返回值
  >              let x = resolveFN(value)
  >              // 返回值如果是Promise,那么等待Promise状态变更,否则直接resolve
  >              x instanceof MyPromise ? x.then(resolve,reject) : resolve(x)
  >            }catch(error){
  >                reject(error)
  >            }
  >         }
  >         // 将包装好的回调 push进当前Promise的成功回调队列中,以保证顺序调用
  >         this.resolveQueue.push(fulfilledFn)
  >         
  >         const rejectedFn = value => {
  >             try{
  >                let x = rejectFn(value)
  >                x instanceof MyPromise? x.then(resolve,reject) : reject(x)
  >             }catch(error){
  >              	reject(error)   
  >             }
  >         }
  >         // 将包装好的回调 push进Promise的失败回调队列中,以保证顺序调用
  >         this.rejectQueue.push(rejectedFn)
  >     })
  >     
  > }
  > ```
  >
  > 

- 现在我们来测试一下我们写好的代码

  - ```javascript
    const P1 = new MyPromise((resolve,reject) => {
    	setTimeout(() => {
            resolve(1)
        } ,1000)
    })
    P1.then(res => {
        console.log(res)
        return new Promise((resolve,reject) => {
            setTimeout(() => {
                resolve(2)
            } ,2000)
        })
    })
    .then(res => {
        console.log(res)
        return 3
    })
    .then(res => {
        console.log(res)
    })
    // 输出
    	1
    	2
    	3
    ```



## 值穿透和状态已经改变的处理

- 值穿透: 根据规范,如果`then()`接收的参数类型不是`function`,那么我们应该忽略它,如果没有忽略,当`then`的回调不是`function`的时候会抛出异常,导致链式调用失败
- 处理状态为`resolve`和`reject`的情况, 我们上面的实现都是对应状态为`pending`的情况,但是有的时候,`resolve`和`reject`再`then`调用的使用就已经执行, (比如: `Promise.resolve().then()`) , 如果这个时候还把`then()`回调push进 `resolve`和`reject`的回调队列中, 那么回调将不会被执行,因此,对于状态已经变为`resolve`和`reject`的情况下,我们就直接进行then回调

```javascript
then(resolveFn,rejectFn){
    // 在函数的开始 , 我们进行参数的校验
    // 这句话的白话文是这样的, 如果resolveFn的类型不是function类型的, 那么将resolveFn指向一个函数,因为在下文要调用函数 , 不然会报错, 无法向下执行
    typeof resolveFn !== 'function' ? resolveFn = value => value : "null"
	// 检查第二个参数
    typeof rejectFn !== 'function' ? rejectFn = reason => {
        throw new Error(reason instanceof Error ? reason.message:reason)
    }:null 
    // then() 方法返回一个Promise
    return new MyPromise((resolve,reject) => {
       // 将then收集的回调函数,重新包装一下,再push进resolve队列,这是为了能获取到回调的返回值
        const fulfilledFn = value => {
           try{
             // 执行第一个Promise的成功回调,并获取返回值
             let x = resolveFN(value)
             // 返回值如果是Promise,那么等待Promise状态变更,否则直接resolve
             x instanceof MyPromise ? x.then(resolve,reject) : resolve(x)
           }catch(error){
               reject(error)
           }
        }
        
        
        const rejectedFn = value => {
            try{
               let x = rejectFn(value)
               x instanceof MyPromise? x.then(resolve,reject) : reject(x)
            }catch(error){
             	reject(error)   
            }
        }
       
        
	    //   我们再这里加入关于状态的处理代码
        switch(this.status){
            case PENDING:
                // 将包装好的回调 push进当前Promise的成功回调队列中,以保证顺序调用
        		this.resolveQueue.push(fulfilledFn)
                 // 将包装好的回调 push进Promise的失败回调队列中,以保证顺序调用
        		this.rejectQueue.push(rejectedFn)
                break
            case FULFILLED:
                fulfilledFn(this._value) // this._value是构造函数的属性,会在初始化实例的时候被赋值	
                break
            case REJECTED:
                rejectedFN(this._value)
                break
        }
    })
    
}
```

## 快了,就快了,我们还要兼容一下同步任务

- 完成了上面`then()`的代码之后, 我们还需要处理一点小细节, 如果我们在new Promise的时候, 构造函数内部如果直接调用`resolve` , 像下面这样:

  - ```javascript
    const P1 = new Promise((resolve, reject) => {
    	resolve(1)
    })
    ```

- 上面的代码会在实例初始化的时候,立刻`resolve` , 这导致我们在下文`then()` 执行的时候, 获取到的值为`undefined`
- Promise 的执行顺序为:`new Promise => then()收集回调 => resolve/reject 执行回调` 这一顺序是建立在`executor`是异步任务的前提上的, 如果`executor`是同步任务,就会出现上面的情况, 获取到的值为`undefinedd`
- 为了兼容这种情况, 我们需要给`resolve/reject`回调包裹一个setTimeout, 众所周知, setTimeout是一个异步操作



> ​	我们来看一下, 比较全面的代码

```javascript
const PENDING = 'PENDING' //等待状态
    const FULFILLED = 'FULFILLED' //执行状态
    const REJECTED = 'REJECTED' //拒绝状态
    // 我们来写一个Promise的类
    class MyPromise {
      constructor(executor) {
        this.status = PENDING //默认状态设置为等待状态
        this.resolveQueue = [] //收集成功状态的函数集合 采用数组队列形式, 可以依次取出任务
        this.rejectQueue = [] // 收集拒绝状态的函数集合
        this._value = undefined

        // 定义要传给executor函数的resolve函数
        const _resolve = value => {
          const run = () => {
            if (this.status !== 'PENDING') return //如果状态不是等待状态,那么说明状态已经被变更,就return 出去
            this.status = FULFILLED //将状态变为执行状态
            this._value = val //保存传入的数据
            // 这里使用一个队列来存储回调, 是为了实现规范要求的 then方法可以被同一个Promise调用多次
            // 如果不使用队列,而是使用一个变量来存储回调, 那么即使多次调用then方法也只会执行一次,因为每次调用then方法会返回一个Promise对象,变量的指向会被每次调用then覆盖掉
            while (this.resolveQueue.length) {
              const FN = this.resolveQueue.shift() // shift可以取出并返回数组第一个元素, 并在数组中删除此元素 , 会改变数组的长度
              FN(value) // 执行取出的函数
            }
          }
          setTimeout(run())
        }
        // 从失败的回调集合中, 依次取出函数并执行
        const _reject = value => {
          const run = () => {
            if (this.status !== 'PENDING') return
            this.status = REJECTED
            this._value = val //保存传入的数据
            // 这里使用一个队列来存储回调, 是为了实现规范要求的 then方法可以被同一个Promise调用多次
            // 如果不使用队列,而是使用一个变量来存储回调, 那么即使多次调用then方法也只会执行一次,因为每次调用then方法会返回一个Promise对象,变量的指向会被每次调用then覆盖掉
            while (this.rejectQueue.length) {
              const FN = this.rejectQueue.shift()
              FN(value)
            }
          }
          setTimeout(run())
        }

        // 当new Promise的时候,会立即执行这个函数, 这就是上文说的我们传入的函数
        executor(_resolve, _reject)
      }
      then(resolveFn, rejectFn) {
        // 在函数的开始 , 我们进行参数的校验
        // 这句话的白话文是这样的, 如果resolveFn的类型不是function类型的, 那么将resolveFn指向一个函数,因为在下文要调用函数 , 不然会报错, 无法向下执行
        typeof resolveFn !== 'function' ? (resolveFn = value => value) : 'null'
        // 检查第二个参数
        typeof rejectFn !== 'function'
          ? (rejectFn = reason => {
              throw new Error(reason instanceof Error ? reason.message : reason)
            })
          : null
        // then() 方法返回一个Promise
        return new MyPromise((resolve, reject) => {
          // 将then收集的回调函数,重新包装一下,再push进resolve队列,这是为了能获取到回调的返回值
          const fulfilledFn = value => {
            try {
              // 执行第一个Promise的成功回调,并获取返回值
              let x = resolveFn(value)
              // 返回值如果是Promise,那么等待Promise状态变更,否则直接resolve
              x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
            } catch (error) {
              reject(error)
            }
          }

          const rejectedFn = value => {
            try {
              let x = rejectFn(value)
              x instanceof MyPromise ? x.then(resolve, reject) : reject(x)
            } catch (error) {
              reject(error)
            }
          }

          //   我们再这里加入关于状态的处理代码
          switch (this.status) {
            case PENDING:
              // 将包装好的回调 push进当前Promise的成功回调队列中,以保证顺序调用
              this.resolveQueue.push(fulfilledFn)
              // 将包装好的回调 push进Promise的失败回调队列中,以保证顺序调用
              this.rejectQueue.push(rejectedFn)
              break
            case FULFILLED:
              fulfilledFn(this._value) // this._value是构造函数的属性,会在初始化实例的时候被赋值
              break
            case REJECTED:
              rejectedFN(this._value)
              break
          }
        })
      }
    }
```

> 我们来测试一下我们的代码
>
> 

```javascript
const P1 = new MyPromise((resolve, reject) => {
      setTimeout(() => {
        resolve(1)
      }, 1000)
    })
    P1.then(res => {
      console.log(res)
      return 2
    })
      .then()
      .then(res => {
        console.log(res)
        return new MyPromise((resolve, reject) => {
          setTimeout(() => {
            resolve(3)
          }, 2000)
        })
      })
      .then(res => {
        console.log(res)
        throw new Error('reject测试')
      })
      .then(undefined, error => {
        console.log(error)
      })

	// 代码输出:
			1
			2
			3 (2秒后)
		    Error: reject测试
```

## 大功告成,我们手写了Promise, 并且可以跑通,难道不值得鼓舞人心吗, 答案是肯定的

# Promise还内置了几个静态方法

> 有关静态方法的知识,我在此放上阮一峰大神的链接, 会比我们都写得好,而且全面
>
> [ES6阮一峰]: https://es6.ruanyifeng.com/?search=mutation&amp;x=12&amp;y=1#docs/promise#Promise-prototype-finally
>
> 