Good Morning, Functional JS (Day 22, 減少副作用的方法)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## 純化
在談到純化函數，最好的方式就是一開始就把他們設計成純函數。

但總有些時候我們很難將不純的重構為純的，這時候可以將副作用從函數提取出來，放在函數調用處展示，讓副作用更明顯，看個例子[^1]：
```
function addMaxNum(arr) {
    var maxNum = Math.max( ...arr );
    arr.push( maxNum + 1 );
}

var nums = [4,2,7,3];

addMaxNum( nums );
```
我們透過 `addMaxNum` 的副作用修改陣列 `nums` ，但我們可以把副作用 push 的部分提取出，使 `addMaxNum` 成為純函數，使副作用更容易被觀察到：

```
function addMaxNum (arr) {
    var maxNum = Math.max(...arr)
    return maxNum + 1
}

var nums = [4, 2, 7, 3]

num.push(addMaxNum(nums))
```
接著繼續看更多處理 side effects / causes 的方式。

## 處理自由變數
如果 side effects / causes 是來自於自由變數，可以使用作用域來封裝進行隔離：

還記得 [Day 20]()

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
> 純化階乘函數

純化此函數的方法，是在自由變數和函數的周圍建立一個 IIFE 的容器，讓 cache 變成 local variable，使外部不會察覺到 factorial 創造的 cache 副作用。

無論這個重構技巧是否有用，因為很多時候我們沒辦法去修改函數周圍的程式，不過這邊我們認知到一個事實：**函數的純度，是從外部判斷的**

![](https://i.imgur.com/2gVbd1a.png)

不管函數內部如何，只要函數使用使用表現是純的，那麼它就是純函數。

不過要小心，使用不純的技巧（如上例中的 cache 技巧），需適度，即使使用純函數封裝，它仍然造成讀者困惑潛在原因，整個來說，盡力減少副作用，而不只是掩飾它。

## 掩飾副作用 Part 2
但有時候，我們無法將自由變量封裝在某作用域[^2]：

```
var nums = [];
var smallCount = 0;
var largeCount = 0;

function generateMoreRandoms(count) {
    for (let i = 0; i < count; i++) {
        let num = Math.random();

        if (num >= 0.5) {
            largeCount++;
        }
        else {
            smallCount++;
        }

        nums.push( num );
    }
}
```

透過一個介面隔離副作用，步驟如下：
1. 保留初始狀態
2. 建立初始狀態的副本供輸入
3. 運行 impure 函數
4. 保存產生副作用的狀態
5. 恢復原本狀態
6. 返回運行後產生副作用的狀態

```
function interface_generateMoreRandoms(count, initial) {
    // 1. 保留初始狀態
    var origin = {
        nums,
        smallCount,
        largeCount
    }
    
    // 2. 建立副本供輸入
    nums = nums.slice()
    
    // 3. 運行 impure 函數
    generateMoreRandoms( count );
    
    // 4. 保存產生副作用的狀態
    var sides = {
        nums,
        smallCount,
        largeCount
    }
    
    // 5. 恢復原本狀態
    nums = origin.nums
    smallCount = origin.smallCount
    largeCount = origin.largeCount
    
    // 6. 返回運行後產生副作用的狀態
    return sides
}
```

call generateMoreRandoms( count )

```

interface_generateMoreRandoms( 5 )

console.log(interface_generateMoreRandoms( 5, initialStates ))

// {
//  largeCount: 0,
//  nums: [0.4677814841565002, 0.4642303003099153, 0.28710410178139156, 0.2701034055081073, 0.45926971284495854],
//  smallCount: 5
// }

console.log(nums)
// []
console.log(smallCount)
// 0
console.log(largeCount)
// 0

```
> 透過介面呼叫 impure 函數，[^JSBIN](https://jsbin.com/lorenetisa/edit?js,console)實作

過程可以看到我們花了很多力氣手動處理副作用，但這種努力是值得的，可以讓我們的意外更少。

但是，這種方法只適合處理同步程式，異步動作就不能靠這種方式處理了。

## 結論
將一個 impure 重構為 pure 方法很多，但最好的是一開始就設計成 pure，當無法重構時，可以嘗試封裝，或者建立一個介面來隔離。

純函數的好處很多，給定相同輸入會有相通輸出，更有引用透明性，讓我們更專注在整體，而不是去擔心每一次函數調用產生的改變。

沒有沒副作用的函數，但我們可以花力氣盡量減少副作用，或者控制在一個範圍，這樣當錯誤發生時，也比較容易除錯。


## 參考資料
^1:[Purifying](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch5.md#purifying)

^2:[Covering Up Effects](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch5.md#covering-up-effects)
