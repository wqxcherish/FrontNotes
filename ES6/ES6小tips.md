# ES6

## 扩展运算符之复制数组
```
//es6做法
const a1 = [1, 2]
const a2 = [...a1]
//or
const [...a2] = a1
a2[0] = 2
a1 //[1,2] 修改a2不会对a1有影响

```
注：要注意的一点是，concat和扩展运算符用做合并数组时，都是浅拷贝。

## 数组实例的find()和findIndex()

find方法，用于找出第一个符合条件的数组成员，它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined。

```
//es6做法
[1,5,10,15].find(function(value,index,arr) {
    return value > 0
}) //10
```

## 数组实例的includes()

该方法返回一个布尔值，表示某个数组是否包含给定的值。

# 对象升级

## Object.assign()

用于对象的合并，将原对象的所有可枚举属性，复制到目标对象上。
object.assign拷贝的属性是有限制的，只拷贝源对象自身的属性（不拷贝继承属性，也不拷贝不可枚举的属性）

```
//为对象添加属性和方法
//es6
class Point {
 constructor(x, y) {
   Object.assign(this, {x, y});
 }
}
Object.assign(Point.prototype, {
 addPoint(arg1, arg2) {
   ···
 }
});
```

复制代码这个方法很常用，但一些细节问题还是要拎出来提醒自己注意一下。

1. 浅拷贝
Object.assign方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。

2. 同名属性的替换
对于这种嵌套的对象，一旦遇到同名的属性，Object.assign的处理方法是替换，而不是添加。要特别注意！！

3. 数组的处理
Object.assign可以用来处理数组，但是会把数组视为对象。

## 属性的遍历
ES6 一共有 5 种方法可以遍历对象的属性。
1. for...in
for...in循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。
2. Object.keys(obj)
Object.keys返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。
3. Object.getOwnPropertyNames(obj)
Object.getOwnPropertyNames返回一个数组，包含对象自身的所有属性（不含Symbol属性，但是包括不可枚举属性）的键名。
4. Object.getOwnPropertySymbols(obj)
Object.getOwnPropertySymbols返回一个数组，包含对象自身的所有 Symbol 属性的键名。
5. Reflect.ownKeys(obj)
Reflect.ownKeys返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

# 函数升级
1. reset参数
用于获取函数的多余参数，这样就不需要使用arguments对象了。
```
// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();
```
2.  箭头函数
ES6 允许使用“箭头”（=>）定义函数。
```
// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();
```
复制代码注意点：
（1）函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。
（2）不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
（3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

# for...of循环
for...of循环可以使用的范围包括数组、Set 和 Map 结构、某些类似数组的对象（比如arguments对象、DOM NodeList 对象）、后文的 Generator 对象，以及字符串。
1. 用法
数组：
```
//为对象添加属性和方法
//es6
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
  console.log(v); // red green blue
}
复制代码Set 和 Map 结构
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
 console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
 console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
复制代码类数组的对象
// 字符串
let str = "hello";

for (let s of str) {
 console.log(s); // h e l l o
}

// DOM NodeList对象
let paras = document.querySelectorAll("p");

for (let p of paras) {
 p.classList.add("test");
}

// arguments对象
function printArgs() {
 for (let x of arguments) {
   console.log(x);
 }
}
printArgs('a', 'b');
// 'a'
// 'b'
```

2. 与其他遍历语法的比较
for...in循环有几个缺点。

数组的键名是数字，但是for...in循环是以字符串作为键名“0”、“1”、“2”等等。
for...in循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键。
某些情况下，for...in循环会以任意顺序遍历键名。
总之，for...in循环主要是为遍历对象而设计的，不适用于遍历数组。

forEach循环的缺点是无法中途跳出forEach循环，break命令或return命令都不能奏效。
与之相对的，for...of有如下优点：

有着同for...in一样的简洁语法，但是没有for...in那些缺点。
不同于forEach方法，它可以与break、continue和return配合使用。
提供了遍历所有数据结构的统一操作接口。

