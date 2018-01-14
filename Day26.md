Good Morning, Functional JS (Day 26, Tail Calls 尾調用)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## STACK OVERFLOW!
![image alt](https://cdn-images-1.medium.com/max/1600/1*tqkykdU69DFrxi82JOWLbQ.png)
> 相信各位開發時一定會遇過這種 Error，這是什麼意思呢？

每一次 call function 都會劃一塊 stack frame 放在 Memory，如上圖黃綠色格子的樣子，每一塊都內含函數的資訊，當函數執行到一半，呼叫第二個時，就往上疊一個，如果又呼叫第三個，再往上疊一個...以此類推。

stack 的消耗是由上到下，就像書堆是從上面開始拿，如果剛剛是 `func1 -> func2 -> func3` 目前 stack 就會疊三格，然後當 `func3` 執行完，stack pop掉一格剩兩格，引擎接著會看 `func2` stack frame 裡面的紀錄，到暫停的地方繼續執行。

上圖中的函數：
```
function foo () {
    return foo()
}

foo()
```

當引擎預測如果這個程序繼續運行，記憶體會被耗盡，所以引擎拋出 RangeError 的錯誤，是為了保護系統。

不同的引擎、不同的設備有不同的限制，所以無法保證 call stack 最大的大小，這些限制使得開發者必須注意遞迴的效能。

## Tail Calls 尾調用
為了避免 stack overflow，有一個很有用的技巧叫做 **Tail Call**。

**Tail Call** 的概念只要一句話：一個函數的在最後一步是 call 另外一個函數。

以 `func1 -> func2 -> func3` 為例，當 call `func2` 的動作在 `func1` 的對底部執行，那 `func1` 的 stack frame 就不需要了，也叫表示記憶體可以釋出（因為 `func1` 做完了），以此類推：

![](https://i.imgur.com/ybO5suT.jpg)

如果 **Tail Call** 自己，又稱為 **tail-recursive** 尾遞迴，這時候只會存在一個 stack frame，所以理論上遞迴 stack 可以一直運算下去。

## Tail Call Optimizations (TCO) 尾調用優化
**TCO** 是使得 **Tail Call** 更高效運行的優化，要做到 **TCO**，必須正確書寫 **Tail Call**。在 ES6 問世之後，對 **Tail Call** 有了明確的規範，只要在 ES6 中，正確使用 **Tail Call** 就不會有 RangeError 的異常。

正確書寫 **Tail Call** 的技巧稱作 PTC (Proper Tail Calls，正確的尾調用)。

首先，必須要開啟嚴格模式 `"use strict"`，並在函數最後一步 call 下一個 function，並且 return 下一個 function 的 return 值。

好吧！看例子最清楚：

### **不正確** PTC
```
function factorial (n) {
    if (n === 0) {
        return 1
    }
    return n * factorial(n - 1)
}
```
此非 PTC，因為 `factorial(n - 1)` 執行後，`n *` 才會執行，記憶體最多需要保存 n 個 stack frame，複雜度 `O(n)`。

### **正確** PTC
```
"use strict";
function factorial(n, partialFactorial = 1) {
  if (n === 0) {
      return partialFactorial
  }
  return factorial(n - 1, n * partialFactorial);
}

factorial(5) // 120
```

正確的 PTC 形式，使用 `"use strict"`，同時記憶體只需要保存一個 stack frame，複雜度 `O(1)`。


### 補充：**非** PTC：

```
// e.g 1
foo( .. );
return;

// e.g.2
var x = foo( .. )
return x

// e.g.3
return 1 + foo( .. );
```

## 結論
一個問題解法通常有遞迴和迭代兩種解法，遞迴的精神是將問題拆成小問題，最後再將結果一層層返回；迭代則是一步一步往結果收斂。沒有一定哪個好，要依問題的本質而定。

如果想使用遞迴，卻又超出了 JS 引擎的 memory stack，就需要重構，使能夠符合 PTC 規範（或着避免 callback hell）。

可讀性强的程式，才是我们的目標，如果使用遞迴反而會造成程式難以閱讀/理解，那就不要刻意用，試試別的方法吧。


## 參考資料
^1: [ECMAScript 6 Proper Tail Calls in WebKit](https://webkit.org/blog/6240/ecmascript-6-proper-tail-calls-in-webkit/)

^2: [尾调用优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)
