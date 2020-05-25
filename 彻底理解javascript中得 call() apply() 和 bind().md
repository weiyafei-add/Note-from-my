# 彻底理解javascript中得 call() apply() 和 bind()

> 是的, this的指向有时候会将你难倒
>
> `.call`、`.apply`、`.bind`和 `new` 关键字是Javascript比较令人困惑的地方
>
> call() apply() 和 bind() 都可以改变this的指向, 以完成正确的代码流程
>
> `this`关键字允许在调用函数或方法时决定哪个对象应该是焦点 , 我们希望能够在不同的上下文或在不同的对象中复用函数或者方法

#### 正文

如何判断`this`指向哪里 ,  一个最重要的问题就是 **这个函数在哪里被调用** 

> 现在让我们来看一段代码



```javascript
function greet(name){
	alset(`hello,my name is ${name}`)
}
```

- 很显然, 如果只给一个函数的定义, 是无法知道函数会打印什么内容
- 想知道函数打印了什么,就需要看函数被调用的过程

```javascript
greet('bill')
```

> 判断`this`关键字引用(我理解为指向)什么 也是同样的道理, 它会随着函数调用方式的变化而变化

现在我们知道为了判断`this`的引用(指向)必须先看函数的定义 , 在实际地查看函数定义时, 我们设立了四条规则:

1. 隐式绑定
2. 显式绑定
3. new绑定
4. window绑定

#### 隐式绑定

我们的目标是查看使用`this`关键字的函数定义, 并判断`this`的指向.

我们有这样一个对象

```javascript
const user = {
 		name:'tony',
 		age:26,
 		greet(){
 			alert(`hello, my name is ${this.name}`)
 		}
}
```

现在, 如果你要调用`user`对象上的`greet`方法, 你就需要这样来调用

```
user.greet()
```

> 这就把我们带到了隐式绑定规则的主要关键点. 为了判断`this`关键字的指向 , **函数在调用时先看一看点号左侧**  如果有点就查看点左侧的对象, 这个对象就时`this`关键字的指向
>
> 很显然 , 上面代码中, `this`指向的时`user`对象

这里引用一段国外大牛的博客文章

*Whenever a function is called, we must look at the immediate left side of the brackets / parentheses “()”. If on the left side of the parentheses we can see a reference, then the value of “this” passed to the function call is exactly of which that object belongs to, otherwise it is the global object. 

译为 :  **每当调用一个函数时，我们必须查看括号 / 圆括号“()”的直接左侧。 如果在括号的左侧我们可以看到一个引用，那么传递给函数调用的“ this”的值正是该对象所属的值，否则它就是全局对象**

如下代码:

​	

```javascript
function bar() {
    alert(this);
}
bar(); // 这里的this指向的就是全局对象, 非严格模式下是window, 严格模式下是 undefined

var foo = {
    baz: function() {
        alert(this);
    }
}
foo.baz(); // 正如上面所述,如果在括号的左侧我们可以看到一个引用, 也就是`对象.方法`的形式,this就指向foo
```

[这里放上博客链接]: http://davidshariff.com/blog/javascript-this-keyword/#first-article

言归正文, user在'点号左侧'意味着`this`指向了`user`对象 , 所以就好像在`greet`方法内部Javascript解释器把`this`变为了`user`

```javascript
const user = {
 		name:'tony',
 		age:26,
 		greet(){
 			//	alert(`hello, my name is ${this.name}`)
            alert(`hello, my name is ${user.name}`)
 		}
}
```

现在, 我们看一个稍微高级一点的例子, 我们的对象不仅要拥有`name`,`age`和`greet`属性, 还要被添加一个`maother`属性, 并且也拥有`name`和`greet`属性

```javascript
const user = {
	name:'tony',
    age:26,
    greet(){
        alert(`hello, my name is ${this.name}`)
    },
    mother:{
        name:'david',
        age:25,
        greet(){
           alert(`hello, my name is ${this.name}`)
        }
    }
}
```

现在我们可以运行以下代码得知:

```
user.greet()	// hello, my name is tony
user.mother.greet() // hello,my name is david
```

每当判断`this`的指向时, 我们都需要查看调用过程, 并确认`点的左侧`是什么,或者引用上文的一段话

> 每当调用一个函数时，我们必须查看括号 / 圆括号“()”的直接左侧

大约在80%的情况下在`点的左侧`都会有一个对象, 这就是为什么在判断`this`指向时查看`点的左侧`时你要做的第一件事, 如果没有点, 那么我们就需要下一条规则

#### 显式绑定 (call,apply,bind)

如果`greet`函数不是`user`对象的函数, 而是一个独立的函数

```javascript
function greet(){
	alert(`Hello,my name is ${this.name}`)
}
const user = {
 name:'tony',
 age:26
}
```

我们知道为了判断`this`的指向, 我们首先必须查看这个函数的调用位置, 现在就引出了一个问题, 我们怎么能让`greet`方法调用的时候将`this`指向`user`对象 ? 我们不能再像之前那样简单的使用`user.greet()`,因为`user`并没有`greet`方法, 再Javascript中, 每个函数都包含了一个能让你恰好解决这个问题的方法, 这个方法的名字叫做`call`

> "call"是每个函数都有的一个方法, 它允许你在调用函数时为函数指定上下文

考虑到这一点, 用下面的代码可以在调用`greet`时用`user`做上下文

