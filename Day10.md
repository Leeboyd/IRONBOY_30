Good Morning, Functional JS (Day 10, Partial Application 偏函數應用)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)


## 一些現在傳，一些等一下再傳
> Partial Application，偏函數應用

如果一個函數接受多個參數，你可以會先傳入部分參數，剩下的稍後傳入，（待全部參數確定後再執行）。

模擬一個場合，要發一個 API 請求，但資料和 callback handler 要稍後才知道（可能在等待上一個 API 請求）。

```
function ajax( url, data, cb ) {

}
```

創建一個函數，手動設置 `ajax` 第一個參數，並等待另外兩個參數 data 和 callback

```
function getOrder(data, cb) {
    ajax( "http://some.api/order", data, cb );
}
```

當然可以再進一步手動做這樣的操作，

```
function getLatesrOrder(cb) {
    getOrder( { id: ORDER_ID }, cb );
}
```

相信讀者應該看出模式了，身為一個工程師應該很習慣在重複的行為中，找到轉換為邏輯的方法，上面三段手動修改只是為了演示，現在來檢查其中的概念。

## Partial Application
請原諒我直接用術語來說明： `getOrder(data,cb)` 是 `ajax` 的偏函數 *partial application* ，`getOrder` 套用 (apply) 了實際參數 `"http://some.api/order"` 到形式參數 url 上，

換個說法，原先的 `ajax` 定義了三個形式參數 (parameter)，假設我已知 url ，我先套用 `"http://some.api/order"` 到 url，先套用部分參數，接下來只要呼叫剩下兩個參數的 `getOrder(data,cb)` 版本即可，

還是很混亂？來段英文吧

