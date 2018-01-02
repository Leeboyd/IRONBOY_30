Good Morning, Functional JS (Day 13, 參數順序調整)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## 關於參數順序
我們已經討論過 **currying** 和 **partial application** 分次處理多參數函數的優點，在討論這兩個技巧時，有時候須透過一些技巧來修正參數順序，像透過 **reverseArgs(..)** 小工具來調整順序，這些小工具會導致我們的程式碼語意被混淆，有沒有辦法能幫助我們處理參數順序呢？

## Destructuring，解構賦值
參考 ES6 的 **destructuring**

```
function move({x = 0, y = 0, z} = {}) {
  return [x, y, z];
}

// 任意順序
move({y: 8, x: 3}); // [3, 8, undefined]

// x 沒傳解構失敗，等於預設值
move({y: 8}); // [0, 8, undefined]
```
> 範例 1 函數參數的解構賦值

範例1中， `foo(..)` 參數是一個物件，解構成兩個形式參數 `x` 和 `y` ；而在函數內部有兩個區域變數命名為`x` 和 `y` ，接著在呼叫時，傳入命名實參 `y` ，便會自動映射形參 `y` 上。

有發現了嗎？這種命名實參加上 **destructuring** ，讓我們不必再苦惱參數順序，因此提高了可讀性，

接著運用這些概念來實作一個小工具。

### 重構 **partial application** 工具

```javascript=
function partialProps(fn,presetArgsObj) {
    return function partiallyApplied(laterArgsObj){
        return fn( Object.assign( {}, presetArgsObj, laterArgsObj ) );
    };
}

```

### 重構 **currying** 工具
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

> 範例2 透過命名實參來自動映射到形參，完全解決了順序的煩惱

使用這兩個新工具來重構範例1：

```
function move({x = 0, y = 0, z} = {}) {
  return [x, y, z];
}
```

### **partial application**

```
var f2 = partialProps( move, { y: 2 } );

f2({ x: 1, z: 6})

// [1, 2, 6]
```

### **currying**

```
var f1 = curryProps( move, 3 );

f1({ x: 2 })({ z: 7 })({ y: 3})

// [2, 3, 7]
```

> 範例3 實際到 [JSBIN](https://jsbin.com/kehisofaro/1/edit?js,console) 測試吧

不必為函數簽名形參順序煩惱，也不需要考慮傳實參的先後，所以我們再也不需要 `partialRight(..)` 和 `reverseArgs(..)` 了。

## 回到現實

那如果一個函數，形參是各自獨立(沒有形參解構 parameter destructuring)，而且不能改變函數簽名 (signature)，又該如何處理呢？

```javascript=
function move2(x, y, z) {
  return [x, y, z];
}
```
> 範例4 形參獨立分開的 move2

或許可以利用 spreadArgs 小工具的概念，將傳入的 `{key: value}` 物件， `...` 展開成獨立實參，

```javascript=
function spreadArgs(fn) {
    return function spreadFn(argsArr){
        return fn( ...argsArr );
    };
}
```
> 範例5 spreadArgs 的概念

但要小心的是，spreadArgs 處理 Array 時，參數的順序是明確的，現在處理的是物件，**物件屬性的順序是不明確的**，故需要傳入一個類似 `["x","y","z"]` 這樣的陣列，告訴小工具處理實參屬性的順序。

剛好，JS 函數有 `toString()` 的方法，我們可以透過解析字串獲取函數簽名的順序，

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
> 範例6 spreadArgProps 解析並解構形參[^1](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#spreading-properties)

spreadArgProps 能處理簡單形參 (含預設參數)的順序問題，複雜的形參（如解構）則不行，使用的方式像這樣，

```
function move2(x, y, z) {
  return [x, y, z];
}
```

### **currying**

```
var f3 = curryProps( spreadArgProps( move2 ), 3 );

f3( {y: 2} )( {x: 1} )( {z: 3} );

// [1, 2, 3]
```

### **partial application**

```
var f4 = partialProps( spreadArgProps( move2 ), { y: 2 } );


f4( { z: 3, x: 1 } );

// [1, 2, 3]
```

## 小結

透過物件形參（object parameters）解構和命名實參（named arguments）模式，減少為調整參數順序的干擾，明顯提高了可讀性。

另外，這些重新定義函數的工具要求你知道**參數的名字**，看到 **fn**，要想成接收函數的形參叫做 **fn**，而不只是把某個函數傳入。


## 參考資料
^1: [Spreading Properties](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#spreading-properties)
