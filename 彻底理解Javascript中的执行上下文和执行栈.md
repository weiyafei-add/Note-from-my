# 彻底理解Javascript中的执行上下文和执行栈

你必须知道Javascript程序内部是如何执行的, 理解执行上下文和执行栈对于理解其他Javascript概念至关重要, 因为他们彼此关联紧密, 比如: (变量声明提升, 作用域和闭包)

> 通常, 大多数人更喜欢把执行栈称为`函数调用栈`, 这样似乎更加形象一些

####  什么是执行上下文

> 执行上下文是评估和执行JavaScript代码的环境的抽象概念, 每当JavaScript代码在运行的时候, 它都是在执行上下文中运行
>
> `我把执行上下文理解为一张a4纸, 我们的代码就书写在a4纸上,从页头执行到页脚`

#### 执行上下文的类型

- 全局执行上下文 =>  这是默认或者说基础的上下文, 任何不在函数内部的代码都在全局上下文中, 它会执行两件事

  1. 创建一个全局的window对象(浏览器中运行)
  2. 设置`this`的值等于这个全局对象, 一个程序中只会有一个全局对象

  > 我们可以理解为整张a4纸, 从上到下,属于全局执行上下文

- 函数执行上下文  =>  每当一个函数被调用时, 都会该函数创建一个新的上下文, 每个函数都有自己的执行上下文, 不过是在函数被调用的时候创建的. 

  > 我们可以理解为整张a4纸中间的自然段, 把自然段想象为一个个函数

- Eval 函数执行上下文  =>  涉及不多, 这里不会讨论

#### 执行栈(函数调用栈)

函数调用栈是一种拥有LIFO(后进先出)数据结构的栈, 用来存储代码运行时创建的所有执行上下文

当我们的JavaScript代码开始执行的时候,  JavaScript引擎会创建一个全局的执行上下文并且压入当前执行栈 . 每当引擎遇到一个函数调用时, 它会为该函数创建一个新的执行上下文并压入栈的顶部

引擎会执行位于栈顶的函数. 当该函数执行完毕时, 执行上下文(就是该函数)从栈中弹出, 控制流程到达当前执行栈的下一个上下文(也就是下一个函数)

我们来书写一段代码, 加以理解:

```javascript
function first(){
    console.log('hello, my name is first')
    second()
}
function second(){
    console.log('my name is second')
}
first()
console.log('done')
```

现在, 我们来看一下JavaScript引擎是如何管理执行上下文的

1. 首先, JavaScript引擎遇到这段代码(假如这是一段完整的js代码), 引擎会先创建一个全局上下文并压入执行栈中(一直会是这样)
2. <img src="C:\Users\weiyafei\Documents\MyNote\Saved Pictures\流程1.png" style="zoom:50%;" />
3. 引擎遇到`first()`这行代码, 意味着要执行该函数, JavaScript引擎会为该函数创建一个新的执行上下文并把它压入当前执行栈的顶部
4. <img src="C:\Users\weiyafei\Documents\MyNote\Saved Pictures\流程2.png" style="zoom:50%;" />
5. 此时控制台会打印 `hello, my name is first` , 然后引擎遇到了`second()`函数调用
6. JavaScript引擎为`second()`函数创建一个新的执行上下文并把它压入当前执行栈的顶部, 此时`first()`函数执行上下文依然在执行栈中, `second`函数开始执行, 此时控制台打印出: `my name is second`
7. <img src="C:\Users\weiyafei\Documents\MyNote\Saved Pictures\流程3.png" style="zoom:50%;" />
8. `second`函数执行完毕, 它的执行上下文会从当前调用栈弹出, 并且控制流程会到达下一个执行上下文,  即`first()`函数的执行上下文
9. <img src="C:\Users\weiyafei\Documents\MyNote\Saved Pictures\流程4.png" style="zoom:50%;" />
10. 当first执行完毕后, 它的执行上下文会从栈中弹出, 控制流程到达最底部的全局执行上下文, 一旦所有代码都指向完毕, JavaScript引擎会从当前栈中弹出全局指向上下文
11. 

<img src="C:\Users\weiyafei\Documents\MyNote\Saved Pictures\流程1.png" style="zoom:50%;" />

> 这样一来,我们就很直观的能理解JavaScript是怎样管理上下文的

#### 怎么创建执行上下文

创建执行上下文分为两个阶段:

1. **创建阶段**
2. **执行阶段**

让我们先来看一看**创建阶段**:

在JavaScript代码执行之前, 执行上下文将经历创建阶段, 在创建阶段会发生三件事:

1. 确定**this**值的指向
2. 创建**词法环境**组件
3. 创建**变量环境**组件

执行上下文在概念上表示如下:

```
ExecutionContext = {
	ThisBinding = <this value>		// this值得指向
	LexicalEnvironment = {....}		// 词法环境
	VariableEnvironment = {...}		// 变量环境
}
```

**`this`的指向**

- 在全局执行上下文中, `this`的值指向全局对象. (在浏览器环境中,this指向window对象)
- 在函数指向上下文中, `this`的值取决于该函数时如何被调用的, 如果它被一个引用对象调用, 那么`this`会被设置为那个对象, 否则`this`的值被设置为全局对象或者`undefined`(严格模式下)

```javascript
let foo = {
	baz:function(){
		console.log(this)
	}
}
foo.baz()	// 此时this的指向为:foo对象

let bar = foo.baz
bar()		// 此时this的指向为window对象
```

**词法环境**

词法环境是一种持有**标识符----变量映射**的结构

