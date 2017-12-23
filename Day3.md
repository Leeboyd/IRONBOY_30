Good Morning, JS (Day 3, Why Functional Programming?)
===

> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## Why I Learn?
先說說為什麼會探討 Functional Programming (FP) ，無非是這幾年相關討論、關注度來到空前的高峰，讓我感到 FP 的重要性已達不容忽視的程度，就趁著這次鐵人賽將學習歷程記錄下來，將相關概念掌握與熟悉。

## Before and After
直接來一段 code 比較 FP v.s. non-FP [^1](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch1.md/#at-a-glance) :

![](https://i.imgur.com/gXxmerW.png)

接著，來一段 FP 來實做同樣邏輯

![](https://i.imgur.com/cdtn4yo.png)

試著以 Functional Programmer 角度來思考這段程式吧：

![](https://i.imgur.com/jmvAIbm.png)

乍看第二段，初學者（含我本人）可能感到困惑或沮喪，並且認定第一段是 舒適/可讀性高/維護性高，且讓我們暫時離開舒適圈，跟隨著 Kyle 的腳步，相信屆時看到第二段，反而會認為這種寫法更自然，甚至會偏好第二段寫法 (反正我本人是信了) 

## 可讀性曲線
![](https://github.com/getify/Functional-Light-JS/raw/master/manuscript/images/fig17.png)

在學習 FP 的過程可能如圖中曲線，讓我們一起堅持下去，爬過陡峭的學習曲線吧。

## Imperative v.s. Declarative

`Imperative` (命令式)，著重在 **How** (如何做)，如第一段 code，透過：
* `if` 敘述、
* `for` 迴圈、
* 暫存變數、
* 重賦值 (reassignments) ，code 中累加的部分
一步步指導電腦 `如何` 完成一件事。

`Declarative` (宣告式，聲明式)，著重在 **What** (描述結果的樣子)，如第二段 code，透過 FP 技巧：
* filter
* reduce
* compose
* pipe
更專注於描述最終的結果。

來試著描述，
1. numbers 陣列進入 `printSum(..)` 
2. 經過 `sumOfChooseNumbers(..)` 處理，得到篩選後的總和
3. `formatMsg(..)` 格式成字串
4. `console.log(..)` 輸出字串

看到這邊，可能讀者完全沒有被說服，還是會認為第一段的 `Imperative` 寫法，很大的原因在於我們對一段相對熟悉，使得認為第一段可讀性較高[^2](dl.acm.org/citation.cfm?id=1850615)。

## 溝通

遙想過去，在我尚未接觸程式時，看到第一段，也認為跟天書一樣，也是經過一番段努力，才能組織與編寫出這套與電腦溝通的方式；再回到現在，可以想像成 FP 將這套 **與電腦溝通** 的方式，**轉換**，成更高層次的 **與人溝通** 。

透過熟悉 FP 的技巧，寫程式的重點將會在如何運用這些工具製作 `樂高積木` (程式片段)，並且透過編排這些 `樂高積木` ，敘述我們要的結果，

舉個例子，一但學會了 `map(..)` 這塊積木，在任何程式見到 `map` ，你可以不需要進去一行一行追蹤，就有把握推測出大致結果（將會輸出經過處理的陣列），相對來說 `for` ，雖然語法上我們較為熟悉，但要推測出一段 `for` 的結果，就非得一行一行去追蹤不可。

## 銀彈？
假如你想說採用 FP 重構之後， code 將立刻變得更簡潔、更優雅和更智慧的話——這...有點不切實際，真的要說的話，從 `Imperative` 到 `Declarative` 並不是一體兩面，而是一個緩慢漸進的過程，可能需要多次嘗試並重構，才能寫出符合 FP 原則的程式。

作者建議，採用 `teach it later` 的方式，就是说，寫完一段 code 之後，過幾個小時或一天再回來看會有不一樣的感覺。並且試著假裝自己在教學一樣解釋這一段 code，重構完成前的 code 通常很混亂不堪，所以這個過程需要反覆進行。

## 接受它，擁抱它
無可避免地，學習 FP 將會遭遇相當多的專有名詞，為避免將初學者（含我本人）淹沒在這些行話，我將從基本的原則去探索 FP，這不是說我們將迴避掉這些名詞，畢竟專有名詞可讓簡化交流的成本，但與其將這些專有名詞琅琅上口，不如去探討這些名詞背後的意義，畢竟最終我們要掌握這些名詞，來組合出我們要的 `樂高積木`。

## 取得平衡
學習 FP 的過程可能會看大量且有趣的開發模式，但不一定你的專案一定要採用這些模式。

在掌握 FP 原則之後，寫程式的基本原則一樣重要——現在編寫的每一行 code 之後都會有人來維護，這個可能是團隊夥伴，包括未來的自己，都沒有人會樂見炫技式的寫法。

![](https://i.imgur.com/zwhgTLc.png)

## 學習資源
作者 Kyle 撰寫時參考的資源，我直接把連結列出，有興趣研究者讀者務必參考，[參考資源](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch1.md/#resources)[^3](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch1.md/#resources)

### Libraries
本人特別把目前流行的 FP Libraries 列出，供之後學習完畢，撰寫真正的 code 時的參考，
* [Ramda](http://ramdajs.com/)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

### 參考資料
[^1]: [At a Glance](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch1.md/#at-a-glance)
[^2]: Buse, Raymond P. L., and Westley R. Weimer. “Learning a Metric for Code Readability.” IEEE Transactions on Software Engineering, IEEE Press, July 2010, dl.acm.org/citation.cfm?id=1850615.
[^3]: [Resources](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch1.md/#resources)
