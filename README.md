## interview-js-184

### 1. 简述JavaScript中map和foreach的区别？

JavaScript中map和foreach共同点：

1.都是循环遍历数组中的每一项。
2.forEach()和map()里面每一次执行匿名函数都支持3个参数：数组中的当前项item,当前项的索引index,原始数组input。
3.匿名函数中的this都是指Window。
4.只能遍历数组。

JavaScript中map和foreach的不同点：

1.forEach()：没有返回值，即返回值为undefined
理论上这个方法是没有返回值的，仅仅是遍历数组中的每一项，不对原来数组进行修改；但是可以自己通过数组的索引来修改原来的数组，或当数组项为对象时修改对象中的值；
2.map()：有返回值，可以return 出来。
区别：map的回调函数中支持return返回值；return的是啥，相当于把数组中的这一项变为啥（并不影响原来的数组，只是相当于把原数组克隆一份，把克隆的这一份的数组中的对应项改变了）；
1.forEach()返回值是undefined，不可以链式调用。
2.map()返回一个新数组，原数组不会改变。

### 2. 解释下JavaScript中this是如何工作的？

实际上this 是在运行时进行绑定的，并不是在编写时绑定，它的上下文取决于函数调 用时的各种条件。this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。 总结: 函数被调用时发生 this 绑定，this 指向什么完全取决于函数在哪里被调用。 一、this 的绑定规则 this 一共有 4 中绑定规则，接下来一一介绍每种规则的解释和规则直接的优先级 默认绑定（严格/非严格模式） 隐式绑定 显式绑定 new 绑定 1.1 默认绑定（严格/非严格模式） 独立函数调用： 独立函数调用时 this 使用默认绑定规则，默认绑定规则下 this 指向 window（全局对象）。 严格模式下： this 无法使用默认绑定，this 会绑定到 undefined。

一、问题的由来
学懂 JavaScript 语言，一个标志就是理解下面两种写法，可能有不一样的结果。
```
var obj = {
	foo: function () {}
};
var foo = obj.foo;
// 写法一
obj.foo()
// 写法二
foo()
上面代码中，虽然obj.foo和foo指向同一个函数，但是执行结果可能不一样。请看下面的例子。

var obj = {
  foo: function () { console.log(this.bar) },
  bar: 1
};
var foo = obj.foo;
var bar = 2;
obj.foo() // 1
foo() // 2
```
这种差异的原因，就在于函数体内部使用了this关键字。很多教科书会告诉你，this指的是函数运行时所在的环境。对于obj.foo()来说，foo运行在obj环境，所以this指向obj；对于foo()来说，foo运行在全局环境，所以this指向全局环境。所以，两者的运行结果不一样。
这种解释没错，但是教科书往往不告诉你，为什么会这样？也就是说，函数的运行环境到底是怎么决定的？举例来说，为什么obj.foo()就是在obj环境执行，而一旦var foo = obj.foo，foo()就变成在全局环境执行？
本文就来解释 JavaScript 这样处理的原理。理解了这一点，你就会彻底理解this的作用。

二、内存的数据结构
JavaScript 语言之所以有this的设计，跟内存里面的数据结构有关系。
var obj = { foo: 5 };
上面的代码将一个对象赋值给变量obj。JavaScript 引擎会先在内存里面，生成一个对象{ foo: 5 }，然后把这个对象的内存地址赋值给变量obj。
也就是说，变量obj是一个地址（reference）。后面如果要读取obj.foo，引擎先从obj拿到内存地址，然后再从该地址读出原始的对象，返回它的foo属性。
原始的对象以字典结构保存，每一个属性名都对应一个属性描述对象。举例来说，上面例子的foo属性，实际上是以下面的形式保存的。
```
{
  foo: {
    [[value]]: 5
    [[writable]]: true
    [[enumerable]]: true
    [[configurable]]: true
  }
}
```
注意，foo属性的值保存在属性描述对象的value属性里面。

