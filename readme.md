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
            verbose: true,
            wait_ms: 15000
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

        window.onload = function () {
            unit.start()
        };


[sample output](http://gkindel.com/okjs/test/quickstart.html)

Reference
-------------
* okjs(options)

    Returns an instance of the testing harness.

    Available options:
    * verbose - if set to false, tests that pass are hidden
    * wait_ms - default delay for callbacks and events
    * exceptions - false by default. Setting to true will disable the catching of exceptions, which
        can help debugging tools.

* .test(msg, fn)

    Defines a group of test assertions. Groups are executed sequentially. Exceptions in the function will
    halt the group functions execution.  All outstanding events are resolved before moving on.

* .equal(a, b, msg)

    Basic assert.  Strictly compared.  If objects are passed in, it looks for b.equals(a),
    if defined. Otherwise does a strict (===) compare.

* .nequal(a, b, msg)

    Basic negative assert. Strict.

* .callback(msg, delay, fn, context)

    Returns a function which is intended to be asynchronously executed, by intervals, observers, eventlisteners, etc. The
    callback needs to be executed in 'delay' time or the test fails, even if "fn" was left null.  The "context" parameter
    will be used as the "this" property, preserving scope for object methods.

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

