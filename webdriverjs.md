# Web Driver JS #

## JS ##

###Getting Started###

To get started with WebDriverJS for Node, you will need to download a copy of the ChromeDriver and 
ensure it can be found on your system PATH. All other browsers can be tested using the 
stand-alone Selenium server. Once you've obtained the ChromeDriver and placed it on your 
PATH, you can run your first test:

```javascript
var webdriver = require('selenium-webdriver');

var driver = new webdriver.Builder().
   withCapabilities(webdriver.Capabilities.chrome()).
   build();

driver.get('http://www.google.com');
driver.findElement(webdriver.By.name('q')).sendKeys('webdriver');
driver.findElement(webdriver.By.name('btnG')).click();
driver.wait(function() {
 return driver.getTitle().then(function(title) {
   return title === 'webdriver - Google Search';
 });
}, 1000);

driver.quit();
Reference API Docs

Installing from NPM
% npm install selenium-webdriver
Building from Source
% git clone https://code.google.com/p/selenium/
% cd selenium
% ./go //javascript/node:selenium-webdriver
```

The commands above will clone the Selenium repository and build the selenium-webdriver module. 
This module will be placed in the ./build/javascript/node directory of your Selenium client.

###Using the Stand-alone Selenium Server###

You can manage the life and death of the Selenium server using the selenium-webdriver/remote module:

```js
var webdriver = require('selenium-webdriver'),
    SeleniumServer = require('selenium-webdriver/remote').SeleniumServer;

var server = new SeleniumServer(pathToSeleniumJar, {
  port: 4444
});

server.start();

var driver = new webdriver.Builder().
    usingServer(server.address()).
    withCapabilities(webdriver.Capabilities.firefox()).
    build();
// ...
```

###Writing Tests###

The selenium-webdriver/testing module may be used to write tests that may be run with Mocha:

```js
var assert = require('assert'),
    test = require('selenium-webdriver/testing'),
    webdriver = require('selenium-webdriver');

test.describe('Google Search', function() {
  test.it('should work', function() {
    var driver = new webdriver.Builder().build();

    var searchBox = driver.findElement(webdriver.By.name('q'));
    searchBox.sendKeys('webdriver');
    searchBox.getAttribute('value').then(function(value) {
      assert.equal(value, 'webdriver');
    });

    driver.quit();
  });
});
```

A full example is provided in the selenium-webdriver/example package. You may also find 
selenium-webdriver's self-tests informative:

```js
% npm install mocha selenium-webdriver
% mocha -R list --recursive node_modules/selenium-webdriver/test
```

###Understanding the API###

Unlike the other language bindings, which all provide blocking APIs, WebDriverJS is purely asynchronous. This document describes the libraries used to simplify working with WebDriver and JavaScript's asynchronous event loop.

The simplest asynchronous APIs use basic continuation passing to indicate when an operation has completed. Unfortunately, this can quickly lead to excessively verbose code that is difficult to understand. For instance, consider the canonical WebDriver example, a simple Google search:

```js
driver.get("http://www.google.com");
driver.findElement(By.name("q")).sendKeys("webdriver");
driver.findElement(By.name("btnG")).click();
assertEquals("webdriver - Google Search", driver.getTitle());
Rewritten in JavaScript with continuation passing, this example quickly gets out of hand:

driver.get("http://www.google.com", function() {
 driver.findElement(By.name("q"), function(q) {
   q.sendKeys("webdriver", function() {
     driver.findElement(By.name("btnG"), function(btnG) {
       btnG.click(function() {
         driver.getTitle(function(title) {
           assertEquals("webdriver - Google Search", title);
         });
       });
     });
   });
 });
});
```

Instead of using continuation passing, WebDriverJS uses "promises" to track the state of each operation. Using promises, the Google search example can be rewritten as follows

```js
driver.get("http://www.google.com");
driver.findElement(By.name("q")).sendKeys("webdriver");
driver.findElement(By.name("btnG")).click();
driver.getTitle().then(function(title) {
 assertEquals("webdriver - Google Search", title);
});
```

###Promises###

A promise is an object that represents a value, or the eventual computation of a value. Every promise starts in a pending state and may either be successfully resolved with a value or it may be rejected to designate an error. Once a promise transitions from the pending state, it becomes immutable and cannot change state again.

You can register observers with a promise using the then() function. This function takes two (optional) functions as arguments: a callback to invoke with the resolved value and an errback to invoke if the promise was rejected. The observer will be invoked immediately if the promise has already been resolved. Otherwise, it will be invoked as soon as it is resolved. If multiple observers have been registered, they will be called in the order added:

