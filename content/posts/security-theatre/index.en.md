---
title: "Security Theatre? More like Security Circus"
date: 2023-02-13T20:20:00+01:00
draft: false
tags: [security theatre, infosec, rants]
categories: [general-knowledge]
---

Working in the field, I have seen many organizations invest a significant amount of time and resources (read: people and money) into security measures that provide little to no actual security.
<!--more--> 
I've witnessed organizations perform a never-ending clown show in the name of security. 
This is commonly referred to as "security theatre."

## What is the Security Theatre

Security theatre, in its simplest form, is a grand illusion. It's security measures put in place not because they actually prevent security breaches, but because they look good on paper, pacify stakeholders, and make everyone feel like they're doing something to protect themselves.

[Oh, how I love the smell of compliance in the morning](https://www.youtube.com/watch?v=vRp7tYWnJJs).

Take, for instance, the requirement of having complex passwords that are regularly changed. This, is a classic example of security theatre. 

While this may seem like a good idea on the surface, it actually does little to prevent security breaches. And let's not forget the inevitable result: password fatigue, causing employees to use ones that are easier to remember or simple variations of the one they previously used. There is a reason (multiple, actually) if [Microsoft](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/expansion-of-fido-standard-and-new-updates-for-microsoft/ba-p/3290633), [Google](https://blog.google/technology/safety-security/one-step-closer-to-a-passwordless-future/) and [Apple](https://www.apple.com/newsroom/2022/05/apple-google-and-microsoft-commit-to-expanded-support-for-fido-standard/) want to move away from passwords.

{{< figure src="https://imgs.xkcd.com/comics/password_strength.png" title="fix your password policy">}}

Another example of security theatre is implementing security controls that are not properly configured or maintained. For instance, firewalls, IDS and IPS are a common security measure that are often poorly configured or not kept up-to-date with the latest security patches

## Root cause

So, what can organizations do to avoid this never-ending circus? First and foremost, they need to understand the real threats they face and implement controls that address those specific threats. This must include in fist place educating employees on how to recognize and respond to attacks. 

Cybersecurity after all, is the Gold Rush 2.0, lots of money are on the table and everyone wants to join, so finding skilled individuals is hard and if you hire consultants is even worse because you will fully depend on them for your security posture.

## Wrapping up
In conclusion, security theatre, is a common problem in the IT security world. To avoid being part of the show, organizations must focus on implementing effective controls, regularly assessing their effectiveness, and having a comprehensive response plan in place. And, for the love of god, stop with the pointless policies already!

{{< admonition type=tip title="Tip" open=true >}}
[Here](https://www.philvenables.com/post/ceremonial-security-and-cargo-cults) you can find an interesting article from Phil Venables, Google CISO, about Cerimonial Security and Cargo Cults
{{< /admonition >}}

