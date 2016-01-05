---
layout: post
title:  "The Limits of ES6 Destructured Assignment"
date:   2016-01-05 15:25:00
author: "Matt Molnar"
categories: es6 javascript
published: true
---

In working with a React component today, I wanted to copy some, but not all, properties from props to state. My immediate thought was that there is certainly a way to do this with ES6 destructuring assignment. After failing to find any relevant examples online, I eventually found my way to [this SO answer][so] which concludes that it can’t be done with the current spec.

Still wanting something a bit less redundant than stating the object names for every assignment, I eventually thought up a partial “solution”. Use destructured assignment to initialize the required properties as local variables. Then again to set the properties of an object literal.

{% highlight javascript linenos %}
let props = {x:1, y:1, z:2, j:4};
let state;

{ 
  const {x, y, z} = props;
  state = {x, y, z};
  console.log(x);
}

console.log(state);
console.log(x);
{% endhighlight %}

Results:

{% highlight text %}
1
{"x":1,"y":1,"z":2}
x is not defined
{% endhighlight %}

There are a few things I don’t like about this so I’m not sure if I’ll continue to use it. For one, you end up polluting your local namespace. Hence me wrapping the assignment in it’s own code block. This makes the local destructured variables unavailable throughout the rest of the function. It sort of solves that issue, but you still end up with a new code block and indenting. Also, I have a feeling it may not be very clear to other programmers what’s going on here.

[so]: http://stackoverflow.com/a/29621542/119909
