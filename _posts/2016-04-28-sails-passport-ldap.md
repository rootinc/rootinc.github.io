---
layout: post
title:  "Sails Passport LDAP"
date:   2016-04-28 17:00:00
author: "Dan Jewett"
categories: javascript sailsjs passport ldap
published: true
---
This tutorial is based on this [link][link1]:

I will point out differences:

1: We used this version of bcrypt `"bcrypt-nodejs": "0.0.3"`.  Also don't forget to add `"passport-ldapauth": "^0.5.0"`.

2: My user model looks like this:
{% highlight javascript linenos %}
var bcrypt = require('bcrypt-nodejs');

module.exports = {
  attributes: {
    email: {
      type: 'email',
      required: true,
      unique: true
    },
    firstName:{
      type:'string',
      required:true
    },
    lastName:{
      type:'string',
      required:true
    },
    password: {
      type: 'string',
      minLength: 6,
    },
  },
  beforeCreate: function(user, cb) {
    if (user.password)
    {
      bcrypt.genSalt(10, function(err, salt) {
        bcrypt.hash(user.password, salt, function(err, hash) {
          if (err) {
            cb(err);
          } else {
            user.password = hash;
            cb();
          }
        });
      });
    }
    else
    {
      cb();
    }
  }
};
{% endhighlight %}
The difference here is `password` is not required (because we use LDAP first, then local, and with LDAP system, password is stored in the cloud).  I also removed the `toJSON()` method.  Finally, in the `beforeCreate()` method, I made sure that there was a `user.password` before bcrypting.  In an LDAP system, since we have no password, there would be nothing to bcrypt.  

3: Here's what I ended up doing:
{% highlight javascript linenos %}
var passport = require('passport');

module.exports = {
  _config: {
    actions: false,
    shortcuts: false,
    rest: false
  },
  
  getLogin:function(req,res){
    res.view('login',{
      message:req.flash ? req.flash('error') : "",
      layout:'home-layout'
    });
  },

  postLogin: function(req, res) {
    passport.authenticate(['ldapauth','local'], {
      successRedirect:'/',
			failureRedirect:'/login',
			failureFlash:true
		})(req, res);
  },

  logout: function(req, res) {
    req.logout();
    res.redirect('/');
  }
};
{% endhighlight %}
First, `passport.authenticate` can take more than one type of authentication method.  In our case, we needed to do 'ldapauth' then 'local'.  We route this to `postLogin`.  If a successful login, we redirect to root.  Otherwise we redirect to login again.  
We add `getLogin` which is different in the other tutorial.  This is so we can display an error message, like `invalid username/pass`.  

4: Login view is pretty much the same.  Difference here is we add `<%= message %>` to echo the error message.  We no longer have a signup form, since this should be done IT side.  Local, we will add users manually if we need this.  
Here is the code:  
{% highlight html linenos %}
<h1>Login</h1>
<p><%= message %></p>
<form method="post" action="/login">
  <input type="input" name="username" placeholder="username">
  <input type="password" name="password" placeholder="password">
  <input type="submit" value="submit">
</form>
{% endhighlight %}
5: Not much to routes.  Here is the code: 
{% highlight javascript linenos %}
'get /login':'AuthController.getLogin',
'post /login':'AuthController.postLogin',
...
{% endhighlight %}
6: Nothing to change/add here.

7: This step strays the most from the tutorial.  First, in the config file, we make a file called `passport.js`, which acts as our config for passport.  It looks like this:
{% highlight javascript linenos %} 
module.exports.passport = {
  local:{
    usernameField: 'email',
    passwordField: 'password'
  },
  
  ldapauth:{
    server:{
      url: process.env.LDAP_URL,
      bindDn:process.env.LDAP_BIND_DN,
      bindCredentials:process.env.LDAP_BIND_CREDENTIALS,
      searchBase: 'dc=rootiverse, dc=net',
      searchFilter: '(mailnickname={{username}})'
    }
  }
}
{% endhighlight %}
This file holds the `config` information for our passport settings.  If you need a local config file, simply add a `local.js` to the config folder with your passport settings.  

Next, which is what the tutorial originally had in the `config/passport.js` file, we moved to `services/passport.js`.  That looks like this:  
{% highlight javascript linenos %}
var passport = require('passport'),
LocalStrategy = require('passport-local').Strategy,
LdapStrategy = require('passport-ldapauth').Strategy,
bcrypt = require('bcrypt-nodejs');

passport.serializeUser(function(user, done) {
  done(null, user.id);
});

passport.deserializeUser(function(id, done) {
  Users.findOne({ id: id } , function (err, user) {
    done(err, user);
  });
});

passport.use(new LdapStrategy(sails.config.passport.ldapauth,function(rootUser,done){
  var email = rootUser.mail.toLowerCase();

  Users.findOrCreate({email:email},{
    email:email,
    firstName:rootUser.givenName,
    lastName:rootUser.sn
  },done);
}));

passport.use(new LocalStrategy(sails.config.passport.local,function(email, password, done) {
  Users.findOne({ email: email }, function (err, user) {
    if (err || !user)
    { 
      return done(err,false);
    }

    bcrypt.compare(password, user.password, function (err, res) {
      if (err || !res)
      {
        return done(err, false);
      }
      else
      {
        return done(null, user);
      }
    });
  });
}));
{% endhighlight %}
We call Passport's `serializeUser` and `deserializeUser` functions.  After that, simply use the `LdapStrategy` and `LocalStrategy` in passport's `use` function.  In the callbacks, do your own server's logic to authenicate/create a user.  

8: Nothing to change/add here.  

9: This part is a little tricky because it is more custom to your app.  Learn more about it [link2][here]:

This is what we ended up using:  
{% highlight javascript linenos %}
'*': ['passport'],

'AuthController': {
  '*': true
},
 
'ProjectsController':{
  'css':true,
  'exportTemplates':true,
  'exportPage':true,
  'preloader':true
}
{% endhighlight %}
Essentially, all routes need to run passport policy (of step 8) first `'*': ['passport']`.  If you are using the `AuthController`, all routes can exclude the passport policy: `'*': true`.  If you are using the `ProjectsController`, css, exportTemplates, exportPage and preloader exclude the passport policy.

That is it!  With that configuration, we were able to get LDAP working in a Sails app using passport.

[link1]: http://www.iliketomatoes.com/implement-passport-js-authentication-with-sails-js-0-10-2/
[link2]: http://sailsjs.org/documentation/concepts/policies
