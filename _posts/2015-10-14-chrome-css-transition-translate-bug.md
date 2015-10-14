---
layout: post
title:  "Chrome CSS Transition Translate Bug"
date:   2015-10-14 9:00:00
author: "Dan Jewett"
categories: css javascript
published: true
---
I came across an annoying thing today working through one of our modules.  The effect was simple -- translate a box from off the screen onto the screen.  I achieved this by doing something like this:

{% highlight css linenos %}
.mainBox{
  transition: transform 1s ease-out 0s;
  -webkit-transition: -webkit-transform 1s ease-out 0s;
  -moz-transition: -moz-transform 1s ease-out 0s;
  -ms-transition: -ms-transform 1s ease-out 0s;

  transform: translate(1024px, 0px);
  -webkit-transform: translate(1024px, 0px);
  -moz-transform: translate(1024px, 0px);
  -ms-transform: translate(1024px, 0px);
}

.mainBox.animate{
  transform: translate(0px, 0px);
  -webkit-transform: translate(0px, 0px);
  -moz-transform: translate(0px, 0px);
  -ms-transform: translate(0px, 0px);
}
{% endhighlight %}

However, only in Chrome, click events and `cursor:pointer` were not working as expected.  It would take an intermittent  amount of time for click events and `cursor:pointer` to begin working again.  You can check it out a simple version here:

<a href="http://jsfiddle.net/udqa4jxb/">http://jsfiddle.net/udqa4jxb/</a>

The effect is much more detrimental in the module.  My solution wasn't pretty, but it fixes the annoying issue nevertheless.  When I add the class, I do this afterwards:

{% highlight javascript linenos %}
this.chromeTimeoutBug = setTimeout(function(){
  that.el.find('.mainBox').css('-webkit-transition','all 0s ease 0s');
  that.chromeTimeoutBug = null;
},1010);
{% endhighlight %}

and when clearing the page,

{% highlight javascript linenos %}
if (this.chromeTimeoutBug)
{
  clearTimeout(this.chromeTimeoutBug);
  this.chromeTimeoutBug = null;
}

this.el.find('.mainBox').css('-webkit-transition','-webkit-transform 1s ease-out 0s');
{% endhighlight %}

Essentially, I found that the issue is related somehow to the transition property.  If I get rid of the transition of effect, and only addClass/removeClass with translate, the glitch does not occur.  Therefore, in my code, I set a timeout to the duration of the animation, in my case 1 second (1000ms), and override the `-webkit-transition` property to its default, and when I reset the page, I set the `-webkit-transition` property back to what I had in the css.

I hope this helps someone who encounters this issue.  Hopefully the bug will get resolved soon.
