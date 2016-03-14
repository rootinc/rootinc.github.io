---
layout: post
title:  "Moving to Babel 6 on the Server"
date:   2016-03-14 13:20:00
author: "Matt Molnar"
categories: javascript babel sailsjs nodejs
published: true
---

Decided it was time to upgrade my server-side code to run on Babel 6. Below is a synopsis of all the issues I ran into and resolved while upgrading my 0.11.3 SailsJS server to run with Babel 6 transpilation. 

The upgrade to Babel 6 itself is easily achieved in Sails by upgrading the `sails-hook-babel` package.

### Missing preset
`couldn't find preset "stage-0" relative to directory`
Just because a preset is on the official Babel preset page, doesn’t mean that Babel comes with it. Simple fix by installing the [package][1] from npm. Read more [here][2].
[1]: https://www.npmjs.com/package/babel-preset-stage-0
[2]: http://jamesknelson.com/the-six-things-you-need-to-know-about-babel-6/

### Need strict mode everywhere
`Block-scoped declarations (let, const, function, class) not yet supported outside strict mode`
I didn’t have to worry about this before, and I don’t feel like worrying about it now. There is a Babel plugin that will automatically add “use strict” to your server-side files so you don’t have to:
`npm install babel-plugin-transform-strict-mode`
Then add it to Babel’s plugin settings:
`"plugins": ["transform-strict-mode"]`
### Don’t forget ES2015
`babel unexpected token import`
So I thought incorrectly that ‘stage-0’ (which I already added as a preset since I knew I would need it) would also backfill to all of ES6. Not the case, ‘es2015’ must be added as a preset to include full es2015/ES6 support.
`presets:['es2015', 'stage-0']`
### No await* anymore
`await* has been removed from the async functions proposal. Use Promise.all() instead.`
I knew this one was coming. An easy enough fix, the array that used to be an opperand of `await*` is now instead wrapped in `Promise.all()` and you simply `await` that call. Some developers will go as far to globalize with `const all = Promise.all.bind(Promise)` and then simply `let [v1, v2] = await all([p1, p2])`.

### Imports have been fixed
This issue was by far the biggest, trickiest and most time consuming. TL;DR, Babel 5 allowed a broken use of ES6 import/export syntax. There’s a great article on this [here][3]. Which was then fixed in Babel 6. I happened to make significant use of the broken syntax.
[3]: https://medium.com/@kentcdodds/misunderstanding-es6-modules-upgrading-babel-tears-and-a-solution-ad2d5ab93ce0#.1u5q5vjwf
There is, of course, a [plugin][4] to make the broken syntax work again. I opted to just fix the code and be done.
[4]: https://www.npmjs.com/package/babel-plugin-add-module-exports
It started out with issues binding my routes to controller actions:
```error: Ignored attempt to bind route (/users) to unknown controller.action :: user.users
//… various others```
I won’t fully explain the issue here as kentcdodds’ article above does a very good job of that. In short, SailsJS is using the CommonJS import syntax which is not 100% translatable to ES6 import. Specifically with defaults, which now have to be named. Thus, I have two options for controllers, either name all the controller actions, or go back to CommonJS exporting. I opted for the former with some recent smaller files, older files I just reverted back.
#### Named exports solution
`export async function users(req, res){...`
#### CommonJS pattern
```module.exports = { 
	async users(req, res){...```

For controllers, I think I may use the named export syntax moving forward since it removes an indent level from these files. 
For models, you really have to just switch back to CommonJS. It may be possible to perform named exports on all the attributes in the model but I didn’t bother to try as this would be a lot of work and would likely make the models much less readable.
### All routes hang
After clearing up all the syntax issues that I was getting explicit errors on, my application was technically running. However, all my routes seemed to hang in the browser. After much trial and error, I found that this was again the Babel 6 import fix at work. I had incorrectly used `export default {...` everywhere, and this time it was in middleware for one of my policies. So, I did the same CommonJS fix as above on any middleware. I also had to perform this change for any services files.


### Done!
Back up and running with all tests passing! It is a good day.
