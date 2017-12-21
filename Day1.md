# Good Morning, JS (Day 1)
## Office Hours with Kyle Simpson, Author of "You Don't Know JavaScript" Book Series

[![Yes](https://img.youtube.com/vi/YC8AQuuszOo/0.jpg)](https://www.youtube.com/watch?v=YC8AQuuszOo)

> 這部影片是 Kyle 的線上解題，來自教學網站 Frontend Masters, 該平臺有 Kyle 的許多影音教學，截至 11/30 已累積超過 300,000 小時的觀看時數

## 內容分享 (PART 2)

> 影片涵蓋許多 Kyle 對 JavaScript 深入理解後的獨特見地，以及如何學習這個語言的正確 MindSet，推薦有興趣的朋友。以下是我看完的一些整理

## Q2. Javascript 如果要發展得像 C++, Python or Java 般成熟，可以往哪個方向去發展
![](https://i.imgur.com/5fEDzhe.png)

Kyle 提到這個問題問他可能會有偏誤，因為 Kyle 是 JavaScript 的擁護者，不過他也認為，當你越喜歡一個語言，對語言越有了解，才越有資格去評價一個語言，因為你會希望這個語言更好。


* Pattern matching
有興趣讀者可以科普一下，[ECMAScript Pattern Matching Syntax](https://github.com/tc39/proposal-pattern-matching)

* types

要看情況，如果你是一個 .NET 開發者，與其直接轉移到 JavaScript 這種 type free 的語言，選用 TypeScript 是一個明智的選擇。

但是這不表示 JavaScript 要往 TypeScript 的方向發展，因為不是每個人都有從強型別語言轉 JavaScript 的過程。

* 關於 Coercion

![](https://i.imgur.com/CSg2iin.png)

很多人認為 JS type Coercion 感覺像魔術，甚至覺得這是設計缺陷，但其實是不了解其運作的方式，比如說 Abstract Equality Comparison (`==`)，有興趣讀者可以參考，[ECMAScript® 2017 Language Specification](https://www.ecma-international.org/ecma-262/8.0/index.html#sec-type-conversion)

* moduliation and bundling

Kyles 這邊談到 ES6 modules 規範與 node ecosystem 實作上的差異，以及 node 實作又與 CommonJS 規範的不同，其他的語言倒是沒有這些令人挫折的掙扎。
![](https://i.imgur.com/VlrTV1J.png)
---

## Q3. Which course from Kyle covers the most advanced JavaScript topics.
![](https://i.imgur.com/5ctSWpM.png)

現在很多線上課程，會把 `advanced` 課程定位在實際應用，涉及專案結構等軟體開發的議題，許多人都期望看完這些課程找到銀彈，實際上，也沒有一套能打遍天下的方法，這也不是 Kyle 製作線上教學的目的。

Kyle 反而喜歡用 `Depth` 這個字，在課程用的 code 都非常基礎，但都談得非常深入，這是他課程的特色。 

![](https://ci6.googleusercontent.com/proxy/rFOtOShBD3GMseSyUPZa4KwKn8Rjl1ySV8nI_BvjP5TapYvW51JopcaMyO_4gSIn6oWexvs5UYKaotX3AbbeM8GONMpHKhJM1vvYqI3sD9SDBqG8D8AAOChUwbOE3HUCDAuuMNB5Lpc6_Ti75EJxLe8oEOXHrxgh=s0-d-e1-ft#https://downloads.intercomcdn.com/i/o/40676029/eb9ecc8558fe6ee941a6d8a9/Kyle-Simpson-banner-v2.jpg)

以下 Kyle 課程來自教學網站 [Frontend Masters](https://frontendmasters.com/teachers/kyle-simpson/) 

* Functional-Light JavaScript, v2
* Deep JavaScript Foundations 
* ES6: The Right Parts 
* Rethinking Asynchronous JavaScript 
* Organizing JavaScript Functionality
* Coercion in JavaScript
* Introduction to JavaScript Programming
* Real-Time Web with Node.js
---
## Q7. How one can keep up with the fast-growing developments in Javascript
![](https://i.imgur.com/I2Qn9mT.png)

與其盲目地追求所有新出的技術，該做的是掌握技術發展的趨勢，知道一項新技術在整個生態圈的角色，舉例來說，Webpack，你可能不需要繪手刻 Webpack bundling，但你要說得出幾點 Webpack 的特色，以及所扮演的角色，例如：
* 與 Gulp 不同的地方是，webpack 是 Configuration-based 的 bundling tool.
* 高度可客製化的 loader
* 實作 AMD、CommonJS 以及 ES6 module 的規範

那什麼時候要學呢？當你要用的時候。

![](https://i.imgur.com/cJ3il6q.png)

首先，Kyle 從來不自稱自己是 JS 專家，他只是不斷地分享自己所學，得到反饋，然後持續學習進步

最後與大家分享，

![](https://i.imgur.com/tk5XGy3.png)
