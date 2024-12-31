[toc]

# 1 变量

## 变量声明

```javascript
let age = 18;
```



## 变量类型

- JavaScript 中有**八种**基本的数据类型

  - 七种原始数据类型：
    - `number` 用于任何类型的数字；整数或浮点数
    - `bigint` 用于任意长度的整数
    - `string` 用于字符串
    - `boolean` 用于 `true` 和 `false`。
    - `null` 用于**未知**的值
    - `undefined` 用于**未定义**的值
    - `symbol` 用于唯一的标识符
  - 一种非原始数据类型：
    - `object` 用于更复杂的数据结构



## 运算符

- `+`，`-`，`*`，`/`，`%`，`**`
- `=`，`+=`，`-=`，`*=`，`/=`，`%=`，`**=`
- `==`，`===`，`!=`，`!==`，`>`，`>=`，`<`，`<=`
  - 如果<u>*数据类型不同*</u>，`==` 和 `!=` 会尝试转为相同的数据类型，而 `===` 和 `!==` 会直接返回 false
- `&&`，`||`，`!`
- `+` (字符串拼接)

- 三元运算符：

  ```javascript
  (company == 'Netscape') ? alert('Right!') : alert('Wrong.');
  ```

- `typeof`：查看数据类型

  > `typeof null` 会返回 `"object"` —— 这是 JavaScript 编程语言的一个错误，实际上它并不是一个 `object`



## 字符串

```javascript
let str = 'Hello, ' + 'world!';

alert(`1 + 2 = ${sum(1, 2)}.`);
```





# 2 向控制台输出

```javascript
console.log('Hello, world!'); // 向控制台输出

alert('Hello, world!'); // 在页面中弹窗输出
```





# 3 集合

## 数组

```javascript
// 数组声明
let arr = [1, 2, 3];

// 增
arr.push(4) // 在尾部添加
arr.unshift(0) // 在头部增加

// 删
arr.pop(); // 在尾部删除
arr.shift(); // 在头部删除

// 访问元素
arr[1]
```

## 字典

```javascript
// 字典声明
let person = { name: "John", age: 30 };

// 增
map.set(key, value);

// 访问元素
map.get(key)
```





# 4 流程控制

## `if-else if-else`

```javascript
if (year < 2015) {
  alert( 'Too early...' );
} else if (year > 2015) {
  alert( 'Too late' );
} else {
  alert( 'Exactly!' );
}
```

## `for`

```javascript
for (let i = 0; i < 3; i++) {
  alert(i);
}

// for ... of ... 用于数组
const arr = [1, 2, 3];
for (const value of arr) {
  console.log(value);
}

// for ... in ... 用于字典
const obj = { a: 1, b: 2, c: 3 };
for (const key in obj) { // 访问键; 但不可以访问值
  if (obj.hasOwnProperty(key)) { 
    console.log(key, obj[key]);
  }
}
```

## `while`

```javascript
while (i < 3) {
  alert( i );
  i++;
}
```





# 5 函数

## 函数定义

```javascript
// 函数声明
function functionName(params) {
  // 函数体
};

// 函数表达式
var functionName = function(params) {
  // 函数体
};
```

## 函数调用

```javascript
sayHello("World");
```

## 箭头函数

- 箭头函数中没有 `this`，`this` 为函数的调用者

```javascript
let func = (arg1, arg2, ..., argN) => expression;
```





# 6 对象

## 对象声明

```javascript
// 字面量
let user = {     
  name: "John", 
  age: 30  
};

// 构造函数
function User(name) { // 构造函数本身与普通函数的声明方式一致, 结合new操作符进行对象声明
  this.name = name;
  this.age = age;
}
let user = new User("John", age)
```

## 继承与原型

- 如果在当前对象中<u>*找不到某个类属性*</u>，那么会沿着 `__proto__` (原型链) 查找





# 7 模块

- 一个 `.js` 文件是一个**模块**

- **`export`**

  ```javascript
  // 在module.js中导出
  export const name = 'Alice';
  export function sayHello() {
    console.log('Hello!');
  }
  
  // 在use.js中导入
  import { name, sayHello} from './module.js'; // 必须使用{}包裹
  ```

  **`default export`**：一般当一个 `.js` 文件中<u>*只导出一个*</u>时使用

  ```javascript
  // 在module.js中导出; 只导出一个函数
  export default function sayHello() {
    console.log('Hello!');
  }
  
  // 在use.js中导入
  import sayHello from './module.js'; // 不需要{}
  ```

  

