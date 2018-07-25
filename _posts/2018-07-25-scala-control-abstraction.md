---
layout: post
title: Scala Control Abstraction
excerpt: Scala Control Abstraction
keywords: scala, functional programming, control structure, repeat until, custom, by-name
---
> পড়তে সময় লাগবে ১০ মিনিট

`Scala` তে খুব বেশি `built-in control structure` নেই, কারণ আমরা খুব সহজেই নিজস্ব `control structure` তৈরি 
করতে পারি। এখানে আমরা তেমন একটি `control structure`, `repeat-until` লুপ তৈরি করব। 

আমরা আমাদের `repeat-until` তৈরি করব ধাপে ধাপে, এবং সেইসাথে `scala` এর কিছু `feature` এর সাথেও পরিচিত হব। 
আমাদের `implementation` শেষ হবার পর আমরা নিচের মত কোড লিখতে পারব। 

```scala
var i = 0
repeat {
  println("Hello, world!")
  i = i + 1
} until (i < 10)
```

উপরের কোড দেখে আপনাদের কি মনে হচ্ছে না যে `repeat-until` `scala` তে `built-in`? আমার কিন্তু প্রথমে তাই মনে হয়েছিল। চলুন দেখি কিভাবে 
আমরা এই `control structure` টি `implement` করতে পারি।  

###  প্রথম ধাপঃ repeatN
প্রথম ধাপে আমরা `implement` করব `repeatN` ফাংশন। চলুন দেখি `repeatN` এর টাইপ সিগনেচার।

```scala
def repeatN(f: () => Unit, n: Int): Unit = ??? 
```

টাইপ সিগনেচার দেখে আমরা বুঝতে পারছি যে `repeatN` হল এমন একটি ফাংশন যেটা ২টি `parameter` নেয়, একটা
কোড ব্লক `f` (`parameter` ও `return value` ছাড়া ফাংশন) ও একটা `integer n`, এবং `repeatN` ঐ কোড ব্লকটিকে `n` সংখ্যক বার `execute` করে। `Implement` করার পরে 
আমরা `repeatN` কে নিচের মত করে ব্যবহার করতে পারব।  

```scala
repeatN(() => {
    println("Hello, world!")
  }, 3)
```

উপরের code টি তিন বার `Hello, world!` প্রিন্ট করবে। চলুন দেখি `repeatN` এর `implementation`:

```scala
def repeatN(f: () => Unit, n: Int): Unit = {
  if (n > 0) {
    f()
    repeatN(f, n - 1)
  }
}
```

