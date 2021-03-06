---
layout: post
title: A CSRF Protection Bypass Technique
date: 2015-04-30 18:14:31.000000000 -07:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- AppSec
- CSRF
meta:
  _publicize_pending: '1'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
author:
  login: abhartiya
  email: anshuman.bhartiya@gmail.com
  display_name: abhartiya
  first_name: ''
  last_name: ''
---

This technique can be used to bypass CSRF protections in some applications by using a static CSRF token (for all users of that application) that looks like a specific format string.

So, to begin with, have you ever noticed CSRF tokens being something like this:
```
RHAU3cgmTvWy6RWSj+NdJy2v8y8Z0g2U5qTQg4ap/lqeLEfA==
```

If you have, have you ever looked at it more closely? The above string is basically divided into 3 parts separated by some delimiters. In the above example, the first part `RHAU3cgmTvWy6RWSj` and the second part `NdJy2v8y8Z0g2U5qTQg4ap` are separated by the delimiter `+`. 

The second part `NdJy2v8y8Z0g2U5qTQg4ap` and the third part `lqeLEfA` are separated by the delimiter `/`. 

The string finally ends with `==`.

So, considering the above example, if you encounter an application that uses CSRF tokens as shown above, try fiddling with the actual string making sure you keep the format consistent i.e. 3 parts separated by some delimiters and so on and so forth.

In my case, the format ended up being `xxxxxxxxxxxxxxxxxxxxxxxxx%2Bxxxxxxxxxxxxxxxxxxxxxxxxxx%2Fxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxw%3D%3D` which when decoded is `xxxxxxxxxxxxxxxxxxxxxxxxx+xxxxxxxxxxxxxxxxxxxxxxxxxx/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxw==`. 

The length of the above string (or the number of x's) would depend on different applications. So, consider this as just a PoC. In a nutshell, what I observed was that an attacker can just trick a victim in order to submit a POST request with the above string as the csrf token as a POST parameter and the application server would gladly accept it because it was only looking to ensure the tokens met a specific format and didn't really compare the actual value received to the value stored on the server side. As a result of this, I was able to bypass CSRF protections throughout the entire application.

There are some more nuances to the above scenario as well. Consider the case of a double-submit CSRF protection. What that entails is that the CSRF tokens need to be sent in two places - one as a session cookie and one in the POST body. Or, maybe one as a custom header and one in the session cookie. Or, maybe one as a custom header and one in the POST body. There can be multiple possible combinations.

The jist is that they both need to be the same. This is mostly done to prevent the headache of storing the CSRF values on the server side. In such cases, bypassing the protection is not easy because as an attacker, you don't really have any control over a victim's browser to be able to set custom headers or session cookies. The most you can do is to trick a victim in order to submit a malicious POST request. But, since the browser sends the headers and/or cookies automatically, the chances of those values matching your value in the POST request are negligible. Hence, the protection, if implemented properly, can be quite effective.

But, when you consider the example discussed above, it was observed that even though the browser was sending a custom header and/or session cookie automagically along with the attacker tricked value `xxxxxxxxxxxxxxxxxxxxxxxxx%2Bxxxxxxxxxxxxxxxxxxxxxxxxxx%2Fxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxw%3D%3D` in the POST body, the server was only looking to ensure that the format matched and not the actual values. So, again, this was a complete CSRF protection bypass because it didn't matter what CSRF values the browser was sending (as headers and/or cookies) as long as an attacker could trick a victim to submit a POST request with the above static CSRF string.

I am not sure if this technique was already known. If it was, pardon my ignorance. I found this during testing and thought it was pretty interesting hence this post.

Cheers!
