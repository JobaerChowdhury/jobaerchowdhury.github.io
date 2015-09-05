---
layout: post
title: A Picture Language in Elm
---


<div class="message">
  TL;DR: This is an implementation of the <a target="_blank" href="https://mitpress.mit.edu/sicp/full-text/sicp/book/node36.html">picture language example from SICP</a> in Elm. 
  You can copy the code from <a target="_blank" href="https://github.com/JobaerChowdhury/picture-language/blob/master/PictureLanguage.elm">here</a> 
  and paste it on <a target="_blank" href="http://elm-lang.org/try">Try Elm</a> to see the <a href="#fig1">result</a>. Have fun!
</div>


## Background
I remember I was fascinated by the example “A picture language”, while I was reading 
<a target="_blank" href="https://mitpress.mit.edu/sicp/">SICP</a>. However, I was unable to 
try that example out on my own, due to my limited knowledge on Scheme or Lisp. I am recently learning 
<a target="_blank" href="http://elm-lang.org">Elm</a>, and while running <a target="_blank" 
href="http://elm-lang.org/examples">some examples,</a> I found it’s very easy to draw something on the browser. And the 
first thing I wanted to do is to simulate the picture language example using Elm. This example 
shows us how we can define some primitive operations, and build complex structure uniformly from those operations. We 
will start with a basic painter that looks like <a href="#fig2">Figure-1.2</a>. 

<div id="fig1" class="image">
    <img src="/public/picture-language/painter-final.png" /> 
    <div class="caption">Figure-1.1: Final result</div>
</div>

After that we'll define some procedures which transforms our basic painter to this beautiful, 
complex pattern shown in <a href="#fig1">Figure-1.1</a>. I recommend you to read the <a href="https://mitpress.mit.edu/sicp/full-text/sicp/book/node36.html">book example</a>. 
Although reading that is not mandatory to follow the code examples here.

<div id="fig2" class="image">
    <img src="/public/picture-language/primitive-painter.png" /> 
    <div class="caption">Figure-1.2: A primitive painter</div>
</div>

## Vectors, Frames and drawLine 

First we define a type alias Point which is just a pair of Float. 

{% gist 05a4f960891ba39528e4 %}

Now we define three primitive vector operations for working with Points. Those are addVect for vector addition, subVect for subtraction and scaleVect are for scaling vectors.  

{% gist 72509bab1defc684ee5f %}

Elm is influenced by **Haskell**. We can give the type annotation with each definition. Here *addVect/subVect* is taking 
two points as parameters and produces a new point. *ScaleVect* is taking a float and a point as parameter, and returns a point. 

Next we define a type called Frame. Frame has three properties, the origin, first edge and second edge. 

{% gist af2be00c00a7bbd1bccf %}

Next we define the *frameCoordMap* as exactly defined in the book text. Here *frameCoordMap* will take two parameter, 
a frame and a Point. And it will map that point inside the frame and return the coordinates of the mapped point. 
So that the point will be shifted and scaled to fit the frame. 

{% gist fb2489311f63d9c8a147 %}

Our primitive drawing operation is drawLine. We will use that operation to implement complex paint procedures. 
For implementing drawLine we are using Elm’s segment function from Graphics module. The segment function takes 
two coordinates and creates a path between them. And the traced function takes a LineStyle and a path and returns 
a Form. Combining these two we can write the following function. 

{% gist 6914b7ce13e707722dce %}

We have chose to return Form because it’s very flexible type in elm’s graphics package. 
We can combine a collection of Forms into a single Form which will come handy when we combine painters. 
Here **|>** is the forward function application, **x |> f == f x**. This function is useful for avoiding parenthesis and 
writing code in a more natural way. We could write the drawLine function also like this.

{% gist 8390e4fc349cf66a0cde %}


But to me the former looks more readable. Now we are ready to define our painters. 

## Painters
Let’s define the type painter. Since we are following the book example, we will define the Painter type as something 
which takes a frame as parameter and returns a Form. 

{% gist 45011f2d71f194e3fade %}

Notice the difference between Point/Frame and Painter. Where Point and Frame represents some sort of values, 
the Painter represents a function. Invoking that function with a frame as parameter will produce our intended drawing. 
Representing Painter as function will turn out to be a powerful technique when we combine multiple painters.

The first painter we will define is called segmentsPainter. It takes a list of line segments (start and end coordinate) 
and draws the lines. That’s it. 

