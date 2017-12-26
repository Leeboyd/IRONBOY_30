Good Morning, JS (Day 6, Higher-order function)
===
> 本系列文章，內容以探討 Kyle Simpson. ***Functional-Light JavaScript*** 一書內容為主
>* 目標：是讀懂 FP，能用 code 與人交流，而不是被壓在 FP 的術語大山下喘不過氣。
>* 原文地址：[Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS)

## Functions are values

在 JS ，function 同時也是值，這表示可以賦予某變數為 function，記得前面討論的 `function expression`，宣告一個變數，將一個匿名函數賦值給變數，舉例，

```javascript=
var tripple = function (x) {
    return x * 3
}

// function 如同值可以再賦值給其他變數
var multiplyingThree = tripple

multiplyingThree(30)  // 90
```

除了可以賦值給變數，當然也可以當成參數傳入 function。

## Higher-order function
根據定義[^1]，higher-order function 是至少滿足下列一個條件的函數：

* 接受一個或多個函數作為輸入
* 輸出一個函數

把 function 當成參數傳入另一個 function，以 forEach 示範如下，

```javascript=
var dogs = ['Lucky', 'Toby', 'Black', 'White']

var print = function (name) {
    console.log(name)
}

dogs.forEach(print)
```
`forEach` 滿足條件：受一個或多個函數作為輸入，故為 higher-order function 。

把一個函數輸出的方式可以像這樣，

```javascript=
var dogs = ['Lucky', 'Toby', 'Black', 'White']

var printer = print();

dogs.forEach(printer)

function print() {
    return function inner(name){
        return console.log(name);
    };
}
```

![](https://www.visitrussia.org.uk/blog/wp-content/uploads/2016/12/nesting-doll.jpg)
是不是有點像俄羅斯套娃[^2]？

### 💡 小提示
FP 學習者們將來會遇到的 Composition ，概念就是來自 higher-order function ，請務必掌握。

## Closure

讀一下 Closure 的定義，

>Closure is when a function remembers and accesses variables from outside of its own scope, even when that function is executed in a different scope

當一個函數可以記住並存取到不同 `scope （作用域）` 的變數，甚至這個函數在不同 scope 被執行，稱之 Closure 

```javascript=
function greaterThan(n) {
  return function inner (m) { return m > n; };
}

var greaterThan10 = greaterThan(10);

console.log(greaterThan10(11)); // true
```

在 `greaterThan` scope 中， `n` 在內部被引用，當 `greaterThan` 在 line 5 執行時， 內部函數 `inner` 建立，`inner` 可以存取到 `n`，在 `return` 後仍然可以。

當 `greaterThan` 執行完畢，內部 scope 應被回收，區域變數 `n`應不存在，但 `inner` 已被不同 scope 的 `greaterThan10` 參考，只要這個參考關係存在， `n` 就會一直保留。

我們稱 `n` 被內部函數 inner `closure` 。

再看一個來自 Moz 的例子[^3]，將兩值相加，一個已知，另一個之後才會知道，我們可以透過 `closure` 記住第一個輸入，等待 y 被指定，

![](https://i.imgur.com/2jAZE6C.png)

### 💡 小提示
這個技巧，在 FP 中相當常見，有兩種形式 `partial application (偏函數應用)` 和 `currying （柯里化）`，會在日後詳細深入。

前面提到函數可作為值，所以也可以透過 `closure` 來記住，

```javascript=
function formatter(formatFn) {
    return function inner(str){
        return formatFn( str );
    };
}

var lower = formatter( function (v){
    return v.toLowerCase();
} );

var capitalize = formatter( function (v){
    return v[0].toUpperCase() + v.substr( 1 ).toLowerCase();
} );

lower( "LOL" );           // lol
capitalize( "boy" );      // Boy
```

FP 相當鼓勵開發者將常用的功能，簡單優雅的方式封裝成 function ，而不是在整個 code 中到處散落 toLowerCase()、toUpperCase()，也就是我在 Day3 提到的 `樂高積木` 

範例中，我們做了兩塊 `樂高積木` ，`lower(..)` 和`capitalize(..)`。

現在試著組合這兩塊積木 `capitalize(lower( "LOL" ))` ，即可得到 `Lol`。

## 小結：

撇開 FP 不說，`closure` 是程式中基礎中的基礎，不管是學習 JS ，請務必務必一定要掌握。



### 參考資料 
[^1]: [Higher-order function](https://en.wikipedia.org/wiki/Higher-order_function)
[^2]: [nesting-doll.jpg](https://www.visitrussia.org.uk/blog/wp-content/uploads/2016/12/nesting-doll.jpg)
[^3]: [closure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)



