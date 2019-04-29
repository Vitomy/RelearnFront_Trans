### 重学前端
![前端知识架构图](./img/front_end.jpg "前端知识架构图")

基本上不动笔墨不读书<br>
JavaScript模块会从运行时、文法和执行过程三个角度剖析JS的知识体系。
#### 为什么有的编程规范要求用void 0代替undefined
🔫Undefind、null、String、Boolean、Number、Object和Symbol七种类型。Undefined、Null。Undefined类型表示未定义，它的类型只有一个值。undefined，任何变量赋值前是undefined类型，值为undefined。undefined是一个变量，而非一个关键字。为了避免无意中被篡改，建议用void 0来获取undefined值。<br>
什么情况下会被篡改？建议使用void 0来获取undefined值。<br>
🔫Boolean，true与flase<br>
🔫String类型，String最大长度2^53-1最大长度是指UTF16编码，字符串的操作charAt(位置寻址),charCodeAt(获得字符Ascii码),length(获取字符长度)<br>
🔫Number类型，通常意义上的数字，对应数学中的有理数(2^64-2^53+3)个值。<br>
正确的比较方式(console.log(Math.abs(0.1+0.2-0.3)<=Number.EPSILON));<br>
🔫Symbol类型本身，它是一切非字符串的对象key的集合，在ES6规范中，整个对象系统被用Symbol重塑。<br>
```js
var mySymbol = Symbol("my symbol");
var o = new Object

    o[Symbol.iterator] = function() {
        var v = 0
        return {
            next: function() {
                return { value: v++, done: v > 10 }
            }
        }        
    };

    for(var v of o) 
        console.log(v); // 0 1 2 3 ... 9

```
这段我不是很明白为什么。稍后查询一下❓❓❓
🔫Object类型，一切有形和无形物体总称<br>
对象的定义分为数据属性与访问器属性，二者都是key-value结构，key可以是字符串或者symbol类型。<br>
3与new Number(3)是完全不同的值，一个是number类型一个是对象类型。<br>
Number、String和Boolean，三个构造器是两用的，当跟new搭配时，产生对象，直接调用时，表示强制类型转换。<br>
```js
Symbol.prototype.helloTrans = ()=>{console.log("hello trans!")};
var a = Symbol("a");
console.log(typeof a);
a.helloTrans();
```
为Symbol添加原型链属性。<br>
.运算符提供装箱操作，根据基础类型构造一个临时对象，使得我们能在基础类型上调用对应对象的方法。<br>
#### 为什么给对象添加的方法能用在基本类型上？原型链属性
#### 装箱与拆箱操作
#### 装箱转换 基本类型转换成对象类型
每一种基本类型Number、String、Boolean、Symbol在对象中都有对应的类，所谓装箱转换，正是把基本类型转换为对应的对象，它是类型转换中一种相当重要的种类。<br>
```js
var symbolObject = (function(){return this;}).call(Symbol("a"));
var symbolObject = Object(Symbol("a"));// 显式转换

```
#### 拆箱转换 对象类型转换基本类型
拆箱转换会尝试调用valueOf和toString来获得拆箱后的基本类型。如果valueOf和toString都不存在或者没有返回基本类型，则会产生类型错误TypeError<br>
```js
    var o = {
        valueOf : () => {console.log("valueOf"); return {}},
        toString : () => {console.log("toString"); return {}}
    }

   String(o)// o*2; // valueOf toString
    // toString
    // valueOf
    // TypeError
```
#### js代码实现String到Number的转换？不用原生的Number和ParseInt
