---
layout: post
title:  "Multiple Instruction Arrow Functions in ES6"
date:   2016-02-05 16:45:00
author: "Matt Molnar"
categories: es6 javascript
published: true
---

Recently, I had a bit of JavaScript code that was a small series of map and reduce functions. To keep things nice and tight, I of course wanted to use single line arrow functions as much as possible. Implicit returns, no brackets, no ‘function’ syntax. I had one reduce() call though where my logic was this:
{% highlight javascript linenos %}
arr.reduce((list2, b) => {
          if (list2.indexOf(b) === -1){
            list2.push(b);
          }
          return list2;
        }, []);
{% endhighlight %}
High level: only add item b from array arr to a new array if it doesn’t already exist in the new array. Simple duplicate removal for the original array. This works, and is probably the easiest to understand, but doesn’t make for the best grouping along with other calls on the same array. How could I modify this a bit so that I could use a bracketless arrow function with an implicit return?
{% highlight javascript linenos %}
arr.reduce((list2, b) =>
          ((list2.indexOf(b) === -1 && list2.push(b)) || 1) && list2
        , []);
{% endhighlight %}
This is certainly shorter in terms of vertical space, and a bit less code overall. However, it’s not very straight-forward to read. The logical AND operator ensures push() is only called on list2 if the item ‘b’ is not found in list2. In the case that ‘b’ is indeed in list2, the value’s index will not be -1 and the AND expression will short circuit and return false. Which would return false for the whole function, which we don’t want. So, I use the logical OR operator and 1 to ensure we get a truthy return value and then another AND to force list2 to be returned.

During all this, I was reminded of part of the JS spec about the comma operator. According to [MDN][comma]:

“The comma operator evaluates each of its operands (from left to right) and returns the value of the last operand.”

Nice! We can simplify this even more:
{% highlight javascript linenos %}
arr.reduce((list2, b) =>
          (list2.indexOf(b) === -1 && list2.push(b), list2)
        , []);
{% endhighlight %}
Which transpiles in ES5 to:
{% highlight javascript linenos %}
arr.reduce(function (list2, b) {
          return list2.indexOf(b) === -1 && list2.push(b), list2;
}, []);
{% endhighlight %}
We’ve removed two of the three logical operators and one of the parens wrappings. Overall, I’d say it’s cleaner than the 2nd iteration and preferable to it as well. However, the use of the comma operator like this is probably not immediately known to the random developer reading the code. Thus I’m a tad hesitant moving forward with it and may go with the more verbose, but easier to understand, bracketed arrow function. Neat trick though, especially as you can chain more commas and expressions to your heart’s desire.

[comma]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comma_Operator
