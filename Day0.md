Good Morning, JS (Day 0)
===


Kyle Simpson 是 You don't know JS 系列的作者，本身也是開源社群 JS 的熱情傳教士，本系列文章以 Kyle 的教學材料，順著 Kyle 的思路，重新認識，深入了解 JavaScript。

-----

## Office Hours with Kyle Simpson, Author of "You Don't Know JavaScript" Book Series
[![Yes](https://img.youtube.com/vi/YC8AQuuszOo/0.jpg)](https://www.youtube.com/watch?v=YC8AQuuszOo)

> 這部影片是 Kyle 的線上解題，來自教學網站 Frontend Masters, 該平臺有 Kyle 的許多影音教學，截至 11/30 已累積超過 300,000 小時的觀看時數


## 內容分享 (PART 1)

> 影片涵蓋許多 Kyle 對 JavaScript 深入理解後的獨特見地，以及如何學習這個語言的正確 MindSet，推薦有興趣的朋友。以下是我看完的一些整理

### Q1. 字串操作： `concat()` method vs. the `+` operator 

![](https://i.imgur.com/EsX3J4q.png)

* **String are immutable**，
    字串操作實際上不會改變該字串，就算叫 `.concat()`，其實底層只是把原值當 input，而輸出新值

* In general, function call tend to take up more processing time (microseconds)
    但實際上有時候會跟我們的直覺相反，跟 JS 的 runtime 環境實作有關，常會發生: 如果一個 pattern 如果使用者較多，通常會特別對這些 pattern 進行優化


* 正確 MindSet
很多人看到這邊就直接下結論 `好吧！把全部的字串連接都改成` + `吧`， Kyle 指出以下幾點提醒
    * line 4 與 line 6 在不同的執行環境效能可能不同
    * 效能測試比較單行，Not really a good idea.
    * 應該比較的是 module 和 module 之間，比如說**同一套登入流程**
    Ａ寫法 v.s. B寫法

* more base on syntax, more devlarative
```javascript=
var s1 = "Good Morning"
var s2 = "JavaScript"

var s = `${s1} ${s2}`
```

* trade-off 效能 v.s 易讀性
隨著專案進行，code 一定不只這幾行，除了效能更要考慮易讀性，以及兩者之間的取捨
![](https://i.imgur.com/sUpldEq.png)


