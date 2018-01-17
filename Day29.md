Good Morning, Functional JS (Day 29, functor 函子)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* **提醒**：本文中各種的 FP 小工具，僅為邏輯演示，實際上並不適合在 production 中使用，建議使用 FP library。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

今天要討論的是 functor，終究還是要接觸術語了，在進入主題之前，先複習一下 `map(..)` 的概念

## map 映射
映射的作用就是將一個值轉換成另一個值，`map(..)` 就是對陣列中所有的值轉換新陣列的值，如下圖：

![](https://github.com/getify/Functional-Light-JS/blob/master/manuscript/images/fig9.png?raw=true)
> Figure from [Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

先看一個簡單的例子
```
var portugueseFood = [
    'galinha',
    'vaca',
    'milho'
]

portugueseFood.map(capitalize)
// ['Galinha', 'Vaca', 'Milho']

```
來看看 map(..) 如何運作：
1. map(..) 是一個函數，接受一個參數 (mapper function)
2. 傳入 capitalize 為 mapper function 
3. map(..) 將每一個元素一一傳入 mapper function 
4. 最後將被 mapper function 轉換後元素，放到新的陣列，並返回

![](https://i.imgur.com/QORqH76.png)
> Picture from Mattias Petter Johansson

看一個有趣的例子，將一組函數放在陣列裡，然後每個都組合 double，最後再執行：
```
var increment = v => ++v;
var decrement = v => --v;
var square = v => v * v;

var double = v => v * 2;

var result = [increment,decrement,square]
.map( fn => R.compose( fn, double ) )
.map( fn => fn( 3 ) );

console.log(result)
// [7, 5, 36]
```

可以這樣操作的原因是 map(..) 會返回陣列，代表可以在 map(..) 之後繼續串鏈執行下一個列表操作。

### map(..) 還是 forEach(..)
各位一定有聽過一句話：*Using the right tool in the right way*，map(..) 是用來映射值的，不是來產生副作用的，畢竟 mapper function 都是純函數（正常的情況），如果要傳入帶有副作用的函數，建議還是使用 `forEach(..)` 或者乾脆寫 loop 避免造成困惑。

## functor
> A functor is something that can be mapped over.

上面這一句話就是我找到 functor 最簡單的定義，來自 [Haskell](https://hackage.haskell.org/package/base-4.8.1.0/docs/Data-Functor.html) 和 [Fantasy Land specification](https://github.com/fantasyland/fantasy-land)。

那 something 是什麼？就是一組值放在某個容器（集合）裡，容器就是指這些值怎麼擺放，比如說陣列（依序列）或者物件（用 key 來取值...等）。

那什麼叫 that can be mapped over？也就是上面 map(..) 做的事，把每個值經過 mapper(..) 得到新值，最後再把新值依照放進同樣結構的容器後 return 。

寫 JS 的開發者，八成使用過 functor ，只是不知道術語（我也是現在才知道），在 JS 中，最常見的 functor 就是 Array，為什麼這樣說呢？因為你可以對陣列 (something)做 map 操作 (that can be mapped over)。


### 學術定義
第一眼見到 functor ，可能認為大概是 function 的一種，實際上兩者沒有任何關連。

> function 與 functor，儘管都是用來表示映射關係，但兩者目標不同。

functor 是一種範疇，包含了轉換成另一個範疇的映射關係（由外部傳入方法）。

**範疇（category）** 可以看作是同一類型的集合，以 [阮一峰 - 函数式编程入门教程](http://www.ruanyifeng.com/blog/2017/02/fp-tutorial.html) 中的圖片為例：

![](http://www.ruanyifeng.com/blogimg/asset/2017/bg2017022203.png)

> 左邊表示人名的範疇，透過傳入一個 f 的映射關係，可轉換成右邊的早餐的範疇。

整理定義：

* 一個容器，或一個集合
* 包含了值的映射關係（map）
* 轉換後也是另一個 functor

### 實例
任何具有 `map` 方法（映射關係）的資料結構，都可以視為 functor：

```
class Wrapper {
  constructor (value) {
    this.value = value
  }
  map (f) {
    return new Wrapper(f(this.value))
  }
}
```
咦？你說只有一個值也叫做集合嗎？

沒錯！class Wrapper 確實是一個 functor，它是一個只能放一個值的容器，其中含有 map 方法接受一個函數 `f` 當參數，然後 return 一個新的 functor，而裡面的值是被映射後的值。

試試看：

```
let something = new Wrapper(2)
console.log(something)
// => { value: 2 }

something.map(function (value) {
  return value + 3;
})

// => Wrapper(5)
```

來個圖解：
functor 本身有對外的 API，比如說 map(..)，透過傳入 mapper function，使內部值轉換
![](http://adit.io/imgs/functors/fmap_apply.png)
> Picture by [Aditya Bhargava](http://adit.io/index.html) from [*“Functors, Applicatives, And Monads In Pictures”*](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)

## Why functor ?
把值裝進一個容器，而且只能使用對外的 API 處理值，這樣做有什麼好處？換個方式問：讓容器自己去運用函數的好處是什麼？答案就是抽象，也更 declarative，我們換個角度看，當我們傳入一個函數給 map() 時，我們請求容器幫我們對值執行這個函數，這不就是 FP 的概念嗎？

還記得我們一開始學習 FP 說的比喻嗎？將函數當成 `樂高積木`，今天討論的 functor 就是可以與多種函數組合，我們到目前為止學習的各種重構，也是為了讓各種函數能夠組合。

