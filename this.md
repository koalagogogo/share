## 什么是 this

>当一个函数被调用时，会创建一个活动记录（ 有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this就是记录的其中一个 属性，会在函数执行的过程中用到。

## 为什么要使用 this

```javascript
function identify() {
    console.log("Hello,I'm " + this.name);
}
var me = {
    name: "fingal"
};
var you = {
    name: "tff"
};
identify.call(me); // Hello,I'm fingal
identify.call(you); // Hello,I'm tff
```

可以在不同的对象中复用函数identify，不用针对每个对象编写一个新函数。

>this提供了一种更优雅的方法来隐式’传递’一个对象的引用，因此可以将API设计得更加简洁并且易于复用。

## 绑定规则

### 一、默认绑定

在非严格模式下，默认绑定的this指向全局对象，严格模式下this指向undefined，不能应用其他规则的情况下，就是默认规则

```javascript
var a = 2;
function foo() {
    console.log(this.a); // this指向全局对象
}
foo(); // 2
function foo2() {
    "use strict"; // 严格模式this绑定到undefined
    console.log(this.a); 
}
foo2(); // TypeError:a undefined

```
### 二、隐式绑定

函数在调用位置，是否有上下文对象，如果有，那么 this 就会隐式绑定到这个对象上，对象属性引用链中 this 指向最后一层的对象


```javascript
var a = "global";
function foo() {
    console.log(this.a);
}
var obj1 = {
    a: "obj1 a",
    foo: foo
};
var obj2 = {
    a: "obj2 a",
    obj1: obj1
};
obj1.foo(); // obj1 a this指向调用函数的对象
obj2.obj1.foo(); // obj1 a this指向最后一层调用函数的对象

// 隐式绑定丢失
var bar = obj1.foo; // bar只是一个函数别名 是obj1.foo的一个引用
bar(); // "global" - 指向全局
```

实际上就是函数调用时，并没有上下文对象，只是对函数的引用，所以会导致隐式绑定丢失

### 三、显示绑定

我们可以通过apply、call、bind将函数中的this绑定到指定对象上

```javascript
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2
};
foo.call(obj); // 2
```

### 四、new绑定

使用构造函数调用的时候，this 自动绑定在 new 期间创建的这个对象上

```javascript
// 回忆 new 的过程

// 1. var obj = {};
// 2. obj._proto_ = Person.prototype;
// 3. Person.call(obj, args...)  咦，好像就是显示绑定

function Person(name) {
    this.name = name; // this 被绑定在 bar 上
}

var person1 = new Person("fingal");  // new 操作符调用函数
console.log(person1.name);
```

### 五、箭头函数中的 this

继承外层非箭头函数的 this 绑定对象，且一旦被绑定，不会被改变。

```javascript
function foo() {
    return () => {
        console.log(this.a);
    };
}
let obj1 = {
    a: 2
};
let obj2 = {
    a: 22
};
let bar = foo.call(obj1); // foo this指向obj1
bar.call(obj2); // 输出2 这里执行箭头函数 并试图绑定this指向到obj2，氮素不会成功

```

## 优先级

* 显式绑定 > 隐式绑定 > 默认绑定
* new绑定 > 隐式绑定 > 默认绑定

```javascript
obj1.foo.call(obj2); // this指向obj2 显式绑定比隐式绑定优先级高。
new obj1.foo(); // this指向new新创建的对象 new绑定比隐式绑定优先级高。
```

## 判定 this

> 1. 函数是否在new中调用（new绑定）？如果是的话this绑定的是新创建的对象。
> 2. 函数是否通过call、apply（显式绑定）或者硬绑定调用？如果是的话，this绑定的是指定的对象。
> 3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。
> 4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。var bar = foo()就是这样。

总结：随着函数使用场合的不同，this的值会发生变化。但是有一个总的原则，那就是this指的是函数运行时所在的环境(箭头函数除外)。


## 测试

```javascript
var a = "global";
function foo() {
    console.log(this.a);
}
foo(); // global

var bar = new foo(); // globale new 绑定

var obj1 = {
    a: 1
};
var obj2 = {
    a: 2
};

obj1.foo = foo; // 隐式绑定
console.log(obj1.foo()); // 1

obj1.foo.call(obj2); // 2 this 指向 obj2  显示绑定 > 隐式绑定
```

## 常见场景

### 一、在函数中使用(默认绑定)

在函数被直接调用时this绑定到全局对象。在浏览器中，window 就是该全局对象

```javascript

console.log(this);

function fn1(){
    console.log(this);
}

fn1();

```

### 二、内部函数使用(默认绑定)

函数嵌套产生的内部函数的this不是其父函数，仍然是全局变量

```javascript

function fn0(){
    function fn(){
        console.log(this);
    }
    fn();
}

fn0();

```

### 三、setTimeout、setInterval(默认绑定)

这两个方法执行的函数this也是全局对象

```javascript

document.addEventListener('click', function(e){
    console.log(this);
    setTimeout(function(){
        console.log(this);
    }, 200);
}, false);

```

### 四、构造函数中(new 绑定)

```javascript
function Person(name){
    this.name = name;
}

Person.prototype.printName = function(){
    console.log(this.name);
};

var person1 = new Person("fingal");

```

### 五、作为对象方法调用 (隐式绑定)

```javascript

var obj1 = {
    name: 'Byron',
    fn: function(){
        console.log(this);
    }
};

obj1.fn();
```

```javascript
// 下边这种情况，this 指向 全局对象
var fn2 = obj1.fn;

fn2();
```