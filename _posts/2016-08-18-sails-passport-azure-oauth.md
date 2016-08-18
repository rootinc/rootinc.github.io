---
layout: post
title:  "Sails Passport Azure OAuth"
date:   2016-08-18 17:00:00
author: "Dan Jewett"
categories: javascript sailsjs passport azure-oauth
published: true
---
This tutorial is based on this [link][link1]:

There are a couple things we need to do in addition to the tutorial previously made a few months ago.  I will point out the differences here:

1: First, make sure you `npm install passport-azure-oath` and update `package.json` to reflect this.  You can remove and npm uninstall `passport-ldap`.

2: Next, we need to set up an app with M$ Office 365 Dev Tools.  Let's go [here][link2].  You will need to login with your root credentials and allow the M$ dev app.  From there, use the form -- it's pretty easy.  Redirect URI was a little tricky for me, in that it should be the route to your app that callbacks the login request.  I ended up using the same as you will see in a few steps.

3: Here's what I ended up changing in `AuthController.js`:
{% highlight javascript linenos %}
  ..
  localcallback: function(req, res, next) {    
    passport.authenticate('local', {
      successRedirect:'/',
  		failureRedirect:'/login',
  		failureFlash:true
  	})(req, res);
  },
    
  azurecallback: function(req, res, next){
    passport.authenticate('azureoauth', {
      successRedirect:'/',
  		failureRedirect:'/login',
  		failureFlash:true
  	})(req, res);
  },
  ..
{% endhighlight %}
Notice I now have a localcallback and azurecallback.  These functions are essentially the posts/gets now.  On local post, we use the localcallback, and on azurepost, we use azurecallback.  If a response comes back from M$, we use the azurecallback to essentially "retry". 

4: The passport service (`passport.js`) changed quite a bit.
Here is the code:  
{% highlight html linenos %}
var passport = require('passport'),
LocalStrategy = require('passport-local').Strategy,
AzureStrategy = require('passport-azure-oauth').Strategy,
bcrypt = require('bcrypt-nodejs');

passport.serializeUser(function(user, done) {
  done(null, user.id);
});

passport.deserializeUser(function(id, done) {
  Users.findOne({ id: id } , function (err, user) {
    done(err, user);
  });
});

passport.use(new AzureStrategy(sails.config.passport.azureoauth,function(accessToken, refreshToken, profile, done){  
  var email = profile.rawObject.unique_name.toLowerCase();

  Users.findOrCreate({email:email},{
    email:email,
    firstName:profile.rawObject.given_name,
    lastName:profile.rawObject.family_name
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
The biggest thing to pont out is that we are using the AzureStrategy now, instead of the ldap.  The data in the profile object is slightly different than what was coming back from ldap, so tak enote of that.

5: `Passport.js config`
{% highlight javascript linenos %}
  ..
  azureoauth:{
    clientId: process.env.AZUREOAUTH_CLIENTID,
    clientSecret: process.env.AZUREOAUTH_CLIENTSECRET,
    tenantId:"*************",
    resource: "https://graph.windows.net"
  }
  ..
{% endhighlight %}
Removed the ldap config, and added ldap config.  Note, from step one, here is where your clientId and clientSecret go.

6: `Route.js`
{% highlight javascript linenos %}
  ..
  'post /login/local':'AuthController.localcallback',
  'post /login/azure':'Authcontroller.azurecallback',
  
  'get /login/azurecallback':'Authcontroller.azurecallback',
  ..
{% endhighlight %}
This structure changed a lot -- in favor of how a lot of other passport services work -- which is to use a callback method.  Again, I am using the same function in `AuthController` as they do the same thing.

7: The login view slightly changed -- mainly to post to our new routes.  Here is the code:
{% highlight javascript linenos %} 
<h1>Login</h1>
<p><%= message %></p>
<h2>Root Login</h2>
<form method="post" action="/login/azure">
  <input type="submit" value="Connect with Office 365">
</form>
<div>OR</div>
<h2>Outside Login</h2>
<form method="post" action="/login/local">
  <input type="input" name="username" placeholder="username">
  <input type="password" name="password" placeholder="password">
  <input type="submit" value="submit">
</form>
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

9: This part is a little tricky because it is more custom to your app.  Learn more about it [here][link2]:

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

[link1]: http://rootinc.github.io/2016/04/28/sails-passport-ldap/
[link2]: https://dev.office.com/app-registration
