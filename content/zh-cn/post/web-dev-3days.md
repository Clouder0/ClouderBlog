---
title: "Frontend 入门笔记"
date: 2022-08-19T15:46:29+08:00
slug: "web-dev-3days"
type: post
tags:
  - web
  - math
categories:
  - Programming
---

跟着 [The Odin Project](https://www.theodinproject.com) 快速入门一下网页开发。

作为一个~~功底深厚~~闲得蛋疼的 CSer，当然是选择 Full Stack 路线啦。

按天来记录学习内容。

## Day 1

Skip 掉 Introduction 部分。

Prerequisites 也可以 Skip 掉。

Git 学过也可以 Skip 掉，直接进入 HTML Foundations 部分。

### HTML Foundations

HTML，全称 HyperText Markup Language，是一种标记语言，和 Markdown 是同类。

HTML 用于定义网页的结构与内容，而不负责网页的具体显示细节。后者归 CSS 管。

HTML 中最主要的单元是 element，一般来说由两个 tag 包裹一定的内容。

```html
<p>A paragraph.</p>
```

可以标记属性(attributes)：

```html
<p style="color:black;">black text.</p>
```

HTML 文件开头标注 DOCTYPE，默认使用 HTML5.

```html
<!DOCTYPE html>
```

整个文件结构也类似于一棵树。

```html
<!DOCTYPE html>
<html>

<head>
</head>

<body>
</body>

</html>
```

head element 用于放置 metadata，比如 title 啊 description 啊之类的。

body element 则用于放置网页的主体内容。

---

#### Working with Text

在 HTML 中，多于一个的空格空格、空行都是被忽视的，与在 Markdown 中差不多。

两个段落就应该使用两个 paragraph element. 或者用 <br><br />

```html
<p> Paragraph One. </p>
<p> Paragraph Two. </p>
```

标题支持从 h1 到 h6.

* strong，重点标记，默认样式是粗体(bold)
* em，emphasis，强调标记，默认样式是斜体(italics)

这里要区别 strong/bold em/i 的区别，前者是语义标签(semantic)，后者是样式标签。
前者注重的是标签本身的意义，而显示效果只是默认的 style，事实上可以用 css 修改。而后者则只注重显示效果。

HTML 中的注释：

```html
<!-- Single Line -->
<p> test </p> <!-- Inline -->
<!--
Multiple
Lines
-->
```

可以把注释也看成一种 element，`<!--` 是 opening tag，`-->` 是 closing tag.

#### Lists

两种 List：Unordered/Ordered，分别对应 ul 和 ol.

List 中的 Item 的 tag 是 li.

List 可以嵌套之类的。

没啥特别的。

#### Links and Images

在 HTML 中创建链接，使用 Anchor Element. 用 href 指定目标，target 指定打开方式。

```html
<a href="https://www.google.com/" target="_blank">Click Me!</a>
```

链接可以是 Absolute Path，也可以是 Relative Path.

Image 也很简单。

```html
<img src="../images/dog.jpg" alt="Alt Text">
```

---

这个 Odin 的教程有点轻量，跟 W3Schools 的比不够全面，但入门确实轻松快速。

#### Project Recipes

第一个 Project.

简单的写几个 HTML 文件就完事了。考察了 ol/ul anchor headings img，没了。

### CSS Foundations

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220817161509-vwmxbh3.png)​

CSS 的基础语法，selector 加上若干键值对。

selector：

* `*`，通配符。
* `div`，按照 element name 匹配。
* `.class`，按照 class name 匹配。
* `#id`，按照 id 匹配。
* `.class1, .class2`，这个叫 grouping selector，对多个目标生效。
* `p.class1`，`class1.class2#id`，chaining selector，任意组合。
* 还有一些 combinator

  * descendant selector (space)
  * child selector (>)
  * adjacent sibling selector (+)
  * general sibling selector (~)

常用的属性：

* color
* background-color
* font-family
* font-size
* font-weight
* text-align
* height
* width

CSS 的生效过程中可能会有 override，此时依据 Cascade 规则，越是具有特异性指向的规则就有越高的优先级。

* HTML Inline Style
* ID Selector
* Class Selector
* Type Selector

如果有相同的优先级，那么后出现的会覆写前面的规则。

向网页添加 CSS 有三种方式：

* External CSS，在 head 中引入外部文件

  ```css
  <link rel="stylesheet" href="styles.css">
  ```
