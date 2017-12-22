Good Morning, JS (Day 2, Currying)
===

Currying 為 functional programing 中重要的技巧，本篇為簡介，日後會更進一步討論本文提到的專有名詞以及相關細節。

## 目的：
將多參數函數，轉化一次僅接受單一參數的函數

![](https://i.imgur.com/Xh4HjGa.png)
## 說明：

每一次 call function 傳入部分的參數，回傳一個 SPECIALIZED function 去處理剩下的參數。

Currying 是將一個函式

`f: X + Y + Z => Result`

轉換成

`f': X => (Y => Z => Result)`

我們調用 f' 與第一個參數 X 。其結果是一個函式，直到傳入所有的參數得到結果。

在 currying 之前這樣調用

`f(X, Y, Z)`

在 currying 之後這樣調用

`f'(X)(Y)(Z)`


## 實作 currying 

![](https://i.imgur.com/IVp5xkh.png)

> 範例3 SPECIALIZED function

## 實測 curry function（一）

![](https://i.imgur.com/pS03Q8K.png)

![](https://i.imgur.com/suTtHHg.png)

## 實作 currying （二）

![](https://i.imgur.com/e4HoM1f.png)

## 實測 curry function（二）
![](https://i.imgur.com/rpfVZwg.png)

## 總結

透過簡單的傳遞參數，就能夠動態的建立實用的新 function，本篇是 currying 範例是來自 Kyle Simpson [getify/Functional-Light-JS](https://github.com/getify/Functional-Light-JS) ，這也是我看過最易理解的 curry 實作，與各位分享。
