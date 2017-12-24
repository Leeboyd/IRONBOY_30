Good Morning, JS (Day 4, You don't know Functions)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## function 是什麼？

FP 不是僅僅用 `function` 這個關鍵字來編寫。如果這麼簡單，大家就可以洗洗睡開心迎新年了，無論如何， `function` 是 FP 的核心，`function` 的概念並不只針對 FP ，對學習非 FP 也是很重要的知識，旅行的第一步就從打好 `function` 基礎開始吧。

## 回顧
論及 FP ，不免俗的還是要簡單回顧一下，畢竟這個 FP 的概念是從數學演進過來的。

在學習程式以前，我所知道的 `function` 長得是這樣 `f(x)`，還有方程式 (equation) 長這樣 `y = f(x)`

舉例，一個 equation 定義 <code>f(x) = 2x<sup>2</sup> + 3</code>  ，
* 代入 x = 2，得到值 11
* 對於任意 x ，我們都能得到對應的 **返回值**，也就是 `y`，兩個值可以組合成一個座標，`(2, 11)、(-1, 5)、(3, 21)...`可以做圖如下：
![](https://github.com/getify/Functional-Light-JS/raw/master/manuscript/images/fig1.png)

在數學中，方程式會有 `input(s)`，且總會有 `output`；如同在程式裡，也會定義各種 `input` 和 `output`，而在 FP 中，描述這種映射關係的專有名詞是 `morphism`。

> 所以，為什麼討論數學？因為 FP 就是使用 equation 的數學意義作為 `function`

## 你說的 function 不是 function
在學習 FP 以前，`procedure` 和 `function` 的區別可能沒那明顯，現在我們可以定義 `procedure` (程序) 如下

* 任意功能 code 的集合，做為邏輯上的切割，可以被調用一次到多次
* input：可能有一個或多個，也可能沒有
* output：可能有，也可能沒有

更進一步定義 `function` (函數) 如下

* `function` 接收 input，並一定有明確的 `return`

接下來的學習都會盡量使用 `function` (函數) 而不是  `procedure` (程序) ，並有明確的 `input` 與 `return`

## Input

承上定義，所有 `function` 都需要 `input`，
參數在英文有 arguments、parameters 以下解釋差異：

### 📚 arguments v.s. parameters
![](https://i.imgur.com/38jyKAG.png)

* 3, 6 為 arguments
* x, y 即為 parameters

### 💡 筆記 - 數量不一致時

在 js 中，傳入的 arguments 個數不一定要與 parameters 相等，
* arguments > parameters，多出的參數仍可用如 `arguments` 存取 [^1](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Functions/arguments)
* arguments < parameters，缺少的參數將被賦值 `undefined`

### 計算參數個數
function 會有預期接受的參數，而表示參數個數的名詞為 `arity`，以上方 `function binary (x, y)` 為例，`arity` 為 2。

* parameters 的數量，可使用 `function` 的唯獨屬性 `length`，以上方 `function binary (x, y)` 來說 `function.length = 2`。

* arguments 的數量，也就是實際傳入的參數數量，可已透過 `arguments` 物件的 `length` 取得。

### 💡 筆記 - arguments 的雷點
在未來版本替代 `arguments.length` 之前，使用 `arguments.length` 是 OK 的，也僅使用 `arguments.length` ，別使用 `arguments[index]` 去訪問值，為什麼？

![](https://i.imgur.com/sTz78ha.png)

結果為

![](https://i.imgur.com/R7X1qR3.png)

原因是因為，function 中 parameters 位置會對應 arguments，而這兩者的值都會參考到同一地方，所以對 line 6 `favoriteColor = "green"`，其實也連動改到 `arguments[0]` 的值，以下類推，相關討論請參考 [^2](https://spin.atomicobject.com/2011/04/10/javascript-don-t-reassign-your-function-arguments/)

### 如何寫出 Declarative 的 Input
還記得 Day 3 討論過的 `Imperative v.s. Declarative` 嗎？ 試著觀察下方兩段處理 Input `[1,2,3]` 的方式，

![](https://i.imgur.com/khigwf4.png)

**Imperative code**，如上方 `iStyle`，專注在如何得掉這個結果，我們如果知道這段的結果，就需要一行一行讀過程式，不是那麼顯而易見。

**Declarative code**，如 `dStyle`，專注在描述結果，也就是我們希望 Input 的樣子，

把握一個原則，也是這一系列文章的精神，盡可能使用 `declarative` 風格，能自我解釋的 code。

### 💡 筆記 - 探討與 FP 的關係
回顧一下上方提到一個術語 `arity` ，在 FP 中我們希望 `function` 的 `arity` 為 1，使用 `object destructuring` 或是 ES6 的 rest [^3](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) ，即可傳入多個參數，並且保持 `arity` 為 1，方便進行 `compose` ，這部分在接下來的文章會討論。


### 參考資料 
[^1]: [Arguments 物件](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Functions/arguments)
[^2]: [JavaScript: Don’t Reassign Your Function Arguments](https://spin.atomicobject.com/2011/04/10/javascript-don-t-reassign-your-function-arguments/)
[^3]: [Rest parameter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
