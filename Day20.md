Good Morning, Functional JS (Day 20, Pure Function 純函數)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## Pure Function
在 [Day 19](https://ithelp.ithome.com.tw/articles/10196484) 有稍稍提到數學定義的純函數，在程式裡，若一個函數符合下列需求，即可被認定為純函數：[^1]

* 沒有任合可觀察的 side causes/effects
* 相同的輸入，永遠會得到相同的輸出
* 輸入輸出資料串流全是顯式（Explicit）的

看個例子[^2]

```
// impure
var minimum = 21;

var checkAge = function(age) {
  return age >= minimum;
};


// pure
var checkAge = function(age) {
  var minimum = 21;
  return age >= minimum;
};
```
> 範例1 不純 v.s. 純

在 **impure** 的寫法，可以看到 `checkAge` 的結果取決於自由變數 `minimum`，也就是 side cause，換句話說，除了參數 `age` 之外，還有 `minimum` 這個隱式（Implicit）輸入，違反我們對 **pure** 的定義。

在 **pure** 的寫法，所有的參數都是顯式（Explicit）的，沒有引用自由變數，每一次相同的輸入，都會得到相同的輸出，所以我們可以稱之為 **Pure Function**。

不過，並非使用自由變數就是不純，只要這個自由變數不是 side cause 就好，例如：

* 讓 `minimum` 為常數，不可以重新賦值
* `Object.freeze`:
```
var immutable = Object.freeze({
    minimum: 21
})

// immutable.minimum
```
**impure** 為什麼不受歡呢？因為結果是不可預測的，需要花力氣去追蹤每一次 function call 後的變化。

## 追求純粹
在 JS 討論一個函數是否純粹時要特別注意，因為 JS 的動態型別非常容易產生意想不到的副作用。

看個例子[^3]

```
function rememberNumbers(nums) {
    return function wrapper(fn){
        return fn( nums );
    };
}

var list = [1,2,3,4,5];

var simpleList = rememberNumbers( list );
```
`rememberNumbers` 看起來似乎是一個純函數，內部用函數 wrapper 閉包了一個陣列 nums。

真的是這樣嗎？做幾個檢驗試試：

1. 從參數來看，我們可能基於同樣 list 傳入，判斷會得到同樣結果，現在我們不改變 list 與 nums 的參考關係，試著這樣做：

```
function getLength(nums) {
    return nums.length;
}

simpleList( getLength ) // 5

list.push(6)

simpleList( getLength ) // 6
```
> 範例2 JS dynamic value

到底是純還是不純呢？看是以什麼角度看，在沒有 `list.push(6)` 的情況是純函數，透過一些修改可以改善這種參考關係造成的不純：

```
function rememberNumbers(nums) {
    // 陣列複製
    nums = nums.slice()
    
    return function wrapper(fn){
        return fn( nums );
    };
}
```
> 解法1 透過陣列複製避免副作用

```
function rememberNumbers(...nums) {
    return function wrapper(fn){
        return fn( nums );
    };
}

var simpleList = rememberNumbers(...list)
```
> 解法2 spread 和 rest 操作合用

這兩種 `...` 操作是將 list 複製到 nums 中，而不是用參考了。

2. **impure** ＋ **pure**
看起我們處理完 list 與 nums 的參考關係造成的副作用了，現在把一個 **impure** 函數傳入 fn 會是怎樣呢？

```
function getLastValue (nums) {
    return nums.reverse()[0];
}

simpleList( lastValue ) // 5

// 檢查一下，是否影響到 list
console.log(list)       // [1, 2, 3, 4, 5] Good!

simpleList( lastValue ) // 1 No! 不一樣結果
```
> 範例3 **impure** x **pure** [^JSBIN](https://jsbin.com/kovuqerisi/edit?js,console)

`reverse()` 會 return 陣列反序，實際修改到陣列，我們檢查 list ，ＯＫ！還是 `[1, 2, 3, 4, 5]` 沒受影響，喔！那是誰被改到了呢？原來是被閉包的 `nums`！

我們需要防止 `fn(...)` 改變它的閉包變數：

```
function rememberNumbers(...nums) {
    return function wrapper(fn){
        // 再 return 一個複製陣列
        return fn( nums.slice() );
    };
}
```
> 解法1 透過陣列複製避免被更動

看起來很穩了，現在可說 `rememberNumbers(..)` 是純函數了嗎？很抱歉還是不行。

3. 傳入有副作用的 fn
我們前面都提出控制副作用都是針對參考關係的，只要傳入有副作用的函數，都會污染 `rememberNumbers(..)` 的 pure

```
function output (nums) {
    console.log(nums.length);
}

simpleList(output)
```
> 範例4 無法控制的 fn

## 小結
**結論：** 我們無法定義出完美純粹的 `rememberNumbers(..)`，我們的目標不是寫百分百的純，我們花力氣提高純度，這樣對我們的程式信心就越高，進而使得可讀性更高。

## 參考資料
[^1]: 維基百科：[純函數](https://zh.wikipedia.org/wiki/%E7%BA%AF%E5%87%BD%E6%95%B0)

[^2]: mostly-adequate-guide: [Oh to be pure again](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch3.html#oh-to-be-pure-again)

[^3]: [ch5 Purely Relative](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/ch5.md#purely-relative)
