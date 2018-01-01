Good Morning, Functional JS (Day 12, Why Currying 柯里化)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## 視覺化 Curry

回想昨天的範例，

```javascript=
function sum(...args) {
    var sum = 0;
    for (let i = 0; i < args.length; i++) {
        sum += args[i];
    }
    return sum;
}

sum( 1, 2, 3, 4, 5 );                        

// 運用 Currying
// 因為是不定長度參數 ...args，需指定個數
var curriedSum = curry( sum, 5 );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );  // 15
```
> 範例1 Currying 後的 curriedSum

假設我們今天不借助 **curry** 直接手刻 curriedSum 呢，如何寫呢？

```javascript=
function curriedSum(v1) {
    return function(v2){
        return function(v3){
            return function(v4){
                return function(v5){
                    return sum( v1, v2, v3, v4, v5 );
                };
            };
        };
    };
}
```
> 範例2 視覺化 Curry

很醜的巢狀，但這是一個徹底觀察 **curry** 到底發生什麼事的好方式，將一個多參數函數拆解成一串連續的函數，每一層巢狀會 return 另一個函數處理下一個參數，一直持續直到收到所有參數為止。

也改成 ES6 的版本看看，風格各有優缺，端看個人取向

```javascript=
curriedSum =
    v1 =>
        v2 =>
            v3 =>
                v4 =>
                    v5 =>
                        sum( v1, v2, v3, v4, v5 );

// 一行式
curriedSum = v1 => v2 => v3 => v4 => v5 => sum( v1, v2, v3 v4, v5 )
```
> 範例3 視覺化 Curry ES6 版本

## Why

好，連續兩天的語法討論以及重構，到底為什麼要這樣做？為什麼要把 sum 改成這樣：

* **currying**：`sum(1)(2)(3)`
* **partial application**: `partial(sum,1,2)(3)`

這兩種風格都比原來的簽名 (signature) `sum(1,2,3)` 怪，請讓我試著解釋

原因ㄧ，兩種方式可以讓你切割傳參數的時空背景（時間、程式的不同區塊），原本的方式需要你在呼叫的當下立刻傳入所有的參數，如果你的函數中有些參數待會才傳入，可以考慮使用 **currying** 和 **partial application**。

原因二，就是方便函數組合 (Composition)，在接下的連載會討論到，簡單解釋：`f(x)` 和 `g(x)` 如果要合成為 `f(g(x))`，有一個隱藏的假設，就是 `f(x)` 和 `g(x)` 都只能接受一個參數。如果是多個参数，比如 `f(x, y)` 和 `g(a, b, c)`，就很難實現函數組合。藉由 **currying** 和 **partial application** 調整成 `Arity = 1` 將可方便實現。

最重要的原因，也就是本文的主旨：提高可讀性。那...到底為什麼這樣的重構可以提升可讀性？回想我們的 `ajax(url, data, cb)` 的範例：

```
ajax (
    "http://some.api/order",
    { id: ORDER_ID },
    function handler(order) { // ... }    
)
```

直接使用 `ajax` ，在呼叫的當下，就需要準備所有會用到的參數，如果這些參數改成在不同時機傳入，語意又是如何呢？

```javascript=
var getOrder = partial(
    ajax,
    "http://some.api/order",
    { id: ORDER_ID }
);

// 待要處理 Order 時，再叫

getOrder( function handler(order){ /* .. */ } );
```
> 範例4

範例4中，我們先定義出 **getOrder** ，並僅將此刻需要的參數傳入 `url`、`data`，當待會要處理訂單時，再呼叫 **getOrder** ，並同時把此刻需要的 **handler** 傳入。

這就是所謂的抽象化 (abstraction)，我們將一句 `ajax` 呼叫，拆成兩個細節：

* 取得訂單
* 處理訂單

不管用 **currying** 和 **partial application** ，傳入部分的參數，都會 return 另一個 **closure** 已傳入參數的 特定性更強的函數 (SPECIALIZED function，特殊化函數)，但別忘記抽象化的目的，**隱藏細節，增強可讀性**。