```js
var promise = webdriver.getTitle();

promise.then(function(title) {
 console.log("title is: " + title);
});

promise.then(function(title) {
 console.log("title still is: " + contents);
});

// title is: foo
// title still is: foo
```

###Value Propagation and Chaining###

Each call to then() will return a new promise that represents the observer's result. If the callback (or errback) returns a value (no return is treated as returning undefined), the promise will be resolved. Conversely, if the handler function throws an error, the promise will be rejected. If you omit one of the handler functions from a call to then(), the corresponding value/error will simply propagate to the resulting promise. With these properties, you can construct processing pipelines:

```js
loadWebApp().
   then(login).
   then(openUserPreferences).
   then(changePassword).
   then(null, function(err) {
     console.error("An error was thrown! " + err);
    });
```

In the example above, the first three calls to then() omit an errback function. If any of these actions (or the initial loadWebApp) throws, the error will propagate to the final call to then(), effectively defining the catch clause in a try-catch:

```js
try {
  loadWebApp();
  login();
  openUserPreferences();
  changePassword();
 } catch (err) {
  console.error(
      "An error was thrown! " + err);
 }
 loadWebApp().
    then(login).
    then(openUserPreferences).
    then(changePassword).
    then(null, function(err) {
      console.error(
          "An error was thrown! " + err);
    });
 ```
 
### Error Handling ###

A rejected promise is considered handled if an observer with an errback function has been registered. Chained promises are implicitly handled by the next promise in the chain.

```js
webdriver.promise.rejected(new Error('boom')).  // (1)
   then(callback1).                             // (2)
   then(callback2).                             // (3)
   then(null, onError);                         // (4)
```
   
The example above defines a chain on a rejected promise. The first three promises in the chain (lines 1-3) are all implicitly handled by the next. The final promise, on line 4, is explicitly handled by the onError function.

If a rejected promise is left unhandled, the offending error will be passed to the active control flow in the next turn of the JavaScript event loop. You can receive notifications of unhandled rejections by listening for an "uncaughtException" event on the ControlFlow object:

```js
webdriver.promise.controlFlow().on('uncaughtException', function(e) {
 console.error('Unhandled error: ' + e);
});

webdriver.promise.rejected(new Error('boom'));
console.log('Leaving rejection unhandled');

// Leaving rejection unhandled
// Unhandled error: Error: boom
```

If no event listeners have been registered, the ControlFlow will rethrow the error to the global error handler. In Node.js, this will trigger an "uncaughtException" event on the global process object; in a browser, window.onerror will be notified.

###Deferred Objects###

There are two halves to WebDriver's promise API: a promise, and a deferred. The promises discussed so far represent the consumer half of the API and provide functions for interacting with a computed value. Sometimes you will need to produce a promise and control when its value is set. For this, you will use a Deferred object.

A deferred object is a special version of a promise that provides two additional functions for setting a promise's value: fulfill() and reject(). Calling fulfill() will set the promise value and invoke the callback chain. Likewise, call reject() to set an error and trigger any error-handlers. The example below demonstrates how to create a promise that will resolve after a set amount of time:

```js
function timeout(ms) {
 var d = webdriver.promise.defer();
 var start = Date.now();
 setTimeout(function() {
   d.fulfill(Date.now() - start);
 }, ms);
 return d.promise;
}

function printElapsed(ms) {
 console.log('time: ' + ms + ' ms');
}

timeout(750).then(printElapsed);
timeout(500).then(printElapsed);

// time: 500 ms
// time: 750 ms
```

In the example above, the timeout() function returns the deferred object's promise property. 
This property provides consumers access to the then() function while reserving 
reject() and fulfill() for timeout().

Two functions are provided for quickly creating promises from known values:

```js
// This...
var p1 = webdriver.promise.fulfilled(123);
var p2 = webdriver.promise.rejected(new Error('boom'));

// ...is shorthand for:
var d1 = webdriver.promise.defer();
d1.fulfill(123);
var p1 = d1.promise;

var d2 = webdriver.promise.defer();
d2.reject(new Error('boom'));
var p2 = d2.promise;
```

###Cancelling a Promise###

Occasionally, the need arises to cancel the computation of a promised value. This can be accomplished using the cancel() function. Calling cancel() will cause the promise to be rejected. A cancellation handler may be provided when creating a Deferred object; upon calling cancel(), this handler will be invoked before rejecting the promise.

The example below demonstrates using cancel() to abort a repeating timer:

