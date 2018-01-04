Good Morning, Functional JS (Day 15, Composition part 1)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

轉眼連載 Kyle 的 ***Functional-Light JavaScript*** 到了一半，相信大家對於 Functional Programing 中使用 function 應該都有概念了。

![Photo by Iker Urteaga on Unsplash](https://images.unsplash.com/photo-1493217465235-252dd9c0d632?auto=format&fit=crop&w=400&q=80)
> 函數就像積木 Photo by Iker Urteaga on Unsplash

函數就像一塊塊的 `樂高積木`，例如這幾天討論的各種小工具（詳細看 [Day 14](https://ithelp.ithome.com.tw/articles/10195632)註解中有整理）我們明確定義各種小工具如何運作，以及能夠解決什麼問題，當然也可以藉由組合這些小積木成為新的工具，做更複雜的應用，這就是接下要研究的FP 核心 - **composition 合成**。

## 函數的合成
還記得我們討論過的 `[].map(unary(parseInt))` 嗎？現在來看看其中的運作。

函數組合的原理：**一個函數的輸出作為下一個函數的輸入**。
* `unary(parseInt)` return 一個函數[^1]
* 這個函數作為實際參數傳入 `map(..)` 當作 mapper function
* 最後 `map(..)` return 一個陣列

概念如下：

```
[] <-- map <-- unary <-- parseInt
```
唸一次：
parseInt 為 unary 的輸入，unary 的輸出為 map 的輸入，map的輸出為 []。

原作者 Kyle 使用工廠的生產線做類比，解釋函數的合成。

![](https://cdn.dribbble.com/users/434031/screenshots/2397782/_gif2.gif)
> 生產線 by Travis Fadjo for Treehouse on Dec 9, 2015

### 首先定義出兩個工具
```
function splitString(str) {
    return String( str )
        .toLowerCase()
        .split( /\s|\b/ )
        .filter( function alpha(v){
            return /^[\w]+$/.test( v );
        } );
}

function deDuplicate(list) {
    return Array.from(new Set(list));
}
```
> code 字串分解、去除陣列重複元素[^2](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch4.md#output-to-input)

### 使用這兩個小工具分析字串
```
var loremString = 'Cras eleifend, in hendrerit tellus feugiat. Nulla non malesuada orci. Sed orci tellus.'

var wordsArray = splitString( loremString )
var wordsUsed = deDuplicate( wordsArray )

console.log(wordsUsed)
```
> 範例1 [JSBIN 實作連結](https://jsbin.com/hekedurevi/1/edit?js,console)

### 目前為止這條生產線運作的還行

* 原料：**loremString**
* 輸入機器 **splitString** ，輸出半成品 **陣列 wordsArray**
* 輸入第二台機器 **deDuplicate**，輸出成品 **陣列 wordsUsed**

### 改進
中間有一個步驟輸出半成品 **陣列 wordsArray**，然後再作為 **函數 deDuplicate** 的輸入，我們這樣想像：生產線不需要輸送帶了，直接把兩台機器接再一起，上一個機器的輸出直接輸入到下一個：

```
var wordsUsed = deDuplicate( splitString( loremString ) )
```
> 範例2 機器接在一起，節省空間
> 注意：這邊函數的閱讀順序是**由內到外** 先`splitString` 再來是 `deDuplicate`，**從左到右**的模式，在 FP 稱作 **pipe**

### 隱藏細節
我們的生產線概念如下：
```
wordsUsed <-- deDuplicate <-- splitString <-- text
```
現在我們知道組合順序，並知道輸入一個字串會輸出一個陣列，定義一個函數將中間細節包裝起來：

![](https://i.imgur.com/OE8WIiX.png)
> 範例3 兩個函數合成變一個

透過生產線的類比，應該可以比較了解函數合成了。

### 函數合成工具
每次都要靠手動合成函數太麻煩，定義一個新工具，自動合成兩個函數：

```
function compose2fn (fn2, fn1) {
    return function composed (value) {
        return fn2( fn1( value ) )
    }
}

// ES6
var compose2fn = (fn2, fn1) =>
    value =>
        fn2( fn1( value ))
```
> 小工具 建立兩個函數的組合[^3]

注意到了嗎？參數順序為 `fn2`, `fn1`，卻是第二個參數會先運作，然後才是第一個，原因是為了與大部分 FP 函數庫中 **compose(..)** 由右到左的工作方式一致。

```
var deDuplicateString = compose2fn (deDuplicate, splitString)
```
> 範例4 自動化合成 v.s. 範例3 手動合成

### 變化合成方向
既然是積木，就可以任意組合，如果將生產線概念改成：
```
 <-- splitString <-- deDuplicate <-- text
```

搭配使用小工具 **compose2fn**：

```
var uniqueCharacter = compose2fn(splitString, deDuplicate)
```
> 範例5 另一種組合

## 小結
一個函數的輸出是另一個函數的輸入，今天的範例簡介函數組合的概念，明天繼續討論。

## 參考資料
[^1]: `var unary = fn => arg => fn( arg );`

[^2]: [Output to Input](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch4.md#output-to-input)

[^3]: [Machine Making](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch4.md#machine-making)
