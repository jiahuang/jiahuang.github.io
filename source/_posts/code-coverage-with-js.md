---
title: Using Istanbul for code coverage
date: 2016-12-10 20:27:00
tags:
---

I've been using [Istanbul](https://github.com/gotwarlost/istanbul), a JS [code coverage](https://en.wikipedia.org/wiki/Code_coverage) library. Istanbul outputs a report that shows which lines are hit during unit tests:

![istanbul report](/images/istanbul-overview.png)

Istanbul can be installed with:
```
npm install -g istanbul
```

## How Istanbul works
Istanbul wraps all your code with functions that track how often each function, line, and branch in the code got hit while running unit tests. It stores this data in a `coverage.json` file which can then be used to generate html, lcov, or other kinds of reports.

The process of wrapping code with tracking functions is called "instrumentation". Istanbul relies on the [esprima](http://esprima.org/) package to parse JS source code. Then [escodegen](https://github.com/estools/escodegen) is used to add in the coverage code and to output a new instrumented source file.

Given a source file `person.js` that looks like:

```js
function Person(name, dob) {
  this.name = name;
  this.dob = dob;
}

Person.prototype.isMinor = function () {
  var currDate = new Date();
  if ((currDate.getTime() - this.dob.getTime())/(1000 * 60 * 60 * 24 * 365.25) < 18.0) {
    return true
  } else {
    return false
  }
}

module.exports = Person;
```

Istanbul can create the instrumented code via: `istanbul instrument --no-compact -o tmp/instrument/person.js person.js`. This creates a instrumented `person.js` file in `tmp/instrument/person.js`.

The `tmp/instrument/person.js` file looks like this. I added some comments to detail what is happening:

```js
// a temp __cov_X9N6pGXtRbutT8pzNmQHeA obj is created
var __cov_X9N6pGXtRbutT8pzNmQHeA = (Function('return this'))();

// there's a __coverage__ obj that is used to track coverage. The name of this object can change depending on settings, but __coverage is the default.
if (!__cov_X9N6pGXtRbutT8pzNmQHeA.__coverage__) { __cov_X9N6pGXtRbutT8pzNmQHeA.__coverage__ = {}; }
__cov_X9N6pGXtRbutT8pzNmQHeA = __cov_X9N6pGXtRbutT8pzNmQHeA.__coverage__;
if (!(__cov_X9N6pGXtRbutT8pzNmQHeA['/Users/jia/code/jiahuang.github.io/person.js'])) {

  // the initialized __coverage__ obj. This already has details for all of the main source code (such as line numbers, functions, and branches that have been tested)
   __cov_X9N6pGXtRbutT8pzNmQHeA['/Users/jia/code/jiahuang.github.io/person.js'] = {"path":"/Users/jia/code/jiahuang.github.io/person.js","s":{"1":1,"2":0,"3":0,"4":0,"5":0,"6":0,"7":0,"8":0},"b":{"1":[0,0]},"f":{"1":0,"2":0},"fnMap":{"1":{"name":"Person","line":1,"loc":{"start":{"line":1,"column":0},"end":{"line":1,"column":27}}},"2":{"name":"(anonymous_2)","line":6,"loc":{"start":{"line":6,"column":27},"end":{"line":6,"column":39}}}},"statementMap":{"1":{"start":{"line":1,"column":0},"end":{"line":4,"column":1}},"2":{"start":{"line":2,"column":2},"end":{"line":2,"column":19}},"3":{"start":{"line":3,"column":2},"end":{"line":3,"column":17}},"4":{"start":{"line":6,"column":0},"end":{"line":13,"column":1}},"5":{"start":{"line":7,"column":2},"end":{"line":7,"column":28}},"6":{"start":{"line":8,"column":2},"end":{"line":12,"column":3}},"7":{"start":{"line":9,"column":4},"end":{"line":9,"column":15}},"8":{"start":{"line":11,"column":4},"end":{"line":11,"column":16}}},"branchMap":{"1":{"line":8,"type":"if","locations":[{"start":{"line":8,"column":2},"end":{"line":8,"column":2}},{"start":{"line":8,"column":2},"end":{"line":8,"column":2}}]}}};
}

__cov_X9N6pGXtRbutT8pzNmQHeA = __cov_X9N6pGXtRbutT8pzNmQHeA['/Users/jia/code/jiahuang.github.io/person.js'];
function Person(name, dob) {
    // when this function is called, the number of times this function is called gets incremented
    __cov_X9N6pGXtRbutT8pzNmQHeA.f['1']++;
    // and the times this line is called also gets incremented
    __cov_X9N6pGXtRbutT8pzNmQHeA.s['2']++;
    this.name = name;
    __cov_X9N6pGXtRbutT8pzNmQHeA.s['3']++;
    this.dob = dob;
}
__cov_X9N6pGXtRbutT8pzNmQHeA.s['4']++;
Person.prototype.isMinor = function () {
    // similarly these also increment the number of times this function & lines are called
    __cov_X9N6pGXtRbutT8pzNmQHeA.f['2']++;
    __cov_X9N6pGXtRbutT8pzNmQHeA.s['5']++;
    var currDate = new Date();
    __cov_X9N6pGXtRbutT8pzNmQHeA.s['6']++;
    if ((currDate.getTime() - this.dob.getTime()) / (1000 * 60 * 60 * 24 * 365.25) < 18) {
        // because this is in an "if" statement, the number of branches called also gets incremented
        __cov_X9N6pGXtRbutT8pzNmQHeA.b['1'][0]++;
        __cov_X9N6pGXtRbutT8pzNmQHeA.s['7']++;
        return true;
    } else {
        __cov_X9N6pGXtRbutT8pzNmQHeA.b['1'][1]++;
        __cov_X9N6pGXtRbutT8pzNmQHeA.s['8']++;
        return false;
    }
};
```

So as the instrumented code is executed, a tmp object stores all the information about what lines have been called for a particular file. **Note that the `istanbul instrument` command is only useful for browser based tests. I included the instrumented code here just to illustrate how Istanbul keeps track of coverage. For tests that can run on node, the regular `istanbul cover` command automatically instruments the code.**

## Istanbul for Node tests

If we had a [jasmine](https://jasmine.github.io) test file `person.spec.js` like this:

```js
var Person = require('./person');

describe("A person", function() {
  it("can be a minor", function() {
    var p = new Person("me", new Date("2000-03-06"));
    expect(p.isMinor()).toBe(true);
  });
});
```

This can be run with `jasmine person.spec.js`:

```sh
Started
.

1 spec, 0 failures
Finished in 0.005 seconds
```

Now to get the coverage of this file `person.spec.js` run `istanbul cover jasmine person.spec.js` to get the following output:

```sh
Started
.

1 spec, 0 failures
Finished in 0.005 seconds

=============================================================================
Writing coverage object [/Users/jia/code/jiahuang.github.io/coverage/coverage.json]
Writing coverage reports at [/Users/jia/code/jiahuang.github.io/coverage]
=============================================================================

=============================== Coverage summary ===============================
Statements   : 92.86% ( 13/14 )
Branches     : 50% ( 1/2 )
Functions    : 100% ( 4/4 )
Lines        : 92.86% ( 13/14 )
================================================================================
```

The report is generated at `coverage/lcov-report.index.html`:

![person report](/images/istanbul-file.png)

Note that the statement % coverage is very high (88%). This is misleading because the tests only cover half of the branches (50%). Be sure to check coverage % across statements, branches, and functions.

## Istanbul for browser tests

The previous example showed how to run Istanbul on node code. Running Istanbul on the browser requires a little more setup. For this example I'll use the following:

* [PhantomJS](http://phantomjs.org/). Phantom is a headless browser that will allow us to test frontend UI code. [jsdom](https://github.com/tmpvar/jsdom) is another option, however jsdom does not handle things like [hidden/visible elements properly](https://github.com/tmpvar/jsdom/issues/1048).
* [Grunt](http://gruntjs.com/). I'm mainly using Grunt as the static server via [grunt-contrib-connect](https://github.com/gruntjs/grunt-contrib-connect) that can serve up tests for Phantom, and also write down the coverage data.
* [Jasmine](https://jasmine.github.io/) for running the unit tests.
* [grunt-contrib-jasmine](https://github.com/gruntjs/grunt-contrib-jasmine) A Grunt plugin that allows us to run Jasmine tests via phantom.

There are a few steps to get Istanbul tests on the browser:
1. Manually instrument the source code via `istanbul instrument`
2. Make the specs use the instrumented source code instead of the actual code
3. Get the coverage data from `window.__coverage__` after all tests have been run
4. Send the data from `window.__coverage__` to a server so that the coverage data can be saved
5. Run `istanbul report` to generate a report out of the coverage data

The full repo of a working demo is here: https://github.com/jiahuang/istanbul-phantom-jasmine

## NYC

[NYC](https://github.com/istanbuljs/nyc) is the newer Istanbul CLI that works with react & ES6. The problem with getting code coverage for ES6 is that only the babel-transformed version of the code actually runs. So a code coverage tool has to un-babelify the code that was run to get the original source code via a [source map](https://blog.sentry.io/2015/10/29/debuggable-javascript-with-source-maps.html). Istanbul cannot currently do this, but NYC can.

One of the drawbacks of Istanbul is that it only checks files that have been required. There's an [open issue](https://github.com/gotwarlost/istanbul/issues/112) to fix this, but for now a workable solution seems to be just require all files. Being able to see code coverage makes it easier to discover fragile pieces of untested code, and write tests to cover edge cases.
