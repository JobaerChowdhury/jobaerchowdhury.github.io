---
layout: post
title: Scala Control Abstraction
excerpt: Scala Control Abstraction
keywords: scala, functional programming, control structure, repeat until, custom, by-name
---

## Scala Control Abstraction 

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

উপরের কোড দেখে আপনাদের কি মনে হচ্ছে না যে `repeat-until` `scala` তে `built-in`? আমার প্রথমে তাই মনে হয়েছিল। চলুন দেখি কিভাবে 
আমরা এই `control structure` টি `implement` করতে পারি।  

###  প্রথম ধাপঃ repeatN
চলুন দেখি repeatN এর টাইপ সিগনেচার।

repeatN: (f: () => Unit, n: Int) => Unit

টাইপ সিগনেচার দেখে বুঝা যাচ্ছে যে repeatN হল এমন একটি function যেটা ২টি parameter নেয়, একটা
কোড ব্লক f ও একটা integer n, এবং repeatN ঐ কোড ব্লকটিকে n সংখ্যক বার execute করে।

repeatN( () => {
    println("Hello, repeatN")
  }, 3)

কাজেই উপরের code টি তিন বার "Hello, repeatN" প্রিন্ট করবে। repeatN এর implementation:


def repeatN1(f: () => Unit, n: Int): Unit = {
  if (n > 0) {
    f()
    repeatN1(f, n - 1)
  }
}


repeatN একটি higher order function, যার প্রথম parameter টি একটি function যেটি কোন parameter নেয় না এবং কোন কিছু
return ও করে না। যখন আমরা f() call করি, শুধুমাত্র f এর body execute হয়।




repeatN1(() => {
  println("Hello, repeatN")
}, 3)



### দ্বিতীয় ধাপঃ better repeatN
repeatN এর একটা জিনিস আমার পছন্দ হয়নি, তা হলঃ

  () => {
    println("Hello, repeatN")
  }

ভাল হত যদি আমরা প্রথমের "() =>" অংশটুকু বাদ দিতে পারতাম, নিচের মত।

repeatN2({
  println("Hello, repeatN2")
}, 3)

Scala তে by-name parameters ব্যবহার করে আমরা সেটা করতে পারি। by-name parameter তৈরি করার জন্য আমরা parameter টাইপকে
() => এর পরিবর্তে শুধু => দিয়ে পরিবর্তন করব। নিচের মতঃ

def repeatN2(f: => Unit, n: Int): Unit = ???

চলুন দেখি নতুন repeatN কিভাবে implement করা যায়।


```scala 
def repeatN2(f: => Unit, n: Int): Unit = {
  if (n > 0) {
    f
    repeatN2(f, n - 1)
  }
}

repeatN2({
  println("Hello, repeatN2")
}, 2)
```


### তৃতীয় ধাপঃ until ছাড়া repeat


```scala
def repeatC(b: => Unit)(c: => Boolean): Unit = {
  b
  if (c) repeatC(b)(c)
}

var t = 0
repeatC {
  println("Hello, repeat with condition")
  t = t + 1
}(t < 5)
```

def myUntil(f: => Boolean): Boolean = f

var x = 0
repeatC {
  println("Hello, repeat until (almost)")
  x = x + 1
}(myUntil(x < 4))


class Foo {
  def bar(f: => Boolean) = f
}

val foo = new Foo()
foo.bar(1 > 2)

foo bar (1 > 2)

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


```scala
def repeat(b: => Unit) = new {
  def until(c: => Boolean): Unit = {
    b
    if (c) until(c)
  }
}


var i = 0
repeat {
  println("Hello")
  i = i + 1
} until (i < 500)
```


Summary:

points what we learned

