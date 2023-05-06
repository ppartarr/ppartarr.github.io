---
layout: post
title:  "Go Error Handling"
date:   2021-01-22 10:40:010 +0200
categories: blog
---

In Go, it well known that [errors are values](https://blog.golang.org/errors-are-values). When I first started programming in Go I was dissatisfied at the number of times I was writing error handling blocks like:
```go
if err != nil {
    // do something with error
    log.Println(err.Error())
}
```

<!--more-->

Although I could use some syntactic sugar to make a one-liner I figured it would be much more fun to have a keyboard binding that simpy inputs the error handling text.

My solution for this is [keypress](https://github.com/ppartarr/keypress).

I simply press the windows key & b simultaneously and the [uinput](https://www.kernel.org/doc/html/v4.12/input/uinput.html) linux kernel module does the rest!

