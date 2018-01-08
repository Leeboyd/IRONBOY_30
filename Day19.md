Good Morning, Functional JS (Day 19, Side effects)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

接觸過 FP 的人，可能多多少少都會聽過萬惡的副作用，接下來將學習各種副作用，並探討對程式碼造成的影響，

說在前面，作者提醒：要寫出沒有副作用的程式碼是不可能，畢竟零副作用的程式就是沒有用的程式，跟一個空括號 `{}` 沒有差異。


## 副作用
![](https://i.imgur.com/5dXpS9p.png)
副作用是指計算過程，系統狀態產生的變化，或者與外部世界的可觀察交互作用。

在電腦科學中，函數副作用指當調用函數時，除了回傳函數值之外，還對主調用函數產生附加的影響。例如修改全域變數（函數外的變數）或修改參數[^2]。


一些常見的副作用：
* 存取檔案系統
* 資料庫 insert
* http 請求
* log
* DOM 操作

請注意，副作用本身並沒有問題，畢竟寫程式就是處理輸入處理後輸出（也就是與外界交互作用），在 FP 我們學習的是讓副作用在我們可以控制的範圍，也就是能觀察到明確的因果關係，看看這段程式：

```
function foo(x) {
    return x * 2;
}

var y = foo( 3 );
```
這段程式有很明確的因果，因：呼叫 foo(3)，**直接**影響果：賦值 6 給 y。

再看這一段：

```
function foo(x) {
    y = x * 2;
}

var y;

foo( 3 );
```
這一段沒有直接的因果關係，呼叫 foo(3)，是**間接**影響 y，這邊就是我們說的副作用。

再看個長一點的程式：

```
var y = 1;

fn1();

fn2();

fn3();

console.log(y)
```
如果你不知道這些 fn 會不會有副作用，你能確定最後的 y 是你要的嗎？除非你從第一行開始追蹤，確認每一次呼叫後的改變。

反過來說如果知道這些 fn 不會有副作用，這表示我們可以花少一點心力追蹤，並且使程式碼更可靠。

### Side causes
有副作用 (side effect) 也有 side causes，看一段程式：
```
function func (x) {
    return x + y
}

var y = 3

func(1)     // 4
```
這裡的 y 並不會隨 foo() 改變，但如果這樣做：

```
var y = 6

func(1)     // 7
```
兩次的 func(1) 結果竟然不同！

這次的問題是出在因，func 間接的因（取決於外部的 y），影響了程式的可讀性，為了避免不可控的副作用，所有決定函數的因素應清楚，能夠觀察到明確的因果關係。

這並不是說我們的程式不能依賴自由變數，請看這段：

```
const PI = Math.PI

function circlePerimeter (r) {
    return 2 * PI * r
}
```

這次函數依賴外部變數 `PI` 上，那 `PI` 算 side cause 嗎？
我們這樣檢查副作用：

1. 每一次相同的輸入，都回得到同樣的結果嗎？Yes
2. `PI` 在程式中並沒有被重新賦值。

在這種清況下，`PI` 不是函數狀態的一部分，`PI` 是固定，不可重新賦值的，我們不需要追蹤 `PI` 的狀態，`PI` 也不會以不可預測的方式變化，影響我們的程式結果。


### 關於相同得輸入，相同的輸出
> A function is a special relationship between values: Each of its input values gives back exactly one output value.[^3]

還記得我們對函數的定義是來自於數學？每次的輸入都會有輸出，下圖是一個合法的函數關係：

![](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/images/function-sets.gif)

下圖就是一個不合法的函數，相同的輸入卻有不同的值：
![](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/images/relation-not-function.gif)
>圖片取自 mathisfun.com

而數學上的函數，在 FP 的世界裡，會特別稱作 **Pure Function**，純函數。

## 小結
學習 FP 時，寫出有 side effect / cause 是很正常的，如何辨識出因果關係，如何避免副作用是我們的重點，因為這不只會影響可讀性和推理，甚至會影響我們對自己程式碼的信任，不可不謹慎。

## 參考資料
[^1]:[mostly-adequate-guide](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch3.html#side-effects-may-include)
[^2]:維基百科：[函數副作用](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E5%89%AF%E4%BD%9C%E7%94%A8)
[^3]:function：[mathisfun.com:](http://www.mathsisfun.com/sets/function.html)
