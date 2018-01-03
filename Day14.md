Good Morning, Functional JS (Day 14, Pointfree 無點風格)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## Point-free style

**Point-free**（又寫成 Pointfree，中文：無參數，無點），正式名稱為：**tacit programming**，其中的 **point**（點）指的就是函數的 **parameter**（形式參數）。

寫在前面，Pointfree 透過隱藏 **parameter** - **argument** （**形參** - **實參**對應），減少視覺上的干擾，上層操作不直接操作數據，只合成運算過程，不過先提醒讀者，並非任何情況都適合 Pointfree，在本系列一直提到的：請**權衡可讀性**。

### Pointfree 改寫相同簽名的範例

```
function double(x) {
    return x * 2
}

[1,2,3,4,5].map( function mapper( v ){
    return double( v );
} )

// [2,4,6,8,10]
```
> 範例1 一般寫法（含有 Point 點）

`double` 與 `mapper` 有相同的函數簽名 (signature) ，`mapper`**形參（v）** 可以直接對應到 `double` **實參（v）**，因此我們可以去掉 `mapper` 包裝，改寫成 Pointfree：

```
[1,2,3,4,5].map( double )
```

### Pointfree 改寫不同簽名的範例

回顧 [Day 7](https://ithelp.ithome.com.tw/articles/10194258)

```
["1","2","3"].map( function mapper(v){
    return parseInt( v );
} );
```
> 範例2

`mapper` 使得僅有一個參數能通過，排除 `map(..)` 傳入的 `index` ，避免 `parseInt` 錯把 `index` 當 `radix` 進行整數解析，還記得我們使用 `unary` 小工具進行處理如下：

```
// 小工具
// var unary = (fn) => (arg) => fn( arg )


["1","2","3"].map( unary( parseInt ) )
```
> 範例2' Pointfree

邏輯：我們藉由 `unary` 小工具，提取第一個形式參數，對映到 parseInt 的第一個實參，再如同範例一改寫成 Pointfree 風格，將不同簽名的`map(..)` 與 `parseInt` 組合。

### 注意
還記得 [Day 10](https://ithelp.ithome.com.tw/articles/10194837) 的 **partialRight** 嗎？或許你想這樣使用

```
map(partialRight(parseInt,10)) 
```

將 10 **right-partially apply** (右偏應用) 在 `parseInt` 的 `radix`，但請注意 `map(..)` 本身會傳 3 個參數 `value`, `index`, `array`，所以 10 會被當成第四個參數傳入 `parseInt`，**partialRight** 相關討論請看 [Day 10](https://ithelp.ithome.com.tw/articles/10194837)。

## Pointfree 的本質

FP 的目的就是透過組合基本的函數，完成複雜的工作，記得 `樂高積木` 的概念嗎，而 Pointfree 風格就是組合函數最好的體現。在看另一個例子：

定義一些基本函數：

```
// 輸出
function output(txt) {
    console.log( txt );
}

// 字串長度是否小於等於 5
function isShortEnough(str) {
    return str.length <= 5;
}

// 字串長度是否大於 5
function isLongEnough (str) {
    return !isShortEnough(str)
}

```

首先印出長度小於 5 的字串

```
function printIf( testfn, msg ) {
    if (testfn( msg )) {
        output( msg )
    }
}

var msg1 = "Good";
var msg2 = msg1 + " Morning";

printIf( isShortEnough, msg1 )            // Good
printIf( isShortEnough, msg2 )
```

印出長度大於 5 的字串

```
printIf( isLongEnough, msg1 )
printIf( isLongEnough, msg2 )       // Good Morning
```

看到 **isLongEnough** 的**點**了嗎？ 形參 `str` 傳遞令你困擾，試著改成 Pointfree 吧！

### 小工具：取反（取補）
介紹一個小工具 `not(..)` 取否定，在 FP libraries 中常稱為 `complement(..)` ：

```javascript=
function not (testerfn) {
    return function negated (...args) {
        return !testerfn(...args)
    }
} 

// ES6
var not = testerfn =>
    (...args) =>
        !testerfn(...args)

```
> FP 小工具 not(..)[^1](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#no-points)

使用 `not(..)` 改寫 `isLongEnough`：

```
var isLongEnough = not(isShortEnough)

printIf( isLongEnough, msg2 )       // Good Morning
```
> 範例3 isLongEnough Pointfree 版本

### 進階：printIf 重構

再介紹一個小工具 `when(..)` 改寫 `if` 條件判斷，

```javascript=
function when (testerfn, fn) {
    return function conditional (...args) {
        if (testerfn(...args)) {
            return fn(...args)
        }
    }
}

// ES6
var when = (testerfn, fn) =>
    (...args) =>
        testerfn(...args) ? fn(...args) : undefined
```
> FP 小工具 when(..)[^2](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#no-points)
console.log(f2)

console.log(f2(isShortEnough))
console.log(f2(isShortEnough)('Morning'))

**步驟1**：

首先將 `output` **right-partially apply** (右偏應用) 在 `when` 的 `fn`，得到一個期望接受第一個參數 `testerfn` 的函數。

```
rightPartial( when, output )

// (...laterArgs) => when ( ...laterArgs, output)
```

**步驟2**：

將判斷 `testerfn` 傳入後，會產生另外一個等待參數 `...args` (如下說明碼，也就是要輸出的字串) 的函數，

如果這樣印出來，就會比較清楚了

![](https://i.imgur.com/n1AejUb.png)

**步驟3**：
經過上面兩步驟的處理，函數簽名會變成 `fn(testerfn)(str)`，想要調整成跟原來 `printIf(testerfn, str)` 相同，可以使用 `uncurry(..)` [註：關於 uncurry，請參考原文](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#no-curry-for-me-please)：

```
var printIf = uncurry( rightPartial( when, output ) );
```
> 範例4 Pointfree 的 printIf

## 小結：
這 5 天花了很多力氣深入探討函數參數的調整方式，也開始接觸函數的組成，小整理如下：

* **Partial Application**，偏函數應用，返回一個預設部分參數的函數，等待剩下全部的參數。

* **Currying**，是 Partial 的特殊形式，將參數數量減少到 1 個，每一次傳入參數都會返回一個另一個等待接受下一個參數的函數，直到收到所有參數，再執行。

* 還有介紹許多小工具（整理如下），包含解構，反序、反 curry...等，都是 FP 函式庫常用的工具。

* **Pointfree** 無點風格，透過隱藏 parameter - argument （形參 - 實參對應），目的在於提高程式碼的可讀性。


## 參考資料
^1, ^2: [No Points](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#no-points)

^3: FP 小工具整理，截至Day 14 (2018-01-03)
> All for one, unary
```
var unary = fn => arg => fn( arg )
```

> One on one, identity
```
var identity = v => v
```

> Unchanging One, constant (for certain APIs)
```
var constant = v => () => v
```

> SpreadArgs, apply(..)
```
var spreadArgs = fn => argsArr => fn( ...argsArr )
```

> GatherArgs, unapply(..)
```
var gatherArgs = fn => ...argArr => fn(argArr)
```

> **Partial Application, partial(..)**
```javascript=
var partial = 
    (fn, ...presetArgs) =>
        (...laterArgs) =>
            fn(...presetArgs, ...laterArgs)
```

> **Partial Application, partialRight(..)**
```javascript=
var partialRight =
    (fn, ...presetArgs) =>
        (...laterArgs) => 
            fn(...laterArgs, ...presetArgs)
```

> **Currying, curry(..)**
```javascript=
var curry = (fn, ARITY = fn.length, nextCurried) =>
    (nextCurried = prevArgs =>
        nextArg => {
            var args = [...prevArgs, nextArg]

            if (args.length >= ARITY) {
                return fn(...args)
            } else {
                return nextCurried(args)
            }    
        }
    )( [] )
```

> No Currying, uncurry(..)
```
var uncurry =
    fn =>
        (...args) => {
            var ret = fn;

            for (let arg of args) {
                ret = ret( arg );
            }

            return ret;
        };
```

> 重構 **partial application** 工具 (參數解構, 命名實參)

```javascript=
function partialProps(fn,presetArgsObj) {
    return function partiallyApplied(laterArgsObj){
        return fn( Object.assign( {}, presetArgsObj, laterArgsObj ) );
    };
}

```

> 重構 **currying** 工具 (參數解構, 命名實參)
```javascript=
// 請注意，arity 需指定，預設為 1
function curryProps(fn, arity = 1) {
    return (function nextCurried(prevArgsObj){
        return function curried(nextArgObj = {}){
            var [key] = Object.keys( nextArgObj );
            var allArgsObj = Object.assign( {}, prevArgsObj, { [key]: nextArgObj[key] } );

            if (Object.keys( allArgsObj ).length >= arity) {
                return fn( allArgsObj );
            }
            else {
                return nextCurried( allArgsObj );
            }
        };
    })( {} );
}
```

> spreadArgProps(..) 解析並解構形參
```javascript=
function spreadArgProps(
    fn,
    propOrder =
        fn.toString()
        .replace( /^(?:(?:function.*\(([^]*?)\))|(?:([^\(\)]+?)\s*=>)|(?:\(([^]*?)\)\s*=>))[^]+$/, "$1$2$3" )
        .split( /\s*,\s*/ )
        .map( v => v.replace( /[=\s].*$/, "" ) )
) {
    return function spreadFn(argsObj) {
        return fn( ...propOrder.map( k => argsObj[k] ) );
    };
}
```

> not(..)，取反 (取補集), `complement(..)` 
```
var not = testerfn => (...args) => !testerfn(...args)
```

> when(..)，重構 if
```
var when = (testerfn, fn) =>
    (...args) =>
        testerfn(...args) ? fn(...args) : undefined
```