* Internal CSS，直接将内容写在 head 中

  ```css
  <head>
    <style>
      div {
        color: white;
        background-color: black;
      }

      p {
        color: red;
      }
    </style>
  </head>
  <body>...</body>
  ```
* Inline CSS，写在 HTML 的 element attribute 中

  ```css
  <body>
    <div style="color: white; background-color: black;">...</div>
  </body>
  ```

然后去做一下 [Exercise](https://github.com/TheOdinProject/css-exercises).

#### Box Model

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220817165419-i6wdkhj.png)​

这张图就差不多足够了。

一切 element 都是 box.

纵向的 margin 是可以重叠的，横向的不重叠。

设置 element 大小时，默认指向的是 content box. 实际的整个 box 会更大一些。
可以改 box-sizing property 为 border-box，这样设置的就是 border 以内的整个 box 的大小，再根据 border padding 的 size 计算出 content 的 size.

#### Block & Inline

暂时不想了解过多细节。

## Day2

### Flexbox

调整 Layout 用的好东西。

Flex Items 将在 Flex Container 中按一定的方式排列。

```css
.flex-container {
	display: flex;
}

.flex-item {
	flex: 1;
}
```

事实上，flex 是 flex-grow flex-shrink flex-basis 的 shorthand，用于修饰 flex-item.

flex-grow 是容器大小增大时，item 的大小增大的 factor，默认为 0. 如果设置为 2，那么该 item 增大速率就是其他默认 item 的两倍，因此大小也是两倍。

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220818113938-iv4uypk.png)​

flex-shrink 则是缩小倍率，而 flex-basis 是初始大小。flex-basis 设置为 auto 会选用 item 本身的 width/height.

flex-direction 可以改变 flex-container 放置 item 的方向。比如 `row` 是横向排列（默认），`column` 是纵向排列。

flex container 内部的 alignment 可以通过 justify-content 和 align-items 调整。

justify-content 调整的是 main axis 上的排版，main axis 是 flex direction 对应的方向。

align-items 则是 cross axis 方向的排版。

比如 flex-direction 为 row 时，main axis 对应横向，cross axis 对应纵向。

可以设置 container 的 gap 属性(row-gap column-gap) 来调整 item 的间距，类似于在 item 中设置 margin.

然后去做 Exercises.

学完 Flexbox 之后，基本上能动手做出来简单的 layout 了。

#### Project Landing Page

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220818144709-e7y0277.png)​

照着这张图片做个 landpage 出来。

感觉似乎也没那么困难了吧。

根据背景颜色从上到下划分出 5 个 Section，用 Flex 分割。

然后每个小块中再照葫芦画瓢即可。

感觉很多小细节的地方写的有问题，唉。

