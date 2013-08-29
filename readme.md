# okjs v2.0

---

A tiny, asynchronous-friendly JavaScript unit test framework.



## About

JavaScript and the web form a heavily asynchronous environment, which can pose particular challenges for JS based unit
testing.  In particular, Ajax, click handlers, and other observer models require that exceptions be caught in various
execution blocks and without interfering with object scope.  Additionally, these methods need to execute in a specified
amount of time. Lastly, once many test pages are written, there needs to be a way to run these stand-alone pages as part of
a larger suite.

Pronounced "OK JS", the library directly attends to these hurdles, and is specifically intended for use in test driven
development.

[Live Example](http://gkindel.github.io/okjs/test/okjs-unit.html)


## Quick Start

        var unit = okjs();

        unit.test("basics:  unit.assert() and unit.forbid()",  function () {
            var foo = 12345;
            unit.assert("value is true-ish", foo);
            unit.assert("strictly compare value", foo, 12345);

            var bar = 0;
            unit.forbid("value is false-ish", bar);
            unit.forbid("strictly forbid value", bar, null);
        });

        unit.test("asynchronous callbacks: unit.callback()",  function () {
            setTimeout( unit.callback("test callback is fired"), 500);
            setTimeout( unit.callback("callback fired, with subtests", function () {
                unit.assert("in callback, true == true", true,true);
            }),  500);
        });

        unit.test("event testing: unit.event() ",  function () {
            unit.event("document was clicked", document, "click", function (e) {
                unit.assert("was a mouse click event", e.type, "click");
                unit.assert("currentTarget is the document", e.currentTarget, document);
            }, 10000);

            unit.log("waiting 10s for user to click page...");
        });

        unit.test("expected errors",  function () {
            unit.exception("undefined method called", function () {
                null.missingFunction();
            });

            unit.exception("throws an exception", function () {
                throw "catch me"
            });
        });

        unit.start()


[Quickstart Output](http://gkindel.github.io/okjs/test/quickstart.html)


The okjs library consists of a harness, a test block, and a grouping of assertions.  It will catch exceptions
in both the passed in test function as well as any asynchronous callbacks or event handlers.  Any asynchronous methods
which do not fire in the configured time (which defaults to 5sec) will fail and be ignored when they do fire.

For larger projects, okjs has the ability to reference other test pages, displaying summaries and aggregating the total
test results.

        var unit = okjs();

        unit.url("quick start demo", "quickstart.html");
        unit.url("okjs unit test", "okjs-unit.html");

        unit.start()

## Reference

### Setup

* `okjs(options)`

    Returns an instance of the testing harness.

    Available options:
    * `verbose` - if set to false, tests that pass are hidden
    * `timeout` - default delay for callbacks and events
    * `exceptions` - Disabled the catching of exceptions, which can help debugging with browser tools which
        break on exceptions. Defaults to false.
    * `logger` - An optional object for rendering test results. Defaults to HTML output.

* `.test(msg, fn)`

    Defines a group of test assertions. Groups are guaranteed to be executed sequentially. Asynchronous callbacks and events
    will block the test until resolved either by firing or by timing out. All other okjs statments must be done within
    a test group. Tests are queued until unit.start() is called.

    * `msg` : Associated test message.
    * `fn` : An anonymous function with no arguments.

* `.start()`

    Begins test execution, to be called sometime after the window onLoad event.

### Assert Statements

* .`assert(msg, result)`

    Tests for truth.  Passes if `result` evaluates to true.

    * `msg` : A string description of the test.
    * `result` An object or property to test

* .`assert(msg, result, expected)`

    Strictly test for a value.  Passes if `result` === `expected`. For Objects and Arrays, a deep comparison is done via
    comparing the output of JSON.stringify() on the two values.  If objects contain a .equals() property, then it is
    called instead with an expeced boolean response.

    * `msg` : A string description of the test.
    * `result` An object or property to test
    * `expected` An object or property to be strictly compared to `result`

* .`forbid(msg, result)`

    Tests for falsehood.  Passes if `result` evaluates to false.

    * `msg` : A string description of the test.
    * `result` An object or property to test

* .`forbid(msg, result, forbidden)`

    Strictly forbid a value.  Fails if `result` === `forbidden`

    * `msg` : A string description of the test.
    * `result` An object or property to test
    * `expected` An object or property to be strictly compared to `result`

* `.callback(msg, fn, context, timeout)`

    Returns a function which must be executed in the alloted time or the
    test fails, even if "fn" was left null.

    * `msg` : A string description of the test.
    * `fn` : Optional callback which specifies a function to be called, with any parameters provided.
    * `context` : Optional scope, to be used for `this` in the callback.
    * `timeout`: Optional custom time restriction for this callback only

* `.event(type, source, msg, fn, delay)`

    Asserts an event will be fired. Event listener with callback is automatically added, waited
     for, and removed once either the event is caught or timeout occurs.

    * `msg` : A string description of the test.
    * `source` : Object which is expected to fire the event, supporting source.addEventListener(type, callback).
    * `type` : Required event type (eg: "click", "resize").
    * `fn` : Optional callback which specifies an event listener handler, with any parameters provided.
    * `context` : Optional scope, to be used for `this` in the callback.
    * `timeout`: Optional custom time restriction for this callback only

* `.exception(msg, fn, context)`

    Asserts that the passed in function, when executed, will trigger an exception.  Useful for testing expected
    error handling.

    * `msg` : A string description of the test.
    * `fn` : Optional callback which specifies a function to be called, with any parameters provided.
    * `context` : Optional scope, to be used for `this` in the callback.

* `.url(msg, url)`

    Loads and external okjs test page into an iframe and includes its results in the aggregate summary.

    * `msg` : A string description of the test.
    * `url` : A url to another okjs test page, on the same domain.

* `.log(msg)`

    Logs a message with no effect on test results. Useful for notes and user interaction prompts.

    * `msg` : An informational text message.



## Customizing Output

For basic style tweaks, the simple output provided by okjs can easily be set with CSS styles.

Advanced or progmmatic output can be achieved by by providing a `logger` object in the options provided to the constructor.

Any logger must implement the following interface:

* `logger.onTest({ message: ... })`

* `logger.onSuccess({ message: ... })` : Called when a test passes.

* `logger.onFail({ message: ..., error: ...})` : Called when a test fails.

* `logger.onInfo({ message: ..., url: ... })` : Called by `.log()` to display a comment, with an optional url.

* `logger.onFinish( okjs )` : Called when the current okjs instance, once it has finished all tests.

* `logger.onRemote( okjs )` : Called when an iframe test page finishes. Receives the remote okjs instance, once finished.

In addition, the following interfaces are made available by okjs to help with output:

* `.passed`

    The total number of tests which have passed so far.

* `.failed`

    The total number of tests which have failed so far.

* `.time()`

    Returns the current time since the harness started, in milliseconds.


## License

okjs is open source, made available under the [MIT License](http://www.opensource.org/licenses/mit-license.php).


## More Info

Currently, the best place to start is to look at the framework's self-test page, in the [/test/](okjs-unit/) folder.


## Bugs & Contributions

This is an open source project, contributions and bug fixes are welcome.  Forking is encourage, or contact me (see Author section) to get access to the development branch.  Additionally, I encourage the use of GitHub for issue tracking.


## Author

Twitter: [@gkindel](http://twitter.com/#!/gkindel)

