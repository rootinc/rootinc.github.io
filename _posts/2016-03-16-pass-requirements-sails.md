---
layout: post
title:  "Changing Password Requirements with SailsJS and Passport"
date:   2016-03-16 17:00:00
author: "Matt Molnar"
categories: javascript babel sailsjs passport
published: true
---
If you perform an installation of [Passport][passport] with [SailsJS][sails] using the [Sails Passport Auth Generator][sails-generate-auth] you get several files in your app already configured for you. If you then use passport-local, you will already have a complexity requirement on the password. It defaults to requiring 8 characters minimum, letters, numbers, and symbols. 

What if you want to change this requirement? In the generated model file `Passport.js`, you should see a line that says `provider   : { type: 'alphanumericdashed' },` and `password    : { type: 'string', minLength: 6 }`. The minLength is an easy and obvious change. What about the complexity requirement though? This stumped me for a bit. There doesn’t seem to be any mention of these keywords or providers on the Passport official site, nor anything in the [Passport-local repository][passport-local]. I believe I found the answer, they appear to be ORM validations from Sails itself. You can find a list [here][validations]. 

For my project, I simply switched to `alphanumeric` and I no longer need symbols in my passwords. I suppose anything on the list linked above should work. I didn’t try it but I suppose you could even set it to `ipv6` and require that every user’s password be a valid ipv6 formatted address. Not sure I would recommend that though.

[passport]: http://passportjs.org/
[sails]: http://sailsjs.org/
[sails-generate-auth]: https://github.com/kasperisager/sails-generate-auth
[passport-local]: https://github.com/jaredhanson/passport-local
[validations]: http://sailsjs.org/documentation/concepts/models-and-orm/validations#/validation-rules
