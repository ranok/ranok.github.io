---
layout: post
title:  "Deferring execution in Cloudflare Workers (Python)"
date:   2026-05-18 10:40:00
categories: cloudflare, worker, python
---
I'm a big fan of the developer experience of Cloudflare Workers. They reduce the friction to get something online and scaling is seamless. I have an existing body of Python code, and I wanted to reuse some of
it in a Worker, which now offers beta support for the language. While the docs state that all of the Runtime APIs from Javascript are supported, there are some documentation gaps.

One of my favorite Workers features is the ability to defer the resolution of some Promises after the initial HTTP request has been responded to. In Javascript, the `ctx.waitUntil()` function takes a Promise
which the Worker will await for up to 30 seconds after the `fetch` function completes. In Python, there is no documentation of how to use this, but it Just Works (TM). Using `self.ctx.waitUntil()` in Python
with an argument that's a function call to an `async` function will be deferred without any other wrangling of Python <-> JS.

