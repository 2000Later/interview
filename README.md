## 前端面试题

### 1.预编译习题 [文章地址](https://zhuanlan.zhihu.com/p/50236805)

```js
function fn(a, c) {
  console.log(a); // function a() {}
  var a = 123;
  console.log(a); // 123
  console.log(c); // function c() {}
  function a() {}
  if (false) {
    var d = 678;
  }
  console.log(d); // undefined
  console.log(b); // undefined
  var b = function () {}; // 这里b声明进行了域解析
  console.log(b); // function () {}
  function c() {}
  console.log(c);
}
fn(1, 2);

// 预编译
// 作用域的创建阶段 预编译阶段
// 预编译的时候做了哪些事情
// js的变量对象 AO对象 供js引擎自己去访问的
// 1 创建AO(Activation Object)对象
// 2 找形参和变量的声明 作为AO的属性名 值是undefined
// 3 实参和形参相统一
// 4 找函数声明 会覆盖变量声明
// GO (Global Object)
//
AO: {
  a: undefined 1 function a() {}
  c: undefined 2 function c() {}
  b: undefined
  d: undefined
  b: undefined
}
// js的解析执行 逐行执行
```

### 2. js this 指向

#### 在函数中直接使用

```js
function get(content) {
  console.log(content);
}
get("您好");
get.call(window, "您好");
```

#### 函数作为对象的方法被调用(谁调用我 我就指向谁)

```js
 var person = {
   name: '张三',
   run(time) {
     console.log(`${this.name} 在跑步 最多${time}min就不行了`)
   }
   person.run(30)
   person.run.call(person, 30)
 }
```

```js
// 1.函数的直接调用 this指向window
// 2. 作为对象的方法调用 this指向该对象
// 阿里笔试题
var name = 222;
var a = {
  name: 111,
  say() {
    console.log(this.name);
  },
};
var fun = a.say;
fun(); // fun.call(window) 222
a.say(); // a.say.call(a)  111

var b = {
  name: 333,
  say(fun) {
    fun(); // fun.call(window)
  },
};
b.say(a.say);
b.say = a.say;
b.say();
```

#### 箭头函数中的 this

- 箭头函数中的 this 是定义在函数的时候绑定的，而不是在执行函数的时候绑定。
- 箭头函数中，this 执行的固定化，并不是因为箭头函数内部有绑定 this 的机制，实际原有是箭头函数根本没有自己的 this,导致内部的 this 就是外层的代码块的 this。正是因为它没有 this。所以就**不能用作构造函数**
- 箭头函数中的 this 是定义函数的时候绑定

```js
var x = 11;
var obj = {
  x: 22,
  say: () => {
    console.log(this.x);
  },
};
obj.say(); // 11
```

- 所谓的定义时候绑定，就是 this 是继承自父执行上下文中的 this，比如这里的箭头函数中的 this.x，函数本身与 say 平级以 key:value 的形式，也就是箭头函数本身所在的对象为 obj，而 obj 的父执行上下文就是 window，因此这里的 this.x 实际上表示 window.x，因此输出的是 11

```js
var obj = {
  birth: 1990,
  getAge() {
    var b = this.birth;
    var fn = () => new Date().getFullYear() - this.birth;
    return fn();
  },
};
obj.getAge(); // 31
```

- 例子中箭头函数本身是在 getAge 方法中定义的，因此，getAge 方法的父执行上下文是 obj，因此这里的 this 指向则为 obj

### 3. 浅拷贝 深拷贝

#### 目标

1. 什么是浅拷贝
2. 实现的方式
3. 在 vue 中的使用

#### 前置知识

- js 的一般数据类型的存储
- js 的引用类型的数据存储

- 浅拷贝是创建一个新对象，这个对象有着原始的对象属性的一份精确拷贝。**如果属性是基本类型，拷贝的就是基本类型的值**，**如果属性是引用类型，拷贝的就是内存地址**，所以如果其中一个对象改变了这个地址，就会影响到另一个对象。

- 深拷贝是将一个对象从内存中完整的拷贝一份出来，从堆内存开辟一个新的区域存放新对象，且修改新对象不会影响原对象。

#### 针对引用类型来说 赋值 深拷贝 浅拷贝的区别

1. 浅拷贝 赋值的区别

