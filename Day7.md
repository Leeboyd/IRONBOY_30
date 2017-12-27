Good Morning, JS (Day 7, 敲敲打打玩轉 function)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

前幾天討論過關於函數的 Input、Output，並稍微提到在 FP 中我們期望 `Arity = 1`，使用如 `destructuring` 和 `...` 讓 parameters 為 1，並談到函數的互相組合，或許你已經躍躍欲試地將你的程式碼重構，但你很快會發現現實中 parameters 的數量/順序/型別都不是那麼容易重構，現在，將進一步討論一些技巧將你的 `function` 更 `functional`。

## All for One (只要 1 個)
場合：將兩個函數組合，比如說把 A function 傳入 B function ，但此時 B function 傳入的參數跟 A function 數量不符合。比如說：

* B：map(...) 傳 3 個變數 `value`, `index`, `array`
* A：parseInt(...) 接收兩個變數 `string`, `radix`

`map(..)` 對一陣列的每個元素呼叫一個 mapper function，並傳入 3 個變數，如果使用 `parseInt(..)` 當作 mapper function，會：

![](https://i.imgur.com/obZy6k7.png)

### 技巧
設計一個單參數函數 `unary`，使得僅有一個參數能通過，如下方例子中多一層 `onlyOneArg` 的包裝：

```javascript=
function unary(fn) {
    return function onlyOneArg(arg){
        return fn( arg );
    };
}
```
很多 FP 開發者會寫成，ES6 `=>` 形式， 

```javascript=
var unary = 
  fn =>
    arg =>
      fn( arg )

// 或

var unary = (fn) => (arg) => fn( arg )
```
> 在此將各種寫法列出，供學習者比較，請自行在易寫與可讀性作權衡

`unary(..)` 利用了 `onlyOneArg` 包裝，阻擋第一參數外的參數通過，也就是例子中的 `index` 不會被傳進 `parseInt(..)` 了

![](https://i.imgur.com/8AfQ1YY.png)

## One on One (傳一返一)
在 FP 中有一種基礎函數：接受一個參數，然後原封不動返回，

```javascript=
function identical(v) {
  return v
}

// ES6 =>
var identical =
  v => 
    v

// 或

var identical = v => v
```
這到底有什麼用咧？

* 場合1：寫 leetcode 時，常會有拆分字串的題目，輸出的 Array 可能有空白，可以使用 `filter(..)` ，而且把上面 `identical(..)` 當成斷言判斷函數傳入：

```javascript=
var words = "   The JavaScript ecosystem is richer than ever...  ".split( /\s|\b/ );

console.log(words)

// ["", "", "", "The", "JavaScript", "ecosystem", "is", "richer", "than", "ever", "...", "", ""]

words.filter( identical )

// ["The", "JavaScript", "ecosystem", "is", "richer", "than", "ever", "..."]
```

`identical(..)` 將輸入值直接返回， JS 將每個值轉型 `coerces` 成 `true` 或 `false`，`filter(..)` 便可因此判斷每個值應該留下或排除。

### 💡 小提示
另一個常用的 unary 方法為 `Boolean(..)` ，作用是將傳入的值轉變為 `true` 或 `false`。

* 場合2：當成 default function 
```javascript=
function format(msg, formatFn = identical) {
    msg = formatFn( msg );
    console.log( msg );
}

function upper(txt) {
    return txt.toUpperCase();
}

format( "JavaScript", upper );     // JAVASCRIPT
format( "JavaScript" );            // JavaScript
```
還記得高階函數 Higher Order function 嗎？上例中 `identical` 即是當成 `formatFn` 的預設值。

## Unchanging One
有些 API 會規定只能傳入一個 function，禁止直接傳值，比如說 JS Promises 的 `then(..)`，
```javascript=
p1
.then( someHandlerFunc )
.then( function () {
  return p2;
} )
.then( someHandlerFunc );
```
ES6 使用者看到匿名函數就手癢想重構一下， 用的是 `=>`：
```javascript=
p1.then( someHandlerFunc ).then(() => p2).then( someHandlerFunc )
```
而做一個 FP 小工具來處理也不錯：

```javascript=
function constant (v) {
  return function value () {
    return v
  }
}

// 或 ES6 =>
var constant =
  v =>
    () =>
      v
// 或
var constant = v => () => v
```

再比較一下兩者的差異

```javascript=
p1.then( someHandlerFunc ).then(() => p2).then( someHandlerFunc )

// v.s.

p1.then( someHandlerFunc ).then( constant( p2 ) ).then( someHandlerFunc )
```

### 💡 比較
雖然 `() => p2` 語法較 `constant( p2 )` 簡潔，但其實仔細看， `=>` 直接 return 一個外部 scope 的變數 `p2`，在 FP 中稱之為副作用，與 FP 的理念有點矛盾，使用須謹慎。

## 小結：

在 FP 中，盡可能得設計單一參數的函數，用到了高階函數的概念，一層包層，目的都是為了方便 function 與 function 之間的結合，明日將繼續介紹使用 `destructuring`  和 `...` 來玩 `function`
