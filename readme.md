okjs
=============
 ojks - a tiny asynchronous-friendly JS unit test utility

About
-------------
JavaScript and the web form a heavily asynchronous environment, which can pose particular challenges for JS based unit
testing.  In particular, Ajax, click handlers, and other observer models require that exceptions be caught in various
execution blocks and without interfering with object scope.  Additionally, these methods need to execute in a specified
amount of time.

The okjs library directly attends to these hurdles, and is specifically intended for the development process.


Quick Start
-------------

        var unit = okjs({
            verbose: true
        });

        unit.test("test group 1",  function () {
            unit.equal(true, true, "true eq true");
            unit.nequal(true, false, "true neq false");
            unit.nequal(1, true, "1 neq true");
        });

        unit.test("test group 2",  function () {
           setTimeout( unit.callback("timeout callback w/ function", 1000, function () {
                unit.equal(true,true, "in callback function ok")
            }),  500);
        });

       unit.test("test group 3",  function () {
            unit.event("click", document, "document was clicked", function (e) {
                unit.equal(e.type, "click", "was a mouse click event");
                unit.equal(e.currentTarget, document, "currentTarget is the document");
            }, 10000);
            unit.log("waiting 10s for user to click page... ");
        });

        window.onload = function () {
            unit.start()
        };


[sample output](http://gkindel.com/okjs/test/quickstart.html)

Reference
-------------
* `okjs(options)`

    Returns an instance of the testing harness.

    Available options:
    * `verbose` - if set to false, tests that pass are hidden
    * `wait_ms` - default delay for callbacks and events
    * `exceptions` - Disabled the catching of exceptions, which can help debugging with browser tools which
        break on exceptions. Defaults to false.

* `.test(msg, fn)`

    Defines a group of test assertions. Groups are executed sequentially. Exceptions in the function will
    halt the group functions execution.  All outstanding events are resolved before moving on.

    * `msg` : Associated test message.
    * `fn` : An anonymous function with no arguments.

* `.equal(a, b, msg)`

    Basic assert.  Strictly compared.  If objects are passed in, it looks for b.equals(a),
    if defined. Otherwise does a strict (===) compare.

    * `a` , `b` : objects to be compared.
    * `msg` : Associated test message.


* `.nequal(a, b, msg)`

    Basic negative assert. Strict.

    * `a` , `b` : objects to be compared.
    * `msg` : Associated test message.

* `.callback(c, fn, context, options)`

    Returns a function which is intended to be asynchronously executed, by intervals, observers,
    ajax responses, event listeners, etc. The callback needs to be executed in time or the
    test fails, even if "fn" was left null. Arguments are preserved.

    * `msg` : Associated test message.
    * `fn` : Optional callback which specifies an event listener function.
    * `context` : Scope, to be used for `this` in the callback.
    * `options`:
        * `wait_ms` - custom time allocation for this callback only

* `.event(type, source, msg, fn, delay)`

    Asserts an event will be fired. Event listener with callback is automatically added, waited
     for, and removed once either the event is caught or timeout occurs.  Event arguments are preserved.

    * `type` : Required event type (eg: "click", "resize").
    * `source` : Object which is expected to fire event, supporting DOM event syntax.
    * `msg` : Associated test message.
    * `delay` : Optional delay in msec. Overrides default timeout.
    * `fn` : Optional callback which specifies an event listener function.

* `.log(msg)`

    Logs a message with no effect on test results. Useful for notes.

    * `msg` : Associated text message.

More Info
-------------

Currently, the best place to start is to look at the framework's self-test page, in the [/test/](test/) folder,
 or [online here](http://gkindel.com/okjs/test/test.html)


Bugs & Contributions
-------------

This is an open source project, contributions and bug fixes are welcome.  Please contact greg.kindel@gmail.com if you
wish to become a contributor.


License
-------------
okjs is open source, made available under the [MIT License](http://www.opensource.org/licenses/mit-license.php).


Author
-------------
Twitter: [@gkindel](http://twitter.com/#!/gkindel)