- 当我们把一个对象赋值给一个新的变量时，赋值的其实是该对象的在栈中的地址，而不是堆中的数据。也就是两个对象指向的是同一个存储空间，无论哪个对象发生改变，其实都是改变的存储空间的内容，因此，两个对象是联动的。

- 浅拷贝：重新在堆中创建内存，拷贝前后对象的基本数据类型互不影响，但拷贝前后对象的引用类型共享一块内存，会相互影响。
- 深拷贝：从堆内存中开辟一个新的区域存放对象，对对象中的子对象进行递归澳贝，拷贝前后的两个对象互不影响。

#### 深拷贝的实现方式

- JSON.parse(JSON.stringify())：只能序列化对象的可枚举的自有属性，例如 如果 obj 中的对象是有构造函数生成的， 则使用 JSON.parse(JSON.stringify(obj))深拷贝后，会丢弃对象的 constructor；
- 递归操作
- deepClone
- Jquery.extend()

date: new RegExp('\\w+')

data: [new Date(1625501533431), new Date(1625501533431)]

| Javascript 拷贝 | 和原数据是否指向同一对象 | 第一层数据为一般数据类型       | 第一次数据不是原始类型         |
| --------------- | ------------------------ | ------------------------------ | ------------------------------ |
| 赋值            | 是                       | 改变会使原始数据一同改变       | 改变会使原始数据一同改变       |
| 浅拷贝          | 否                       | 改变不会使原始数据类型改变     | 改变会使数据一同改变           |
| 深拷贝          | 否                       | 改变不会使原始数据类型一同改变 | 改变不会使原始数据类型一同改变 |

### 4. 防抖节流

当持续触发事件 一定时间内没有再触发事件 事件处理函数才会执行一次。如果设定的时间到来之前 又一次触发了事件 就重新开始延时

触发事件 一段时间内 没有触发 事件执行 肯定是定时器
(在设定的时间内 又一次触发了事件 重新开始延时 代表的就是重新开始的定时器)
(那么意味着上一次还没有结束的定时器要清除掉 重新开始)

```js
let timer;
clearTimeout(timer);
timer = setTimeout(function () {}, delay);
```

#### 实际应用

使用 echarts，改变浏览宽度的时候，希望重新渲染。
echarts 的图像，可以使用此函数，提升性能。(虽然 echarts 里有自带的 resize 函数)
典型的案例就是输入搜索：输入结束后 n 秒才进行搜索请求，n 秒内又输入的内容, 就重新计时。
解决搜索的 bug， 拿到键盘停下那一刻输入的值进行搜索

#### 节流

当持续触发事件的时候 保证一段时间内 只调用一次事件处理函数
一段时间内 只做一件事情

#### 实际应用 表单的提交

典型的案例就是鼠标不断点击触发按钮，规定在 n 秒内多次点击只有一次生效。

### 4. js 的作用域

#### 作用域说明：一般理解指一个变量的作用范围

1. 全局作用域
   (1) 全局作用域在页面打开时被创建,页面关闭时被销毁
   (2) 编写在 script 标签中的变量和函数,作用域为全局,在页面的任意位置都可以访问到
   (3) 在全局作用域中有全局对象 window,代表一个浏览器窗口,由浏览器创建，可以直接调用
   (4) 在全局作用域中声明的变量和函数会作为 window 对象的属性和方法保存

2. 函数作用域
   (1) 调用函数时,函数作用域被创建,函数执行完毕,函数作用域被销毁
   (2) 每调用一次函数就会创建一个新的函数作用域,他们之间是相互独立的
   (3) 在函数作用域中可以访问到全局作用域的变量,在函数外无法访问函数作用域内的变量
   (4) 在函数作用域中访问变量、函数时,会先在自身作用域中寻找,若没有找到,则会到函数上一级作用域中寻找,一直找到全局作用域

#### 作用域的深层次理解

- 执行期的上下文

  - 当函数执行的前期 会创建一个执行期上下文的内部对象 AO (作用域)
  - 这个内部的对象是预编译的时候创建出来的 因为当函数被调用的时候 会先进行预编译
  - 在全局代码执行的前期会创建一个执行期的上下文的对象 GO

- 有关 js 的预编译

#### 函数的作用域预编译

1. 创建 ao 对象 AO{}
2. 找形参和变量声明 将变量和形参名 当作 AO 对象的属性名 值为 undefined
3. 实参形参相统一
4. 在函数体里面找到函数声明 值赋予函数体

#### 全局作用域的预编译