এখানে `repeatN` একটি [higher order function](https://docs.scala-lang.org/tour/higher-order-functions.html), 
যার প্রথম `parameter` একটি `function` (যা কোন `parameter` নেয় না এবং কোন কিছু
`return` ও করে না) ।  যখন আমরা `f`কে  `call` করি, শুধুমাত্র `f` এর `body execute` হয়। 


### দ্বিতীয় ধাপঃ better repeatN
`repeatN` এর একটা জিনিস আমার পছন্দ হয়নি, তা হলঃ

```scala
() => {
  println("Hello, world!")
}
```

ভাল হত যদি আমরা প্রথমের `() =>` অংশটুকু বাদ দিতে পারতাম (এটা দিয়ে বুঝানো হচ্ছে যে এই কোড ব্লকটি কোন `parameter` নেয় না), 
এবং নিচের মত করে ব্যবহার করতে পারতাম।

```scala
repeatN({
  println("Hello, world!")
}, 3)
```

`Scala` তে [by-name parameters](https://docs.scala-lang.org/tour/by-name-parameters.html) ব্যবহার করে আমরা সেটা করতে পারি। 
`by-name parameter` তৈরি করার জন্য আমরা `parameter` টাইপকে `() =>` এর পরিবর্তে শুধু `=>` দিয়ে পরিবর্তন করব। নিচের মতঃ

```scala
def repeatN(f: => Unit, n: Int): Unit = ???
```

চলুন দেখি নতুন `repeatN` কিভাবে `implement` করা যায়।

```scala 
def repeatN(f: => Unit, n: Int): Unit = {
  if (n > 0) {
    f
    repeatN(f, n - 1)
  }
}
```


### তৃতীয় ধাপঃ কন্ডিশন সহ repeat  
এই ধাপে আমরা `implement` করব `repeatC` - যেটাকে আমরা নিচের মত করে ব্যবহার করতে পারি।  

```scala
var t = 0
repeatC {
  println("Hello, repeat with condition")
  t = t + 1
} (t < 5)
```

`repeatC` এবং `repeat-until` এর মধ্যে একমাত্র পার্থক্য হল, `repeatC` তে `until` `keyword` টা নেই (যার কারণে এর implementation 
কিছুটা সহজ)। চলুন দেখে নেওয়া যাক `repeatC` এর `implementation`। 

```scala
def repeatC(block: => Unit)(condition: => Boolean): Unit = {
  block
  if (condition) repeatC(block)(condition)
}
```

এখানে দুটি বিষয় লক্ষণীয়। 
* `repeatC` একটি [recursive function](https://alvinalexander.com/scala/scala-recursion-examples-recursive-programming)
* `parameter` গুলি দুটি group এ নেয়া হয়েছে, যাতে আমরা `,` ব্যবহার না করে `()` দিয়ে `parameter` গুলোকে আলাদা করতে পারি।  

### চতুর্থ ধাপঃ until সহ প্রথম প্রচেষ্টা 

`until keyword` ছাড়া `repeat` আমার কাছে `fluent` শোনায় না, তাই আমরা এবার `until keyword implement` করার চেষ্টা করব।  

```scala
def until(f: => Boolean): Boolean = f

var x = 0
repeatC {
  println("Hello, repeat until (almost)")
  x = x + 1
} (until (x < 4))
```
এখানে `until` তেমন কিছুই করছে না, যে `condition` `parameter` হিসাবে পাচ্ছে সেটাকেই `execute` করছে। 
এবং আমরা এখানে আমাদের আগের `repeatC` ফাংশানই বেবহার করতে পারছি। এখানে একমাত্র বাজে বেপার হচ্ছে `until` অংশটুকু `parantheses` এর ভিতরে। 
যদি আমরা `(until (x < 4))` এর পরিবর্তে `until (x < 4)` লিখতে পারতাম, তাহলে আমাদের `implementation` এখানেই শেষ হয়ে যেত। কিন্তু 
এখন আপনি সেটা করতে গেলে `compile fail` করবে (কেন বলতে পারবেন?)। 

### পঞ্চম ধাপঃ Anonymous object 

নিচের কোডটুকু খেয়াল করুন। 

```scala
class Foo {
  def bar(f: => Boolean) = f
}
val foo = new Foo()

foo.bar(3 > 2)  // evaluates to true  
foo bar (3 > 2) // evaluates to true  
```

`foo object` এর `bar method` আমরা দুইভাবে `call` করতে পারি, `.` দিয়ে, অথবা স্পেস দিয়ে।  

class FooWithBody(body: => Unit) {
  def bar(f: => Boolean): Unit = {
    if (f) body
  }
}

new FooWithBody {
  println("Foo with body class")
} bar (2 > 1)

```scala
def fooBody(body: => Unit) = new {
  def bar(f: => Boolean): Unit = {
    if (f) body
  }
}

fooBody {
  println("Foo body anonymous func")
} bar (2 > 1)
```

### শেষ ধাপঃ repeat until 
 
```scala
def repeat(b: => Unit) = new {
  def until(c: => Boolean): Unit = {
    b
    if (c) until(c)
  }
}


var i = 0
repeat {
  println("Hello, World!")
  i = i + 1
} until (i < 10)
```

### পরিশিষ্ট
`repeat-until` দেখানোর উদ্দেশ্য হল, কিছু `language feature` ব্যবহার করে কিভাবে আমরা সহজেই `built-in control structure` এর মত 
`control structure` তৈরি করতে পারি। এইরকম `control structure`, `api` কে `fluent` করে তুলতে অথবা `DSL implement` করতে কাজে লাগে।    
