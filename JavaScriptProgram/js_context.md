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
Javascript代码执行过程中，除了依靠函数调用栈来搞定函数的执行顺序外，还依靠任务队列(task queue)来搞定另外一些代码的执行。<br>

一个线程中，事件循环是唯一的，但是任务队列可以拥有多个。<br>
macro-task大概包括:scripts，setTimeout，setInterval，setImmediate，I/O，UI rendering<br>

micro-task大概包括:process.nextTick,Promise,Object.observe,MutationObserver<br>


