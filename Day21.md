Good Morning, Functional JS (Day 21, Referential Transparent 引用透明)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

昨天，我們已將純函數的下列特性進行討論：

* 沒有任合可觀察的 side causes/effects
* 相同的輸入，永遠會得到相同的輸出

今天，我們進一步討論純函數另一個廣為人知的特性：**引用透明（Referential Transparent）**

## Referential Transparent 引用透明

> Referential transparency means that "equals can be replaced by equals"[^1]

引用透明是指把一個函數可以用它的輸出值取代，並且整體程式的行為不變[^2]；換句話說，如果程式中任意兩處具有相同輸入值的函數調用能夠互相置換，而不影響程式的運作，那麼該程式就具有引用透明性。

由於純函數相同的輸入總是會返回相同的結果，這也保證了引用透明性，來看個例子：

![](https://i.imgur.com/znlAwLa.png)

觀察 line 6，右邊使用 function return `9` 取代 function call `calculateAverage(nums)`，而程式其他部分行為不變，所以 `calculateAverage(nums)` 具有引用透明性，是一個純函數。

## 透明度更高的程式碼
雖說一個具引用透明性的函數可以被它的輸出值取代，但並不是真的要用它的值去替換掉！

引用透明真的的用意：當你在 review 程式時，假如說你已經知道某個函數的結果時，下一次再看到同一個函數時，你不必再花力氣去思考去腦內編譯。

就像宣告 `const` PI = 3.14，下次在別的地方看到 PI，我們腦內會自動把它換成 3.14，完全不會花力氣去思考它是什麼。

這也是致力於追求函數純粹性的原因，讀者看到同一個函數，不必去擔心結果是不是會跟其他的不同（或造成副作用），不必去重新計算（追蹤）程式的結果。

寫具有引用透明的純函數，這樣我們就可以專心在更重要的事情上。

## 追求純的理由 - Cacheable 可快取性
根據純函數的特性，我們能夠對同樣輸入的函數做快取：

```
var cache = []

function factorial(n) {
    // 如果我們計算過了
    // 直接 return 快取直
    
    if (cache[n] !== undefined) {
        return cache[n];
    }
    
    var finalVal = 1;
    
    for (var i = 1; i <= n; i++) {
        finalVal = finalVal * i;
    }
  
    cache[n] = finalVal
  
    return cache[n]
}

console.log(factorial(4))
// 24

console.log(factorial(11))
// 39916800

console.log(factorial(20))
// 2432902008176640000
```
> 範例1 有快取的階乘函數

做些檢查：
* 給定相同輸入，會有相同輸出
* 用結果替代函數呼叫，不影響程式，具有引用透明性

這個函數是計算階乘，如果是大數字運算，將結果緩存，會提高程式的效能。

`factorial` 引用透明性成立的關鍵是在於，cache 只能被 `factorial` 存取和更新，也許有人會認為這樣並不符合純函數，**因為還是觀察得到，只是不觀察而已**。

這種性能優化產生的副作用，會藉由隱藏快取結果來避免，在 FP library 實作快取的方法叫做 `memoize`。

來手刻一個 `memoize`：
```
var factorial = (function memoization(){
    var cache = [];

    return function factorial(n){

        if (cache[n] !== undefined) {
            return cache[n];
        }

        var finalVal = 1;

        for (var i = 1; i <= n; i++) {
            finalVal = finalVal * i;
        }

        cache[n] = finalVal

        return cache[n]
    };
})();
```
> 範例2 抑制副作用

現在我們透過一個 IIFE 將 `factorial` 的副作用 (side effect/ causes) `cache` 閉包住，現在程式的其他部份都觀察不到他們了。

### Ramda 範例
```
let checker = 0
const factorial = R.memoize(n => {
    
    // 計算一次，+1
    checker++
    
    var finalVal = 1;
    
    for (var i = 1; i <= n; i++) {
        finalVal = finalVal * i;
    }
  
    return finalVal
});

console.log(factorial(5))

console.log(factorial(5))

console.log(factorial(5))

console.log(checker)
//=>  1
```
> 範例3 Ramda memoize 快取運算過的結果

範例3中使用一個計數器，確認函數在第一次執行過後，之後再有同樣輸入值會返回快取結果，而不是重算。實作請查看[^JSBIN](https://jsbin.com/padamuyodo/edit?js,console)

## 小結
減少副作用 (side effect/ causes) ，讓副作用盡可能地少，會使得程式碼更容易被理解，讀者也會對程式什麼會發生、什麼不會發生會更有信心。



## 參考資料
[^1]: O. Nierstrasz PS — *Functional Programming* ch4.3 References

[^2]: John C. Mitchel (2002). *Concepts in Programming Languages*. Cambridge University Press. p. 78
