---
layout: post
title:  "HTTP signature validation in AWS Lambda"
date:   2024-03-05 15:55:04
categories: aws, lambda, python
---
I was recently implementing a webhook handler that processed fairly sensitive data, and the HTTP POST request was signed using an HMAC SHA256 algorithm. The general design was such that both the sending and receiving code had a shared secret, the sender would generate a timestamp (added to HTTP headers), and concatenate that timestamp POST body, then put the SHA256 digest into another header.

In order to validate this, I had to perform the reverse, parse out the signature and timestamp from the headers, pull out the body, and then calculate the HMAC to validate the request. I ran into an issue where in a Python3 Lambda, the `event` object passed to the handler
from the HTTP request (via Function URL) would come pre-processed as a dict, and converting it back to a string would change the format slightly, thus invalidating the signature.

To get around this, I set up an API Gateway trigger for the Lambda, and in the API Gateway settings I marked `application/json` as a binary format. This did not cause the gateway to base64 encode the request body, but now it was treated as a string and I was able to successfully validate the HMAC signature.
