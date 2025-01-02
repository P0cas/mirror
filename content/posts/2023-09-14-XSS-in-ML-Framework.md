---
layout: post
title: "($10,890) What is XSS in the ML/AI ecosystem, not only web3.0?"
description: The article is about how i have got $10k via xss vulnerability in the ML/AI ecosystem. i am sure that many researcher on web2.0 found the xss vulnerability but not all guys gets big bounty for xss. so today i wanna introduce what we need to do after finding xss. Actually every bug can give you guys nice bounty >  even if it's xss
date: 2023-09-14 02:00:53
logo: code
---

Hi guys, i wanna introduce a way to earn $10k bounty on an open-source projects. a month ago, i reported a bug in the ML Framework. and the bug i found was not critical so i needed to find a way to escalate it low-severity to high-severity. first of all, the bug i found was xss. i know that most people think that is not the bug which has high severity but me and others know a way to exploit scenario via low-severity bug.

- Account Takeover
- CSRF
- SSRF

First of all, they can exploit to ato, csrf, ssrf, rce with xss when some researcher found an xss. However in real, if you succeeded in other than major companies like google, apple, facebook, shopify, you cannot earn substantial rewards from them with high percentage. In general, client-side bugs don't yield high bounty but this changes significantly when the target is operates under the web3.0 or AI/ML

In the web ecosystem on the present era, web services are being developed under the influence of Web 3.0 and AI/ML technologies. web 3.0 services are primarily focused on areas like [cryptocurrency](https://en.wikipedia.org/wiki/Cryptocurrency) and [NFT](https://en.wikipedia.org/wiki/Non-fungible_token) exchanges and the AI/ML is typically conversational like [ChatGPT](https://en.wikipedia.org/wiki/ChatGPT), [Bing AI Chatbot](https://www.bing.com/new), etc. that means you can earn more bounty in web3.0 or AI/ML services than when you find a bug in modern web services

Also, many exchanges manage bug bounty programs and pay higher bounty compared to modern web services. that is why many researchers focus on bug bounty hunting in cryptocurrency exchanges. on the other hand, i have rarely seen people doing bug bounty work on AI/ML frameworks. Based on what i mentioned earlier, this means we can earn high bounty not only in Web 3.0 services but also in AI/ML services even if you found a bug like xss.

Returning to my story, i actually found an XSS in an ML framework. however, i couldn't earn a high bounty just by reporting the bug because it lacked impact. but the ML model's data was saved in the web service, so  tried to find another bug to link it with the XSS. after diving back into it for three hours, i discovered an LFI bug and linked it with the XSS. then, i reported it again, and this time, they accepted the bug for a high bounty! i also recommend that you participate in bug bounty programs not only for general web services but also for these types of services. thx sm

*do you guys think there is a better bug than XSS for exploitation?*