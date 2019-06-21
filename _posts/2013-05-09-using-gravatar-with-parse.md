---
layout: post
title: Using Gravatar with Parse
date: 2013-05-09 20:35 -0700
---

Gravatar is an service that allows users to specify a public avatar image linked to an email address. I recently decided to allow users to use their Gravatar avatar with my Parse-backed app rather than build in my own avatar system. To simplify things, I've hooked beforeSave for the User object to hash the email address and add it to the object. That ensures that any email address change is automatically hashed on the server and allows me to send the hash down to clients rather than the email address to protect privacy.

```
function MD5(s){ /* one of many publicly available MD5 functions */ }

Parse.Cloud.beforeSave(Parse.User, function(request, response) {
    var email = request.object.get("email");
    request.object.set("emailHash", email? MD5(email) : "");
    response.success();
});
```