```js
function logForever() {
 var key = setInterval(function() {
   console.log('hello');
 }, 100);

 return webdriver.promise.defer(function() {
   console.log('goodbye');
   clearInterval(key);
 }).promise;
}

var promise = logForever();

setTimeout(function() {
 promise.cancel();

 // Swallow the resulting cancellation error.
 promise.then(null, function() {});
}, 300);

// hello
// hello
// hello
// goodbye
```

Cancellation handlers are inherited by chained promises. When you call cancel(), the root promise will cancelled, allowing the resulting rejection to propagate through the chain:

```js
function onCancel() {
 console.log('The promise was cancelled');
}

webdriver.promise.defer(onCancel).
   then(step1).
   then(step2).
   then(step3).
   then(null, function(e) {
     console.error('There was an error: ' + e);
   }).
   cancel(new Error('boom'));

// The promise was cancelled
// There was an error: Error boom
```

Note the cancel() operation is available on both the Promise and Deferred objects. To prevent a promise consumer from cancelling an operation, simply define a custom handler that throws.

```js
function foo() {
 var d = webdriver.promise.defer(function() {
   throw Error('nice try!');
 });
 return d.promise;
}

foo().cancel();

// Error: nice try!
```

###Control Flows###

Recall the Google search example:

```java
// In Java
driver.get("http://www.google.com");
driver.findElement(By.name("q")).sendKeys("webdriver");
driver.findElement(By.name("btnG")).click();
assertEquals("webdriver - Google Search", driver.getTitle());
```

This can be rewritten using the promise library as:

```js
driver.get("http://www.google.com").
   then(function() {
     return driver.findElement(By.name("q"));
   }).
   then(function(q) {
     return q.sendKeys("webdriver");
   }).
   then(function() {
     return driver.findElement(By.name("btnG"));
    }).
   then(function(btnG) {
     return btnG.click();
    }).
   then(function() {
     return driver.getTitle();
   }).
   then(function(title) {
     assertEquals("webdriver - Google Search", title);
    });
```    
    
Unfortunately, this is really verbose, even for such a simple example - longer scripts could easily 
get out of hand. In order to provide an API that cleanly handles asynchronous actions 
without impeding readability, WebDriverJS uses a promise "manager" to coordinate the 
scheduling and execution of all commands.

The promise manager maintains a queue of scheduled tasks, executing each once the one 
before it in the queue is finished. The WebDriver API is layered on top of the promise manager, 
allowing us to simplify the search example:

```js
driver.get("http://www.google.com");
driver.findElement(webdriver.By.name("q")).sendKeys("webdriver");
driver.findElement(webdriver.By.name("btnG")).click();
driver.getTitle().then(function(title) {
 assertEquals("webdriver - Google Search", title);
});
```

At the heart of the promise manager is the ControlFlow class. You can obtain an instance of this class using webdriver.promise.controlFlow(). Tasks are enqueued using the execute() function. Tasks always execute in a future turn of the event loop, once those before it in the queue (if there are any) have completed.

```js
var flow = webdriver.promise.controlFlow(),
   num = 0,
   start = Date.now();

function printStepNumber() {
 num += 1;
 console.log("task #" + num);
 return webdriver.promise.delayed(250);
}

flow.execute(printStepNumber);
flow.execute(printStepNumber);
flow.execute(printStepNumber).then(function() {
 var elapsed = Date.now() - start;
 console.log("All done; elapsed time: " + elapsed + " ms");
});
console.log("All tasks scheduled!");

// All tasks scheduled!
// task #1
// task #2
// task #3
// All done; elapsed time: 750 ms
```

Each command in the WebDriverJS API wraps a call to ControlFlow#execute(), ensuring that all commands execute in the correct order - no extra effort required.

###Manual promise chaining###

```js
driver.get(“http://www.google.com”).
    then(function() {
      return driver.findElement(webdriver.By.name('q'));
    }).
    then(function(q) {
      return q.sendKeys('webdriver');
    }).
    then(function() {
      return driver.findElement(webdriver.By.name('btnG'));
    }).
    then(function(btnG) {
      return btnG.click();
    }).
    then(function() {
      return driver.getTitle();
    }).
    then(function(title) {
      console.log(title);
    });

// webdriver - Google Search
Using the promise manager

driver.get(“http://www.google.com”);
driver.findElement(webdriver.By.name('q')).sendKeys('webdriver');
driver.findElement(webdriver.By.name('btnG')).click();
driver.getTitle().then(function(title) {
  console.log(title);
});

// webdriver - Google Search
```

###Framing###

