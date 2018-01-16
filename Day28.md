Good Morning, Functional JS (Day 28, Trampolines)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)


這兩天的討論
* [Day 27, CPS](
https://ithelp.ithome.com.tw/articles/10197332)
* [Day 26, Tail Calls 尾調用](https://ithelp.ithome.com.tw/articles/10197230)

在處理的都是在避免遞迴造成的 stack overflow，這些 **PTC、CPS** 的寫法，是把當前遞迴的結果透過傳值或閉包給下一個遞迴，這種優化叫做 **TCO**，為什麼會稱作優化呢？因為我當前的遞迴執行結束，不需要再回來，所以可以將 stack frame 移走，避免堆疊更多 stack frame。

在引擎支援 PTC 的情況下，執行時會做 TCO 將遞迴轉換成迭代形式而不會產生 stack overflow 錯誤。

那如果引擎不支援 TCO 呢？請注意這裡說的不支援不是說不能寫，我們還是可以寫 PTC 或 CPS 但引擎不會幫忙做最佳化，有另外一種技巧叫做 **Trampolines**，適合在沒有 TCO 的環境使用。

## Trampolines
![image alt](https://cdn.dribbble.com/users/597165/screenshots/1835197/me800x600.gif)
> **"trampoline"** from [Sara Farnsworth ](https://dribbble.com/sarafarnsworth)


trampoline 在我的認知就是自己把遞迴的形式轉換成迭代，它不再是 function call function，所以不會堆高 stack（遞迴），而是 call function 得到 return 值，不斷循環執行每個 return 的函數，直到 return 值不是函數就直接 return，

之所以會命名 trampoline (跳跳床)，個人猜測是因為這個函數的運作方式： call -> return -> call -> return -> call ... ，然後以 Stack 角度來看就是，push -> pop -> push -> pop ...，故稱之。

trampoline 是一個函數，具體是這樣工作的：
* 接受一個函數 fn 當參數
* call fn
* 如果返回值是函數，就再 call 得到返回值
* 如果返回值不是函數，就直接返回


來看一下 trampoline(..)[^1]：

```
function trampoline(fn) {
    return function trampolined(...args){
        var result = fn( ...args )
        
        // 檢查返回結果的類型，當返回一個函數時，loop 繼續
        while (typeof result === "function") {
            result = result()
        }
        
        return result
    }
}
```
> FP工具 trampoline

我們以交互遞迴的例子說明用法，例子摘自 
*Functional JavaScript* [^2]：

```
function toSteven(n) {
  if (n === 0)
    return 'boom!'
  else
    return toJob(Math.abs(n) - 1);
}

function toJob(n) {
  if (n === 0)
    return undefined
  else
    return toSteven(Math.abs(n) - 1);
}

console.log(toSteven(2))
// "boom!"
console.log(toJob(3))
// "boom!"
```
兩個函數互相 call，倒數某個絕對值，直到一方達到零則停止，偶數傳給 toSteven 或奇數傳給 toJob 會 `boom!`。

縱使我們寫好了 Tail Call 形式，但引擎沒支援 TCO，只要 `toSteven(100000);` 就會：
![](https://i.imgur.com/16lSFLm.png)
> stack overflow!

## 步驟1：重構
為了能夠使用 trampoline 我們需要重構，像 CPS 一樣，我們為每個 continuation function (也就是接著要做的函數) 創建**閉包**，

```
function toSteven(n) {
  if (n === 0)
    return 'boom!'
  else
    return () => toJob(Math.abs(n) - 1);
}

function toJob(n) {
  if (n === 0)
    return undefined
  else
    return () => toSteven(Math.abs(n) - 1);
}
```
> 重構以使用 trampoline


### 確認一下重構後，兩者在基本條件都能運作
```
console.log(toSteven(0))
// "boom!"

console.log(toJob(0))
// undefined
```

### 確認一下能夠手動遞迴
```
console.log(toSteven(2))
// () => toJob(Math.abs(n) - 1)


console.log(toSteven(2)())
// () => toSteven(Math.abs(n) - 1)


console.log(toSteven(2)()())
// "boom!"
```
> 這種叫做展開（閉包）

看起來都行！那現在就來實驗 trampoline 吧！

## 步驟2：使用 trampoline
```
var trampolined_toSteven = trampoline(toSteven)

var trampolined_toJob = trampoline(toJob)
```

見證奇蹟的時刻到了！

```
console.log(trampolined_toSteven(200000))
// "boom!"

console.log(trampolined_toJob(3333333))
// "boom!"
```
> stack never blow~

![image alt](https://cdn.dribbble.com/users/398023/screenshots/3401511/cake-for-dribble_1x.jpg)
> **"Jump Around!"** from [Dawn Moseke](https://dribbble.com/dawnmoseke)

**為什麼不會爆？**
剛剛提到我們同 CPS 傳一個閉包給下一個函數，但不同點是，每個返回的函數，我們都立即執行完畢，所以引擎不會累積越來越多的閉包。

**完整實作範例請參考** [^JSBIN](https://jsbin.com/wuhuzeboxu/edit?js,console)

## 結論：
Trampoline 相對於 CPS 風格，可讀性較高，缺點是要特別重構將遞迴閉包供給 trampoline 工具函數使用，但我們不必特地修改函數簽名，在不支援 TCO 的環境下，它不失為另一個選擇方案，讓我們在 imperative iterate (命令式的迭代) 和 declarative recursive (聲明式的遞迴) 之間取得平衡。

我們這幾天花很多力氣處理遞迴 stack 爆掉的問題，我們使用了 **Tail call 尾調用**，用在遞迴就叫做尾遞迴，如果引擎有支援辨識 Tail call，就會進行 **Tail Call Optimization, TCO 尾調用優化**。

而寫出尾調用的規範叫做 **Proper Tail Call, PTC**，把遞迴放在函數結束點，其中，return 表達式，這樣引擎就會做 TCO，使得遞迴不論多深，都只會佔一個 call stack frame，不會爆掉。

雖然這幾天的討論的東西，實際用途並不是很顯著，範例有點抽象，但過程的推理都滿有意思的，藉由這次鐵人把過去看過的東西整理整理也是不錯。

## 參考資料
[^1]:[Trampolines](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch8.md#trampolines)
[^2]:Fogus, Michael. *Functional JavaScript*. O'Reilly Media, 2013.

[^3]: [[翻译] JS的递归与TCO尾调用优化](https://segmentfault.com/a/1190000004018047)

[^4]: [JavaScript 中的递归优化](http://yoyoyohamapi.me/2016/06/28/JavaScript-%E4%B8%AD%E7%9A%84%E9%80%92%E5%BD%92%E4%BC%98%E5%8C%96/)
