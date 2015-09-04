---
layout: post
title: A Picture Language in Elm
---


<div class="message">
  TL;DR: This is an implementation of the <a target="_blank" href="https://mitpress.mit.edu/sicp/full-text/sicp/book/node36.html">picture language example from SICP</a> in elm. 
  You can copy the code from <a target="_blank" href="https://github.com/JobaerChowdhury/picture-language/edit/master/PictureLanguage.elm">here</a> 
  and paste it on <a target="_blank" href="http://elm-lang.org/try">Try Elm</a> to see the result. Have fun!
</div>


## Background
I remember I was fascinated by the example called “A picture language” while I was reading 
<a target="_blank" href="https://mitpress.mit.edu/sicp/">SICP</a>. But I was unable to 
try that example out on my own due to my limited knowledge on Scheme or Lisp. I am recently learning 
<a target="_blank" href="http://elm-lang.org">Elm</a> and while running <a target="_blank" 
href="http://elm-lang.org/examples">some examples</a> I found it’s very easy to draw something on the browser. And I 
decided to simulate the picture language example using Elm. This picture language example shows us how we can define 
some primitive operations and build complex structure from those primitive operations. We will start with a basic 
painter which looks like this - 

And after that we define some procedures which ultimately transforms our basic painter to this beautiful, 
complex pattern. I recommend you to read the <a href="https://mitpress.mit.edu/sicp/full-text/sicp/book/node36.html">book example</a>. 
Although reading that is not mandatory to follow the code examples here.

## Vectors, Frames and drawLine 

First we define a type alias Point which is just a pair of Float. 

{% gist 05a4f960891ba39528e4 %}

Now we define three primitive vector operations for working with Points. Those are addVect for vector addition, subVect for subtraction and scaleVect are for scaling vectors.  

{% gist 72509bab1defc684ee5f %}

Elm is influenced by **Haskell**. We can give the type annotation with each definition. Here *addVect/subVect* is taking 
two points as parameters and produces a new point. *ScaleVect* is taking a float and a point as parameter, and returns a point. 

Next we define a type called Frame. Frame has three properties, the origin, first edge and second edge. 

{% gist af2be00c00a7bbd1bccf %}

Next we define the frameCoordMap as exactly defined in the book text. Here frameCoordMap will take two parameter, 
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
image like picture 1.1

{% gist ef160bfeaa4faab26321 %}

todo - image 1. 

## Transforming painters 
Now we will write a function which will transform painters. Let’s define the function first. 

{% gist c2a6fca1cdde476f89e2 %}

“Painter operations are based on the procedure transform-painter, which takes as arguments a painter and 
information on how to transform a frame and produces a new painter. The transformed painter, when called on a frame, 
transforms the frame and calls the original painter on the transformed frame.The arguments to transform-painter 
are points (represented as vectors) that specify the corners of the new frame: When mapped into the frame, the first 
point specifies the new frame's origin and the other two specify the ends of its edge vectors. Thus, arguments within 
the unit square specify a frame contained within the original frame.”

Now we can use the transformPainter to define various transformations. For example the flipVert and flipHoriz like this. 

{% gist b2734592c1192521f51e %}

Result of flipHoriz on wave painter is given on figure 2. todo 

We can also define other transformations like beside or below. For example here is the below function. 

{% gist 120b55055389b2c424b6 %}

What below does is splits the frame into two half. and draws the two painters passed as parameters in the one half each.
 Result of (below wave (flipHoriz wave) is shown in figure 3. todo. The implementation of beside is very similar. 

We can also define some recursive functions as painters. For example we can define a painter which will split the 
initial painter in smaller part with each subsequent call. 

{% gist 472fd348305dec45a96b %}

The result of performing (rightSplit wave 6) is shown in figure 6. todo. Similarly we can define another method upSplit. 

We are pretty close to our final painter. First we define the function cornerSplit. 

{% gist 95d740e4693850c93f87 %}

It is a simple recursive function. We only have to calculate the smaller pieces and place them appropriately. 
Result of performing (cornerSplit wave 6) is shown in figure 7. 

Finally we define the squareLimit function as below. 

{% gist e3d8a6f4acbbb21e0d91 %}

It is also a simple function where we compute the smaller parts (quarter and half) of the frame. And then we place 
the smaller parts appropriately. Now we can apply the squareLimit function to our wave painter and get the beautiful 
pattern shown in figure 7. 

{% gist 946274f47f75fc30ee34 %}

## Final thoughts 
One of the most important thing is that painter is a primitive operation. And all the operations produce painter 
themselves. Which is called closures. And using these property we can build arbitrarily complex systems from very 
few basic operations. 

You can check the full code <a href="https://github.com/JobaerChowdhury/picture-language">here</a>.  
