---
layout: post
title: Tail recursion in Scala
excerpt: Tail recursion in Scala
keywords: scala, functional programming, recursion, tailrec, tail recursion
---
> পড়তে সময় লাগবে `<TODO>` মিনিট

### ভূমিকা
আপনারা অনেকেই হয়ত recursion ব্যবহার করেছেন। কিছু কিছু সমস্যা সমাধানে recursion এর ব্যবহার খুবই স্বাভাবিক, যেমন  gcd, fibonacci, factorial ইত্যাদি। 

```scala 
def factorial (n:Int): Int = 
    if (n == 0) 1 
    else (n * factorial(n-1))
```
Scala তে recursive ফাংশান লিখার সময় একটি জিনিস আমাদের জানা খুব জরুরি, তা হল `tail recursion`. আমাদের লিখা `recursive` ফাংশান যদি `tail recursive` না হয়, তাহলে প্রোগ্রাম চলার সময় `StackOverflowError` হবার সম্ভাবনা থেকে যায়। চলুন দেখি বিস্তারিত। 

### Tail recursion কি? 
আমরা জানি recursive ফাংশান হল সেই ফাংশান, যে তার বডি এর মধ্যে নিজেকেই আবার কল করে। কোন `recursive` ফাংশান এর একদম শেষের এক্সপ্রেশনটি যদি হয় তার নিজেকে কল করা, তাহলে সেই ফাংশান কে আমরা বলি `tail recursive` ফাংশান। যেমন নিচের `gcd`  ফাংশানটি একটি `tail recursive` ফাংশান, কারণ ফাংশানটি তার শেষ এক্সপ্রেশন এ নিজেকে কল করেছে । 

```scala
def gcd(a: Int, b: Int): Int = {
    if (b == 0) a
    else gcd(b, a % b)
}
```

### Tail recursion এর গুরুত্ব 
জাভা ভার্চুয়াল মেশিন এ প্রতিটি ফাংশান কল এর জন্য একটি করে `stack` ফ্রেম ব্যবহার হয়। `recursive` ফাংশান এর ক্ষেত্রে ব্যাপারটা অনেকটা নিচের মত। 
* যখন একটি `recursive` ফাংশান নিজেকে কল করে, তখন ঐ ফাংশানটির ইনফর্মেশন এর একটি কপি `stack` এ পুশ করা হয়। 
* প্রতিবার ফাংশানটি যখন নিজেকে কল করে, নতুন করে ফাংশানটির ইনফর্মেশন `stack` এ পুশ করা হয়। এ কারণে `recursion` এর প্রত্যেকটি লেভেল এর জন্য একটি করে `stack` ফ্রেম দরকার হয়। একটি `recursive` ফাংশান যদি ১ লাখ বার নিজেকে কল করে তাহলে ১ লাখটি `stack` ফ্রেম তৈরি হবে। 
* সমস্যা হল জাভা ভার্চুয়াল মেশিন প্রতিটি থ্রেড এর জন্য `stack` এর আকার সীমাবদ্ধ করে দিয়েছে। কাজেই যখন সেটার অধিক মেমরি আমাদের ফাংশান নিতে চাইবে, ভার্চুয়াল মেশিন তখন `StackOverflowError` থ্রো করবে।  

`Tail recursive` ফাংশান এর সুবিধা হল, এর প্রত্যেকটি লেভেল এর জন্য নতুন করে কোন `stack` ফ্রেম এর প্রয়োজন হয় না, কাজেই `StackOverflowError` হবার কোন সম্ভাবনা নেই। চলুন একটি উদাহরণ এর মধ্যমে ব্যাপারটি দেখে নেওয়া যাক। 

```scala 
def sum(ls: List[Int]): Long = ls match {
    case Nil => 0                // if the list is empty, return 0
    case x :: xs => x + sum(xs)  // otherwise with the current element, add the rest of the element's sum to get the result
}
```
উপরের ফাংশানটি একটি `recursive` ফাংশান, যা একটি লিস্ট এর যোগফল বের করছে। বলতে পারবেন যে উপরের ফাংশানটি কি `tail recursive`? 

আপাতদৃষ্টিতে মনে হচ্ছে ফাংশানটি যেহেতু তার শেষ লাইন এ নিজেকে কল করেছে, সেহেতু ফাংশানটি হয়ত `tail recursive`, কিন্তু আসলে তা নয়। ফাংশানটির শেষ লাইন করে ভেঙ্গে যদি আমরা নিচের মত করে লিখি তাহলেই জিনিসটা ধরতে পারব। 

```scala
val current = x
val restSum = sum(xs)
current + restSum     
```