{% gist 8dcd922395e0646a4d50 %}

Notice the lambda notation represented by \frame ->. This means the segmentsPainter function is returning a function. 
For each pair of coordinated we first map them inside our frame. And then call the drawLine function to draw a line 
between them. Finally we combine a list of Forms into a single form by using the ‘group’ function. 

Using the segmentsPainter we can define different kind of painters. For example the ‘wave’ painter. The coordinate 
 values for the wave painter is taken from <a target="_blank" 
 href="http://www.billthelizard.com/2011/08/sicp-244-245-picture-language.html"> Bill the Lizard's page</a>.

{% gist f58186576f0509e52a7c %}

Now if we define a frame and call the main function to draw the wave painter like following, we will get a 
image like <a href="#fig2">Figure-1.2</a>.

{% gist ef160bfeaa4faab26321 %}

## Transforming painters 
Now we will write a function which will transform painters. Let’s define the function first. 

{% gist c2a6fca1cdde476f89e2 %}

Explanation from the book. 

<div class="message">
“Painter operations are based on the procedure transform-painter, which takes as arguments a painter and 
information on how to transform a frame and produces a new painter. The transformed painter, when called on a frame, 
transforms the frame and calls the original painter on the transformed frame.The arguments to transform-painter 
are points (represented as vectors) that specify the corners of the new frame: When mapped into the frame, the first 
point specifies the new frame's origin and the other two specify the ends of its edge vectors. Thus, arguments within 
the unit square specify a frame contained within the original frame.”
</div>

Now we can use the transformPainter to define various transformations. For example the flipVert and flipHoriz like this. 

{% gist b2734592c1192521f51e %}

Result of flipHoriz on wave painter is shown in <a href="#fig3">Figure-1.3</a>. 

<div id="fig3" class="image">
    <img src="/public/picture-language/flip-horiz.png" /> 
    <div class="caption">Figure-1.3: flipVert and flipHoriz on wave painter</div>
</div>

Now we will define two more transformations - beside and below.  

{% gist 120b55055389b2c424b6 %}

What below does is splits the frame into two half. and draws the two painters passed as parameters in the one half each.
Result of *(beside wave (flipHoriz wave)* is shown in <a href="#fig4">Figure-1.4</a>. The implementation of beside is very similar.
  
<div id="fig4" class="image">
    <img src="/public/picture-language/beside.png" /> 
    <div class="caption">Figure-1.4: Result of (beside wave (flipHoriz wave)</div>
</div>


We can also define some recursive functions as painters. For example we can define a painter which will split the 
initial painter in smaller part with each subsequent call. 

{% gist 472fd348305dec45a96b %}

The result of performing *(rightSplit wave 6)* is shown in <a href="#fig5">Figure-1.5</a>. 
Similarly we can also define another method upSplit, 
which will split recursively the image in upward location. 

<div id="fig5" class="image">
    <img src="/public/picture-language/right-split.png" /> 
    <div class="caption">Figure-1.5: Result of (rightSplit wave 6)</div>
</div>


We are pretty close to our final painter. First we define the function cornerSplit. 

{% gist 95d740e4693850c93f87 %}

It is a simple recursive function. We only have to calculate the smaller pieces and place them appropriately. 
Result of performing *(cornerSplit wave 6)* is shown in <a href="#fig6">Figure-1.6</a>. 

<div id="fig6" class="image">
    <img src="/public/picture-language/corner-split.png" /> 
    <div class="caption">Figure-1.6: Result of (cornerSplit wave 6)</div>
</div>

Finally we define the squareLimit function as below. 

{% gist e3d8a6f4acbbb21e0d91 %}

It is also a simple function where we compute the smaller parts (quarter and half) of the frame. And then we place 
the smaller parts appropriately. Now we can apply the squareLimit function to our wave painter and get the beautiful 
pattern shown in <a href="#fig1">Figure-1.1</a>. 

{% gist 946274f47f75fc30ee34 %}

## Final thoughts 
The important thing I learnt from this example is how various painter operations work in an uniform way. And we can 
build complex systems from very few basic operations.  

<div class="message">
“The fundamental data abstractions, painters, are implemented using procedural representations, which enables the 
language to handle different basic drawing capabilities in a uniform way. The means of combination satisfy 
the closure property, which permits us to easily build up complex designs.” -- SICP. 
</div>

You can check the full code <a href="https://github.com/JobaerChowdhury/picture-language">here</a>.  
