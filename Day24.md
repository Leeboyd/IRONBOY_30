Good Morning, Functional JS (Day 24, 使用 Immutable.js)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

不知道讀者有沒有注意到，我們處理副作用的手法 - 當需要改變程式中的狀態時，我們不改變已存在的值，而是創造並追蹤新值，此時就會衍生新的問題 - **效能**。

每當要改變陣列或物件時，我們使用`Array.prototype.slice()` 或 `Object.assign` ...等方式，創造一個新副本，返回後不再使用，然後被垃圾回收，每個動作都會佔用額外的運算和記憶體資源。

如果這樣的複製操作出現一兩次，效能耗損可以忽略；但如果是頻繁的複製、資料結構複雜、或是屬於核心邏輯，那就要慎重考慮效能問題了。

> 關於效能以及優化的論述，請參考原作者 Kyle Simpson. ***Functional-Light JavaScript*** : Ch6 Value Immutability - [Performance](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch6.md#performance) 一節。

## [Immutable.js](http://facebook.github.io/immutable-js/)

這節介紹以 Immutable.js 這套函式庫，用來實作值不可變性的資料結構，讓開發者更容易追蹤狀態變化，更有提供效能優化，降低資源耗用（相對於手動複製）。

Immutable.js 支援多種資料結構，包括 `List` 與 `Map`，**類似**原生 JS 的 `Array` 與 `Object`。 

### List
相當於 JS 的陣列，使用 `List.of` 建立一個 List，此外 JS 陣列方法，在 Immutable.js 都有實作，可以無縫使用：
```
var list = Immutable.List.of(...['薯條', '漢堡', 'Pizza', '冰', 'Beer', '熱狗', '餅', 'tacco' ])

list.push('香腸')

list.shift()

list.set(6, '玉米');

console.log(list.size);
// 大小不變

console.log(list.toArray().slice());
// 內容不變
```

從上例可以看出，就算使用不純的 Array.prototype 方法：
`splice(..)`、`pop(..)`、`push(..)`、`shift(..)`、`unshift(..)`、`reverse(..)`... 操作 list，或者透過 `set()` 修改值都沒有動到 list（其實這些操作都是產生了新的 list 版本），

```
var list2 = list.set(6, '玉米')

list2 === list
// false，參考位置不同

list2.get(6)
// '玉米'

list.get(6)
// '餅'

console.log(list2.toArray().slice())
// ['薯條', '漢堡', 'Pizza', '冰', 'Beer', '熱狗', '玉米', 'tacco' ]
```
我們可以視 `list` 和 `list2` 為兩個不同版本的陣列，Immutable.js 其實是在原本的資料位置上加一個參考，並沒有為 list2 重新 allocate 一塊新的記憶體[^1]，請看下圖：

![Screenshot from Immutable.js Tutorial - The Advantages of Using Immutable.js](https://i.imgur.com/Ksxz3wA.jpg)
> Screenshot from [Fullstack Academy](https://youtu.be/cHKG4b2XNig)

橘色的表示原本的 List；藍色的為 List2，圖中可以看到有75％的資料節點仍繼續使用，只是換了 Parent Node，並沒有額外佔用記憶體（僅多出要修改的部分），這就是效能優化的部分。

### Persistence
在原始 List 創建後，仍舊能夠保持原來的樣子，且每一個版本都是可用的
```
var list = Immutable.List.of(...);
var list2 = list.push('香腸');
var list3 = list.shift()
var list4 = list.concat(list2, list3);
```
> That's Persistence

當然不排除可以手刻實現 **Immutability 值不變性** ，必須追蹤每一次的狀態變化讓每個更新版本可用、管理記憶體...等，開發難度會非常大。


### 關於 `Immutable.js` ，我整理一些我參考的資源：
* [Immutable.js: An Introduction with examples written for humans](http://untangled.io/immutable-js-an-introduction-with-examples-written-for-humans/)淺顯易懂附範例
* [Immutable.js 簡介](https://rhadow.github.io/2015/05/10/flux-immutable/) 中文，介紹了 List, Map, Squence
* [Immutable.js Tutorial - The Advantages of Using Immutable.js](https://youtu.be/cHKG4b2XNig) 影音教學，本文範例的參考來源
{%youtube cHKG4b2XNig %}

## 結論
這幾天我們研究了副作用、純函數的特性與好處，並且我們在減少副作用上下了很多功夫，
* [Day 23, Immutability 值不可變性]()
* [Day 22, 減少副作用的方法]())
* [Day 21, Referential Transparent 引用透明](https://ithelp.ithome.com.tw/articles/10196689)
* [Day 20, Pure Function 純函數](https://ithelp.ithome.com.tw/articles/1019656)
* [Day 19, Side effects](https://ithelp.ithome.com.tw/articles/1019648)

學習 FP 的過程中，真的裡裡外外地複習了 JS 的諸多特性，以及從不同的角度重新學習 JS，如在處理副作用時，延伸探討 Immutability 值不變性，回頭研究了基本型別，以及 ES6 `const` 宣告並非原本想像般的運作，與值不可變根本無關。

以及為了實現 Immutability，學習了陣列與物件複製與 `Object.freeze(..)`，其中的效能議題，則透過 `Immutable.js` 使用不可變的資料結構。而 Immutability 並不是指不去改變值，而是一種讓程式碼可讀性更高、更穩定的約束。

## 參考資料
[^1]: [Introduction to Immutable.js and Functional Programming Concepts](https://auth0.com/blog/intro-to-immutable-js/)
