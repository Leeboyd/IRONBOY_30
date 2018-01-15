Good Morning, Functional JS (Day 27 Continuation-passing style)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## Continuation-passing style，CPS
![](http://canacopegdl.com/images/continuation/continuation-5.jpg)
> Photo from http://canacopegdl.com/

不是所有的遞迴都可以按照 PTC 規範重構（Proper Tail Calls），如多重遞迴 （multiple recursion），或者是交互遞迴（mutual recursion），這些是函式內，呼叫多次遞迴，這些都很難做到 PTC 得到優化（因為沒有辦法把多個遞迴放到最後）。

我們借用 Continuation-passing style，CPS，詳細定義參考 [wikipedia](https://en.wikipedia.org/wiki/Continuation-passing_style)，本篇只帶一點概念，我盡量用我理解的方式解說，在 JS 中，*continuation* 表示函數結束後下一步驟的 callback，也就是接著要做的事，而 CPS 就是在函數結束後把要做的事指定給下一個函數（當作參數）。


以加總為例：

```
"use strict";

var sumRecursion = (function IIFE(){

    return function sum(...nums) {
    
        return recursion(nums, v=>v)
        
    }

    function recursion([result, ...nums], cont) {
    
        if (nums.length == 0) return cont(result)
        
        return recursion( nums, function(v) {
        
            return cont(result + v)
            
        })
        
    }

})();

console.log(sumRecursion( 3, 1, 17, 94, 8 )); 
```

一開始執行時，把第一個 continuation 函數傳入 `cont` ，此時 `cont` 一個單純回傳的函數，

```
(v) => v

// or

function (v) {
    return v
} 
```

CPS中每個函數都有一個額外的參數用来表示當一個函數執行完畢該做什麼。

基本條件，只有一個參數，直接呼叫 (v) => v。

否則，傳給下一個遞迴 continuation 函數，裡面是這樣的：

以第一次的循環為例

* closure 當前的 result = 3，以 `3 + v` 的形式
* closure 當前的 cont `(v) => v`

一直持續下去，到剩最後一個 `8`，滿足基本條件，開始執行 `cont`，請注意我們一直到最後才執行 `cont` 哦。這邊是遞迴的邏輯：滿足基本條件，一層層逐步返回部分結果。

只不過在實作 PTC 時（下方註解有範例供回憶^1），我們把部分結果傳遞給下一個遞迴，在 CPS 時，我們將運算步驟閉包在一個 continuation callback函數中，並延遲 (defer) 到最後一步才開始執行。

所以，在最後一個步驟會得到一個很深的巢狀函數：

`cont( 3 + cont( 1 + cont( 17 + cont( 94 + cont( 8 ) ) ) ) )`

此外請觀察範例，在每一輪遞迴最後的樣子，`return recursion( .. )`，這裡符合 PTC 的規範。不過 CPS 也有一些缺點

* 原生並無 CPS 規範，得自己設計
* CPS 這種多層巢狀的風格基本上可讀性很差，上面的例子算是最簡單的例子了（原作者的例子我看不懂）

詳細範例請操作 [JSBIN](https://jsbin.com/reyawojese/edit?js,console) 練習。

有興趣讀者，可參考更多 CPS 的資料：
* [By example: Continuation-passing style in JavaScript](http://matt.might.net/articles/by-example-continuation-passing-style/)
* [Continuation-passing Style介绍及应用](http://www.voidcn.com/article/p-firtuucm-bec.html)
* [What is continuation-passing style in functional programming?](https://www.quora.com/What-is-continuation-passing-style-in-functional-programming)

## 註解
^1: 符合 PTC 加總函數
```
"use strict";

function sum(num1,num2,...nums) {
    num1 = num1 + num2;
    if (nums.length == 0) return num1;
    return sum( num1, ...nums );
}

sum( 3, 1, 17, 94, 8 );        // 123
```


