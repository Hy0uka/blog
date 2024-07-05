+++
title = '常见JavaScript混淆与反混淆'

description = "JavaScript混淆"

tags = [ "技术" ]

date = 2024-07-05T15:35:25+08:00

+++

JavaScript 混淆（Obfuscation）是指通过一系列技术手段，使 JS 代码变得难以理解和分析，增加代码的复杂性和混淆度，阻碍逆向工程和代码盗用。实际上就是一种保护 JS 代码的手段。JS最早被设计出来就是为了在客户端运行，直接以源码的形式传递给客户端，如果不做处理则完全公开透明，任何人都可以读、分析、复制、盗用，甚至篡改源码与数据，这是网站开发者不愿意看到的。

压缩工具开发的初衷是减小 JS 文件体积，但 JS 代码经过压缩替换后，其可读性也大大降低，间接起到了保护代码的作用。但是后来主流浏览器的开发者工具都提供了格式化代码的功能，压缩技术所能提供的安全保护收效甚微。于是专门保护 JS 代码的技术：JS 加密和 JS 混淆。

#### 常见混淆手段

变量名/函数名的替换，将有意义的变量名函数名替换为随机生成的名称

```
/*
function calculateArea(radius) {
  return Math.PI * radius * radius;
}
console.log(calculateArea(5));
*/
function _0x2d8f05(_0x4b083b) {
  return Math.PI * _0x4b083b * _0x4b083b;
}
console.log(_0x2d8f05(5));
```

字符串混淆，将代码中的字符串替换为编码或加密的形式，可以防止字符串被轻易读取。

```
// console.log("Hello, world!");
console.log("\x48\x65\x6c\x6c\x6f\x2c\x20\x77\x6f\x72\x6c\x64\x21");
```

控制流混淆，改变代码的执行顺序或结构。例如，可以使用条件语句和循环语句来替换简单的赋值操作。

```
/*
let a = 1;
let b = 2;
let c = a + b;
console.log(c);
*/
let a = 1;
let b = 2;
let c;
if (a === 1) {
  if (b === 2) {
    c = a + b;
  }
}
console.log(c);
```

花指令，即在源码插入一些不会被执行的代码。

```
/*
let a = 1;
let b = 2;
let c = a + b;
console.log(c);
*/
let a = 1;
let b = 2;
if (false) {
  console.log(a - b);
}
let c = a + b;
console.log(c);
```

代码转换，将代码转换为等价的，但更难理解的形式。

```
/*
let a = 1;
let b = 2;
let c = a + b;
console.log(c);
*/
let a = 1;
let b = 2;
let c = a - (-b);
console.log(c);
```

#### 常见反调试手段

实现防止他人调试、动态分析自己的代码，我们可以预先在代码中做处理，防止用户调试代码。

无限 debugger。比如写个定时器死循环禁止调试。

```
var c = new RegExp("1");
c.toString = function () {
    alert("检测到调试")
    setInterval(function() {
        debugger
    }, 1000);
}
console.log(c);
```

内存耗尽。更隐蔽的反调试手段，代码运行造成的内存占用会越来越大，很快会使浏览器崩溃。

```
var startTime = new Date();
debugger;
var endTime = new Date();
var isDev = endTime - startTime > 100;
var stack = [];

if (isDev) {
    while (true) {
        stack.push(this);
        console.log(stack.length, this);
    }
}
```

检测函数、对象属性修改。攻击者在调试的时，经常会把防护的函数删除，或者把检测数据对象进行篡改。可以检测函数内容，在原型上设置禁止修改。

```
function eval() {
    [native code]
}

window.eval = function(str) {
    console.log("[native code]");
};

window.eval = function(str) {
};

window.eval.toString = function() {
    return `function eval() {[native code]}`
};

function hijacked(fun) {
    return "prototype" in fun || fun.toString().replace(/\n|\s/g, "") != "function" + fun.name + "() {[nativecode]}";
}
```

#### 在线混淆工具

[javascriptobfuscator.dev]: https://javascriptobfuscator.dev/

#### 反混淆

JS 反混淆（Deobfuscator ）是指对经过混淆处理的代码进行还原和解析，以恢复其可读性。Deobfuscator 可以通过对代码进行静态分析和动态分析等方式来实现。需要注意的是，Obfuscation 只能降低可读性，不能完全避免逆向攻击，而 Deobfuscator 也并不能完全还原混淆过的代码。

#### 在线反混淆工具

[javascript-deobfuscator]: https://droneshakti.org/
[Raz1ner JavaScript Deobfuscator]: https://dev-coco.github.io/Online-Tools/JavaScript-Deobfuscator.html
[synchrony deobuscator]: https://deobfuscate.relative.im/
[js-beauty]: https://beautifier.io/

以及浏览器前端中的Source窗口，配合console.log（）函数非常好用。

目前的需求是，一段经过混淆的恶意代码，能否直接在前端执行出解混淆后的结果。目前认知来看只有编码类混淆可以，而且存在执行到C2的风险。
