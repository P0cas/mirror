---
layout: post
title: "0-Day, Copy and Paste ReDoS in github.com"
description: The article is about 0-day, ReDos vulnerability in github.com. ReDoS vulnerability is a type of DoS vulnerability that occurs within a regular expression engine. This vulnerability occurred in the paste-markdown module on GitHub.
date: 2023-08-05 09:00:53
logo: code
---
## Summary
In summary, I found a Self-ReDoS vulnerability in the issue feature of github.com and reported it to HackerOne.
<hr>

## Analysis

```javascript
function onPaste$1(event) {
    const { currentTarget: el } = event;
    if (shouldSkipFormatting(el))
        return;
    if (!event.clipboardData)
        return;
    const textToPaste = generateText(event.clipboardData);
    if (!textToPaste)
        return;
    event.stopPropagation();
    event.preventDefault();
    const field = event.currentTarget;
    if (field instanceof HTMLTextAreaElement) {
        insertText(field, textToPaste);
    }
}
```
Once the user performs a paste action, the onPaste$1() function is executed first. Upon examining the code of this function, you can observe that it calls the generateText() function, passing the ClipboardData as an argument.

```javascript
function generateText(transfer) {
    if (Array.from(transfer.types).indexOf('text/html') === -1)
        return;
    const html = transfer.getData('text/html');
    if (!/<table/i.test(html))
        return;
    const parser = new DOMParser();
    const parsedDocument = parser.parseFromString(html, 'text/html');
    let table = parsedDocument.querySelector('table');
    table = !table || table.closest('[data-paste-markdown-skip]') ? null : table;
    if (!table)
        return;
    const formattedTable = tableMarkdown(table);
    return html.replace(/<meta.*?>/, '').replace(/<table[.\S\s]*<\/table>/, `\n${formattedTable}`);
}
```
In the generateText() function, the first logic checks the type of the received Clipboard data. If the type is 'text/html', the function immediately returns. However, since the type is divided using Array.from(), it bypasses the aforementioned filtering. Additionally, the Clipboard data must contain the value \`&lt;table\` to meet the condition. Once these conditions are met, the function uses DomParser to parse the Clipboard data and checks if there is a table tag within the parsed Element. Subsequently, the function employs replace() method to replace the meta tags in the html variable with an empty value.

### First Redos gadget
```javascript
/<meta.*?>/
```
However, looking at the regular expression used here, it appears vulnerable to ReDos.

![](https://media.discordapp.net/attachments/962997469757702177/1131356373108662293/2023-07-20_00.46.13.png?width=1264&height=938)
Indeed, when I tried using ReDos Checker, I was able to confirm that it is vulnerable. 

According to the above, performing an infinite check on the value `<meta` causes ReDoS (Regular Expression Denial of Service). However, when using .getData('text/html') to retrieve values, it automatically adds the value `<meta charset='utf-8'>` to the beginning of the content, preventing ReDoS from occurring.

### Second Redos gadget

```javascript
/<table[.\S\s]*<\/table>/
```
However, the second regular expression is also vulnerable to ReDoS, so I have decided to perform a ReDoS attack using it. According to the aforementioned report, the parsed data must contain the FORM tag element unconditionally.

The ReDoS payload will be written as `<table<table<table<table` and similar patterns. However, since this is not a valid table tag, it cannot bypass the IF statement. But if we close it with `</table>` and then use `<table>` to insert the table tag element, ReDoS will not occur. Therefore, to bypass this logic, I have inserted an arbitrary `<div>` tag to add the `<table>` tag.

```javascript
'<table'.repeat(99999) + '<div><table></div>'
```
So, the payload has been created as described above

### 0-day validation
<style>
    .myvideo {
        width: auto;
        max-width: 100%;
        height: auto;
    }
</style>
<video class="myvideo" controls="" preload="">
    <source src="https://cdn.discordapp.com/attachments/962997469757702177/1137343390254637158/ReDos_-_v2.mov" type="video/mp4">
</video>

Now, it is confirmed that ReDoS is functioning properly. By applying this payload in a GitHub issue, it should work as intended. The exploit is based on the table tag rather than the meta tag, enabling the execution of a 0-day ReDoS attack.

---
## Applying 0-Day to Github.com
![](https://cdn.discordapp.com/attachments/962997469757702177/1137344178787987506/bf-2.png)

For this test, I have set up the breakpoint as described above.
If ReDoS does not occur within the generateText() function, the code execution will proceed directly without any delay from line 352. However, if ReDoS occurs within the generateText() function, due to the DoS effect, line 352 will not be executed immediately after line 351.

<video class="myvideo" controls="" preload="">
    <source src="https://drive.google.com/u/0/uc?id=1NZBdhqwVXubTxENytjhvtC6R75DXu-AN&export=download" type="video/mp4">
</video>

Let's watch the video!

In the first scenario, the result shows the insertion of a non-vulnerable payload, where ReDoS does not occur within the generateText() function. As a result, the code execution proceeds smoothly without any delays, starting from line 352.

In the second scenario, the result displays the insertion of a ReDoS payload. ReDoS occurs within the generateText() function, causing a significant delay in that section of the code. As seen in the video, generateText() function stays for an extended period.

---
## Reference
- [Hackerone Report#2076605](https://hackerone.com/reports/2076605)
- [Hackerone Add Report#2087681](https://hackerone.com/reports/2087681)
- [Github Ticket#2270535](https://support.github.com/ticket/2270535)
- [Pull Request](https://github.com/github/paste-markdown/pull/89)