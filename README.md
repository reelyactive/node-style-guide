# reelyActive's Node.js Guide

Here we summarise the best practices, coding standards and tools for all our Node.js development.


## Existing Standards

Fortunately, we aren't the first to code for Node.js, and there are some good style guides out there from which to draw inspiration.  Remember, the reason for following (de facto) standards is to facilitate both collaboration and consistency.


### Felix's Node.js Style Guide

[Read it on GitHub](https://github.com/felixge/node-style-guide).  Read _all_ of it.  Seriously.  All of it.  We'll wait patiently.  Now that you've read it you'll recall and appreciate the _select_ highlights below:

#### Two Spaces

> Use 2 spaces for indenting your code and swear an oath to never mix tabs and spaces - a special kind of hell is awaiting you otherwise.

#### 80 Characters Per Line

> Limit your lines to 80 characters. Yes, screens have gotten much bigger over the last few years, but your brain has not. UUse lowerCamelCase for variables, properties and function names
Use UpperCamelCase for class names
Use UPPERCASE for Constants
se the additional room for split screen, your editor supports that, right?

#### Case

- Use lowerCamelCase for variables, properties and function names
- Use UpperCamelCase for class names
- Use UPPERCASE for Constants


### Google's JavaScript Style Guide

[Read it](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml).  You can skim through the bits that Felix already touched upon, because in the event of a conflict, Felix's Style Guide shall prevail.  Please take extra care in reading the following three sections:

#### Naming

The very explicit [Naming Style Rules](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml?showone=Naming#Naming) provide more than simple DOs and DONTs, they also explain why the given practices are encouraged.  They also reinforce Felix's naming guidelines, for example:

- functionNamesLikeThis,
- variableNamesLikeThis,
- ClassNamesLikeThis,
- EnumNamesLikeThis,
- methodNamesLikeThis,
- CONSTANT_VALUES_LIKE_THIS,
- foo.namespaceNamesLikeThis.bar, and
- filenameslikethis.js

#### Code Formatting

The very explicit [Code Formatting Style Rules](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml?showone=Code_formatting#Code_formatting) provide all the examples you need to resolve ambiguities.  Keep in mind that Google follows the C++ formatting rules in spirit.  How about an example pertaining to a common source of contention: Function Arguments?

> When possible, all function arguments should be listed on the same line. If doing so would exceed the 80-column limit, the arguments must be line-wrapped in a readable way. To save space, you may wrap as close to 80 as possible, or put each argument on its own line to enhance readability. The indentation may be either four spaces, or aligned to the parenthesis.

#### Comments

The very explicit [Comments Style Rules](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml?showone=Comments#Comments) also follow the C++ rules in spirit and specifically use [JSDoc](http://usejsdoc.org/).  There are many illustrative examples.  Read them!  And remember:

> Inline comments should be of the // variety.


## reelyActive Standards

The existing standards above cover a lot, but they don't cover everything.  So we're establishing a few of our own internal standards which are specifically suited to our projects and workflow.

### Coding Style

#### Copyright

Every JavaScript file should start with the following comment:

    /**
      * Copyright reelyActive 2014-2015
      * We believe in an open Internet of Things
      */

The use of a copyright indicates that the code is intellectual property created by reelyActive.

Note that the date range should extend between the year the file was initially created to the year the file was last modified.  This is a simple way to provide a brief historical perspective of the code.

The line _"We believe in an open Internet of Things"_ serves as a reminder of our vision and should inspire the developer to maintain the code in a way befitting of an open project with many collaborators.

#### Self: what is this exactly?

What is _this_?  In [Mixu's Node Book](http://book.mixu.net/node/ch4.html), the _this_ keyword is considered the #1 gotcha in V8 and JavaScript.  Therefore, in a class constructor and its class methods, we always use a _self_ variable to refer to the class object instance.  In other words, the _self_ unamibuously refers to the class instance.

A static method can accept an _instance_ parameter in order to work on a specific class object instance.

The example below illustrates our use of the _self_ variable and _instance_ parameter.  And [this video clip](https://vimeo.com/59037145) represents an amusing meme you can use to recall our motivation for the _self_ variable.

```javascript
/**
 * An example class constructor
 * @constructor
 */
function SomeClass() {
  var self = this;                               // Always on the first line

  self.app = express();
  self.httpPort = 80;

  self.app.listen(self.httpPort, function() {    // this.httpPort would work
    console.log("Listening on ", self.httpPort); // this.httpPort wouldn't work
  });
};

/**
 * An example class method
 */
SomeClass.prototype.doSomething = function() {
  var self = this;                               // Always on the first line

  someStaticMethod(self);
};


/**
 * An example static method
 * @param {SomeClass} instance The given class object instance.
 */
function someStaticMethod(instance) {
  instance.app.doSomethingElse();
};
```

### RESTful Structure

#### Keep server.js simple

The class constructor is contained in a file named server.js.  A developer should be able to quickly read the file and identify all the supported routes.  Imagine it as a table of contents for the REST API.

Below is an example server.js file.

```javascript
var express = require('express');

/**
 * An example RESTful class constructor
 * @constructor
 */
function RESTfulClass() {
  var self = this;

  self.app = express();

  self.app.use(function(req, res, next) {
    req.instance = self;                         // Sneak the class object
    next();                                      //   instance into the req
  });

  // Each route is in a separate external file
  self.app.use('/someroute', require('./routes/someroute'));
  self.app.use('/anotherroute', require('./routes/anotherroute'));

  self.app.listen(80);
};

module.exports = RESTfulClass;
```

#### One route per file

Each API route is in a separate external file within a routes subfolder.  A developer should be able to quickly read the file and identify any middleware used, all the routes and subroutes as well as the supported HTTP methods for each.

Each route is an [express router object](http://expressjs.com/api.html#router).

Each method (GET, POST, PUT, DELETE, ...) for a given route will be implemented via a function call.  This allows a developer to quickly navigate to the implementation of any given route.  For clarity, the function name will have a prefix corresponding to the given HTTP method, specifically:
- GET = retrieve
- POST = create
- PUT = replace
- DELETE = delete

In keeping with the previous example, the file routes/someroute.js would resemble the following:

```javascript
var express = require('express');
var router = express.Router();

router.use(function someMiddleware(req, res, next) {
  next();
});

router.route('/')                                // Supports GET, POST
  .get(function(req, res) {
    retrieveSomething(req, res);
  })
  .post(function(req, res) {
    createSomething(req, res);
  });

router.route('/:id')                             // Supports GET, PUT, DELETE
  .get(function(req, res) {
    retrieveSomethingSpecific(req, res);
  })
  .put(function(req, res) {
    replaceSomethingSpecific(req, res);
  })
  .delete(function(req, res) {
    deleteSomethingSpecific(req, res);
  });

function retrieveSomething(req, res) { }         // GET = retrieve
function createSomething(req, res) { }           // POST = create

function retrieveSomethingSpecific(req, res) { } // GET = retrieve
function replaceSomethingSpecific(req, res) { }  // PUT = replace
function deleteSomethingSpecific(req, res) { }   // DELETE = delete

module.exports = router;
```



