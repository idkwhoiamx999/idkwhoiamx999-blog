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


#### Vulnerable backend logic:


```javascript
const user = await findUserByEmail(email);
if (!user) {
    return "Email not found";
}
if (!verifyPassword(password)) {
    return "Password incorrect";
}
```


#### Secure backend logic:


```javascript
const user = await findUserByEmail(email);
if (!user || !verifyPassword(password)) {
    return "Invalid email or password";
}
```

So this should be info we have to get more impact.

----


Now let’s dig a little deeper.

Since I already knew the email address was valid, I started thinking about brute-forcing the password. At first, I noticed there was a CAPTCHA, so I almost gave up on the idea because I assumed it would stop automated attacks. But I decided to test it anyway.


I entered a valid email address with an incorrect password, completed the CAPTCHA, and repeatedly clicked the Login button to observe the application’s behavior. I wanted to see whether it would trigger rate limiting, ask me to solve another CAPTCHA, temporarily lock the account, or introduce any delay after multiple failed attempts.

After around 10 to 15 failed login attempts, nothing happened. There were no rate-limit errors, no additional CAPTCHA challenges, and no account lockout. To confirm my observations, I entered the correct password using the same session and was immediately logged in. At this point, it became clear that the CAPTCHA protection was ineffective against repeated authentication attempts so good i didnt even need to try bypass it more time saved haha.

The next step was to intercept the login request using Burp Suite and send it to Intruder. I configured a password wordlist and intentionally placed the correct password at the end of the list. Burp successfully sent thousands of authentication requests, and every single one was processed by the server without any rate limiting or account lockout. When the final request containing the correct password was reached, the application authenticated the session successfully, confirming that the login endpoint accepted unlimited authentication attempts.

Good lets go deep more.

----


After confirming that the issue affected the main login page, I wanted to determine whether it was limited to a single endpoint or part of a broader authentication implementation.

I began searching for additional login portals across <mark>*.example.com</mark> To discover them, I used a combination of
**Google Dorks for ex :**


> site:example.com inurl:login

> site:example.com inurl:signin

> site:example.com inurl:auth

> site:example.com “Sign In”

> site:example.com “Log In”


I used GoSpider to crawl the target and discover additional endpoints. You can find it here: 


## [Gospider](https://github.com/jaeles-project/gospider)


so Google Dorks helped me identify publicly indexed authentication-related endpoints, while GoSpider was used to crawl the target and enumerate additional URLs and endpoints that were not immediately visible.

Once I had a list of potential login endpoints, I tested each authentication portal individually. To my surprise, several of them exhibited the same behavior, indicating that the issue was not isolated to a single login page but affected multiple authentication portals across the application.


----


At this point, I had gathered everything I needed. It was time to document my findings, write the report, and submit it to the security team.


<img src='https://raw.githubusercontent.com/idkwhoiamx999/idkwhoiamx999-blog/refs/heads/main/images/post1/imagepost1.png' alt='firstImage'>


> #### The reporte get triaged very fast it was my favorite programe <3


The report was initially classified as **Medium** severity. Based on my findings and the potential impact, I believed the issue warranted a higher severity rating, so I continued analyzing the authentication flow and discussing the findings with the security team.


<img src='https://raw.githubusercontent.com/idkwhoiamx999/idkwhoiamx999-blog/refs/heads/main/images/post1/imagepost11.png' alt='secondeImage'>


So, I decided to dig even deeper. I had a feeling there was more to uncover, so I continued searching for additional endpoints. After testing a few more authentication portals, I finally came across another vulnerable endpoint.



> ### https://admin-portal-example.com/login


And there it was — a vulnerable endpoint. I immediately updated my report with the new findings and sent it to the security team. After reviewing the additional evidence, they upgraded the severity from **Medium** to **Critical.**



<img src='https://raw.githubusercontent.com/idkwhoiamx999/idkwhoiamx999-blog/refs/heads/main/images/post1/imagepost111.png' alt='thirdImage'>


> ### Finaly ❤

----


And that’s the end of this write-up!

I hope you enjoyed reading it and found it useful. This is just the beginning — I have plenty of interesting research, CTFs, bug bounty stories, and security write-ups that I’ll be sharing over the coming weeks, so stay tuned.

If you enjoyed this article, feel free to share it and tag me on X. I’d love to hear your thoughts, answer your questions, or discuss different approaches to the findings.

Thanks for reading, and I’ll see you in the next write-up. 🚀

---

## links:

https://x.com/idkwhoiamx999

https://bsky.app/profile/ykx999.bsky.social

