# 2017第九屆 IT 邦鐵人賽

參賽題目: Good Morning, Functional JS (Day 30)
===
> 標題 Good Morning，其實是希望本系列能為 Functional Programming 學習雲霧中透進一絲微光

![Flaming Deku Stick by Tatiana Bischak](https://cdn.dribbble.com/users/844597/screenshots/3542008/ocarinaoftimeweapon_dekustick_final_1x.png)


會寫本系列文章，主要是因為想要學習 Functional Programming，也想對 JS 有更深入的了解，內容以探討 Kyle Simpson. [***Functional-Light JavaScript***](https://github.com/getify/Functional-Light-JS) 一書內容為主軸，

![](https://i.imgur.com/dIHQ3kp.png)

算是邊讀邊翻譯，也算給自己做筆記，但並不是逐字翻譯，而是以原文的概念加上我自己的理解，難免會與原文失真，下方我有留資訊歡迎討論。

以下是我參考的資訊，供有興趣讀者也參考：
### 1. Frisby [***Mostly Adequate Guide to Functional Programming***](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/README.md)

> 中譯本：[JS 函数式编程指南](https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details)

![](https://i.imgur.com/pTPyqWk.png)

以生動的舉例解說各種 FP 概念，不過內文舉例非原生 JS ，有使用函式庫。

### 2. 阮一峰 [函数式编程入门教程](http://www.ruanyifeng.com/blog/2017/02/fp-tutorial.html)
深入淺出談 Functional Programing 搭配範例。

### 3. Functional Programing 名詞大全
快速查詢專有名詞的定義與簡單範例
[Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functor)

### 4. Functional Programing 相關函式庫
* [Ramda](http://ramdajs.com/)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)


## 文章目錄以及參考資源

### 熱身前導文
帶大家看影片認識 Kyle Simpson，以及正確的學習心態

* [Day 0](https://ithelp.ithome.com.tw/articles/10192480)
* [Day 1](https://ithelp.ithome.com.tw/articles/10192779)

參考影片：
Office Hours with Kyle Simpson, Author of "You Don't Know JavaScript" Book Series
[![Yes](https://img.youtube.com/vi/YC8AQuuszOo/0.jpg)](https://www.youtube.com/watch?v=YC8AQuuszOo)

### 第一部分 Why
對應原文第一章，學習 FP 的理由，也提到正確的學習心態
* [Day 3 Why Functional Programming](https://ithelp.ithome.com.tw/articles/10193303)

其中作者也列出[參考資源](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch1.md/#resources)，點擊查看

### 第二部分 Function
對應原文第二章，說明function 在 Functional Programming 的定義，以及須滿足的條件，

* [Day 4, You don't know Functions](https://ithelp.ithome.com.tw/articles/10193556)
* [Day 5, You don't know Functions 續](https://ithelp.ithome.com.tw/articles/10193696)

### 第三部分 參數
對應原文第三章，調整參數，使函數能互相組合
#### 收集（gather） 和 展開（spread）操作
* [Day 7, 敲敲打打玩轉 function](https://ithelp.ithome.com.tw/articles/10194258)
* [Day 8, Spread and Gather](https://ithelp.ithome.com.tw/articles/10194509)

#### Partial Application
* [Day 10, Partial Application 偏函數應用](https://ithelp.ithome.com.tw/articles/10194837)

#### Currying
* [Day 2, Currying](https://ithelp.ithome.com.tw/articles/10192884)
* [Day 11, 再探 Currying 柯里化](https://ithelp.ithome.com.tw/articles/10195145)
* [Day 12, Why Currying 柯里化](https://ithelp.ithome.com.tw/articles/10195285)

#### 參數順序處理
* [Day 13, 參數順序調整](https://ithelp.ithome.com.tw/articles/10195432)

#### Point-free 無參數風格
* [Day 14, Pointfree 無點風格](https://ithelp.ithome.com.tw/articles/10195632)

參考資料：
* 阮一峰 [Pointfree 编程风格指南](http://www.ruanyifeng.com/blog/2017/03/pointfree.html)

### 第四部分 函數組合
對應原文第四章，實際使用第三部分的技巧進行函數組合，並使用函式庫 Ramda
* [Day 15, Composition part 1](https://ithelp.ithome.com.tw/articles/10195854)
* [Day 16, Composition part 2](https://ithelp.ithome.com.tw/articles/10196003)
* [Day 17, Composition - pipe](https://ithelp.ithome.com.tw/articles/10196121)
* [Day 18, Ramda 實作](https://ithelp.ithome.com.tw/articles/10196316)

### 第五部分 副作用
對應原文第五章，研究副作用、純函數的特性與好處，並且我們在減少副作用上下了很多功夫，

* [Day 19, Side effects](https://ithelp.ithome.com.tw/articles/1019648)
* [Day 20, Pure Function 純函數](https://ithelp.ithome.com.tw/articles/1019656)
* [Day 21, Referential Transparent 引用透明](https://ithelp.ithome.com.tw/articles/10196689)
* [Day 22, 減少副作用的方法](https://ithelp.ithome.com.tw/articles/10196844)

### 第六部分 Immutability
對應原文第六章，在 JS 中實現數據不可變性，有兩個方法: Array.slice()、Object.freeze()。但是這兩種方法都是 shallow 處理，又有存在性能的問題，這部分藉由 使用 Immutable.js 來實現。 

* [Day 23, Immutability 值不可變性](https://ithelp.ithome.com.tw/articles/10196920)
* [Day 24, 使用 Immutable.js](https://ithelp.ithome.com.tw/articles/10197038)

推薦教學，了解 Immutable.js：
[![Yes](https://img.youtube.com/vi/cHKG4b2XNig/0.jpg)](https://www.youtube.com/watch?v=cHKG4b2XNig)

### 第七部分 遞迴
對應原文第八章，遞迴之所以在 FP 中討論，是因為迭代方式常會需要中間值，這就是副作用，所以在流行的函數庫中都沒有迭代方法，這就是遞迴重要的原因之一

* [Day 25, Recursion 遞迴](https://ithelp.ithome.com.tw/articles/10197134)

#### 處理 stack overflow 技巧
* [Day 26, Tail Calls 尾調用](https://ithelp.ithome.com.tw/articles/10197230)
* [Day 27 Continuation-passing style](https://ithelp.ithome.com.tw/articles/10197332)
* [Day 28, Trampolines](https://ithelp.ithome.com.tw/articles/10197438)

額外參考資料：
* [Recursion in Functional JavaScript](https://www.sitepoint.com/recursion-functional-javascript/)
* [[翻译] JS的递归与TCO尾调用优化](https://segmentfault.com/a/1190000004018047#articleHeader4)


### 番外篇

#### functor 函子
連載最後也是最重要的概念 - functor
* [Day 29, functor 函子](https://ithelp.ithome.com.tw/articles/10197535)

額外參考資料：
* [What is a functor?](https://medium.com/@dtinth/what-is-a-functor-dcf510b098b6)
* [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

#### Higher-order function
說明何謂 Higher-order function，以及特性

* [Day 6, Higher-order function](https://ithelp.ithome.com.tw/articles/10194000)

推薦教學，深入了解如何應用：
[![Yes](https://img.youtube.com/vi/9YkUCxvaLEk/0.jpg)](https://www.youtube.com/watch?v=9YkUCxvaLEk)


#### ES6 Class
因為接觸 Angular ，想了解一下 Class，於是有了這篇

* [Day 9, ES6 Class 地雷](https://ithelp.ithome.com.tw/articles/10194639)

## Not the end, to be continued 
連載到了今天，鐵人完賽，每天花了相當多時間翻資料、整理心得，幾度想放棄，好在跟 [好想工作室](https://ithelp.ithome.com.tw/ironman/signup/team/17) 一起團報連載、一起討論，總算是完成挑戰了！

![](https://i.imgur.com/Hjs72uL.png)
> 12/20 開始連續 30 天的 commit 


但學習不會因此停止，如果有引起你的興趣也請點頁面上方連結，閱讀原作。So far，我們的 FP 工具箱裡的工具還不全，明天 31 日會整理目錄以及一些資源，在未來，如果有新的研究心得，我仍會寫成文章分享。

有任何問題、技術交流、以下是連絡資訊



> 關於我
> ## 李柏毅 *LeeBoy*
> 前端工程師
> * email：jobboy19890101@gmail.com
> * [Github](https://github.com/Leeboyd)