[index.html](https://pastebin.com/KNhmLvY4) [main.css](https://pastebin.com/ZBH2b7fc)

### Javascript

终于进入 js 部分了。

学完 js 估计就能写点小项目了吧。

HTML 可以内嵌 js. 用 script tag 即可。

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Page Title</title>
</head>
<body>
  <script>
    // Your JavaScript goes here!
    console.log("Hello, World!")
  </script>
</body>
</html>
```

可以引入 external script.

```html
<script src="myscript.js"></script>
```

js 输出方式一般有：

* 修改 innerHTML
* 直接输出整页的 HTML，`document.write()`
* window.alert
* console.log

语法入门，不如直接找个什么网站刷点题算了。

定义变量：`let name = var;`

定义函数：

```javascript
function func_name() {
}
```

注释：

```javascript
//single line
/*
multiple
lines
*/
function /* inline */ func_name() {}
```

let 与 const 定义的变量的作用域是 block.

而 var 是更古老的变量定义方法，可以看成是全局变量。

能用 let 尽量用 let 吧。

js 中的 const 并非 immutable，其定义的是一个不可改变的引用。大部分时候，我们定义 const 指向的是 literals.
但如果我们指向可变的对象，比如数组，那么数组本身是可以更改的。

js 的 operator 和 C 差不多。

datatypes 啊之类的也基本一样。

HTML 与 js 之间常常使用 event 建立联系。

```html
<button onclick="document.getElementById('demo').innerHTML = Date()">The time is?</button>
```

更常用的写法是调用自定义的函数。

```html
<button onclick="displayDate()">The time is?</button>
```

js 不区分单引号、双引号。

string interpolation，和 shell 有点类似。

```javascript
let firstName = "John";
let lastName = "Doe";

let text = `Welcome ${firstName}, ${lastName}!`;

let price = 10;
let VAT = 0.25;

let total = `Total: ${(price * (1 + VAT)).toFixed(2)}`;
```

js 中有个 NaN，not a number，但依然是 number 类型。

```javascript
let x = 1 / "a"; //NaN
isNaN(x); // True
typeof NaN; // number
let x = 1 / 0; // Infinity
typeof Infinity; // number
```

**比较两个 js 中的 object 永远返回 false**.
~~那啥，我不能重载 == operator 吗？
应该是说比较非 primitive type 默认返回 false 吧。~~
似乎还真不能重载 operator...

```javascript
let x = new Number(500);
let y = new Number(500);
x == y; // False
```

数组：

```javascript
const cars = ["Saab", "Volvo", "BMW"];
```

可以直接给未定义的下标赋值，数组会自动扩充、、、什么奇怪的特性。

再见，数据越界！

```javascript
const test = [];
test[0] = 1;
test[2] = 2;
test[1] = 4;
```

数组中的对象可以是不同类型，和 python 类似。

```javascript
const fruits = ["Banana", "Orange", "Apple"];
fruits.push("Lemon");  // Adds a new element (Lemon) to fruits
let last = fruits.pop();
```

Array 没有自己的类型，typeof 将会返回 object.
~~Everything is an object!~~
可以用 instanceof 来判断。

？？？Array 的 sort 方法默认按照 string 来比较。

要按数字排序得手写 cmp 函数。

```javascript
function(a, b){return a - b}
```

负数则 a 在 b 前，0 则不变，正数则 a 在 b 后。

Array 对象可以调用 forEach 方法。

```javascript
const numbers = [45, 4, 9, 16, 25];
let txt = "";
numbers.forEach(myFunction);

function myFunction(value, index, array) {
  txt += value + "<br>";
}
```

可以省略一两个参数。

还有一个 map 方法。

```javascript
const numbers1 = [45, 4, 9, 16, 25];
const numbers2 = numbers1.map(myFunction);

function myFunction(value, index, array) {
  return value * 2;
}
```

还有 filter：

```javascript
const numbers = [45, 4, 9, 16, 25];
const over18 = numbers.filter(myFunction);

function myFunction(value) {
  return value > 18;
}
```

reduce 则可以遍历整个 Array 来求某些结果。

```javascript
const numbers = [45, 4, 9, 16, 25];
let sum = numbers.reduce(myFunction, 0);

function myFunction(total, value, index, array) {
  return total + value;
}
```

还有 every、some 检查元素是否满足条件，includes 检查是否包含某元素。其他的函数要用再查吧。

惯例使用 const 定义数组。

if/else/else if 是 C 风格的。

switch case 也是。

for 也是。

没有 foreach，直接用 for. 这个时候似乎并不保证遍历顺序。用 forEach 保证顺序。

```javascript
const numbers = [45, 4, 9, 16, 25];

let txt = "";
for (let x in numbers) {
  txt += numbers[x];
}
```

还有一个 for of？？

* for in 遍历的是 enumrable property，对广泛的对象也可以使用。
* for of 只能遍历 iterable objects，更类似于其他语言的 for in.

总之以后尽量用 for of.

while 一整套跟 C 一样。其实 Control Flow 这一块除了 for in/of 之外都和 C 一样。

set map 两个常用数据结构。list vector 全部由 array 承担了。

typeof 只能停留在 object level，要判断具体的类型有一个奇怪的方法：

```javascript
function isArray(myArray) {
  return myArray.constructor === Array;
}
function isDate(myDate) {
  return myDate.constructor === Date;
}
```

无力吐槽。。。

类型转换用 Constructor. 也有 toString parseInt 之类的函数。

异常处理是 try catch throw finally 那一套。不过似乎总是写 `catch(err)` 再判断 `err.name`

js 有一个叫做 hoisting 的机制，会把各种定义提前到 scope top……但赋值不会。这个大部分情况下没啥影响。

在 scope top 写上 "use strict" 可以打开严格模式。

```javascript
x = 3.14;       // This will not cause an error.
myFunction();

function myFunction() {
  "use strict";
  y = 3.14;   // This will cause an error
}
```

```javascript
"use strict";
myFunction();

function myFunction() {
  y = 3.14;   // This will also cause an error because y is not declared
}
```

过于宽松可能导致出了 bug 还没报错……

JS 的 Arrow function，或者说 lambda.

```javascript
hello = () => {
  return "Hello World!";
}
hello = (val) => "Hello " + val;
hello = val => "Hello " + val;
```

JS 的 class:

```javascript
class Car {
  constructor(name, year) {
    this.name = name;
    this.year = year;
  }
  age() {
    let date = new Date();
    return date.getFullYear() - this.year;
  }
}

let myCar = new Car("Ford", 2014);
document.getElementById("demo").innerHTML =
"My car is " + myCar.age() + " years old.";
```

JS 的 module.

在 lib 中需要 export，类似于 public?

```javascript
export const name = 'square';

export function draw(ctx, length, x, y, color) {
  ctx.fillStyle = color;
  ctx.fillRect(x, y, length, length);

  return { length, x, y, color };
}
```

也可以集中 export

```javascript
export { name, draw, reportArea, reportPerimeter };
```

在需要用到的地方 import:

```javascript
import { name, draw, reportArea, reportPerimeter } from './modules/square.js';
```

HTML 中引入 module:

```html
<script type="module" src="main.js"></script>
```

还可以 export default. 默认只能有一个。

```javascript
const t = 1;
export default t;

import t from './module.js';
// Equals to import (default as t) from './module.js'
```

当然还有 as.

```javascript
const t = 1;
export {t as tt};

import tt from ...;

// inside module.js
export { function1, function2 };

// inside main.js
import {
  function1 as newFunctionName,
  function2 as anotherNewFunctionName,
} from './modules/module.js';
```

可以将 module import 成一个类似于 object 的形式：

```javascript
import * as Module from './modules/module.js';

Module.function1();
Module.function2();
```

可以逐层向上 export.

```javascript
export * from 'x.js'
export { name } from 'x.js'
```

其他的暂且跳过。

JSON，JavaScript Object Notation，众所周知。

```javascript
let text = '{ "employees" : [' +
'{ "firstName":"John" , "lastName":"Doe" },' +
'{ "firstName":"Anna" , "lastName":"Smith" },' +
'{ "firstName":"Peter" , "lastName":"Jones" } ]}';
const obj = JSON.parse(text);
text = JSON.stringify(obj);
```

js 可以在代码中直接下断点。

```javascript
let x = 15 * 5;
debugger;
document.getElementById("demo").innerHTML = x;
```

W3Schools 的 js 教程部分就结束了。

类 C 的风格很容易熟悉。

---

js 本身差不多了，接下来就要与已学的结合起来了。

js 与 html 交互需要通过 DOM(Document Object Model).

操作某个 element 首先需要找到它，可以用 css 中的 selector 进行筛选，也可以直接用 ID 之类的。

DOM 是一棵树，同样可以增删查改。具体的函数到时候查。

event 是可以动态绑定的。

```javascript
const btn = document.querySelector('#btn');
btn.onclick = () => alert("Hello World");

const btn = document.querySelector('#btn');
btn.addEventListener('click', () => {
  alert("Hello World");
});
```

几个 project 感觉都没啥意思，跳过了。

前端入门完成！庆祝一下，两天的功夫。

### Backend

Odin Project 的 Foundations 里面没有 Backend 的详细教程。后续的路就有很多分叉了。

比如 Ruby on Rails，Python Django 啊 Java 之类的。

按照我的技术栈，估计后续选择 Django 或者 Flask 吧。

这里暂且不提，先完成我的小项目。

## Day3

今天的小目标：写出小学数学计算训练器的 prototype.

一个极度简单的纯前端 single page application.

主要有两个 State: 选择模式与答题模式。

选择模式时，用户通过鼠标选择一个特定的训练模式，而后进入答题模式。

答题模式可以参考这个 UI:

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220819105200-8j9ldbu.png)​

具体的，选择答题模式实际上应该只是选择了一种 exprgen，将这个 function 传参后调用同一套代码就可以了。

在 js 端渲染数学公式，用上 KaTeX.

写 Mobile Version，补习一下 Grid 相关知识。

经过了几个小时的 ~~艰苦的~~ ~~面向 Google 的~~ 开发后，终于完成了。

由于不会写 Responsive，只能单独写个 mobile version...

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220819153901-j1dp31c.png)​

![image](https://assets.b3logfile.com/siyuan/1609132319768/assets/image-20220819153948-zyjjh00.png)​

上手练习：[https://clouder0.github.io/primary-math-trainer/src/](https://clouder0.github.io/primary-math-trainer/src/)

项目 Repo：[https://github.com/Clouder0/primary-math-trainer](https://github.com/Clouder0/primary-math-trainer)