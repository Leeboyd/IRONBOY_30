Good Morning, Functional JS (Day 16, Composition part 2)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## 多函數合成
既然可以合成兩個函數，當然也可以合成多個：
```
output <-- fn1 <-- fn2 <-- ... <-- fnN <--input
```

定義小工具 `compose(..)` 如下：

```javascript=
function compose(...fns) {
    return function composed(result) {
        // 複製 fns 陣列到 list
        var list = fns.slice()
        
        while (list.length > 0) {
            // 從右到左拿出函數並執行
            result = list.pop()(result)
            
        }
        
        return result
    }
}

// ES6
var compose = (...fns) =>
    result => {
        var list = fns.slice()
        while (list.length > 0) {
            result = list.pop()(result)
        }
        return result
    }
```
> 小工具 compose(..)

延伸 [Day15](https://ithelp.ithome.com.tw/articles/10195854) 的範例，使用到的基本運算如下

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

function skipShortWords(words) {    
    return words.filter(word => word.length > 4)
}
```
> code 字串分解、去除陣列重複元素、排除短字串 [^源碼連結](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch4.md#general-composition)

現在要從一串字串中找出長度大於4的單字，手動合成這些函數：

```
var result = skipShortWords( deDuplicate( splitString(str) ) )
```
使用 **compose(..)** 自動合成：

```
var longWords = compose(skipShortWords, deDuplicate, splitString)

var result = longWords(str)
```
> 範例1 找長單字 compose(..)應用，[^JSBIN 練習](https://jsbin.com/duqazuvuno/edit?js,console)

## 應用：結合 partialRight
現在試著應用 partialRight 的概念，實作一個右偏應用的 compose (也就是預先定義 `splitString`, `deDuplicate`)，稱為 `wordsFilter(..)`:

```
var partialRight =
    (fn,...presetArgs) =>
        (...laterArgs) =>
            fn( ...laterArgs, ...presetArgs );
```
> 關於 partialRight，見 [Day 10](https://ithelp.ithome.com.tw/articles/10194837) 討論

```
var wordsFilterBy = partialRight(compose, deDuplicate, splitString)

var longWords = wordsFilterBy( skipShortWords )
```
> 範例2 找長單字 partialRight compose 應用

如果在第二步驟，使用 skipLongWords：

```
var shortWords = wordsFilterBy( skipLongWords )
```
> 範例3 找短單字 partialRight compose 應用

從這邊可以看到在第二步使用不同的 filter (**skipShortWords** 和 **skipLongWords**) 創造出功能不同的函數，這就是 **函數合成 Composition** - FP 最強大的地方。

## 應用：結合 currying
當然也可以使用 **currying** 處理，

```
var curry = (fn, ARITY = fn.length, nextCurried) =>
    (nextCurried = prevArgs =>
        nextArg => {
            var args = [...prevArgs, nextArg]

            if (args.length >= ARITY) {
                return fn(...args)
            } else {
                return nextCurried(args)
            }    
        }
    )( [] )
```
> 關於 currying，見 [Day 11](https://ithelp.ithome.com.tw/articles/10195145) 討論

因為 compose 參數由右到左處理，如果是使用我們自製的 currying，可能需要使用 `reverseArgs` 操作，並且指定參數數目為 3，因為 compose 是可變參數函數，寫法如下：

```
var curried_compose = curry(reverseArgs(compose), 3)

curried_compose(splitString)(deDuplicate)(skipShortWords)
```
> 範例4 找長單字 curry compose 應用，[^JSBIN 練習](https://jsbin.com/vehigevaba/edit?js,console)

## 另一種 Compose (reduce 版)
試著使用 **reduce(..)** 重構 **compose(..)** ：

```
function compose(...fns) {
    return function composed(result) {
        return fns
        .reverse()
        .reduce(function reducer( result, fn ) => {
            return fn(result)
        }, result )
    }
}

// ES6

var compose = (...fns) =>
    result => 
        fns
        .reverse()
        .reduce((result, fn) => fn(result), result)        
    
```
> 小工具 compose(..)，reduce(..)版本

## 另一種 Compose (lazy 版)
到目前為止，我們組合的每一個函數都是單參數函數，如果要傳多實參給第一層，可以加一層 lazy-evaluation function 包裝：

```
function compose(...fns) {
    return fns.reverse().reduce( function reducer(fn1,fn2){
        return function composed(...args){
            return fn2( fn1( ...args ) )
        }
    } )
}

// ES6

var compose = (...fns) =>
        fns
        .reverse()
        .reduce( (fn1,fn2) =>
            (...args) => fn2( fn1( ...args ) )
        );
```
> 小工具 處理多參數輸入的 compose 

### 運作邏輯不同處
* 可處理多參數
* 與其每一次都計算後將結果再進到下一個循環，這次 defer 延遲所有的運算
* 每一次的循環都會返回一個包裹層級更多的函數
* reduce 最後結果得到一個函數
```
function composed(...args){
            return fnN(...fn2( ( fn1( ...args ) )...)
}
```
* 傳入參數後，最終組合函數再由內到外處理參數
* 而之前做法是，每一次 reduce 都從右邊拿一個函數運行，再進到下一個循環

## 小結
今天整理了更多 compose 的方式，以及回顧了 partial 和 curry，結合各種函數的 FP 真的越來越有趣了，明天將繼續探討 compose 的議題。
