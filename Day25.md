Good Morning, Functional JS (Day 25, Recursion 遞迴)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

今天研究的主題是 Recursion 遞迴，我得自首：我承認遞迴是很強大的技巧，但我不太喜歡用 - **強大但令人困惑**，我把它跟正則表達一起擺在技術債的最底部，總是一直沒去碰。

但該來的還是會來，沒想到研究個 FP ，竟然又讓我碰到遞迴，上次看到是在學校學演算法的時候...，今天就來還技術債吧。

## 定義
> 「為了理解遞迴，則必須首先理解遞迴。」[^1]

回憶一下課堂上關於遞迴的定義：指 function 執行過程反覆呼叫自身，持續到基本條件滿足時。三個重點：

* 基本條件，滿足基本條件，遞迴停止，也可說是終止條件。
* 若未滿足，呼叫自己。
* 每一次都逐漸往基本條件的方向收斂。

一個遞迴演算法，必須要有一個基本條件，否則会一直遞迴下去，直到 stack overflow。

![Photo by Mike Wilson on Unsplash](https://images.unsplash.com/photo-1507512003023-e986f0611020?auto=format&fit=crop&w=634&q=80)
> Droste Effect 遞迴的視覺呈現

你說課本的定義太難懂了！學校離我太遠了，那我們就看個例子[^2]
```
function foo(x) {
    // 基本條件
    if ( x < 5 ) return x
    return foo(x / 2)
}
```
呼叫 foo (16)，

![](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/images/fig13.png?raw=true)
> [Photo](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/images/fig13.png?raw=true) by getify on GitHub

* step2 ~ step3：未滿足條件，把 `x / 2` 結果作為參數，呼叫自己
* step4：滿足基本條件 `x < 5`，`return 4`（圖中黑色虛線），不再遞迴。

這邊使用 call stack 解釋，step4 條件滿足時發生的事，如下圖，當基本條件滿足後，step 4 的 return 會觸發 stack 上所有的 function call（並且都執行 return），直到 return 最終結果：

![](https://i.imgur.com/zzxwvZ6.png)
> [Photo](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/images/fig19.png) by getify on GitHub

更多遞迴範例，網路上資源很多
* [遞迴 (RECURSIVE) 介紹與經典題型](https://hellolynn.hpd.io/2017/08/19/%E9%81%9E%E8%BF%B4-recursive-%E4%BB%8B%E7%B4%B9%E8%88%87%E7%B6%93%E5%85%B8%E9%A1%8C%E5%9E%8B/)
* [費式(Fibonacci)數列](http://dhcp.tcgs.tc.edu.tw/c/p014.htm)
* 費式數列交互遞迴範例[Fibonacci Numbers Using Mutual Recursion](https://www.researchgate.net/publication/246180510_Fibonacci_Numbers_Using_Mutual_Recursion)

### 關於交互遞迴
定義：由一函數 A 呼叫另一函數 B，再由另一函數 B 呼叫原函數 A
![](https://i.imgur.com/DBuhQV2.jpg)

## Recursive 遞迴 v.s. Interative 迭代
ＯＫ，我們已經透過定義和簡單的範例複習完遞迴了，其實用遞迴解的問題，同時可以迭代方式解題，在 Stack Overflow 有相當多的討論，例：[連結1](https://stackoverflow.com/a/15688084/5023488)、[連結2](https://stackoverflow.com/a/15688097/5023488)、[連結3](https://stackoverflow.com/a/21616651/5023488)，兩者理論上可以互相轉換。

關於遞迴與迭代解決問題的方式：

* 遞迴：一種透過重複將問題分解為同類的子問題而解決問題的方法。通過呼叫函數本身來進行。

* 迭代：其目的通常是為了接近所需求的目標或結果。每一次對過程的重複被稱為一次"迭代"，每一次迭代得到的結果通常會被用來作為下一次迭代的初始值。

```
// 遞迴解法
function fib(n) {
    if (n <= 1) return n;
    return fib( n - 2 ) + fib( n - 1 );
}

// 迭代解法
function fib (n) {
    var r = 0
    var q = 1
    var p;
    
    if (n <= 1) {
        return n
    } else {
        for (let i = 2; i <= n; i++) {
            p = r + q
            r = q
            q = p
        }
        return p
    }
}
```
> Fibonacci 兩種解法

遞迴使用 call stack 代替 for loop，用 return 形式追蹤每次呼叫的狀態，而非在每次迭代中重新分配 p（將結果存在 p），而 for loop 使用可變的狀態作為計數器（不符合 Immutability 的精神），基本上，FP 開發者都會避免這種重賦值的行為。

好吧，再從別的角度看這兩段程式，最起碼，遞迴的程式碼較精簡，~~減少鍵盤的損耗~~。

再從 Declarative (宣告式) v.s Imperative (命令式) 的角度看，遞迴是一種 Declarative 的寫法，`loop` 屬於 Imperative，遞迴告訴程式做什麼，而不是告訴程式怎麼做。

## 遞迴 - 以 binary tree 為例
昨天研究遞迴的時候，翻了以前的資料結構講義，在二元樹，有一道經典範例：計算深度 (depth)，最大深度是指是從 root 出發（向左或右）找出最長路徑。
![](https://i.stack.imgur.com/8yPi9.png)
> 如圖，最大深度為 3

### iterative 迭代算法
走訪全部路徑計算深度，然後再從中找最大的，也就是要窮舉，網路上有解法，有興趣可以參考：[挑戰演算法 - 二元樹的最大深度](http://mycodetub.logdown.com/posts/236920-day12-algorithm-for-30-day-challenge-the-maximum-depth-of-a-binary-tree)。

### recursive 遞迴算法
如果 root 指向 null，那深度 = 0，
如果 root 不為 null，那深度就比較當前左子樹與右子樹的深度，取最大值，那麼當前的深度就是最大子樹深度 + 1。
基本條件（終止條件），當節點指向 null 時，返回 0。

```
function maxDepth(node){
    if (node == null) {
        return 0;
    }
    // 遞迴計算左子樹的深度
    var depthLeft = maxDepth(node.left);
    // 遞迴計算右子樹的深度
    var depthRight = maxDepth(node.right);

    return 1 + Math.max(depthLeft, depthRight)
}
```
> 遞回求二元樹深度


## 結論
今天透過範例，和大家一起重新學習遞迴。使用遞迴代表的是程式設計模式的轉變（也就是 Imperative 轉成 Declarative），身為人類，我們應該站在更高的抽象層鳥瞰問題，告訴電腦做什麼（What）而不是怎麼做（How），感覺說過好多次了，不過這就是 FP 的精神！


## 參考資料
[^1]:Wiki：[遞迴正式定義](https://zh.wikipedia.org/wiki/%E9%80%92%E5%BD%92)
[^2]:Chapter 8: Recursion [#Definition](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch8.md#definition)