The ControlFlow class's execution queue ensures tasks are executed in order, but what about nested tasks? Nested calls to execute() push a new frame to the ControlFlow's internal call stack. Upon each turn of its execution loop, the ControlFlow will pull a task to execute from the queue of the top-most frame.

```js
flow.execute(function() {
 console.log('a');

 flow.execute(function() {
   console.log('b');
 });

 flow.execute(function() {
   console.log('c');
 });

});

flow.execute(function() {
  console.log('d');
});

// a
// b
// c
// d
```

A new frame will also be allocated for promise callbacks and errbacks, logically grouping tasks:

```js
flow.execute(function() {
 console.log('a');

}).then(function() {

 flow.execute(function() {
   console.log('c');
 });
});

flow.execute(function() {
  console.log('b');
});

// a
// c
// b
```

Consider the following example written with the Java API:

```js
1| driver.get(MY_APP_URL);
2|
3| String title = driver.getTitle();
4| if ("Login Page".equals(title)) {
5|   driver.findElement(By.id("user")).sendKeys("bugs");
6|   driver.findElement(By.id("pw")).sendKeys("bunny");
7|   driver.findElement(By.id("login")).click();
8| }
9|
10| driver.findElement(By.id("userPreferences")).click();
```

This example can be rewritten in JavaScript as:

```js
1| driver.get(MY_APP_URL);
2| driver.getTitle().then(function(title) {
3|   if ("Login Page" === title) {
4|     driver.findElement(By.id("user")).sendKeys("bugs");
5|     driver.findElement(By.id("pw")).sendKeys("bunny");
6|     driver.findElement(By.id("login")).click();
7|   }
8| });
9| driver.findElement(By.id("userPreferences")).click();
```

Since lines 4-6 are executed within a callback, the ControlFlow class will ensure they execute before line 9.

###Error Handling###

When an unhandled rejection is reported to the ControlFlow class, it will check if there is more than one frame on the stack. If so, the ControlFlow will abort the current frame and propagate the rejection to the previous frame.

```js
flow.execute(function() {
 console.log('a');

 flow.execute(function() {
   console.log('b');

   flow.execute(function() {
     console.log('c');
     return webdriver.promise.rejected('boom');
   });

   // The previous task fails, so this will never execute.
   flow.execute(function() {
     console.log('d');
   });
 });

}).then(function() {
 console.log('Success!');
}, function(err) {
 console.log('There was an error! ' + err);
});

// a
// b
// c
// There was an error! boom
```

If an unhandled rejection propagates all the way to the top-most frame, the ControlFlow will use previously described behavior of emitting an uncaughtException event or notifying the global error handler if there are no registered listeners:

```js
flow.execute(function() {
 flow.execute(function() {
   flow.execute(function() {
     throw Error('No soup for you!');
   });
 });
});

flow.on('uncaughtException', function(err) {
 console.log('There was an uncaught exception: ' + err);
});

// There was an uncaught exception: Error: No soup for you!
```

You can error propagation to model try-catch blocks. Consider the Java example below:

```js
try {
  driver.switchTo().alert().dismiss();
} catch (NoAlertPresentException ignored) {
}

try {
  saveButton.click();
} finally {
  logOut();
}
```

This could be rewritten using WebDriverJS as:

```js
driver.switchTo().alert().dismiss().then(null, function(e) {
  if (e.code !== webdriver.ErrorCode.NO_ALERT_PRESENT) {
    throw e;
  }
});
```

saveButton.click().then(logOut, logOut);
Defining Multiple Flows
WebDriverJS supports defining "parallel" flows using webdriver.promise.createFlow(). This function accepts a callbac which will be passed the newly created flow. Tasks scheduled within this flow will be synchronized with each other, but will remain independent of any other control flows. Each call to createFlow() returns a promise that will resolve when the flow has completed.

```js
var flow1 = webdriver.promise.createFlow(function(flow) {
 flow.execute(function() {
   console.log("flow 1 a");

   flow.execute(function() {
     console.log("flow 1 c");
   });
 });

 flow.execute(function() {
   console.log("flow 1 b");
 });
});

var flow2 = webdriver.promise.createFlow(function(flow) {
 flow.execute(function() {
   console.log("flow 2 a");

   flow.execute(function() {
     console.log("flow 2 c");
   });
 });

 flow.execute(function() {
   console.log("flow 2 b");
 });
});

flow1.then(function() {
 console.log('Flow 1 finished');
});

flow2.then(function() {
 console.log('Flow 2 finished');
});

// flow 1 a
// flow 2 a
// flow 1 c
// flow 2 c
// flow 1 b
// flow 2 b
// Flow 1 finished
// Flow 2 finished
````

If an error propagates to the top of a control flow's stack, the promise returned by createFlow() will be rejected.

```js
var result = webdriver.promise.createFlow(function() {
 throw Error('fail');
});

