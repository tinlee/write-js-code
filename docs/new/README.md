
`
疑问：很多文章中，都判断了返回值的类型，如果返回值是`object`则返回，不然会返回新的对象。但是在实际测试中一些简单数据类型，如Number,会出现返回的对象中的值未被定义的状况。这里的类型判断是为了什么呢？
`

# what is new
new的mdn说法为`new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例`。
但是关于创建过程，在原文中出现了两种说法，一种是说`创建一个空的简单JavaScript对象（即{}）；
链接该对象（设置该对象的constructor）到另一个对象 ；`,另外一种是说`一个继承自 Foo.prototype 的新对象被创建`,鉴于前一种方式，个人比较认同，所以采用前一种方式书写本文。
## 标准语法
```javascript
new constructor[([arguments])]
```
其中constructor可以为一个类或者一个函数，而arguments为一个参数列表
# 实现目标
实现一个函数，完成new运算符的相关例子，并可以传入class或者函数。目标代码
```javascript
_new(constructor[, arg1[, arg2[, ...argN]]])
```
# constructor为函数时的实现过程
## 实现原理
new函数的基本原理为：
1.创建一个空的简单JavaScript对象（即{}）；
2.将新对象的__proto__指向构造函数原型；
3.将构造函数中的this指向新对象
4.构造函数中若有返回值，就直接返回；否则返回新对象
## 简单版手写实现
```javascript

/**
 * new的手动实现参数
 * @param {function} Func 构造函数
 * @param {array<any>} arg 传入的其他参数
 */
function _new(Func,...arg) {
    // 1.创建一个新对象
    const _this={}
    // 2.将新对象的__proto__指向构造函数原型
    _this.__proto__ = Func.prototype
    // 3.将构造函数中的this指向新对象
    const returnData=Func.apply(_this, arg);
    // 4.构造函数中若有返回值，就直接返回；否则返回新对象
    return returnData||_this
}
```
## 类型验证
在js中,可以执行new关键字的情况，有以下几种
- Number
- String
- Boolean
- Date
- Object
- Array
- Function
- RegExp
- Error
- Class
- Promise
我们就以上几种类型验证。
```javascript
_new(Number,1) // 1
_new(String,'test') // "test"
_new(Boolean,true) // true
_new(Boolean,false) // Boolean {}
_new(Date,'2020-01-14') // Tue Jan 14 2020 08:00:00 GMT+0800 (中国标准时间)
_new(Object,{key:'value'}) //{key: "value"}
_new(Array,1) // [empty]
_new(Array,1,2,3) // (3) [1, 2, 3]
_new(Function,(arg)=>{return arg}) 
/*
ƒ anonymous(
) {
(arg)=>{return arg}
}*/
_new(Function,'arg','return arg')
/*
ƒ anonymous(
) {
(arg)=>{return arg}
}*/
_new(RegExp,'test') // /test/
_new(Error,'this is a error')
/*
Error: this is a error
    at _new (<anonymous>:7:27)
    at <anonymous>:1:1
*/
Class test{}
_new(test) // Uncaught TypeError: Class constructor a cannot be invoked without 'new'
_new(Promise,function(){}) //Uncaught TypeError: undefined is not a promise

```
我们可以看到验证结果中出现了两种状况，一种是基本数据类型出现了问题。以Number为例。如果我们执行`typeof new Number(1)`。此时的返回值是`"object"`，而我们执行我们函数的时候，`typeoof _new(Number)`返回的是`"number"`。

##  无法模拟的状况
前面提到了，new可以是一个函数或者一个类，但是如果我们传入是Class或者是Promise，发现我们实现的函数是报错的。在某一些情况下是无法进行new 的模拟，即需要在内部判断new.target的情况。

new.target属性允许你检测函数或构造方法是否是通过new运算符被调用的。在通过new运算符被初始化的函数或构造方法中，new.target返回一个指向构造方法或函数的引用。在普通的函数调用中，new.target 的值是undefined。
```javascript
function test(){ 
    console.log( new.target)
}
test() // undefined
new test() // test的新的实例
```
在无法模拟的情况下，均是内部出现了检测new.target的代码.因为new 是关键字，我们无法对它重新定义或者全局赋值，在此情况下，也就无法进行模拟。

在前一段中，基础类型在内部有一个属性`PrimitiveValue`,该值是在初始化的时候进行判断并赋值的，如果遇到了new标签，那么该值会被设置，并返回一个对象。如果没有遇到，则返回基础类型。也就是上一段中出现的bug。

因此在遇到此类数据的时候，模拟的`new`只能返回基础类型的返回值，而不能返回新对象。

`Promise的报错有些问题，可参考`紫云飞`大佬在某乎的回答。`