![](https://i.imgur.com/vESfLhH.png)

翻譯：偏函數 *partial application* 正式來說是一種減少函數參數個數 `Arity` 的流程，`Arity` 指的是形式參數 parameter 的個數，如原先 `ajax` 的 arity 從 3 個減少到 2 個。

有了正式定義，就來實現一個 `partial(..)` 工具函數

```javascript=
function partial(fn, ...presetArgs) {
    return function partiallyApplied(...laterArgs) {
        return fn( ...presetArgs, ...laterArgs )
    }
}

// ES6, =>

var partial =
    (fn, ...presetArgs) =>
        (...laterArgs) =>
            fn( ...presetArgs, ...laterArgs )

```

**建議**：只是掃過看看是不行的喔！這個小工具的概念會一直出現在這系列文，請務必熟悉。

* fn: 將被 partially apply 偏函數應用的函數，就是要被縮減參數個數
* presetArgs: `...` 收集部分先傳入的參數，保存到 presetArgs 陣列做稍後使用。

為什麼可以稍後使用呢？為什麼在 `partial(..)` 結束後，內部函數為何還能引用 `fn` 和 `presetArgs`？ 就是 **closure** 哦！！，在內部呢，創建並 return 一個函數 `partiallyApplied`，`partiallyApplied` 閉包 **closes over** 了 `fn` 和 `presetArgs`。

所以，稍後 `partiallyApplied` 在程式別處執行時，它其實是使用了傳入 `partial` 然後被 close over 的 `fn` 運行原函數，而參數呢？先是 `presetArgs` : 一開始作為 partial application 先傳入的參數（現已被 close over），接著是近一步傳入的 `laterArgs`。

**建議**：困惑嗎？正常的，可以的話參考原文[^2](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#some-now-some-later)，接下來要更深入了。

## 重構
接著，來使用我們的 FP 小工具 `partial` 來重構一開始的三段，

```
var getOrder = partial(ajax, "http://some.api/order")
```

記得有一個 return 嗎，現在試想 `getOrder` 的內部的樣子

```javascript=
var getOrder = function partiallyApplied (...laterArgs) {
    return ajax("http://some.api/order", ...laterArgs)
}

```

那 getLatesrOrder 如何重構呢？

```javascript=
// ver1
var getLatesrOrder = partial(
    ajax,
    "http://some.api/order",
    { id: ORDER_ID }
)

// ver2
var getLatesrOrder = partial( getOrder, { id: ORDER_ID })
```

* ver1: 直接指定 url 與 data 的實際參數來定義
* ver2: 將 getLatesrOrder 定義成 getOrder 的 partial application，並指定 data。

ver2 重用了已定義好的 getOrder，所以表達上更清楚，也比較符合 FP 的精神。

再看看以下這段 code ，試著全盤理解 ver1 和 ver2 的內部運作

```javascript=
// ver1
var getLatesrOrder = function partiallyApplied (...laterArgs){
    return ajax(
        "http://some.api/order",
        { id: ORDER_ID }
    )
}
// ver2
var getLatesrOrder = function outerPartiallyApplied (...outerLaterArgs) {
    var getOrder = function innerPartiallyApplied (...innerlaterArgs) {
        return ajax("http://some.api/order", ...innerlaterArgs)
    }
    return getOrder({{ id: ORDER_ID }}, ...outerLaterArgs)
}
```
**建議**：為何要這樣一層一層包裝，哈！這就是 FP，請務必熟悉與習慣。

## 再來個 Partial Application: 範例
```javascript=
function add (x, y) {
    return x + y
}

// ES6, =>

var add = (x, y) => x + y
```

現在我們有一個成績陣列，因為題目出錯全體加十分，使用的是 `map(...)`

```javascript=
[30, 55, 42, 87, 66].map( ... )

// [40, 65, 52, 97, 76]
```

因為 `add(x, y)` 的函數簽名（signature），不是 `map` 預期，不能直接傳入 `map(...)`，這時候我們就可以利用 **partial application** 幫助調整：

```javascript=
[30, 55, 42, 87, 66].map( partial( add, 10 ))
```

## 參數反序
在思考一個場合，如果 `ajax( url, data, cb )` 現在要先傳 cb 稍後再 傳 `data` 和 `url`（**partially apply**，現在來做一個 `reverseArgs`） FP 小工具：

```javascript=
function reverseArgs(fn) {
    return function argsReversed(...args){
        return fn( ...args.reverse() );
    };
}

// ES6 =>
var reverseArgs =
    fn =>
        (...args) =>
            fn( ...args.reverse() );
```

現在不再從左邊開始，而是從右邊開始偏應用 **partially apply**，另外，為了在 call function 的時候仍然使用原順序，所以在偏應用處理後，又在反序一次：

```javascript=
var getOrderWithHandler = reverseArgs(
    partial(reverseArgs( ajax ), function handler() {
        // ...
    })
)

```

好吧！來試著解釋這一段吧，FP 洋蔥語法從內層開始剝，（僅示意）

1. reverseArgs( ajax ) 將參數反序並 return `(cb, data, url)`
2. partial(...) 將 handler 套用在 `(cb, data, url)` 第一個位置 `cb` 並 return `(data, url)`

3. 也就是 `cb` 已被 **closes over** 了
4. 最後再次使用 reverseArgs ， return (url, data)
5. 故處理後，可以如同原本使用方式，

為了不手動做 reverseArgs 兩次，試著做另外一個 FP 小工具 `partialRight`

## PartialRight

```javascript=
function partialRight(fn, ...presetArgs) {
    return reverseArgs(
        partial( reverseArgs( fn ), ...presetArgs.reverse() )
    );
}
```

如果你還是覺得兩次 reverseArgs 很冗餘，試試這樣

```javascript=
function partialRight(fn,...presetArgs) {
    return function partiallyApplied(...laterArgs){
        return fn( ...laterArgs, ...presetArgs );
    };
}

// ES6 =>
var partialRight =
    (fn,...presetArgs) =>
        (...laterArgs) =>
            fn( ...laterArgs, ...presetArgs );
```

不管是哪一個 partialRight，都不能保證讓一個特定形式參數 parameter 被偏應用 **partially applied**，只能確保傳入的值是最右邊的實際參數：

```javascript=
function some(x,y,z,...rest) {
    console.log( x, y, z, rest );
}

var f = partialRight( foo, "在你右邊" );

f(  );              // "在你右邊" undefined undefined []

f( 1, 2 );          // 1 2 "在你右邊" []

f( 1 );             // 1 "在你右邊" undefined []

f( 1, 2, 3 );       // 1 2 3 ["在你右邊"]

f( 1, 2, 3, 4 );    // 1 2 3 [4,"在你右邊"]
```

## 小結：
終於開始好玩的東西了，今天講的主題 **Partial Application**，偏函數應用，或者叫部份套用函式 Partially applied function，都是屬於翻譯差異，建議讀者有興趣一定要去翻翻相關資料，下個結論：對於某個函數，會先傳入部分實際參數，剩下的稍後傳入，稱之。

## 參考資料
[^1]: [Partial application](https://en.wikipedia.org/wiki/Partial_application)
[^2]: [Some Now, Some Later](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch3.md#some-now-some-later)