```javascript
greet.call(user)
```

再次强调一遍: **`call`是每个函数都有的一个属性,并且传递给它的第一个参数会作为函数被调用时的上下文. 换句话说, `this`将会指向传递给`call`的第一个参数**

这就是第二条规则的基础(显式绑定), 因为我们明确地(使用`.call`)指定了`this`的指向.

现在让我们对`greet`的方法做一点小小的改动. 假如我们像传递一些参数呢? 不仅提示他们的名字, 还要提示他们知道的语言 , 比如:

```javascript
function greet(lang1,lang2,lang3){
	alert(`Hello, my name is ${this.name}, and i know ${lang1},${lang1},${lang1}`)
}
```

现在, 让我们来书写上面的代码实现, 需要在`call`的指定上下文(第一个参数)后一个一个地传入

```javascript
function greet(lang1,lang2,lang3){
	alert(`Hello, my name is ${this.name}, and i know ${lang1},${lang2},${lang3}`)
}
const user = {
    name:'tony',
    age:26
}
const languages = ['javascript','Ruby','Python']
greet.call(user,languages[0],languages[1],languages[2])
```

显然, 方法奏效了, 不过你可能注意到, 必须一个一个传递`languages`数组的元素, 这样有些繁琐, 如果我们能把整个数组作为第二个参数并让Javascript为我们展开就好了, 

有个好的消息, `apply`函数就是为我们做这样的事情. 

`apply`和`call`本质相同, 但不是一个一个传递参数, 你可以用数组传参而且`apply`会在函数中为你自动展开

那么现在用`apply`函数来实现一下

```javascript
function greet(lang1,lang2,lang3){
	alert(`Hello, my name is ${this.name}, and i know ${lang1},${lang2},${lang3}`)
}
const user = {
    name:'tony',
    age:26
}
const languages = ['javascript','Ruby','Python']
greet.apply(user,languages)
```

到目前为止: 我们学习了`call`和`apply`的"显式绑定"规则, 用此规则调用的方法可以让你指定`this`在方法内的指向.

补充几点:

> `call()`提供新的this值给当前调用的函数/方法
>
> 如果使用`call()`方法调用函数, 并且没有传入第一个参数, 那么`this`的值将会指向全局对象
>
> 例如:
>
> ```javascript
> greet.call() // this指向全局
> ```
>
> 如果调用`call()` 的函数不处于严格模式下, `call()`的第一个参数指定为`null`或者`undefined`时 会自动替换为`指向全局对象`
>
> 

**现在就到了bind() 方法了**

`bind()`和`call()`基本完全相同, 主要区别在于 `bind()`方法不会立刻调用函数, 而是返回一个能以后调用的新函数, 我们来用`bind()`方法书写上面的代码

```javascript
function greet(lang1,lang2,lang3){
	alert(`Hello, my name is ${this.name}, and i know ${lang1},${lang2},${lang3}`)
}
const user = {
    name:'tony',
    age:26
}
const languages = ['javascript','Ruby','Python']

const newFn = greet.bind(user,languages)

newFn()
```

#### new绑定

这个就比较浅显易懂了, 其实每当用`new`调用函数时, Javascript解释器都会在底层创建一个全新的对象并把这个对象当作this, 每当用`new`调用函数时, `this`会自然地引用解释器创建的新对象.

这里我来描述一下new的时候, 底层发生的过程:

> 1. 首先声明一个中间对象
> 2. 将该中间对象的原型指向构造函数的原型
> 3. 将构造函数的`this`, 指向该中间对象
> 4. 返回该中间对象, 即返回实例对象



```javascript
function Person(name,age){
    /*
    	JavaScript会在底层创建一个新对象`this`, 它会代理不在Person原型链上的属性,比如this.name 和 this.age 就不在Person原型链上.
    	如果一个函数用`new`关键字调用Pserson, this就会指向解释器创建的新对象
    	
    */
	this.name = name;
    this.age = age
}
const P1 = new Person('pony',20)
```

#### window绑定

**其实非常简单**

```javascript
function sayAge(){
	console.log(`hi,my age is ${this.age}`)
}
const user = {
    name:'pony',
    age:20
}
```

不出意外的话, 你的浏览器控制台会打印出如下

```
hi, my age is undefined
```

因为`this.age`时undefined. 实际上这是因为点的左侧没有任何东西, 我们也没有使用`.call`, `.apply`, `.bind`或者`new`关键字, JavaScript会将`this`默认指向window对象, 这意味着函数会查找window对象上面的age属性

这就是第 4 条规则为什么是 `window 绑定` 的原因。如果其它规则都没满足，JavaScript就会默认 `this` 指向 `window` 对象。

#### 尾

因此, 将所以规则付诸实践, 每当我在函数内部看到`this`关键字时, 这些就是我为了判断它的指向而采取的步骤:

1. 查看函数在哪里被调用
2. 点的左侧有没有对象? 如果有,它就是`this`的指向, 如果没有,那么继续第三步
3. 该函数是不是`call` `apply` `bind`调用的 ? 如果时,它会显式地指明`this`地指向,如果不是,那么继续第4步
4. 该函数是不是`new`调用地, 如果时,`this`的指向就是JavaScript解释器新创建的对象, 如果不是,继续第5步
5. 是否在严格模式下, 如果是, `this`就是`undefined`,如果不是,继续第6步
6. this会指向window

