---
layout: post
title: "Javascript arrays and objects. Wait, what?"
comments: true
---

I received an email from a developer recently asking about some code I had authored which was along the lines of:

``` js 
function addOrUpdateThing(inputThing) {
    var index = self.thingCache.keys[inputThing.thingId];

    // loop through the thing and extract special bits
    Object.keys(self.specialBits).forEach(function(key){
        var soSpecial = inputThing[key].toFixed(3);
        doThatSpecialConcern(soSpecial);
    });
}
```

The question was long the lines of:
> The inputThing is used as both an object and an array, what gives?
> You've got it like an object resembling: 
> 
``` js
inputThing = {
   thingId: 1
};
var index = self.thingCache.keys[inputThing.thingId];
```
> But you're also using it like an array:
> 
> ``` js
inputThing = [1,2,3,4];
var soSpecial = inputThing[key];
```


That's an awesome question, and the kind of thing I love answering. Although it's something that's obvious to many people who develop using javascript, for someone whose javascript experience extends to attaching event listeners using jQuery it had them stumped.

The short answer of course is "they're all the same, everything you knew to be true about arrays and objects is invalid in javascript. Have a cuppa and a lie down"

I apologise in advance, the explanation to get to that point is kind of a long winded but please humour me. Below is my (almost) verbatim response.


## Arrays, 0..n-1 right?

Well kind of... 
You are probably aware that arrays in javascript aren't tied to a length like they are in languages like c#. Which means you can call:

``` javascript 
theArray.push(value)
```

until the cows come home and never have an issue (unless the cows take a long time and you run out of memory). So when you do something like array.push or array.unshift it'll add the index for that item as the next available integral index; meaning 

``` js
var a = [];
a.push('foo'); 
```
will give you <span class='highlight'><code class='language-js'><span class='nx'>a</span>[<span class='mi'>0</span>] === <span class='s1'>'foo'</span>;</code></span> evaluating to true

Now indexes in js arrays also don't have to be contiguous, so it's perfectly legal to do (building on the code from above):

``` js
a[99] = 'bar';
```

Giving you an array of length 100 with 'foo' at index 0 and 'bar' at index 99 (and a whole lot of undefineds in the middle).
Incidentally, a push would give a new element at index 100 here since it's the next available integral index.
So a.push('baz') gives us: `[ 0: 'foo', 99: 'bar', 100: 'baz' ]`

What you may not know is that arrays don't have to be indexed only by numbers (like they do in most typed languages). So we can do some weird looking stuff like…

``` js
a['waitWhat'] = 'mind blown';
```
and we'll have: `[ 0: 'foo', 99: 'bar', 100: 'baz', 'waitWhat': 'mindBlown' ]`

Looking at how I've laid out those examples, it looks a lot like the object syntax (intentionally). Arrays are just a special type of object, and due to how prototypical inheritance works (which is a whole other discussion for a different time) this means that arrays get all of the nice things that objects do. Which means we have:

``` js
a.waitWhat === 'mindBlown'; 
```
evaluating to true.

This leads us to the conclusion that the array initialisation syntax:

``` js
var b = []; 
```

is actually just syntactic sugar over

``` js
var b = new Array(); 
```

Which is also why <span class='highlight'><code class='language-js'>[] === []</code></span> evaluates to false. It also gives us that `push, pop, shift, unshift, filter, map, reduce, etc etc etc` are methods on the Array prototype (take that here as meaning class).

So as we saw before you could do `a['waitWhat']` and `a.waitWhat` which are the same thing, that's because they're acting on the object prototype (think here of it as a base class). And all objects are effectively associative arrays. 

## On to the original question:

inputThing is an object of the form:

``` js
{
    thingId: 1,
    specialProperty: '180.0000345'
}
```

And we're iterating over the keys of the specialBits (which are the property names).

So the second call is effectively:

``` js
inputThing['specialProperty'].toFixed(3);
```

Which is the same as

``` js
inputThing.specialProperty.toFixed(3);
```

or (loosely) the equivalent of (in C#)

{% highlight csharp %}
Type t = inputThing.getType();
PropertyInfo pi = t.GetPropertyInfo(key); // where key is "specialProperty"
Decimal moneyValue = pi.getValue(inputLocation);
{% endhighlight %}

Hopefully that clears it up a bit. If not, I'm more than happy to jump on a screen share and walk through it (and explain prototypical inheritance which does your head in coming from a real language like C#) or try and find something that explains it better than I do