1. 创建 GO 对象
2. 找到变量声明 将变量名作为 GO 对象的属性名 值是 undefined
3. 找函数声明 值赋予函数体

- 作用域链
  - 会保护到一个隐式的属性中去 [[scope]] 这个属性是我们用户访问不到的 但是的的确确存在 让 js 引擎来访问 里面存储的就是作用域链

#### 闭包的形成

1. 基本理解：a 函数内返回 b 函数,b 函数可以访问 a 函数中的变量
2. 深入理解：函数 a 执行时创建了 a 作用域的 AO 对象，定义了 b 函数,**并把 b 函数返回(返回的时候能拿到 a 函数的 AO 对象)**,执行完 a 函数作用域链被销毁。b 函数保存在外面,执行时创建 AO 对象,由于 b 函数还存在,所以能够访问 a 函数的 AO 对象(前面是 a 函数的作用域链被销毁,但是在 b 函数中依然能够访问到 a 函数的 AO 对象)。

### 5. event-loop(事件循环)

1. js 语言的特点：单线程、解释性语言
2. 事件循环机制由三部分组成：调用栈 微任务 消息队列
3. event-loop 执行过程: 刚开始的时候 会从全局一行一行的执行 遇到函数调用 会压入到调用栈 被压入的函数被称之为帧 当函数返回后会从调用栈中弹出

```js
function fun1() {
  console.log(1);
}
function fun2() {
  console.log(2);
  fun1();
  console.log(3);
}
fun2();
```

js 中的异步操作 比如 fetch setTimeout setInterval 压入到调用栈中的时候里面的消息进入到消息队列中去 消息队列中会等到调用栈清空之后在执行

```js
function func1() {
  console.log(1);
}
function func2() {
  setTimeout(() => {
    console.log(0);
  }, 0);
  func1();
  console.log(3);
}
func2();
```

promise async await 异步操作的时候会加入到微任务中去 会在调用栈清空的时候立即执行
调用栈加入的微任务会立马执行 **微任务的优先级大于消息队列**

```js
var p = new Promise((resolve) => {
  console.log(4);
  resolve(5);
});

function func1() {
  console.log(1);
}

function func2() {
  setTimeout(() => {
    console.log(2);
  }, 0);
  func1();
  console.log(3);
  p.then((resolve) => {
    console.log(resolve);
  });
}
func2();
```

### 6. 单例模式的理解

定义：1 只有一个实例 2 可以全局的访问
主要解决：一个全局使用的类 频繁的创建和销毁
何时使用：当你想控制实例的数目 节省系统化资源的时候
如何实现：判断系统是否已经有这个单例 如果有则返回 没有则创建
单例模式的优点：内存中只要一个实例 减少了内存的开销 尤其是频繁的创建和销毁实例(比如说首页页面的缓存)

使用场景：1 全局缓存 2 弹窗

#### ES5 实现单例模式

利用闭包缓存变量

```js
function createLogin(a, b, c) {
  console.log(a, b, c);
  var div = document.createElement("div");
  div.innerHTML = "我是登录弹窗";
  div.style.display = "none";
  document.body.appendChild(div);
  return div;
}
// 单例的职责
function getSingle(fn) {
  var result;
  return function () {
    return result || (result = fn.apply(this, arguments));
  };
}
var create = getSingle(createLogin);
// 多次点击只能创建一个，因为闭包中已经缓存了
document.getElementById("loginBtn").onclick = function () {
  var loginLay = create(1, 2, 3);
  loginLay.style.display = "block";
};
```

#### ES6 实现单例模式

利用 static 缓存实例

```js
class singleClass {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  static getInstance(name, age) {
    // 静态方法, 只能由类本身使用,不会被实例化
    if (!this.instance) {
      // 如果存在就不再被创建了
      this.instance = new singleClass(name, age);
    }
    return this.instance;
  }
}
let lyCom = singleClass.getInstance("xiao", 12);
let lyCom2 = singleClass.getInstance("da", 18);
console.log(lyCom === lyCom2); // true
```

### 7. 策略模式

- 策略模式的定义
  1. 定义一系列的算法 把它们封装起来 并且他们之间可以相互替换
  2. 核心将算法的使用 和 算法的实现分离出来

### 8. 发布订阅

发布-订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变的时，所有依赖于它的对象都将得到通知 **先订阅再发布**

#### 作用

