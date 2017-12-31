Good Morning, Functional JS (Day 11, 再探 Currying 柯里化)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## One at a Time (一次傳一個)
還記得昨天的 **Partial Application** 嗎？今天如果將函數改為一次傳一個參數，然後再次 return 另一個函數來處理下一個參數；

講的學術一點，將一個多參數函數拆解成一串連續的鏈式函數，每個小函數接受單一參數 `Arity = 1` ，並 return 另一個函數接受下一個參數。

> 這種技巧，就叫 Currying (柯里化)

回想一下，我們的 `ajax(url, data, cb)`，如果重構成 Curried  版本是這樣的：

```
curriedAjax(url)(data)(cb)
```

三個連續的 `(..)` 表示鏈式呼叫，也可以分拆成這樣：

```javascript=
var chain1 = curriedAjax(url)

var chain2 = chain1(data)

chain2(cb)
```

> 範例1

我們現在學到了，將一次接受所有參數的 `ajax(..)`，改成

* **Partial Application** 版本：部分實參先傳，部分實參後傳
* **Currying** 版本：一次只接受一個實參

**相同處：**
每一次的 Curry，都將一個實參應用到原函數的形參，一直到傳入所有實參為止

**相異處：**
curriedAjax return 出的 chain1，是一個明確只接受下一個參數 **data** ，並非是接收所有剩下參數的函數。


回想 [Day2](https://ithelp.ithome.com.tw/articles/10192884)，

```javascript=
// before currying

function sum (x, y, z) {
  return x + y + z
}

sum(2, 3, 1) // 5

// after currying 
var curried_sum = curry(sum)

curried_sum(2)(3)(1) // 6

```
> Day2 範例1 currying 前後的 function call 差異

由此可以知道，**Currying** 就是把多參數（Arity > 1）函數轉化成一系列單參數（Arity = 1）的函數。

## 實作 Currying

[Day2 **Currying** 簡介](https://ithelp.ithome.com.tw/articles/10192884) 中的範例

```javascript=
function curry (fn, ARITY = fn.length) {
  return (function nextCurried (prevArgs) {
    return function curried (nextArg) {

      var args = [...prevArgs, nextArg]

      if (args.length >= ARITY) {
        return fn(...args)
      } else {
        return nextCurried(args)
      }

    }
  })( [] )
}
```
> Day2 範例3 實作 curry function（二）

來改寫成 ES6 版本，

```javascript=
var curry = (fn, ARITY = fn.length, nextCurried) =>
    (nextCurried = prevArgs =>
        nextArg => {
            var args = [...prevArgs, nextArg]

            if (args.length >= ARITY) {
                return fn(...args)
            } else {
                return nextCurried(args)
            }    
        }
    )( [] )
```
> 範例4 curry 小工具 ES6 版本

* 參數說明： fn: 原函數; ARITY: 原函數形參個數

1. 初始 `[]` 做為 **prevArgs**，收集已傳入的參數
2. 接每一次傳入的實際參數 **nextArg** 與 **prevArgs** 合併成 **args**
3. 當 **args** 長度小於 ARITY 時，return nextCurried(args)，**args**，也就是下一輪的 **prevArgs**
4. 當收集到足夠的實參，就利用這些實參，呼叫原函數 **fn**。

* 使用限制：預設 `length` 能明確表示函數形參個數，得知 **Currying** 需迭代的次數。
* 如果 length 不明確的函數：包含預設參數、destructing、或不定長度參數 `...args`，便要指定明確的參數個數傳入。

```javascript=
function sum(...args) {
    var sum = 0;
    for (let i = 0; i < args.length; i++) {
        sum += args[i];
    }
    return sum;
}

sum( 1, 2, 3, 4, 5 );                        

// 運用 Currying
// 因為是不定長度參數 ...args，需指定個數
var curriedSum = curry( sum, 5 );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );  // 15
```
> 範例5 不定參數 **Currying**

## 重構時間

```javascript=
// 怕你忘記
function add (x, y) {
    return x + y
}

[30, 55, 42, 87, 66].map( partial( add, 10 ))
```
> 範例6 Day 10 全班加 10 分例子

由於 **Partial Application** 與 **Currying** 概念上相似，改成 **Currying** 方式來重構吧！

```
[30, 55, 42, 87, 66].map( curry( add )( 10 ) )
```

幾乎長得一樣呀？這樣說好了，假如一開始不知道要全班加幾分，可能題目有錯才會加，我先把 `add` 轉化成加 X 分的函數 `addX`

```
var addX = curry( add );

// 過一陣子...題目錯啦，加 3 分
[30, 55, 42, 87, 66].map( addX( 3 ) );

```
> 範例7
> 別只用看的，實際到 [JSBIN](https://jsbin.com/womefizali/1/edit?js,console) 測試吧

## 小結

在 JavaScript 裡，這兩個 FP 技巧都是運用 Closure 的概念實作的，保存部分參數，全部拿到後再執行，比較如下：

* **Partial Application**：先指定部分實參，產出一個接收所有剩下實參的函數
* **Currying**：每傳一次參數進入，就會得到一個特定性更強的函數 (SPECIALIZED function，特殊化函數)

如果想要每次只 **partially apply** 一個也是可以的，不過讓 curry 一次幫你完成是更好的選擇。

## 參考資料
* [函数式编程入门教程](http://www.ruanyifeng.com/blog/2017/02/fp-tutorial.html)
* [第 4 章: 柯里化（curry）](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch4.html#%E4%B8%8D%E4%BB%85%E4%BB%85%E6%98%AF%E5%8F%8C%E5%85%B3%E8%AF%AD%E5%92%96%E5%96%B1)
