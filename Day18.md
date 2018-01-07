Good Morning, Functional JS (Day 18, Ramda 實作)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## Pointfree 風格 與 函數合成

關於 Pointfree 的討論，請[參考 Day14](https://ithelp.ithome.com.tw/articles/10195632)，先來看一段簡單的範例，參考自 阮一峰 的文章 [Pointfree 编程风格指南](http://www.ruanyifeng.com/blog/2017/03/pointfree.html)[^1]，範例中用到 [Ramda](http://ramdajs.com/)，Ramda 是一個 FP library，提供了很多現成的方法，可以直接使用这，省得自己定義一些常用工具函數（也就是我們一直在做的）。

## Pointfree 範例1
延伸 [Day15](https://ithelp.ithome.com.tw/articles/10195854) 的範例，使用到的基本運算如下

```
// 分割字串
function splitString(str) {
    return String( str )
        .toLowerCase()
        .split( /\s|\b/ )
        .filter( function alpha(v){
            return /^[\w]+$/.test( v );
        } );
}

// 去除重複單字
function deDuplicate(arr) {
    return Array.from(new Set(arr));
}

// 找出長度大於 4 的單字
function skipShortWords(arr) {    
    return arr.filter(word => word.length > 4)
}
```
然後，將基本運算合成，

```
var getLongWords = R.pipe(
  splitString,
  deDuplicate,
  skipShortWords
);

getLongWords(str)
```

運算過程中，`str` 是傳入的值，`getLongWords` 是處理這個值得函數，定義 `getLongWords` 時完全沒有提到 `str`，這就是 **Pointfree** 風格。

現在要找出最長的單字，加入兩個基本運算

```
// 返回較大的單字
var getBiggerWord = (a, b) => a.length > b.length ? a : b;

// 返回最大的單字
var findBiggestWord = arr => R.reduce(getBiggerWord, '', arr);
```
一樣將基本運算合成，

```
var getBiggestWords = R.pipe(
  splitString,
  deDuplicate,
  skipShortWords,
  findBiggestWord
);
```
每一個步驟加上語義化的命名，整個過程四個步驟非常清晰，這就是 Pointfree 的優點。

實驗一下吧，請參考[^JSBIN 完整示範](https://jsbin.com/jaluzibile/1/edit?js,console)

## Pointfree 範例2
現在想像要做一個 api 請求，

```
ajax("http://some.api/order", {'id': 1 })
```
需要找出某位員工的訂單，需要輸入 {id: 員工id }這樣的資料，

假設我們要在員工資料表，找出職稱為 'Worker' 的員工 id

```
var employees = [{
  "id": 1,
  "role": "Manager",
  "name": "Amos"
}, {
  "id": 2,
  "role": "Worker",
  "name": "Gawen"
}, {
  "id": 3,
  "role": "IT",
  "name": "Renado"
}]
```
再來先定義一些方法：

* 要取出某位員工的 role 屬性，你可能會這樣寫 `obj.role`，或寫：

```
function extractPropRole (person) {
    return person.role;
}
```

* 但在 FP 世界，我們希望能將共通的邏輯抽出：可以把任意物件的任意 key 值取出：

```
function prop(key, obj) {
    return obj[key];
}
```
> 物件取值

* 同樣的方式，也定義一個物件賦值的方法

```
function setProp(key, obj, value) {

    // 避免動到原本物件（副作用）
    var o = Object.assign( {}, obj );
    
    o[key] = value;
    return o;
}
```
> 物件賦值

* 重新定義一個 extractPropRole，取出物件中的 role，做法是將 role 偏應用到 prop: 

```
// partially applied prop
var extractPropRole = R.partial(prop, ['role'])
```
* 找到 role 為 Worker 的員工

```
function isWorker(role) {
    return role === 'Worker'
}

// 註：使用 find 只會找到第一個符合
var getWorker = R.find(R.pipe(extractPropRole, isWorker))
```

* 到目前為止的資料流：(因為使用 pipe 所以我寫成左到右)

```
employees --> extractPropRole --> getWorker --> worker
```

* 到目前為止還不錯，接著要製作一個物件 { id: id }，傳入ajax

```
function makeObj(key, value) {
    return setProp( key, {}, value );
}
```
> 製作物件的方法

* 從找到的員工，取出該員工的 id，這次對 prop 使用 curry： 

```
// curried function
var extractPropId = R.curry(prop)('id');
```

* 再將 id 偏應用到 makeObj，用來製造給 ajax 的物件（有 id 的）

```
var queryData = R.partial(makeObj, ['id']);
```

* 最後，整個傳起來

```
R.compose( queryData, extractPropId, getWorker );
```
* 整理資料流：(因為使用 compose 所以我寫成右到左)
```
{id} <-- queryData <-- extractPropId <-- getWorker <-- extractPropRole <-- employees
```

* 全部程式碼：，請參考[^JSBIN 完整示範](http://jsbin.com/kagejasibe/1/edit?js,console)

在推導過程中都沒用到 `employees`，這就是 Pointfree 風格，也可以看到 partial 和 curry 在整個函數組合過程的相當實用，重構參數數量使前後輸出輸入間更容易結合。

## 函數組合結論
函數組合是 FP 最重要的核心：

* 一種定義函數的方式，上一個輸出會是下一個函數的輸入。

* 因為 JS 只能 return 單值的特性，所以將參數調整成 `arity = 1` 單參數函數會非常方便做組合。

* 組合更是一種 Declarative 聲明式的資料流寫法，讓我們把關注放在 what (要做什麼，要產生什麼結果)而不是 how（一步步怎麼做）。

## 參考資料
[^1]:[Pointfree 编程风格指南](http://www.ruanyifeng.com/blog/2017/03/pointfree.html)
