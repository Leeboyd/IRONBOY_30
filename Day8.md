Good Morning, JS (Day 8, Spread and Gather)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)


## `...`，收集與解構
回顧一下前面討論過的 `...`  parameter destructing 操作，

```javascript=
function unaryFoo( [x,y,...args] ) {
    // ..
}

unaryFoo( [1,2,3,4,5] );
```

`unaryFoo` 為一個單參數函數 `unary`，但在函數內容中，我可以對陣列元素單獨處理，利用 `...` (spread，展開)，除了 `x`,`y` 其他的會透過 `...` 操作收集在 `args` 陣列。

場合：試著結合下面兩個函數，

```javascript=
function foo(x,y) {
    console.log( x + y );
}

function bar(fn) {
    fn( [ 3, 9 ] );
}
```

`bar(foo)` 直接組合結果失敗，

![](https://i.imgur.com/hLVAbTP.png)

`foo(..)` 預期接受單獨 `x` 和 `y` ，而 fn(..) 僅送出單一參數，可以這麼做，

* 改變 `foo` 宣告，改成 `foo([x, y])`
* 改變 `fn` 呼叫行為，改成 `fn(...[3, 9])`

### spreadArgs

場合2：想像上述兩個 `foo`, `bar` 無法改宣告方式定義，為了調整，需要實作一個輔助工具，把單一參數 Array 展開，

```javascript=
function spreadArgs(fn) {
    return function spreadFn(argsArr){
        return fn( ...argsArr );
    };
}

// ES6

var spreadArgs =
    fn =>
        argArr =>
            fn(...argArr)

// 或

var spreadArgs = (fn) => (argArr) => fn(...argArr)
```

現在可以這樣使用 `spreadArgs`：

```
bar ( spreadArgs ( foo ) )

```

利用 `spreadArgs` 的輸出作為 `foo` 的輸入，這種一個函数的輸出作为另外一個函數的輸入被叫做 `composition（組合）`。

### gatherArgs
除了展開，從另外一個角度想，也可以做收集的工具，

```javascript=
function gatherArgs(fn) {
    return function gatheredFn(...argsArr) {
        return fn( argsArr );
    };
}

// ES6 箭头函数形式
var gatherArgs =
    fn =>
        (...argsArr) =>
            fn( argsArr );

// 或

var gatherArgs = (fn) => (...argArr) => fn(argArr)
```

透過使用 `gatherArgs` 將單獨的實際參數 arguments 一一收集到一個 argArr 陣列，使其成為另外一個單參數函數的輸入。

來看個實際範例，好夥伴 `reduce`
> reduce
> 陣列處理函數，會重複呼叫一個 reducer，其中 reducer 接受兩個參數 `accumlator`, `currentValue`，最終將陣列簡化為單一值。

```
function combineFirstTwo([ v1, v2 ]) {
    return v1 + v2;
}

[1,2,3,4,5].reduce( gatherArgs( combineFirstTwo ) );
// 15
```
`combineFirstTwo` 接收兩個形式參數 parameter，在此利用 `gatherArgs(..)` 將 reduce 傳入的兩個參數收集起來給 `combineFirstTwo(..)`。


## 小結
注意： 在 Ramda 中，這兩個小工具被稱作 `unapply(..)` 與 `apply(..)` 分別對應 `收集（gather）` 和 `展開（spread）` 的概念，這兩個概念可能會幫助學習者理解。 
