# 由concat方法引发的学习

在查阅资料的时候, 偶然间看到MDN上的一段代码, 引起了我的兴趣, 是这样一段代码

```javascript
function combin(){
	let arr = [].concat.apply([],arguments)
	return Array.from(new Set(arr))
}
var m=[1,2,2], n=[2,3,3]
console.log(combin(m,n))
// 输出是[1,2,3]
```

似乎还是学艺不精, 起先我不能理解`[].concat.apply([],arguments)`这段代码的详细意思, 尽管我知道`apply`是改变this指向的

经过网上查阅文档, 我得知以下:

- ​	`[].concat.apply([],arguments)`  concat前面的[]  只是能调用数组的`concat`方法而已, 而apply这个方法是提供一个新的this值给调用它的函数/方法,  这么一来, 就是第二个[]来调用'concat'方法, 然后与`arguments`参数组成一个新的数组

