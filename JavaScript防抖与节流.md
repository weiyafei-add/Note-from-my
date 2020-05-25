# JavaScript防抖与节流

### 防抖: 任务频繁触发的情况下, 只有任务触发的间隔超过指定间隔的时候,任务才会执行

防抖常常用于处理页面中搜索框以及向服务器发送请求的按钮 , 以防止用户恶意的快速点击, 以及每次输入都需要向服务器发送请求, 造成服务器的资源浪费, 更为严重, 会出现BUG, 造成不可预期的渲染

### 	页面中有一个保存按钮, 点击它会向服务器发起一条API请求,保存当前表单的数据

```javascript
function apiRequest () {// 发送请求}

document.getElementById('.btn').addEventListenr('click',function(){
    setTimeout(() => {
         apiRequest()
    },200)
})
  
    // 首先我们能想到的是设定一个定时器, 200ms后发起请求
    
```

上面的代码还是会有问题, 因为每一个点击都还是绑定一个计时器, 假如我们快速点击5下, 则200ms后5个定时器启动, 依旧有5条请求, 这不符合我们的预想

我们应该在每个定时器创建钱, 检查定时器是否已经启动过, 如果有,则清理已有的定时器

```javascript
function apiRequest () {// 发送请求}

var timer
document.getElementById('.btn').addEventListenr('click',function(){
   if(timer) clearTimeout(timer)
   timer = setTimeout(() => {
        apiRequest()
    },200)
})
```

现在代码已经达到我们预期的效果, 我们来提取这段代码以便复用

```javascript
function apiRequest(){}

// 防抖函数
function debounce(func,delay){
    var timer				//	我们返回了一个函数, timer会因为闭包一直存活者
    return function(){
        if(timer){
            clearTimeout(timer)
        }
        timer = setTimeout(() => {
            func()
        },delay)
    }
}
document.getElementById('btn').addEventListener('click',debounce(apiRequest,200))
```

尽管如此, 还是有些不完美的地方, 我们无法在防抖函数中给func函数传入参数, 我们可以这样做

```javascript
function apiRequest(){}

// 防抖函数
function debounce(func,delay){
    var timer				//	我们返回了一个函数, timer会因为闭包一直存活者
    return function(...args){
        if(timer){
            clearTimeout(timer)
        }
        timer = setTimeout(() => {
            func.call(this,...agrs)
        },delay)
    }
}
document.getElementById('btn').addEventListener('click',debounce(apiRequest,200))
```

现在 函数达到我们预期, 多次点击会被清除, 只会在200ms之后执行,但是, 这样又对正常的用户造成些许的惩罚, 用户只点击一次也会被200ms之后执行, 显然我们并不想这样, 以上的例子适合搜索框事件,如果是提交按钮, 我们应该这样写

```javascript
function apiRequest(){}

function debounce(func,delay){
    var timer
    var run = false				//	我们在这里加入一个标志
    return function(...args){
        if(!run){				// 如果run为false则执行
            func.apply(this,args)
        }
        run = true				// 执行完一次后将run置为true
        if(timer)clearTimeout(timer)
        timer = setTimeout(() => {
            run = false			// 我们在延迟delay毫秒后将run设置为fasle, 否则func之后执行一次
        },delay)
    }
}

```



> 防抖函数是一个高阶函数, 如果我们想给func传参, 我们需要这样来写
>
> ```
> var debounceFun = debounce(func,200)
> 
> var searchText = 'javaScript'
> 
> debounceFun(searchText ) 	
> ```
>
> 这样,我们在argments对象中就可以获取到searchText这个参数

# 节流

### 节流: 指定时间间隔内只会执行一次任务

节流的方法又两种

1. 利用时间戳的来计算, 保证一定时间内执行

   ```javascript
   // 一段获取当前时间的函数, 在规定的时间内获取一次, 不能频繁获取
   function getDate(){
       console.log(new Date())
   }
   function throttle(func,delay){
       let old = 0		//设定一个旧的时间
       return function(){
           let now = new Date().valueOf()
           if(now - old > delay){		// 如果间隔的时间差大于delay就执行这个函数, 显然,第一次是肯定会大于的, 在delay期间的所有点击都小于delay, 会被忽略
               func.apply(this)
               old = now		//	func执行完后, 要把now赋值给old
           }
       }
   }
   ```

2. 用一个标记来实现, 结合定时器

   ```javascript
   // 闭包中返回一个标记
   function getDate(){
       console.log(new Date())
   }
   function throttle(func,delay){
      let run = true
      return function(){
          if(!run) return 	// 如果run不是true则return
          run = false		// 如果不把run置为false, 那么会一直增加定时器
          setTimeout(() => {
              func.apply(this)
              run = true
          },delay)
      }
   }
   ```

   > 节流在工作中的应用:
   >
   > 1. ​	懒加载要监听计算滚动的位置, 使用节流按一定时间的频率获取
   > 2. ​     用户点击提交按钮, 假设我们知道接口大致的返回时间的情况下, 我们使用节流, 只允许一定时间内点击一次

   在某些特定的工作场景, 我们就可以使用防抖与节流来减少不必要的损耗

   面试官会问: 为什么说上面的场景不节制会造成过多损耗呢

   再来写一篇有关重绘与回流的文章

   # 重绘与回流

   - 重绘(repaint): 当元素样式的改变不影响布局时, 浏览器将使用重绘对元素进行更新,此时由于只需要UI层面的重新像素绘制, 因此损耗较少
     - 常见的重绘操作有:
       - 改变元素颜色
       - 改变元素背景色
       - 等等不修改元素位置尺寸以及内容的
   - 回流(reflow): 也叫重排. 当元素的尺寸,结构或者触发某些属性时, 浏览器会重新渲染页面, 称为回流. 此时, 浏览器需要重新进行计算, 计算后还需要重新页面布局, 因此是较重的操作.
     - 常见的回流操作有:
       - 页面初次渲染
       - 浏览器窗口大小改变
       - 元素尺寸/位置/内容发生改变
       - 元素字体大小变化
       - 添加或删除可见的DOM元素
       - 激活CSS伪类 (:hover....)
     - **重点: 回流必定会触发重绘, 重绘不一定会触发回流. 重绘的开销较小, 回流的代价较高**
     - 在工作中我们要如何避免大量使用重绘与回流呢
       - 避免频繁操作样式, 可汇总后统一一次修改
       - 尽量使用class进行样式修改, 而不是直接操作样式
       - 减少DOM的操作, 可使用字符串一次性插入

