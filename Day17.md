Good Morning, Functional JS (Day 17, Composition - pipe )
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## 從左到右
前幾天探討的 `compose(..)` 邏輯是從右到左，剛好對應到手動合成函數的樣子，

```
// 右到左
compose(fn2, fn1)
```

```
// 內到外
fn2( fn1() )
```

在 FP 還有另一種合成的方式是從左到右處理，通常在 FP 函數庫稱作 `pipe(..)`。

```javascript=
function pipe(...fns) {
    return function piped(result) {
        var list = fns.slice()

        while (list.length > 0) {
            result = list.shift()(result)
        }
        
        return result
    }
}

// ES6

var pipe = (...fns) =>
    result => {
        var list = fns.slice()
        
        while (list.length > 0) {
            result = list.shift()(result)
        }
        
        return result
    }
```
> 小工具 pipe(…)

邏輯上，其實只要把 `compose(..)` 的參數反轉就能實現，

```
var pipe = reverseArgs( compose )
```

重構找長單字範例

```
compose(skipShortWords, deDuplicate, splitString)

// pipe 版本

pipe(splitString, deDuplicate, skipShortWords)
```
> 範例1 compose 與 pipe 對照

使用 `pipe(..)` 最大的優點是在於以函數的執行順序排列參數，`pipe(splitString, deDuplicate, skipShortWords)` 閱讀順序就是執行順序，先 splitString 再 deDuplicate 最後再 skipShortWords。

## pipe 與 partial application
也順便結合之前所學使用 pipe 與 partial，看看與 compose 的差異，在 [Day 16](https://ithelp.ithome.com.tw/articles/10196003) 我們使用 **partialRight** 將參數順序反轉應用在 compose，
```
var wordsFilterBy = partialRight(compose, deDuplicate, splitString)

var longWords = wordsFilterBy( skipShortWords )
```
> 範例2 找長單字 partialRight compose 應用

對比使用 pipe，不需反轉順序，直接使用 **partial**

```
var wordsFilterBy = partial(pipe, splitString, deDuplicate)
```

## 關於抽象化
抽象化的概念經常被定義為將通用的部分切出去，在多處使用通用邏輯，修修改改常會有忘記改到的風險，常聽到一個抽象化原則：**DRY（don't repeat yourself）**，就是為了避免這個風險。

關於組合，最大的好處就是做到了 DRY。除了 DRY，我們學習 FP 更高的目標是在分解程式碼成為一塊塊的小積木。

分解後，程式碼的目的更明確，處理複雜性問題的方法就是拆解，分離出關注點。
![](https://i.imgur.com/26lVf5h.png)

除了分解，另一個方式就是寫** Declarative** (宣告式，聲明式)風格，對比 **Imperative** (命令式)，聲明式描述輸出的樣子，命令式關心詳細實作。

以 ES6 的 destructing 解構賦值為例：
```
function getData() {
    return [1,2,3,4,5];
}

// Imperative
var tmp = getData();
var a = tmp[0];
var b = tmp[3];

// Declarative
var [ a ,,, b ] = getData();
```
> 範例3 Declarative v.s. Imperative

## 組合與抽象化
回到函數組合，先比較 Declarative v.s. Imperative 的組合方式：
```
// Declarative 聲明式
var longWords = compose(skipShortWords, deDuplicate, splitString)

// Imperative 命令式
function longWords(text) {
    return skipShortWords( deDuplicate( splitString( text ) ) );
}
```
使用函數組合是一種聲明式的抽象，我們關心 `what`:
一個字串經過會 splitString、deDuplicate、skipShortWords；而把 `how`: 一步一步得到結果的細節留在 compose 裡做。

對比使用命令式寫
```
var wordsArray = splitString( loremString )

var wordsUsed = deDuplicate( wordsArray )

skipShortWords(wordsUsed)
```
函數組合不是只有 DRY，更能夠把關注點放在 **What**，提升程式可讀性。

## 小結：

pipe(..) 是函數組合的另一種方式，讓參數閱讀順序就是執行順序，

而函數組合更是強大的抽象化的工具，能夠把命令式的`how` 抽象為可讀性更好的聲明式，讓程式專注在 `what`。
