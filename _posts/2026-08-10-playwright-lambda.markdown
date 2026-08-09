---
layout: post
title:  "Running Containerized Playwright in AWS AWS Lambda"
date:   2026-08-09 10:40:00
categories: aws, playwright, lambda, python
---

When trying to take a containerized Playwright script and run it in AWS Lambda, I was running into immediate target crashes, with even additional memory configured.
I eventually found [this](https://stackoverflow.com/questions/78796910/playwright-browser-launch-issue) StackOverflow question and after tinkering with the flags I found
that the following `launch` command worked in AWS Lambda:

`chromium.launch(headless=True, args=['--no-sandbox', '--disable-gpu', '--single-process', '--no-zygote', '--disable-setuid-sandbox'])`