三、函数
这样的结构是很清晰的，问题在于属性的值可能是一个函数。
var obj = { foo: function () {} };
这时，引擎会将函数单独保存在内存中，然后再将函数的地址赋值给foo属性的value属性。
```
{
  foo: {
  	[[value]]: “函数的地址”
  }
}
```
由于函数是一个单独的值，所以它可以在不同的环境（上下文）执行。
```
var f = function () {};
var obj = { f: f };
// 单独执行
f()
// obj 环境执行
obj.f()
```
四、环境变量
JavaScript 允许在函数体内部，引用当前环境的其他变量。
```
var f = function () {
	console.log(x);
};
```
上面代码中，函数体里面使用了变量x。该变量由运行环境提供。
现在问题就来了，由于函数可以在不同的运行环境执行，所以需要有一种机制，能够在函数体内部获得当前的运行环境（context）。所以，this就出现了，它的设计目的就是在函数体内部，指代函数当前的运行环境。
```
var f = function () {
	console.log(this.x);
}
```
上面代码中，函数体里面的this.x就是指当前运行环境的x。
```
var f = function () {
	console.log(this.x);
}

var x = 1;
var obj = {
  f: f,
  x: 2,
};
  // 单独执行
f() // 1
// obj 环境执行
obj.f() // 2
```
上面代码中，函数f在全局环境执行，this.x指向全局环境的x。
在obj环境执行，this.x指向obj.x。
回到本文开头提出的问题，obj.foo()是通过obj找到foo，所以就是在obj环境执行。一旦var foo = obj.foo，变量foo就直接指向函数本身，所以foo()就变成在全局环境执行。
### 3. 简述异步线程，轮询机制，宏任务微任务？

同步任务： 指的是在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。
异步任务： 指的是不进入主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

1. 同步和异步任务在不同的执行"场所"，同步的进入主线程，异步的进入Event Table执行并注册函数。
2. 当指定的异步事情完成时，Event Table会将这个函数移入Event Queue。
3. 主线程内的任务执行完毕为空，会去Event Queue读取对应的函数，推入主线程执行。
4. js引擎的monitoring process进程会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event
Queue那里检查是否有等待被调用的函数。上述过程会不断重复，也就是常说的Event Loop(事件循环也可以叫事件轮询)。


宏任务（macrotask ）和微任务（microtask ）
macrotask 和 microtask 表示异步任务的两种分类。
在挂起任务时，JS 引擎会将所有任务按照类别分到这两个队列中，首先在 macrotask 的队列（这个队列也被叫做 task queue）中取出第一个任务，执行完毕后取出 microtask 队列中的所有任务顺序执行；之后再取 macrotask 任务，周而复始，直至两个队列的任务都取完。
JavaScript 执行机制：
主线程任务——>微任务——>宏任务
如果宏任务里还有微任就继续执行宏任务里的微任务，如果宏任务中的微任务中还有宏任务就在依次进行

主线程任务——>微任务——>宏任务——>宏任务里的微任务——>宏任务里的微任务中的宏任务——>知道任务全部完成

### 4. JavaScript阻止事件冒泡的方法？

比如现在有一个子盒子和一个父盒子，子盒子和父盒子二者都有点击事件，但是此时，当我们点击子盒子时，只想让子盒子显示点击事件。这里我们就要用到阻止事件冒泡的方法来隔断父盒子的事件显示。
我们应该怎样阻断父盒子的点击事件呢？
可以直接在子盒子内部的点击事件里面添加stopPropagation()方法
如下所示：
```
son.addEventListener('click',function(e){
	alert('son');
	e.stopPropagation();
},false)
```
但是需要注意的是：这个方法也有兼容性问题，在低版本浏览器中（IE 6-8 ）通常是利用事件对象cancelBubble属性来操作的。即直接在相应的点击事件里面添加：
e.cancelBubble = true;
如果我们想要解决这种兼容性问题，就可以采用下述方法：
```
if(e && e.stopPropagation){
	e.stopPropagation();
}else{
	window.event.cancelBubble = true;
}
```
### 5. Javascript 怎样判断array 和 object ?

var obj = {"k1":"v1"};
var arr = [1,2];
1：通过isArray方法
使用方法： Array.isArray(obj); //obj是检测的对象

2：通过instanceof运算符来判断
instanceof运算符左边是子对象（待测对象），右边是父构造函数（这里是Array），
具体代码：
console.log("对象的结果："+(obj instanceof Array));
console.log("数组的结果："+(arr instanceof Array));

3:使用isPrototypeOf()函数
原理：检测一个对象是否是Array的原型（或处于原型链中，不但可检测直接父对象，还可检测整个原型链上的所有父对象）
使用方法: parent.isPrototypeOf(child)来检测parent是否为child的原型;isPrototypeOf()函数实现的功能和instancof运算符非常类似；
具体代码：
Array.prototype.isPrototypeOf(arr) //true表示是数组，false不是数组

4：利用构造函数constructor
具体代码：
console.log(obj.constructor == Array) //false
console.log(arr.constructor == Array) //true

5：使用typeof(对象) + 类型名结合判断：
具体代码：
```
function isArrayFour(arr) {
  if(typeof arr === "object") {
    if(arr.concat) {
      return 'This is Array'
    } else {
      return 'This not Array'
    }
  }
}
console.log(typeof(obj)) //"object"
console.log(typeof(arr)) //"array"
console.log(isArrayFour(obj)) //This not Array
console.log(isArrayFour(arr)) //This is Array
```
