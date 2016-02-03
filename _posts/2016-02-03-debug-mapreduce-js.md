---
layout: post
title:  "Debugging MapReduce in Javascript"
date:   2016-02-03 14:43:00
author: "Matt Molnar"
categories: es6 javascript mapreduce
published: true
---

With single line arrow functions in ES6, it’s getting easier and cleaner to use JS’s native Array.prototype.map() and Array.prototype.reduce() functions for molding our data. I recently decided to go the MapReduce route for a project to filter some data upfront. Inevitably, I ran into some issues with my logic and wasn’t getting what I expected. Thing is, there doesn’t seem to be a simple way to add debugging to a chain of map() and reduce() calls, particularly if you’re using single line arrow functions.
{% highlight javascript linenos %}
let behaviors = track.metrics
  .reduce((list1, m) => list1.concat(m.behaviors), [])
  .reduce((list2, b) =>
    ((list2.indexOf(b) === -1 && list2.push(b)) || 1) && list2
  , [])
  .map(bId => json.behaviors[bId]);
{% endhighlight %}
Here I need to take multiple arrays of id’s, flatten them, remove duplicates, then map in their respective object for the given id. I had an issue with the final value at first (note, above code is correct) and wanted to debug the values between each method call in the chain.

How do I do this? Well, very rough approach would be to switch to bracketed arrow functions, which will allow me to run two statements in the function.
{% highlight javascript linenos %}
let behaviors = track.metrics
  .reduce((list1, m) => {
    console.log(list1);
    return list1.concat(m.behaviors)
  }, [])
  .reduce((list2, b) => {
    console.log(list2);
    return ((list2.indexOf(b) === -1 && list2.push(b)) || 1) && list2
  }, [])
  .map(bId => json.behaviors[bId]);
{% endhighlight %}
This works but suffers a couple drawbacks. You lose the syntactic conciseness of the implicitly returning no bracket arrow functions, resulting in what looks like a borderline ES5 mess. Second, I get way too many calls to console.log as the functions get run for every item in the array. So I would have to add additional logic to each function to prevent that. I’m not even going to bother typing that all out here.

So, I wrote up a rather simple factory function to use in this case. 
{% highlight javascript linenos %}
function MRDbg(...txt){
  return function(prev, cur, i, arr){
    if (i === 1){
      console.log.apply(console, [...txt, arr]);
    }
    return arr;
  }
}
{% endhighlight %}
It expects to be called as a reduce() handler and allows you to label each debug call. So it fits nicely into my current MapReduce chain, and, here’s a big one, I don’t have to modify my existing functions and then worry about changing them back later!
{% highlight javascript linenos %}
let behaviors = track.metrics
  .reduce((list1, m) => list1.concat(m.behaviors), [])
  .reduce(MRDbg('first'))
  .reduce((list2, b) =>
    ((list2.indexOf(b) === -1 && list2.push(b)) || 1) && list2
  , [])
  .reduce(MRDbg('second'))
  .map(bId => json.behaviors[bId]);
});
{% endhighlight %}
Much cleaner! We then get some very simple console outputs.
{% highlight text %}
first [140, 141, 142, 143, 144, 132, 145, 146, 147, 148, 149, 150, 151, 152, 153, 87, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 100, 178, 89, 179, 101, 102, 98, 176, 177, 180, 100, 178, 89, 179, 101, 181, 102, 182, 140, 141, 142, 144, 145, 146, 147, 148, 149, 183, 150, 152, 184, 87, 154, 155, 158, 159, 160, 110, 185, 163, 112, 164, 166, 167, 168, 169, 170, 171, 173, 186, 174, 175]
main.js:115 
second [140, 141, 142, 143, 144, 132, 145, 146, 147, 148, 149, 150, 151, 152, 153, 87, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 100, 178, 89, 179, 101, 102, 98, 180, 181, 182, 183, 184, 110, 185, 112, 186]
{% endhighlight %}
You can find a gist of the function [here][gist] along with an ES5 compatible version.

[gist]: https://gist.github.com/mattcodez/617336258d243078f74f