1. 支持简单的广播通信，当对象状态发生改变时，会自动通知已经订阅过的对象。
2. 可以应用在异步编程中 替代回调函数 可以订阅 ajax 之后的事件 只需要订阅自己需要的部分(那么 ajax 调用发布之后订阅的就可以拿到消息了)(不需要关心对象在异步运行时候的状态)
3. 对象之间的松耦合 两个对象之间都互相不了理解彼此 但是 不影响通信 当有新的订阅出现的时候 发布的代码无需改变 同样发布的代码改变 只要之前约定的事件名称没有改变 也不影响订阅
4. vue react 之间实现跨组件之间的传值

#### 发布订阅模式实现

1. 首先想好谁是发布者
2. 给发布者添加一个缓存列表，用于存放回调函数来通知订阅者
3. 最后就是发布消息，发布者遍历这个换成列表，依次触发里面存放的订阅者回调函数

```js
var Pubsub = (function () {
  var subObj = {};
  return {
    // 发布
    publish() {
      var key = Array.prototype.shift.call(arguments);
      var _pubs = subObj[key];
      if (!_pubs) return;
      for (var i = 0, fn; (fn = _pubs[i++]); ) {
        fn.apply(this, arguments);
      }
    },
    // 保存订阅的消息
    subscribe(type, handler) {
      (subObj[type] || (subObj[type] = [])).push(handler);
    },
    remove(type, handler) {
      // 没有参数移除所有
      if (arguments.length === 0) {
        subObj = {};
      } else if (arguments.length === 1) {
        // 移除相应的类型
        subObj[type] = [];
      } else if (arguments.length === 2) {
        // 移除单个消息
        var _events = subObj[type];
        if (!_events) return;
        for (var i = 0, fn; (fn = _events[i++]); ) {
          if (fn === handler) {
            _events.splice(i, 1);
          }
        }
      }
    },
  };
})();
// 订阅
Pubsub.subscribe("eat", function (arg) {
  console.log("订阅", arg);
});
Pubsub.subscribe("eat", function (arg) {
  console.log("订阅", arg);
});
Pubsub.subscribe("play", function (arg) {
  console.log("打游戏", arg);
});
Pubsub.publish("eat", "饭"); // 发布
Pubsub.remove("play");
Pubsub.publish("play", "玩"); // 发布
```

### 9. 数组扁平化

- 方法一：使用 falt(n)

```js
var arr = [1, [2, 3, [4], 5], 6, [7], [8, [9], [10]], 11];
console.log(arr.flat(Infinity));
```

- 方法二：利用正则 /\[|\]/g 全部匹配 [ 或者 ] 替换为空字符 数字类型会变为字符串！！！

```js
var arr = [1, [2, 3, [4], 5], 6, [7], [8, [9], [10]], 11];
console.log(JSON.stringify(arr).replace(/\[|\]/g, "").split(","));
```

- 方法三：正则改良版

```js
var arr = [1, [2, 3, [4], 5], 6, [7], [8, [9], [10]], 11];
console.log(JSON.parse(`[${JSON.stringify(arr).replace(/\[|\]/g, "")}]`));
```

- 方法四：reduce+递归

```js
const flat = (arr) =>
  arr.reduce(
    (pre, cur) =>
      Array.isArray(cur)
        ? (pre = [...pre, ...flat(cur)])
        : (pre = [...pre, cur]),
    []
  );
```

- 方法五：循环+递归

```js
const flat = (arr, target = []) => {
  for (let i = 0, item; (item = arr[i++]); ) {
    Array.isArray(item) ? flat2(item, target) : target.push(item);
  }
  return target;
};
console.log(flat2(arr));
```

### 10.BFC

#### BFC的理解 
  块级格式化上下文，它是指一个独立的块级渲染区域，只有 Block-level BOX(块级元素) 参与，该区域拥有一套渲染规则来约束块级盒子的布局，且与区域外部无关
#### 没有形成BFC的现象
- 一个盒子不设置height, 当内容子元素都浮动时，无法撑起自身
- 这个盒子没有形成BFC
#### 触发BFC的方法:
  1. float 的值不为 none
  2. position 的值不是 static 或者 relative
  3. display 的值是 inline-block、flex 或者 inline-flex
  4. overflow: hidden;

- BFC 的其他作用：
  - BFC 可以取消盒子的 margin 塌陷
  - BFC 可以阻止元素被浮动元素覆盖
