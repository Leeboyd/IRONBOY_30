Good Morning, JS (Day 9, ES6 Class 地雷)
===
> 今天為番外篇，最近研究 FP ，在 programming paradigm 除了 FP，大家肯定聽過 OOP，其中 class 父子繼承是最常見的模式，在 ECMA-262 提出 Class 語法，現在來探討一下與傳統 Class 的差異

## 比較 ES6 Class 與 Factory Method[^1](https://via.intercom-mail-200.com/e?ob=FRzAGBR3lVktCI0rBBKLzXoZAvjIxHabYGfROwzYt%2BDR00U5thJIfMxT2e9g9n3X&h=e83fcf36f486fc13cbc9089782dc96c175b34bd7-13220203316)
ECMAScript 2015 介紹了類別 `classes` 作為 JavaScript 現有 prototype-based 繼承的語法糖 `syntactical sugar`，也就是說，並非是將傳統的物件導向模式引入，基本上只是一個定義 constructor函數 與 prototype-based 繼承的簡潔寫法而已[^2](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Classes)。

以下將探討語法並比較：
左邊是 ES6 class，右邊是 Factory function (也稱作 module pattern)

![](https://i.imgur.com/cgSW1bT.png)

而產生物件的方式語法 (實例化)，
![](https://i.imgur.com/KHMFBtQ.png)

當然，結果都是一樣輸出 `funny`，效能以及可讀性也差不多。

但如何解讀這兩段可就不同了，對已經熟悉 class 的使用者（C++, JAVA）認為應使用 ES6 class 來結構程式邏輯，但如果用傳統物件導向來理解 JS ES6 class 可能會踩到雷，如前面所說這是一個語法糖衣。

### 繼承
接著比較 ES6 class inheritance  以及物件 composition，

![](https://i.imgur.com/iEucZuY.png)

左邊 line 10 `Button` 繼承 `Widget`，並定義一個方法 `click(..)`，其中 click 呼叫了 `print(..)`，但實例化的 bbb 中並沒有實作 `print(..)` 方法，所以當 line 18 呼叫了 click (也就是 print） ，會透過 `[[Prototype]]` 一路 delegate 給 line 6 Widget 的 print 執行，

![](https://i.imgur.com/Lrm1Htx.png)
> The Prototype Chain[^3](https://medium.com/javascript-scene/3-different-kinds-of-prototypal-inheritance-es6-edition-32d777fa16c9)

在看右邊寫法，捨棄了抽象化的繼承關係，而做到跟左邊 ES6 的繼承方式，Widget 和 Button 彼此都是物件，之間有 `prototype-chain` 相關，差異在 line 16 中，透過 `Object.create()` ， 使 bbb 與 Button 有 `prototype-chain` 相連，因為沒有用 `new` 或 `constructor` 所以在 line 17 做簡單的初始化 `setId`， line 18 就兩邊相同了。

雖然右邊的寫法視覺上較冗長，但其實就是就是三個物件 (Widget、Button、bbb)，之間有 `prototype-chain` 相關，`print(..)` 方法也在這三個物件中共享。

## ES6 Class 的雷點

前面提到習慣左邊寫法者，可能會在 ES6 class 踩到雷，為何這樣說呢？當 line 16 實例化一個新物件 bbb 時，會預期 bbb 會永遠保持相同的 context，也就是耳熟能祥 `this`，結果當我們這樣使用時，

```javascript=
$('#btn').click(bbb.click) // btn
```

結果不是預期的 `'lol'` 而是 `btn`，也就是印出 jQuery 事件監聽對象的 id，而不是初始化傳入的 `lol`，當然右手邊也是同樣的結果。

![](https://i.imgur.com/DNKJEzn.png)

動態的 context 很適合 `prototype-chain` 這種模式，相對的，卻很不適合實作 class 模式，習慣用 class 預期 ES6 class 也會靜態綁定 this，但其實並沒有，你可能會這樣做，

```javascript=
$('#btn').click(bbb.click.bind(bbb)) // 'lol'
```
用 `bind` 或者用 `=>`，

```javascript=
class Widget {
    constructor(id) {
        this.id = id
        this.print = () => {
            console.log( this.id )
        }
    }
}
```

當你這樣做的時候，也就表示承認在 JavaScript 使用 ES6 Class 是一件很尷尬的事，一種 **明明知道 JS 是動態 context，卻硬是靜態綁定 context**，只為了繞過或捨棄 `prototype-chain` 模式；更進一步說，基於 ES6 Class 僅作為語法糖衣，底層還是 `prototype-chain` ，會分享狀態、方法，很多靜態繼承的特色其實很難在 JS 中實現，如 private 關鍵字。

## 小結
在使用一個新語法之前，先行研究與了解才能判斷是否適合，ES6 是一個好的模式嗎？我認為不是，如果從別的語言轉來的，我認爲反而造成混亂，而使用綁定的方式，硬是調整成預期行為，其實是沒有意義的，不如熟悉 JS 的的動態特性以及彈性，我認為這是比較正確的心態。

## 參考資料
[^1]: Jaswinder asked [when to use a Class in JavaScript vs a factory method and also composition vs. inheritance](https://via.intercom-mail-200.com/e?ob=FRzAGBR3lVktCI0rBBKLzXoZAvjIxHabYGfROwzYt%2BDR00U5thJIfMxT2e9g9n3X&h=e83fcf36f486fc13cbc9089782dc96c175b34bd7-13220203316).
[^2]: [Classes](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Classes)
[^3]: [3 Different Kinds of Prototypal Inheritance: ES6+ Edition](https://medium.com/javascript-scene/3-different-kinds-of-prototypal-inheritance-es6-edition-32d777fa16c9)
