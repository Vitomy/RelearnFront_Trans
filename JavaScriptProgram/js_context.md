## JavaScript学习历程
嗯...不知道怎么总结这个js😢。作为一种弱类型脚本语言，从执行上下文➡️变量对象➡️作用域链闭包➡️this➡️chrome浏览器优化➡️函数式编程➡️函数柯里化➡️面向对象、构造函数、原型与原型链➡️面向对象实战➡️jquery扩展插件➡️事件循环机制➡️Promise实现➡️ES6大门<br>
好像每个知识点都知道，倒是深入了解不了。所以需要深入了解，掘金，简书，知乎以及高级程序设计，让知识深深刻在大脑里。<br>
最后深入理解Node➡️Koa➡️浏览器优化➡️MySQL➡️重学前端<br>
### js内存空间详细
结合队列，栈，堆内存，栈存储变量对象，堆存储对象，引用设计。脱离执行环境，垃圾会进行收集。<br>
[javascript内存空间详解，传送门](https://www.jianshu.com/p/996671d4dcc4)
### 执行上下文详细
[执行上下文详细，传送门](https://www.jianshu.com/p/a6d37c77e8db)

Execution Context执行上下文
```js
console.log(a);
var a = 20;
```
每次当控制器转到可执行代码的时候，就会进入一个执行上下文。执行上下文可以理解为当前代码的执行环境，会形成一个作用域。
* 全局环境: JS代码运行起来会首先进入该环境
* 函数环境: 当函数被调用执行时，会进入当前函数中执行代码
* eval

JS引擎会以栈的方式处理多个执行上下文，栈可称为函数调用栈(call stack)。栈底永远都是全局上下文，而栈顶就是当前正在执行的上下文。<br>

按出栈顺序执行。单线程，同步执行，只有栈顶上下文处于执行中，其他上下文需要等待，全局上下文只有唯一一个，浏览器关闭时出栈，函数的执行上下文的个数没有限制。每次某个函数被调用，就会有一个新的执行上下文为其创建，即使是调用自身函数。<br>

### 变量对象详解
[变量对象详解，传送门](https://www.jianshu.com/p/330b1505e41d)

当一个函数激活时，一个新的执行上下文就会被创建。而一个执行上下文的生命周期可以分为两个阶段。<br>
* 创建阶段 执行上下文分别创建变量对象，建立作用域链，以及确定this指向。
* 执行阶段 创建之后，剋是执行代码，完成变量赋值，函数引用，以及执行其他代码

变量对象创建过程需要熟悉一下。<br>

变量对象和活动对象，都是同一个对象，只是处于执行上下文不同的生命周期。不过只有处于函数调用栈栈顶的执行上下文中的变量对象，才会变成活动对象。<br>

function声明会比var声明优先级更高一点。
```js
function test(){
    console.log(a);
    console.log(foo());
    var a = 1;
    function foo(){
        return 2;
    }
}
test();
```
test()执行上下文开始理解，创建过程，进入testEC
```js
testEC = {// 变量对象
    VO:{},
    scopeChain:{}
}
VO = {arguments:{...}//注：在浏览器的展示中，函数的参数可能并不是放在arguments对象中，这里为了方便理解，我做了这样的处理
,foo:<foo reference>,a:undefined}
```
在未进入执行阶段之前，变量对象中的属性都不能访问！但是进入执行阶段，变量对象就转变为了活动对象，里面的属性都能被访问了，然后开始进行执行阶段的操作<br>
执行阶段
```js
VO->AO
AO = {
    arguments:{...},
    foo:<foo reference>,
    a:1,
    this:window
}
```
几个例子比较通俗易懂。<br>

### 详细作用域链与闭包
[作用域链与闭包，传送门](https://www.jianshu.com/p/21a16d44f150)

作用域与作用域链。作用域作为一套规则，这套规则用来管理引擎如何在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找(标识符指的是变量名或者函数名)<br>
* 作用域与执行上下文是完全不同的两个概念。js整个执行过程，分为两个阶段，代码编译阶段与代码执行阶段。编译阶段由编译器完成，将代码翻译成可执行代码，这个阶段作用域规则会确定。执行阶段由引擎完成，主要任务是执行可执行代码，执行上下文在这个阶段创建。<br>

作用域链，是由当前环境与上层环境的一些列变量对象组成，它保证了当前执行环境对符合访问权限的变量和函数的有序访问。<br>

闭包是一种特殊的对象。<br>

两部分组成。执行上下文&在该执行上下文中创建的函数<br>

当执行执行上下文中创建的函数，如果访问了执行上下文中变量对象中值，那么闭包就会产生<br>

执行上下文，执行完毕之后，生命周期结束，那么该函数的执行上下文就会失去引用。其占用的内存空间很快就会被垃圾回收器释放。可是闭包的存在会阻止这一过程。<br>

```js
var fn = null;
function foo() {
    var a = 2;
    function innnerFoo() {
        console.log(a);
    }
    fn = innnerFoo; // 将 innnerFoo的引用，赋值给全局变量中的fn
}

function bar() {
    fn(); // 此处的保留的innerFoo的引用
}

foo();
bar(); // 2
```
bar依然能访问到a的值，通过`fn = innerFoo`，保留下来了引用。可以城foo为闭包。<br>

所以通过闭包，我们可以在其他的执行上下文中，访问到函数的内部变量。<br>

在闭包中，能访问到的变量，仍然是作用域链上能够查询到的变量。<br>

```js
for(var k = 1;k<=5;k++){
    setTimeout(function timer(){
       console.log(num);
},1000);
}
// 实现循环输出12345
for(var k = 1;k<=5;k++){
    setTimeout(function timer(){
       var num = k;
       return function(){
       console.log(num);
       }
}(),1000);
}
```

### this解读
this的指向，是在函数被调用的时候确定的。也就是执行上下文被创建时确定的。因此一个函数中的this指向可以是非常灵活的。<br>
函数执行过程中，this一旦被确定，就不可更改了。<br>
```js
var a = 10;
var obj = {
    a: 20
}

function fn () {
    this = obj; // 这句话试图修改this，运行后会报错
    console.log(this.a);
}

fn();
```
全局对象中的this。
```js
this.a2 = 20;
var a1 = 10;
a3 = 30;
```
函数中的this
```js
var a = 20;
function fn(){ console.log(this.a);}
fn();
```
```js
var a = 20;
function fn(){
    function foo(){
        console.log(this.a);
    }
    foo();
}
fn();
```
```js
var a = 20;
var obj = {
    a:10,
    c:this.a + 20,
    fn:function(){
        return this.a;
    }
}
console.log(obj.c);
console.log(obj.fn());
```
如果调用者函数，被某一个对象所拥有，那么该函数在调用时，内部的this指向该对象。如果函数独立调用，那么该函数内部的this，则指向undefined<br>
但是在非严格模式中，当this指向undefined时，他会被自动指向全局对象。<br>
要想准确确定this指向，找到函数的调用者以及区分他是否是独立调用就变得十分关键<br>
```js
function foo() {
    console.log(this.a)
}

function active(fn) {
    fn(); // 真实调用者，为独立调用
}

var a = 20;
var obj = {
    a: 10,
    getA: foo
}

active(obj.getA);
```
使用call，apply显式指定this<br>
js可以自行手动设置this的指向。<br>

实现继承 call实现一次复制。<br>
```js
function bind(fn,obj){
    return function(){
        return fn.apply(obj,arguments);
    }
}

var obj = {
    a:20,
    getA:function(){
        setTimeout(bind(function(){console.log(this.a)},this),1000)
    }
}
obj.getA();
```
new操作符
* 创建一个新的对象
* 将构造函数的this指向了这个新对象
* 指向构造函数的代码，为这个对象添加属性、方法等
* 返回新对象

### 在chrome开发者工具中观察函数调用栈、作用域链与闭包
[传送门](https://www.jianshu.com/p/73122bb3d262)

函数在被调用执行时，会创建一个当前函数的执行上下文。在该执行上下文的创建阶段，变量对象、作用域链、闭包、this指向会分别被确定。<br>


闭包是一个特殊的对象，它由执行上下文与在该执行上下文中创建的函数共同组成。<br>
闭包的形成需要两个条件
* 在函数内部创建新的函数
* 新的函数在执行时，访问了函数的变量对象

```js
// demo07
function foo() {
    var a = 10;

    function fn1() {
        return a;
    }

    function fn2() {
        return 10;
    }

    fn2();
}

foo();
```
闭包是指这样的作用域(foo)，它包含有一个函数fn1，这个fn1可以调用被这个作用域所封闭的变量a、函数、或者闭包等内容。通常我们通过闭包所对应的函数来获得对闭包的访问。<br>

* 闭包是在函数被调用执行的时候才被确认创建的
* 闭包的形成，与作用域链的访问顺序有直接关系
* 只有内部函数访问了上层作用域链中的变量对象时，才会形成闭包，因此，我们可以利用闭包来访问函数内部的变量

### 函数与函数式编程
[传送门](https://www.jianshu.com/p/69dede6f7e5f)

* 函数声明、函数表达式、匿名函数与自执行函数

两种声明方式 var与function，函数声明比变量声明具有更为优先的执行顺序。

```js
fn();// 可执行
function fn(){};
```

函数表达式，使用了var进行声明，以变量声明规则判断。<br>
函数表达的提升方式与变量声明一致。<br>

```js
fn();// 报错
var fn = function fn(){};
```
执行顺序是
```js
var fn = undefined;   // 变量声明提升
fn();    // 执行报错
fn = function() {   // 赋值操作，此时将后边函数的引用赋值给fn
    console.log('function');
}
```
匿名函数 没有显式进行赋值操作的函数。使用场景，多作为一个参数传入另一个函数中<br>
```js
var a = 10;
var fn = function(bar,num){
    return bar()+num;
}
fn(function(){return a;},20)
```
执行上下文过程
```js
// 变量对象在fn上下文执行过程中的创建阶段
VO(fn) = {
    arguments: {
        bar: undefined,
        num: undefined,
        length: 2
    }
}

// 变量对象在fn上下文执行过程中的执行阶段
// 变量对象变为活动对象，并完成赋值操作与执行可执行代码
VO -> AO

AO(fn) = {
    arguments: {
        bar: function() { return a },
        num: 20,
        length: 2
    }
}
```
函数自执行与块级作用域 由于ES5没有块级作用域，常使用函数自执行的方式来模仿块级作用域，这样就提供了一个独立的执行上下文，结合闭包，为模块化提供了基础。函数自执行，就是匿名函数的应用。<br>
```js
(function() {
    // 私有变量
    var age = 20;
    var name = 'Tom';

    // 私有方法
    function getName() {
        return `your name is ` + name;
    }
})();
```
公有变量和公有方法
```js
(function() {
    // 私有变量
    var age = 20;
    var name = 'Tom';

    // 私有方法
    function getName() {
        return `your name is ` + name;
    }

    // 共有方法
    function getAge() {
        return age;
    }

    // 将引用保存在外部执行环境的变量中，形成闭包，防止该执行环境被垃圾回收
    window.getAge = getAge;
})();
```
函数参数传递方式：按值传递<br>

函数式编程<br>
封装一个实现。
```js
function getNumbers(array){
    var res = [];
    array.forEach(function(item){if(typeof item === 'number'){
        res.push(item);
    }})
    return res;
}
```
函数是以函数实现，只需关心这个方法能做什么而不是他具体怎么实现。<br>
函数是第一等公民。<br>
```js
var getUser = $.get;
```
只用表达式，不用语句。函数式编程要求，只是用表达式，不使用语句。也就是说，每一步都是单纯的运算，而且都有返回值。<br>

纯函数 相同的输入总会得到相同的输出，并且不会产生副作用的函数，就是纯函数<br>
函数式编程强调没有副作用，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行文，尤其是不得修改外部变量的值。<br>
js有很多原生不纯的函数，要注意使用，不能修改原生的数据
```js
var source = [1, 2, 3, 4, 5];

source.slice(1, 3); // 纯函数 返回[2, 3] source不变
source.splice(1, 3); // 不纯的 返回[2, 3, 4] source被改变

source.pop(); // 不纯的
source.push(6); // 不纯的
source.shift(); // 不纯的
source.unshift(1); // 不纯的
source.reverse(); // 不纯的

// 我也不能短时间知道现在source被改变成了什么样子，干脆重新约定一下
source = [1, 2, 3, 4, 5];

source.concat([6, 7]); // 纯函数 返回[1, 2, 3, 4, 5, 6, 7] source不变
source.join('-'); // 纯函数 返回1-2-3-4-5 source不变
```
### 函数柯里化
高阶函数的一些特殊用法。<br>
柯里化是指这样一个函数(createCurry)，他接受函数A作为参数，运行后能够返回一个新的函数，并且这个新的函数能够处理函数A的剩余参数。<br>

柯里化也被称为部分求值，可以不用借助柯里化通用式来转化得到柯里化函数。<br>
```js
[].slice.call(arguments);// 将arguments提取出来转化成数组
// arguments本身并不是数组，而是对象
```
```js
// 简单实现，参数只能从右到左传递
function createCurry(func, args) {

    var arity = func.length;
    var args = args || [];

    return function() {
        var _args = [].slice.call(arguments);
        [].push.apply(_args, args);

        // 如果参数个数小于最初的func.length，则递归调用，继续收集参数
        if (_args.length < arity) {
            return createCurry.call(this, func, _args);
        }

        // 参数收集完毕，则执行func
        return func.apply(this, _args);
    }
}
```
[关于函数柯里化比较难，决定回头再看！传送门](https://www.jianshu.com/p/5e1899fe7d6b)
[张鑫旭的柯里化，传送门](https://www.zhangxinxu.com/wordpress/2013/02/js-currying/)

### 详解面向对象、构造函数、原型与原型链
[面向对象详解，传送门](https://www.jianshu.com/p/15ac7393bc1f)

无序属性的集合，其属性可以包含基本值，对象或者函数

```js
var person = {
    name:'Tom',
    age:18,
    getName:function(){},
    parent:{}
}
```
创建对象
```js
var obj = new Object();
var obj = {};
```
为对象添加方法
```js
var person = {};
person.name = "Tom";
person.getName = function(){return this.name;}
person.age = 10;
```
访问属性名或方法
```js
['name','age'].forEach(function(item){
    console.log(person[item]);
})
```
工厂模式 通过模子复制出需要的对象<br>
```js
var createPerson = function(name,age){
    var o = new Object();
    o.name = name;
    o.age = age;
    o.getName = function(){
        return this.name;
    }
    return o;
}
```
new的实现
```js
function New1(func){
    var res = {}; // 创建一个新对象
    if(func.prototype!==null){
        res.__proto__ = func.prototype;// 将构造函数的作用域赋值给新对象
    }
    var ret = func.apply(res,[].slice.call(arguments,1));// 改变this的指向
    // 返回新对象
    if((ret==="object"||ret==="function")&&ret!==null) return ret;
    return res;
}
```
原型模式 我们创建每一个函数都有一个prototype属性，该属性指向一个对象，这个对象就是我们说的原型<br>
new出来的实例，都有一个__proto__属性，该属性指向构造函数的原型对象，通过这个属性，让实例对象也能够访问原型对象上的方法。

```js
function Person(name,age){
    this.name = name;
    this.age = age;
}
Person.prototype = {
    constructor:Person,// 指向Person
    getName:function(){console.log(name);}
    getAge:function(){console.log(age);}
}
```
这里的prototype对象不再是原来的{},所以需要重新加上constructor<br>
原型链的访问，其实跟作用域链有很大的相似之处，他们都是一次单向查找的过程。<br>
因此，实例对象能够通过原型链，访问到处于原型链上对象的所有属性与方法。<br>
原型链的继承<br>
```js
function Person(name,age){
    this.name = name;
    this.age = age;
}
Person.prototype.getName = function(){return this.name;}
```
构造函数的继承
```js
// 构造函数的继承
function cPerson(name, age, job) {
    Person.call(this, name, age);
    this.job = job;
}
```
原型的继承
```js
// 继承原型
cPerson.prototype = new Person(name, age);

// 添加更多方法
cPerson.prototype.getLive = function() {}
```
更好的继承
```js
var Student = function(name, age, grade) {
    // 通过call方法还原Person构造函数中的所有处理逻辑
    Person.call(this,mname, age);
    // 在子类中实现Person的构造函数
    this.grade = grade;
}


// 等价于
var Student = function(name, age, grade) {
    this.name = name;
    this.age = age;
    this.grade = grade;
}
```
考虑，如何将子类对象的与原型加入到原型链中？只需要让子类对象的原型，称为父类对象的一个实例，然后通过`__proto__`就可以访问父类对象的原型。这样就继承了父类原型中的方法与属性。<br>
先封装一个方法，根据父类对象的原型创建一个实例
```js
function create(proto,options){
    // 创建一个空对象
    var tmp = {};
    // 让这个空对象成为父类对象的实例
    tmp.__proto__ = proto;
    // 传入的方法挂载到新对象上，新的对象将作为子类对象的原型
    Object.defineProperties(tmp,options);// 给tmp添加新属性
    return tmp;
}
```
create方法实现继承
```js
Student.prototype = create(Person.prototype, {
    // 不要忘了重新指定构造函数
    constructor: {
        value: Student
    }
    getGrade: {
        value: function() {
            return this.grade
        }
    }
})
// 等价于
Student.prototype = new Person();
```
通过__proto__就可以访问父类对象的原型。<br>
提供了`Object.create();`作用同上，使用也一样<br>
#### 事件循环机制
[事件循环机制，很重要](https://www.jianshu.com/p/12b9f73c5a4f)
Javascript代码执行过程中，除了依靠函数调用栈来搞定函数的执行顺序外，还依靠任务队列(task queue)来搞定另外一些代码的执行。<br>

一个线程中，事件循环是唯一的，但是任务队列可以拥有多个。<br>
macro-task大概包括:scripts，setTimeout，setInterval，setImmediate，I/O，UI rendering<br>

micro-task大概包括:process.nextTick,Promise,Object.observe,MutationObserver<br>

setTimeout/Promise等我们称之为任务源。而进入任务队列的是他们指定的具体的执行任务。<br>
```js
// setTimeout中的回调函数才是进入任务队列的任务
setTimeout(function() {
    console.log('xxxx');
})
// 非常多的同学对于setTimeout的理解存在偏差。所以大概说一下误解：
// setTimeout作为一个任务分发器，这个函数会立即执行，而它所要分发的任务，也就是它的第一个参数，才是延迟执行
```

来自不同任务源的任务会进入到不同的任务队列。其中setTimeout与setInterval是同源的。<br>

事件循环的顺序，决定了JS代码的执行顺序，从script开始第一次循环。之后全局上下文进入函数调用栈。调用栈清空(只剩下全局)，然后执行所有的micro-task。当所有的可执行的micro-task执行完毕之后，循环再次从macro-task开始，找到其中一个任务队列执行完毕，然后再执行所有的micro-task，这样一直循环下去<br>

其中每一个任务的执行，无论是macro-task还是micro-task，都是借助函数调用栈来完成的<br>

#### Promise 透彻理解promise
[promise传送门](https://segmentfault.com/a/1190000012646402)

```js
function want() {
    console.log('这是你想要执行的代码');
}

function fn(want) {
    console.log('这里表示执行了一大堆各种代码');

    // 返回Promise对象
    return new Promise(function(resolve, reject) {
        if (typeof want == 'function') {
            resolve(want);
        } else {
            reject('TypeError: '+ want +'不是一个函数')
        }
    })
}

fn(want).then(function(want) {
    want();
})

fn('1234').catch(function(err) {
    console.log(err);
})
```
Promise对象有三种状态
* pending: 等待中，或者进行中，表示还没有得到结果
* resolved(Fulfilled): 已经完成，表示得到了我们想要的结果，可以继续往下执行
* rejected: 表示得到结果，但是并不是我们想要的，拒绝执行

```js
new Promise(function(resolve, reject) {
    if(true) { resolve() };
    if(false) { reject() };
})
```
上面的resolve和reject都为一个函数，作用分别是将状态修改为resolved和rejected<br>
Promise对象中的then方法，接收构造函数中处理的状态变化，并分别对应执行。then有两个参数，第一个函数接收resolved状态的执行，第二个参数接收reject状态的执行<br>

```js
function fnpromise(want){
    return new Promise(function(resolve,reject){
        if(typeof want === "number") resolve();
        else reject();
}).then(function(){console.log("num")},function(){console.log("not num");});
}
```
then方法的执行结果也会返回一个Promise对象。因此可以进行then的链式执行，这也是解决回调地狱的主要方式。<br>
```js
function fn(num) {
    return new Promise(function(resolve, reject) {
        if (typeof num == 'number') {
            resolve();
        } else {
            reject();
        }
    })
    .then(function() {
        console.log('参数是一个number值');
    })
    .then(null, function() {
        console.log('参数不是一个number值');
    })
}

fn('hahha');
fn(1234);
```
Promise中的数据传递
```js
var fn = function(num) {
    return new Promise(function(resolve, reject) {
        if (typeof num == 'number') {
            resolve(num);
        } else {
            reject('TypeError');
        }
    })
}

fn(2).then(function(num) {
    console.log('first: ' + num);
    return num + 1;
})
.then(function(num) {
    console.log('second: ' + num);
    return num + 1;
})
.then(function(num) {
    console.log('third: ' + num);
    return num + 1;
});

// 输出结果
first: 2
second: 3
third: 4
```
对于ajax的封装
```js
var url = 'https://hq.tigerbrokers.com/fundamental/finance_calendar/getType/2017-02-26/2017-06-10';

// 封装一个get请求的方法
function getJSON(url) {
    return new Promise(function(resolve, reject) {
        var XHR = new XMLHttpRequest();
        XHR.open('GET', url, true);
        XHR.send();

        XHR.onreadystatechange = function() {
            if (XHR.readyState == 4) {
                if (XHR.status == 200) {
                    try {
                        var response = JSON.parse(XHR.responseText);
                        resolve(response);
                    } catch (e) {
                        reject(e);
                    }
                } else {
                    reject(new Error(XHR.statusText));
                }
            }
        }
    })
}

getJSON(url).then(resp => console.log(resp));
```
#### Promise.all
当有一个ajax的请求，参数需要另外两个甚至更多请求都有返回结果之后才能确定，那么这个时候，就需要用到Promise.all来帮助我们应对这个场景。<br>
Promise.all接收一个Promise对象组成的数组作为参数，当这个数组所有的Promise对象状态都变成resolved或者rejected的时候，才会调用then方法。<br>
```js
var url = 'https://hq.tigerbrokers.com/fundamental/finance_calendar/getType/2017-02-26/2017-06-10';
var url1 = 'https://hq.tigerbrokers.com/fundamental/finance_calendar/getType/2017-03-26/2017-06-10';

function renderAll() {
    return Promise.all([getJSON(url), getJSON(url1)]);
}

renderAll().then(function(value) {
    // 建议大家在浏览器中看看这里的value值 也是一个resolved组成的数组
    console.log(value);
})
```
#### Promise.race都是以一个Promise对象组成的数组作为参数
当一个Promise状态变成resolved或者rejected时，就可以调用.then方法。
```js
function renderRace() {
    return Promise.race([getJSON(url), getJSON(url1)]);
}

renderRace().then(function(value) {
    console.log(value);
})
```
马上调用了then方法，也没有等待后面的数据回来。<br>

#### Es6基础知识汇总
1. 新的变量声明方式let/const 与var不同，新的变量声明方式带来了一些不一样的特性就是提供了块级作用域与不再具备变量提升<br>
```js
{let a = 10;}; console.log(a);
// VM1305:1 Uncaught ReferenceError: a is not defined
//     at <anonymous>:1:28
```
使用ES6，需要全面使用let/const替换var。什么时候用？<br>
let来声明一个值会被改变的变量，而使用const来声明一个值不会被改变的变量，称之为常量。<br>

```js
const obDev = {a:23,b:34};
undefined
obDev = {b:23,a:34};
// VM1438:1 Uncaught TypeError: Assignment to constant variable.
//     at <anonymous>:1:7
// (anonymous) @ VM1438:1
obDev.a = {b:23,a:34};
{b: 23, a: 34}a: 34b: 23__proto__: Object
obDev
{a: {…}, b: 34}
```
#### 箭头函数的使用()=>{}
```js
// ES5
var fn = function(a,b){return a+b;}
// ES6 当函数直接被return时，可以省略函数体的括号
const fn = (a,b)=>a+b;
// ES5
var foo = function(){var a = 20;var b = 30;return a+b;}
//es6
const foo = ()=>{const a = 20;const b = 30;return a+b;}
```
箭头函数可以替换函数表达式，但是不能替换函数声明。<br>
箭头函数中，没有this。如果你在箭头函数中使用了this，那么该this一定就是外层的this。<br>
正是因为箭头函数没有this，因此我们也就无从谈起用call/apply/bind来改变this指向。<br>
```js
var person = {
    name: 'tom',
    getName: function() {
        return this.name;
    }
}

// 我们试图用ES6的写法来重构上面的对象
const person = {
    name: 'tom',
    getName: () => this.name
}

// 但是编译结果却是
var person = {
    name: 'tom',
    getName: function getName() {
        return undefined.name;
    }
};
```
es6会默认采用严格模式，因此this也不会自动指向window对象了。如果必须使用this，可以采用以下的写法
```js
const person = {
    name:"tom",
    getName:function(){
        return setTimeout(()=>this.name,1000);
    }
}

// 编译后
var person = {
    name:"tom",
    getName:function getName(){
        var _this = this;
        return setTimeout(function(){
            return _this.name;
        },1000);
    }
}
```
箭头函数无法访问arguments对象。<br>
#### 模版字符串 解决使用+号拼接字符串的不便利而出现的
```js
const a = 20;
const b = 30;
const string = `${a}+${b} = ${a+b}`;
```
使用``将整个字符串包裹起来，在${}来包裹一个变量或这一个表达式，shell也有类似的用法<br>
#### 解析结构
```js
const props = {
    className:'tiger-button',
    loading:false,
    clicked:true,
    disabled:'disabled'
}
```
当我们需要取得其中的两个值loading与clicked时
```js
// es5
var loading = props.loading;
var clicked = props.clicked;
// es6
const {loading,clicked} = props;
// 给一个默认值，当props对象中找不到loading时，loading就等于该默认值
const {loading = false,clicked} = props;
```
解析结构的减少了很大的代码量，所以大受欢迎。<br>
```js
// 比如
// section1
import React, { Component } from 'react';

// section2
export { default } from './Button';

// section3
const { click, loading } = this.props;
const { isCheck } = this.state;

// more  任何获取对象属性值的场景都可以使用解析结构来减少我们的代码量
```
另外，数组也有属于自己的解析结构
```js
const arr = [1,2,3];
const [a,b,c] = arr;

// es5
var arr = [1,2,3];
var a = arr[0];
...
```
#### 函数默认参数
之前不能为函数指定默认参数，因此很多时候，为了保证传入的参数具备一个默认值
```js
function add(x, y) {
    var x = x || 20;
    var y = y || 30;
    return x + y;
}

console.log(add()); // 50
```
以上方法是常用的，但是es6
```js
function add(x = 20, y = 30) {
    return x + y;
}

console.log(add());
```
而实际开发中，可以这样做
```js
const ButtonGroupProps = {
    size: 'normal',
    className: 'xxxx-button-group',
    borderColor: '#333'
}

export default function ButtonGroup(props = ButtonGroupProps) {
    ... ...
}
```
#### 展开运算符
es6用...来表示展开运算符，可以将数组方法或者对象进行展开。
```js
const arr1 = [1,2,3];
const arr2 = [...arr1,10,20,30];
// 对象
const obj1 = {
  a: 1,
  b: 2,
  c: 3
}

const obj2 = {
  ...obj1,
  d: 4,
  e: 5,
  f: 6
}

// 结果类似于 const obj2 = Object.assign({}, obj1, {d: 4})
```
展开运算符结合解析结构使用。
```js
// 这种方式在react中十分常用
const props = {
  size: 1,
  src: 'xxxx',
  mode: 'si'
}


const { size, ...others } = props;

console.log(others)

// 然后再利用暂开运算符传递给下一个元素，再以后封装react组件时会大量使用到这种方式，正在学习react的同学一定要搞懂这种使用方式
<button {...others} size={size} />
```
展开运算符还用在函数的参数中，来表示函数的不定参。只有放在最后才能作为函数的不定参数，否则会报错。<br>
```js
const add = (a,b,...more) =>{
    console.log(more);// 传进来的more是一个数组
    return more.reduce((m,n)=>m+n)+a+b; // 这里是函数式编程的使用
    // reduce 将参数两项两项相加，第一项加第二项之后又变成了第一项继续相加
}
console.log(add(1,2,3,4,5,6,7,8));
```
#### 对象的字面量与class
```js
// es6
const name = "jane";
const age = 10;
const Person = {name,age}

// es5
var Person = {name:name,age:age};
```
这种方式在任何地方都可以使用，比如在一个模块对外提供接口时。<br>
```js
const getName = ()=>person.name;
const getAge = ()=>person.age;
// commonJS方式
module.exports = {getName,getAge};
// ES6 modules方式
export default {getName,getAge};
```
除了属性之外，对象字面量写法中的方法也有简写方式
```js
/ es6
const person = {
  name,
  age,
  getName() { // 只要不使用箭头函数，this就还是我们熟悉的this
    return this.name
  }
}

// es5
var person = {
  name: name,
  age: age,
  getName: function getName() {
    return this.name;
  }
};
```
在对象字面量中可以使用中括号作为属性，表示属性名也能是一个变量了
```js
const name = 'Jane';
const age = 20

const person = {
  [name]: true,
  [age]: true
}
```
例如ant-design源码实现，大量使用了这种方式来拼接当前元素className
```js
let alertCls = classNames(prefixCls, {
      [`${prefixCls}-${type}`]: true,
      [`${prefixCls}-close`]: !this.state.closing,
      [`${prefixCls}-with-description`]: !!description,
      [`${prefixCls}-no-icon`]: !showIcon,
      [`${prefixCls}-banner`]: !!banner,
 }, className);
```
Es6为创建对象提供了新的语法糖，class语法
```js
// es5
function Person(name,age){
    this.name = name;
    this.age = age;
}
// 原型方法
Person.prototype.getName = function(){return this.name}
// Es6
class Person{
    constructor(name,age){
        this.name = name;
        this.age = age;
    }
    getName(){return this.name;} // 原型方法
    static a = 20; // 等同于Person.a = 20
    c = 20;// 表示在构造函数中添加属性 在构造函数中扽同于 this.c = 20
    // 箭头函数的写法表示在构造函数中添加方法，在构造函数中等同于this.getAge = function(){}
    getAge = ()=>this.age;
    // 箭头函数this指向不能被改变
}
```
继承extends
```js
class Student extends Person{
    constructor(name,age,gender,classed){
        super(name,age);
        this.gender = gender;
        this.classes = classes;
    }
    getGender(){return this.gender;}
}
```
只需要extends关键字就可以实现继承了。不用像es5那样去担心构造函数继承和原型继承。<br>
super方法其实与ES5的call/apply继承构造函数是一样的功能<br>
```js
super(name,age);
Person.call(this);
// 也可以调用父级原型方法
super.getName
```
#### 模块Modules
```js
const a = 10;
const b = 20;

const obj = {
    name:"trans",
    age:"24"
}

const fn = ()=>{
    const a = 20;
    const b = 39;
    return a+b;
}

export default {
    a,b,obj,fn
}

export const bar = ()=>{}
export const foo = 1234;
```
