



### 方法说明

`Object.is()` 方法用来判断两个值是否相等，它接收两个参数，分别是需要比较的两个值。

返回一个 `Boolean` 值标示这两个值是否相等。

```javascript
Object.is(123, 123);  // true
Object.is(123, '123');  // false
Object.is([], []);  // false
Object.is(NaN, NaN); // true
```



我们在开发中，基本上都是使用 `==` 与 `===` 来判断两个值是否相等，可能有的人会问了，他们与 `Object.is()` 有什么区别呢？

#### 与 "==" 比较

​		我们都知道 `==` 比较两个值是否相等，如果两边的值不是同一个类型的话，会将他们转为同一个类型后再进行比较。

```javascript
123 == '123';   // true
'' == false;    // true
false == 0;     // true
NaN == NaN;     // false
```

  		而 `Object.is()` 不会对要比较的内容的类型进行转换

```javascript
Object.is(123, 123);  // true
Object.is(123, '123');  // false
Object.is(undefined, undefined);   // true
Object.is(0, false);    // false
```



#### 与 “===” 比较

​		`===` 不会对类型进行转换，两边的值必须相等且类型相同才会等到 `true` 。

```javascript
123 === 123;     // true
123 === '123';   // false
'' === false;    // false
false === 0;     // false
NaN === NaN;     // false
```

​		需要特殊注意的是，对于 `0` 和 `NaN` 的比较。无论 `0` 的正负，他们都是相等的，而 `NaN` 是与任何值都不相等的，包括他本身。

```javascript
+0 === -0;  // true
0 === -0;  // true
+0 === 0;  // true
NaN === NaN;  // false
```

​		而 `Object.is()` 会将 `NaN` 与 `NaN` 视为相等，无符号的 `0` 归为整数。

```javascript
Object.is(0, +0);    // true
Object.is(-0, 0);    // false
Object.is(-0, +0);    // true
Object.is(NaN, NaN);    // true
```



#### 对于 `引用类型` 的值的比较

JavaScript的数据类型分为 `值类型` 和 `引用类型` 两种：

- 值类型：字符串（string）、数值（number）、布尔值（boolean）、undefined、null
- 引用类型：对象（Object）、数组（Array）、函数（Function）

**区别：**

- 值类型
  - 占用空间固定，保存在栈中
  - 保存与复制的是值本身
  - 使用typeof检测数据的类型

- 引用类型
  - 占用空间不固定，保存在堆中
  - 保存与复制的是指向对象的一个指针
  - 使用instanceof检测数据类型
  - 使用new()方法构造出的对象是引用型

来看几个例子：

```javascript
const a = {text: '123'};
const b = {text: '123'};
const c = a;

Object.is(a, b);  // false
Object.is(a, c);  // true
```

这与 `==` 和 `===` 的结果一致的，因为 a 与 b 不相等因为他们指向了不同的地址，c 是由a赋值而来的，他们指向的是同一个地址，因此是相等的。

```javascript
c.name = "hello";
Object.is(a, c);  // true

console.log(c);   // {text: "123", name: "hello"}
console.log(a);   // {text: "123", name: "hello"}
```

即使我们在后面对 c 的属性进行一些修改，它与 a 仍然是相等的。因为修改的是 a 与 c 共同指向的那个地址的值。



### 如何实现的

`Object.is()` 除了 `IE` 浏览器不支持，其他的浏览器全部支持，如果要需要兼容IE的话，需要做一些兼容处理。

```javascript
if (!Object.is) {
    Object.is = function(x, y) {
        if (x === y) {
            // 判断0与+0
            return x !== 0 || 1/x === 1/y;
        } else {
            // 判断是否都为 NaN
            return x !== x && y !== y;
        }
    }
}
```

通过与 `===` 的比较我们知道，他只在 `0` 和 `NaN` 上的判断有所区别，我们只需要处理这这个特殊情况即可。

- 先判断 `x===y` ，为 `true` 时说明其中没有 `NaN`；如果x为 `+0` 或 `0` ，则 `+0 !== 0` 为 `false`，需要继续判断  `1/x === 1/y`；如果 x 或 y 其中有一个是 `-0` ，则运算后为 `-Infinity === Infinity` ，返回 `false`。
-  如果 `x!==y` 包含一个特殊情况，就是两个都为 `NaN` 的时候应该返回 `true`；因为 `NaN` 是与任何值都不相等的，包括它本身，那么只要 x 和 y 都不等于他们本身，就说们他们都是 `NaN` 。



