যেহেতু ফাংশানটি `tail recursive` নয়, সেহেতু আমার মেশিন এ (ডিফল্ট সেটিংস, java 8, scala 2.12) আমি নিচের মত করে ফাংশানটিকে ব্যবহার করতে গেলে `StackOverflowError` পাই। 

```scala
val myList = (1 to 8000).toList
val result = sum(myList)

...

Exception in thread "main" java.lang.StackOverflowError
```

এবার চলুন আমরা এই ফাংশানটির একটি `tail recursive` ভার্সন লিখি, এবং সেটা চালিয়ে দেখি। 

```scala
  def sumT(ls: List[Int]): Long = {
    @tailrec 
    def sumInternal(ls: List[Int], acc: Long) : Long = ls match {
      case Nil => acc
      case x :: xs => sumInternal(xs, x + acc)
    }

    sumInternal(ls, 0)
  }

  val myList = (1 to 8000).toList
  val result = sumT(myList) 
  // no error this time, result is computed fine. 
```

এখানে Scala compiler যখন `recursive` ফাংশানটিকে `tail recursive` হিসাবে চিহ্নিত করতে পারে, তখন compiler যে বাইটকোড তৈরি করে সেখানে মাত্র একটি `stack frame` ব্যবহার করেই পুরো ফাংশান এর কাজটি সম্পন্ন করে ফেলে। এক্ষেত্রে compiler `recursive` ফাংশানটিকে একটি `loop` এ রূপান্তরিত করে (প্রতিটি নতুন ফাংশান কল হয়ে যায় একটি `jump` ইন্সট্রাকশন)। যে কারণে প্রতিটি `recursion` লেভেল এর জন্য আলাদা আলাদা `stack frame` এর প্রয়োজন হয় না, যে কারণে `StackOverfloError` ও এড়ানো যায়। এই পুরো ব্যাপারটিকে বলা হয় `tail call optimization`। 

কাজেই যখন আমরা নিশ্চিত হতে পারব না যে আমাদের `recursive` ফাংশানটিকে কি পরিমাণ data নিয়ে কাজ করতে হচ্ছে, তখন আমরা অবশ্যই চেষ্টা করব যাতে ফাংশানটিকে `tail recursive` করে লিখা যায়। নইলে রানটাইমে গিয়ে `StackOverfloError` হবার সম্ভাবনা প্রচুর। 

### @tailrec annotation 
যদিও `scala compiler` নিজে থেকেই `tail recursive` ফাংশান চিহ্নিত করতে পারে, তারপরেও আমরা আমাদের `tail recursive` ফাংশান এ `@tailrec` annotation টি ব্যবহার করতে পারি। এই annotation টি ব্যবহারের সুবিধা হল, যদি আমরা কোন ফাংশান যেটি `tail recursive` নয়, সেটাতে এই `annotation` টি ব্যবহার করি, তাইলে কম্পাইলেশন ফেইল হবে। যেমন আমরা যদি প্রথম `sum` ফাংশানটিতে এই `annotation` ব্যবহার করি তাহলে নিচের ঘটনা ঘটবে। 

```scala
Error: could not optimize @tailrec annotated method sum: it contains a recursive call not in tail position
```

### কখন প্রযোজ্য নয়
নিম্নলিখিত ক্ষেত্রে Scala compiler, tail call optimize করবে না। 
* যদি মেথডটিকে `override` করা যায়
* যদি `indirect recursion` অথবা `mutual recursion` হয়
* শেষ কলটি একটা `function value` তে যায়

শুধুমাত্র `private` অথবা `final` অথবা অন্য কোনও মেথড এর ভিতরের মেথড `tail call optimization` এর জন্য বিবেচিত হবে। 

### পরিশিষ্ট
প্রয়োজন না হলে `recursive function` না লিখাই ভাল। আগে দেখতে হবে  যেসব `library` ফাংশান দেয়া আছে, যেমন `fold`, `map`, `reduce` ইত্যাদি দিয়ে কাজ হচ্ছে কিনা। আর যদি `recursive` ফাংশান না লিখে কোনও ভাবেই হাতে থাকা সমস্যাটি সমাধান করা না যায়, তাহলে চেষ্টা করতে হবে যাতে ফাংশানটি `tail recursive` হয়। 

এ বিষয়ে আরও বিস্তারিত জানতে নিচের লিঙ্কগুলো দেখতে পারেন। 
* [Martin Odersky এর বই এর এই অংশটুকু](https://www.artima.com/pins1ed/functions-and-closures.html#8.9)
* [Stack frame নিয়ে বিস্তারিত](https://www.artima.com/insidejvm/ed2/jvm8.html)
* [Scala এ কিছু recursive function এর উদাহরণ](https://alvinalexander.com/scala/scala-recursion-examples-recursive-programming)