result.then(function() { console.log('Success!'); },
           function(e) { console.log('Failure: ' + e); });

// Failure: Error: fail
```

The webdriver.Builder class will automatically attach the created WebDriver instance to the active flow. The following example uses multiple flows to test Google search using various search terms:

```js
var terms = [
   'javascript',
   'selenium',
   'webdriver'
];

var flows = terms.map(function(term) {
 return webdriver.promise.createFlow(function() {
   var driver = new webdriver.Builder().build();

   driver.get('http://www.google.com');
   driver.findElement(webdriver.By.name('q')).sendKeys(term);
   driver.findElement(webdriver.By.name('btnG')).click();
   driver.getTitle().then(function(title) {
     if (title !== (term + ' - Google Search')) {
       throw Error('Unexpected title: ' + title);
     }
   });
 });
});

webdriver.promise.fullyResolved(flows).then(function() {
 console.log('All tests passed!');
});
```

###Working in a Browser###

In addition to node, WebDriverJS may also be used directly in the browser. To compile the browser module, which has a smaller set of dependencies than node, run:

```js
% ./go //javascript/webdriver:webdriver
```

The WebDriverJS client must be used with the Selenium server. Just as with node, 
you can create a WebDriver instance using the webdriver.Builder class:

```html
<!DOCTYPE html>
<script src="webdriver.js"></script>
<script>
  var driver = new webdriver.Builder().
     usingServer('http://localhost:4444/wd/hub').
     withCapabilities(webdriver.Capabilities.chrome()).
     build();

 driver.get('http://www.google.com');
 driver.findElement(webdriver.By.name('q')).sendKeys('webdriver');
 driver.findElement(webdriver.By.name('btnG')).click();
 driver.getTitle().then(function(title) {
   if (title !== 'webdriver - Google Search') {
     throw new Error(
         'Expected "webdriver - Google Search", but was "' + title + '"');
   }
 });

 driver.quit();
</script>
```

###Controlling the Host Browser###

Launching a browser to run a WebDriver test against another browser is a tad redundant (compared to simply using node). Instead, using WebDriverJS in the browser is intended for automating the browser actually running the script. This can be accomplished as long as the URL for the server and session ID for the browser are known. While these values may be passed to the builder directly, they may also be defined using the wdurl and wdsid "environment variables", which are parsed from the loading page's URL query data:

```html
<!-- Assuming HTML URL is
 -- /test.html?wdurl=http://localhost:4444/wd/hub&wdsid=foo1234
 -->
<!DOCTYPE html>
<script src="webdriver.js"></script>
<input id="input" type="text"/>
<script>
 // Attaches to the server and session controlling this browser.
 var driver = new webdriver.Builder().build();

 var input = driver.findElement(webdriver.By.tagName('input'));
 input.sendKeys('foo bar baz').then(function() {
   assertEquals('foo bar baz',
       document.getElementById('input').value);
 });
</script>
```

###Caveats###

There are a few caveats to using WebDriverJS in the browser. First, the webdriver.Builder class may only be used to attach to an existing session. In order to create a new session, you must manually create a session, as shown in the first browser-based example. Second, there are commands which may impact the page running the WebDriverJS script.

webdriver.WebDriver#quit: The quit command will terminate the entire browser process, including the window running WebDriverJS. Do not use this command unless you are absolutely sure you want to lose everything.
webdriver.WebDriver#get: WebDriver is designed to emulate a user as closely as possible. This means that regardless of which frame a WebDriver client is currently focused on, navigation commands (e.g., driver.get(url)) always target the top-most frame. When controlling its host browser, a WebDriverJS script could navigate away from itself by issuing a WebDriver.get command while focused on its own window. If you wish to automate the host browser and still navigate between pages, focus your WebDriver client on a second window (this is similar in concept to Selenium RC's multi-window mode):

```html
<!DOCTYPE html>
<script src="webdriver.js"></script>
<script>
 var testWindow = window.open('', 'slave');

 var driver = new webdriver.Builder().build();
 driver.switchTo().window('slave');
 driver.get('http://www.google.com');
 driver.findElement(webdriver.By.name('q')).sendKeys('webdriver');
 driver.findElement(webdriver.By.name('btnG')).click();
</script>
```

###Supported Browsers###

WebDriverJS is supported in the following browsers:

* IE 8+
* Firefox 10+
* Chrome 12+
* Opera 12+
* Android 4.0+
* 
