---
layout: post
title:  "Quick Deepstream.io Setup Using JSPM"
date:   2016-02-12 12:10:00
author: "Matt Molnar"
categories: es6 javascript jspm
published: true
---
Want to use JSPM rather than Bower for running the Deepstream.io example? Follow these steps. This is basically a duplicate of the [Getting Started tutorial][tutorial] on the [Deepstream.io website][website] but using a really simple JSPM setup. This is a very crude guide where I list everything I had to do to get things running.

Create an empty project folder
npm install deepstream.io
Copy server code verbatim from the Getting Started guide
jspm install npm:deepstream.io-client-js
Hit enter for all the prompts from JSPM

We’re going to modify the client side code a little bit. We have native support for ES6 compiling with JSPM/Babel so we can import the Deepstream client directly:
{% highlight javascript linenos %}
import deepstream from 'deepstream.io-client-js';
let ds = deepstream( 'localhost:6020' ).login();

let record = ds.record.getRecord( 'someUser' );

let input = document.querySelector( 'input' );

input.onkeyup = function(){
    record.set( 'firstname', input.value );
};

record.subscribe( 'firstname', function( value ){
    input.value = value;
});
{% endhighlight %}
Save as index.js

Run `jspm bundle index.js index-bundle.js`

If you have an error about ‘yeast’ missing, run `jspm install npm:yeast`

For the HTML, we’re going to move all the JS into the body so that the `input` exists before we try to find it. Note JSPM requires a couple extra files, which should already be ready to go.
{% highlight HTML linenos %}
<!DOCTYPE html>
<html>
    <head>
    </head>
    <body>
        <input type="text" />
        
        <script src="jspm_packages/system.js"></script>
        <script src="config.js"></script>
        <script type="text/javascript" src="index-bundle.js"></script>
        <script>
        System.import('index.js');
        </script>
    </body>
</html>
{% endhighlight %}
Run `node start.js` and open your index.html page in two browsers, you should now see the input’s staying in sync!

Again, this is a pretty crude “up and running” proof of concept. You’ll have to manually run JSPM’s `bundle` command for any client side changes. Ideally you’ll want a little more automated of an asset pipeline at some point.

[tutorial]: https://deepstream.io/tutorials/getting-started.html
[website]: https://deepstream.io/

