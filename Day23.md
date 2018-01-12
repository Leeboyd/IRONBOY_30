Good Morning, Functional JS (Day 23, Immutability 值不可變性)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)


## 基本型別的值不可變性 Immutability
在 JS 中，基本型別如 `number`, `string`, `boolean`, `null`, `undefined` 具有值不可變性：

```
// 不合法，也沒有用
4 = 2.4
```
> number 的值不可變性

JS 的初學者很容易有個誤解（含我本人） - `string` 和 `array` 很像，所以應該是可變的，例如同樣可以用 `[]` 索引的方式去訪問元素，實際上 `string` 也具值不可變性：

```
var msg = 'Good morning'

msg[5]
// => 'm'

msg[5] = 'M'

msg.length = 10

msg
// => 'Good morning'
```
> string 的值不可變性

儘管可以用 msg[5] 取值，`string` 跟 `array` 還是不同，陣列操作：`msg[5] = 'M'`、`msg.length = 10` 這兩個賦值動作都失敗。

在嚴格模式 `"use strict"` 下，這些賦值操作也都無效，因為這兩個屬性都是唯讀的。

## 值不可變性與副作用
再繼續之前，關於值不可變性的討論，並不是說我們不能在程式裡改變某個值，在 FP 裡，我們借用值不可變性的精神，當需要改變程式中的狀態時，我們不改變已存在的值，我們必須創造並追蹤新值。

```
var s = [1,2,3];

function addValue(arr) {
    var newArr = [ ...arr, 4 ];
    return newArr;
}

console.log(addValue( [1,2,3] ));    
// [1,2,3,4]

console.log(s)
// [1,2,3]
```

記得之前關於副作用的分析嗎？`addValue` 是純函數嗎？
* 給定相同輸入，會有相同輸出嗎？
* 是否具有引用透明性
* 無副作用？
答案都是肯定的。

在 `addValue` 內，透過創造一個新陣列 newArr，不改變 `arr` 引用的原陣列 `s` 來避免副作用。

`arr` 引用的陣列型別是可變的，但我們選擇不改變它，這就是值不可變性 Immutability 的精神。

## ES6 `const` 常數宣告
關於常數，我們嚷嚷上口：一個不能被改變的值、一個不能被改變的變數...之類的。這些答案很接近，但不夠精確。

比較精確的說法是：一個無法被重新賦值 **reassignment** 的變數。

也就是無論常數的值為何，無論值是否具值不可變性，**與值的特性無關**。

下面的範例，使用陣列來說明 [^1]：

```
const x = [ 2 ];

x = 1  // TypeError
```

x 是一個常數，無法被重賦值，但 [ ] 是常數嗎？很明顯不是，下面的動作完全可行：

```
x[0] = 3;

console.log(x)

// [3]
```
儘管用 const 宣告 x，但以結果來看 x 還是被改變了，因為陣列值是可變的。

![](https://i.imgur.com/4BxzkJD.png)
如果 const 宣告並不能保證值不可變性，那對 FP 開發者又有什麼意義呢？

再回頭看看 `const x = [ 2 ];` 想表達的是什麼？

它強烈地暗示讀者這個陣列在程式接下來都不會被修改，如果發現 x 在某個地方被更動了，難道他們不會驚訝嗎？

又或者，我們在某個地方寫了副作用影響到 x ，他們是不是會認為 x 仍然是 [2]，因為他們猜測我用 const 宣告的目的：**這個變數不會更新**。

很多時候 bug 的產生不是因為變數被重賦值，而是沒觀察到值被改變，const 並不是那麼的有用，它並不能保證什麼，

```
const arr = [1,2,3];

somefunc( arr );

console.log( arr[0] ); // ???
```
> const 無法保證值不變


真的要用，就用 const 來宣告一些簡單的常數吧：

```
const PI = 3.141592
```

## Object.freeze
有一種簡單的方法，可以將 `object/array/function` 轉變成不可變（まあまあ 勉勉強強還行）
```
var x = Object.freeze([x])
```

* `Object.freeze` 會走訪 `object/array` 每個屬性和索引，將他們設置為唯讀
* `Object.freeze` 會將 `object/array` 標記為 **non-reconfigurable**，也就是無法被新增或刪除屬性。任何想要改動的嘗試都會失敗, 要不沈默地失敗, 就是丟出一個 TypeError 例外 (此例外最常出現在 strict mode)[^2]

不過，凍結的動作是「淺」的 - 如果資料屬性也是物件，不會遞迴地做凍結的動作

```
obj1 = {
  internal: {}
};

Object.freeze(obj1);

// 無效
obj1.internal = 'aValue';

// 允許改變
obj1.internal.a = 'aValue'
```
> 淺層凍結

如果要深層凍結，就必須手動走訪全部子陣列/子物件套用 `Object.freeze` 

回到剛剛的例子
```
var arr = Object.freeze([1,2,3]);

somefunc( arr );

console.log( arr[0] ); // ???
```
我現在可以很確定答案是 1，`Object.freeze` 會返回一個真的不可變的值。就算把他傳入一個我們不能觀察的地方，依然可以相信這個值不會變。

## 參考資料
^1: [Reassignment](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch6.md#reassignment)

^2: MDN: [Object.freeze()](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
