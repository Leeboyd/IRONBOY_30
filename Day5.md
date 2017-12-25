Good Morning, JS (Day 5, You don't know Functions 續)
===

> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## Output

接下來探討 output，以下三個都有相同的結果：

```
function foo() {}

function bar() {
    return;
}

function baz() {
    return undefined;
}
```

### 💡 筆記
別忘了 FP 的原則：使用 `function` 而非 `procedure`，使用 `function` 必須有明確的 `return`，使用 `return;` 或是沒有 `return`，都會返回 `undefined`， `undefined` 並不是明確的返回值。

## return

> Developers prefer explicit patterns over implicit ones.

還記得 <code>f(x) = 2x^2 + 3</code> ，以 JS 表達：

![](https://i.imgur.com/4MAQIuS.png)

這兩個函數做的事情都一樣，兩者有差別嗎？

**當然有**

前者為一種隱式輸出，透過賦值改變外部 scope 的變數 y ; 後者就是顯式的輸出。

再看看一段，試問這是顯式還是隱式輸出：

![](https://i.imgur.com/aYKgMGc.png)

在 line 9 有明確的 return，但魔鬼藏在細節裡，試著在 line 79 和 line 81 觀察 `nums` 吧。

**line 12** :`[1, 3, 9, 27, undefined, 84]`

**line 15** :`[1, 3, 9, 27, 0, 84]`

我們雖然在 `function sum` 內部操作 list（line 4 將空值 `undefined` 補 0），但還是影響到 `function` 外部的 Array。

為什麼？因為局部變數 list 保持對 nums 的引用 `reference-copy`，而不是 `value-copy`，JS 中對 object, arrays, function 都是使用 `reference-copy` ，所以變動 list 變動，意外變動到外部的 nums 了，在 JS 中，這種事情太常發生了。

### 💡 筆記 - 陣列複製
slice() , Array.from(), `[...]` 都可以做到 shallow copy

## 副作用
這種改變到外部作用域的情況，在 FP 有個術語： `side-effect` (副作用)，相對的，沒有副作用的也有個術語： `pure` (純函數)，在接下來的文章會進一步討論。

## Syntax
![](https://i.imgur.com/GfGfPuO.png)

### 函數名稱

使用 `function declaration` 函數宣告式，需要指定一個名稱，

![](https://i.imgur.com/luFQDii.png)

具名和匿名具體上有什麼差別呢？ `function` 有一個屬性，叫做 `name` ，以字串儲存函數命名。如上方的 `myFunctionName` ，`name` 最常使用在 console、開發者工具等 JS 執行環境，我們在追蹤異常時，stack trace 會顯示。

![](https://i.imgur.com/Cf2nrVD.png)

大家在 debug 時，不知道遇到上圖中的的狀況：`(anonymous function)`，我們無法從 `(anonymous function)` 得知這個 exception 的任何資訊，對追蹤毫無幫助。而好的命名又比無意義的 foo 更有幫助。

在 ES6，`anonymous function expressions` 可以透過此方式獲得名稱，

![](https://i.imgur.com/da6Csth.png)

除了追蹤 bug 外，具名函數的優點還有，可以在函數內部自我引用，在遞迴函數就常使用，也方便事件處理，請看以下三段：

```javascript=
// sync
function flattenArray (array) {
  return Array.isArray(array)
  ? array.reduce((previous, current) => [...flattenArray(previous), ...flattenArray(current)], [])
  : [array]
}
```

```javascript=
// async 
setTimeout( function frogCry(){
    console.log('呱')
    setTimeout( frogCry, 2000 );
}, 2000 );
```

```javascript=
// 解除綁定
document.getElementById( "Btn" )
    .addEventListener( "click", function handleClick(evt){
        // 解除綁定
        evt.target.removeEventListener( "click", handleClick, false );

        // ..
    }, false );
```

還有一個更不常見的例子 **IIFE** (immediately invoked function expressions)，同樣地也應給予命名，如果不知道如何命名，至少使用 IIFE：

```javascript=
(function IIFE(){

    // 

})();

```

### 💡 心得 - 具名的優點
寫匿名函數十分容易，因為我們完全不想在命名上費勁，但因為方便易寫，其實是犧牲了可讀性。

如果實在是想不出好名字，可能沒有深入理解 `function` 的目的，或是功能太廣泛，應試著考慮重構以提高可讀性和維護性。

### `=>`

ES6 `=>` 語法，比較下列兩段

```javascript=
people.map( function getName(person){
    return person.name;
} );

// vs.

people.map( person => person.name );
```

關鍵字消失了，
* ~~function~~
* ~~{}~~
* ~~return~~

甚至， function name `getName` 也消失了，這些語法都被 `=>` 所取代。

如果， `person.name` 因為某些原因沒被定義，拋出 exception ，可以發現 `(anonymous function)` 會在 stack trace 的最上層...

使用 `=>` 語法意味著，
* 拋棄函數命名
* 犧牲可讀性
* debug 困難
* 自我引用？...做不到

開發者使用 `=>` 或多或少會遇到以下狀況，需要做些微調：

```javascript=
// 多個參數? 要加 ( )
people.map( (person,idx) => person.name );

// object destructure？需要 ( )
people.map( ({ person }) => person.name );

// 預設值? 需要 ( )
people.map( (person = {}) => person.name );

// return Object? 需要 ( )
people.map( person =>
	({ preferredName: person.name })
);
```

老實說，我喜歡匿名函數，更喜歡 `=>` 這種簡潔的語法，能夠優化 code 的空間，如同段落標題作者說的，增加或減弱可讀性由你決定。

## You don’t know Functions 小結：

這兩天的內容探討了 `function` 的 Input 、 Output 以及語法，整理了幾個原則：

* `function` 需要有一個或多個（只有一個是最好）Input，和明確 Output
* 注意 `function` 的副作用
* 使用 `anonymous functions` 與 `=>` 須謹慎，易寫但難讀。
