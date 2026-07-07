+++
author = "idkwhoiamx999"
title = "When Missing Rate Limiting Leads to a Critical Authentication Finding: A Real-World Case Study"
date = "2019-03-09"
description = "Chain Lead to a Critical Authentication Finding: A Real-World Case Study"
tags = [
    "markdown",
    "text",
]
weight = 10
+++


Hey It’s been quite a while since my last article, but I’m excited to start publishing consistently again, covering web application security, bug bounty research, CTFs, penetration testing, and the lessons I learn along the way.

Let’s start with one of the most interesting authentication issues I’ve ever reported.


<img src='https://miro.medium.com/v2/resize:fit:720/format:webp/1*McB2hI75sOM7gMXR3_zc7g.jpeg' alt='sss'>

So, let’s talk about the critical vulnerability I found in what we’ll call <mark>example.com</mark>

I started by testing the login functionality of the main application. Since I had only discovered the login page about 10 minutes earlier, I simply began interacting with it, clicking through the functionality and looking for anything interesting.

I created a new account and logged in successfully. My first test was to enter a valid email address with an incorrect password.


| Email  | Password |
| ----- | --- |
| hunterxhunter_@wearehackerone.com   | 1234567  |


The application responded with:

> Password is incorrect.

That immediately caught my attention. Instead of stopping there, I started wondering, “What happens if the email doesn’t exist?”

To test this, I slightly modified the email address by adding a few extra characters while keeping the same password.


| Email  | Password |
| ----- | --- |
| hunterxhunter123_@wearehackerone.com   | 1234567  |


This time, the application responded with:


> This email is not registered.


At this point, I confirmed that the login endpoint returned different error messages depending on whether the supplied email address existed. This allowed an attacker to enumerate valid accounts simply by observing the authentication responses.

The root cause of this issue was improper backend authentication error handling. Instead of returning a generic response such as “Invalid email or password” for all failed login attempts, the backend disclosed whether an email address was registered, resulting in an account enumeration vulnerability.

