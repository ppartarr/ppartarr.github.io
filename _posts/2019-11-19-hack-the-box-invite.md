---
layout: post
title:  "Hack the box invitation"
date:   2019-11-19 21:18:09 +0200
categories: blog
---

This post is the first in of a series in which I attempt to solve [Hack The Box](https://www.hackthebox.eu/) challenges and track my progress. The idea is to note down my thought process in solving the challenges, including my failed attemps. Beware that these blog posts will contain spoilers to the challenges so be sure to attempt to solve them yourself before you read on.

So without further ado, here is the solution to the invite challenge:

I thought the challenge was going to be trivial and I fully expected the invite code to be written as a comment in the html. The first thing I did was to open up the developer console in my browser. And in there I found not an invite code, but a token... Hum... Clearly not as trivial as I originially thought.

![hidden input](/assets/images/hack-the-box-invite.png)

I assumed that this token has to be sent to an API and exchanged for a code, so I started looking at the HTML form. I decided to check the debugger view of the firefox developer tools and in there I find **inviteapi.min.js** - looks like I found what I'm looking for. The code is obfuscated, so I copy the javascript and submit it to the first javascript deobfuscation website I can find, which returns the following code:

```javascript
'use strict';
/**
 * @param {string} code
 * @return {undefined}
 */
function verifyInviteCode(code) {
  var formData = {
    "code" : code
  };
  $.ajax({
    type : "POST",
    dataType : "json",
    data : formData,
    url : "/api/invite/verify",
    success : function(response) {
      console.log(response);
    },
    error : function(response) {
      console.log(response);
    }
  });
}
/**
 * @return {undefined}
 */
function makeInviteCode() {
  $.ajax({
    type : "POST",
    dataType : "json",
    url : "/api/invite/how/to/generate",
    success : function(response) {
      console.log(response);
    },
    error : function(response) {
      console.log(response);
    }
  });
}
;
```

The `makeInviteCode` is the function I'm interested in. Now let's try and send a POST request using curl to that endpoint
```bash
$ curl -X POST https://www.hackthebox.eu/api/invite/how/to/generate -d "UQmSFCHauxO4Fb16mdsuWYoM0Avob3S6BdP48HbE"
{"success":1,"data":{"data":"SW4gb3JkZXIgdG8gZ2VuZXJhdGUgdGhlIGludml0ZSBjb2RlLCBtYWtlIGEgUE9TVCByZXF1ZXN0IHRvIC9hcGkvaW52aXRlL2dlbmVyYXRl","enctype":"BASE64"},"0":200}%
```

Looks like I have a base64 encrypted string. Let's decrypt that.

```bash
$ echo "SW4gb3JkZXIgdG8gZ2VuZXJhdGUgdGhlIGludml0ZSBjb2RlLCBtYWtlIGEgUE9TVCByZXF1ZXN0IHRvIC9hcGkvaW52aXRlL2dlbmVyYXRl" | base64 -d
In order to generate the invite code, make a POST request to /api/invite/generate%
```

```bash
$ curl -X POST https://www.hackthebox.eu/api/invite/generate
{"success":1,"data":{"code":"QldORk4tWEpZVlktSUJHWE0tUkxCRFEtUUhGWkU=","format":"encoded"},"0":200}%
```

Ah ha, there is the code. I copy paste it into the input field. And I get an error. I look at the curl response again and see that the format is encoded, so I decode it using base64 again, paste the code into the input field and carry on to create my account!

```bash
$ curl -X POST https://www.hackthebox.eu/api/invite/generate
{"success":1,"data":{"code":"T1ZCR04tTkpLWkItQ0JWT1YtTFFQU0stRE5WQVM=","format":"encoded"},"0":200}%

$ echo "T1ZCR04tTkpLWkItQ0JWT1YtTFFQU0stRE5WQVM=" | base64 -d
OVBGN-NJKZB-CBVOV-LQPSK-DNVAS%
```