> 这里的**标识符**指的是变量/函数的名字, 而**变量**是对实际对象[包含函数对象类型]或原始数据的引用

词法环境**内部**有两个:

1. 环境记录器
2. 一个外部环境的引用

- **环境记录器**是存储变量和函数声明的实际位置
- **外部环境的引用**意味着它可以访问其父级词法环境(作用域)

**词法环境**有两种类型

- **全局环境**   (在全局执行上下文中)是没有外部环境引用的词法环境, 全局环境的外部环境引用时**null** , (它拥有内建的 Object/ Array / 等、在环境记录器内的原型函数(关联全局对象, 比如window对象) 还有任何用户定义的全局变量, 并且`this`的值指向全局对象
- **函数环境**    函数内部用户定义的变量存储在**环境记录器**中, 并且引用的外部环境可能是全局环境, 也可能是包含此内部函数的外部函数

**环境记录器也有两种类型**

1. **声明式环境记录器**存储变量、函数和参数
2. **对象环境记录器**用来定义出现在**全局上下文**中的变量和函数的关系

简而言之:

- 在**全局环境**中 , 环境记录器是对象环境记录器
- 在**函数环境**中 , 环境记录器是声明式环境记录器

**注意**=> 对于**函数环境** , **声明式环境记录器**还包含了一个传递给函数的`arguments`对象 ( 此对象存储索引和参数的映射 ) 和传递给函数的参数的**length**

抽象地将 , 词法环境在伪代码中看起来像这样: 

```
// 全局执行上下文
GlobalExectionContext = {
	//词法环境
	LexicalEnvironment:{
	// 环境记录器
    	EnvironmentRecord:{
    		Type:'Object'	//上面我们知道, 全局环境中, 环境记录器是对象环境记录器
    		// 这里绑定标识符
    	}
    	// 外部环境的引用
    	outer:<null>
	}
}
// 函数执行上下文
FunctionExectionContext = {
	//词法环境
	LexicalEnvironment: {
		//环境记录器
		EnvironmentRecord: {
			Type:'Declarative' // 上面我们知道, 函数环境中, 环境记录器是声明式环境记录器
			// 这里绑定标识符	
		}
		// 外部环境的引用: 全局上下文引用 或者外部函数上下文引用
		outer: <Global or outer function environment reference>
		
	}
}
```

#### 变量环境

它同样是一个词法环境, 其环境记录器持有**变量声明语句**在执行上下文中创建的绑定关系

如上所述  变量环境也是一个词法环境, 所以它有着上面定义的词法环境的所有属性

在**ES6**中, **词法环境**组件和**变量环境的**不同是:

1. 词法环境被用来存储函数声明和变量(`let`和`const`)绑定
2. 变量环境只用来存储`var`变量绑定

我们来看样例代码来理解上上面的概念

```javascript
let a = 20
const b = 30
var c

function multiply(e,f){
    var g = 20
    return e*f*g
}
c = multiply(20,30)
```

执行上下文看起来像这样:

```javascript
GlobalExectionContext = {				// 全局执行上下文 

	ThisBinding: <Global Object> 		// this绑定:全局对象
	
	LexicalEnvironment: {				// 词法环境
		EnvironmentRecord: {			// 环境记录器
			Type:'Object'
			// 这里绑定标识符
			a: <uninitialized>			// a:未初始化
			b: <uninitialized>			// b:未初始化
			multiply: <func>			// multiply是一个函数类型
		}
		outer:<null>					// 外部引用是null
	},
    
    VariableEnvironment:{				//变量环境
    	EnvironmentRecord:{				//环境记录器
    		Type:'Object'
    		//这里绑定标识符
            c:undefined					// c:初始化为undefined
    	}
        outer: <null>					// 外部引用是null
    }
}

FunctionExectionContext = {
    ThisBinding: <Global Object>	       // this的绑定时全局对象 (就上面代码段而言)
    
    LexicalEnvironment:{				    // 词法环境
    	EnvironmentRecord:{					// 环境记录器, 类型: 声明式
    	    Type:'Declarative'
    		//这里绑定标识符
    	   Arguments:{0:20,1:30,length:2}	// 上面我们知道, 对于函数环境, 声明式函数记录器还包含了一个传递给函数的Arguments对象
		}
        outer:<GlobalExectionEnvironment> 	// 外部引用: 此时是全局执行上下文
	},
    
    VariableEnvironment:{					// 变量环境
        EnvironmentRecord:{
            Type:'Declarative'
            //这里绑定标识符
            g:undefined
        },
        outer:<GlobalLexicalEnvironment>	
    }
}
```

**注意**: 只有遇到调用函数`multiply`时, 函数执行上下文才会被创建

可能你以及注意到`let`和`count`定义的变量并没有关联任何值, 但`var`定义的变量被设成了`undefined`

这是因为在创建阶段时, 引擎检查代码找出变量和函数声明, 虽然函数声明完全存储在环境中, 但是变量最初设置为`undefined`(`var`情况下) , 或者未初始化(`let和const`情况下)

这就是为什么你可以在声明之前访问`var`定义的变量(虽然是`undefined`) , 但是在声明之前访问`let`和`const`的变量会得到一个引用错误

这就是我们说的变量声明提升

#### 执行阶段

这是整篇文章中最简单的部分, 在此阶段, 完成对所有的变量的分配, 最后执行代码

**注意** => 在执行阶段, 如果JavaScript引擎不能再源码中声明的实际位置找到`let`变量的值, 它会被赋值为`undefined`

<img src="C:\Users\weiyafei\Documents\MyNote\Saved Pictures\创建执行上下文.png" style="zoom:80%;" />























