Evently Primer
==============

The Basic Model
---------------

If you've ever built event-based applications with jQuery,
you should feel right at home with Evently.
All it is, is a shortcut for writing jQuery,
and was extracted from applications I've been writing over the last few months.

In jQuery development, it is common to `intermix custom and standard
events <http://yehudakatz.com/2009/04/20/evented-programming-with-jquery/>`_,
so for instance, you may have a DOM element with a ``click`` handler,
a ``hover`` handler and a ``loggedIn`` handler.
We are assuming you're familiar with the first two,
and probably familiar with custom events like the last one as well.


Custom Events Primer
--------------------

In a normal jQuery application, you'd apply these like this::

    $("#myelement").bind("click", function() {
        $(this).text("You clicked me.");
    });

    $("#myelement").bind("mouseenter", function() {
        $(this).text("You moused over me.");
    });

    $("#myelement").bind("loggedIn", function(e, user) {
        $(this).text("You are logged in.");
    });

Of course, custom events need to be triggered,
as the browser has no built-in event by that name,
so ``loggedIn`` won't happen until you trigger it from code somewhere.
Like this::

    $("#logIn").bind("click", function() {
        $("#myelement").trigger("loggedIn", ["Dr. Pepper"]);
    })

Hopefully none of what we've written so far is surprising.
If you learned a few new tricks about jQuery, you can thank us later.
If you already knew all that, keep reading, as the good stuff is about to start.


Back to Evently
---------------

As we mentioned at the top of the article,
Evently is just shortcuts for writing code like the above.
Here is the above widget defined in Evently::

    $("#another").evently({
        click: function() {
            $(this).text("You clicked me.");
        },
        mouseenter: function() {
            $(this).text("You moused me.");
        },
        loggedIn: function(e, name) {
            $(this).text("You are logged in, " + name);
        }
    });

You can see that it is just the same as the original code sample,
except that instead of individual calls to ``.bind``,
there is a JavaScript object structure that specifies the event names.

Just like the above example, custom events need to be triggered by code.
Let's setup an element to trigger the ``loggedIn`` event of the Evently widget::

    $("#anotherLogIn").evently({
        click: function() {
            $("#another").trigger("loggedIn", ["Mr. Pibb"]);
        }
    });

If all you take away from this document is that
Evently is just a shortcut for wiring up jQuery widgets,
you can consider yourself a success.
However, there are four more things to learn.
(I know, that's a lot, but it's a powerful library.)
When you've learned the four things, you will become a true ninja.


Four More Things to Learn
-------------------------

* Evently has a built-in way to handle Ajax calls and Mustache_ templating.
* Evently widgets can contain other Evently widgets,
  nested like matryoshka dolls. This turns out to be hella useful.
* Evently has some magic events,
  including a handler for realtime updates from CouchDB_.
* Evently code can be represented as a deeply-nested filesystem tree,
  when you use the couchapp_ to deploy it.


Mustache Templating
+++++++++++++++++++

So far, we have only looked at Evently events
that have a plain old function as their value.
Evently has a built-in method of rendering Mustache_ templates.
When you use these structured event definitions,
Evently takes care of a lot of the boilerplate so you can just concentrate on your application.

Here is a basic Evently widget using Mustache_::

    $("#basicMustache").evently({
        click: {
            mustache: '<h4>Thanks for clicking me, {{name}}</h4>',
            data: function() {
                var name = prompt("Enter your name");
                return {
                    name: (name || "Anonymous")
                };
            }
        }
    });

The two key elements here are ``mustache`` (the HTML template) and ``data``,
which is a function which feeds the values into the Mustache template.
For more information about Mustache templates,
see ``the Mustache documentations <http://mustache.github.com/>``.


Ajax Requests with Mustache Templates
+++++++++++++++++++++++++++++++++++++

Probably the most useful thing in the world you can do in an Ajax app is,
*load data from the server and render it*.
This is usually an asynchronous operation.
We'll assume you are comfortable with the fundamentals of Ajax requests.

Here is an Evently widget like the previous one, that loads data
from the server using jQuery, before passing it to the Mustache template.
Instead of an jQuery ajax call, you could use any other method of loading data.
Later we'll see some examples that use a CouchDB library to load the data.

::

    $("#ajaxMustache").evently({
        mouseenter: {
            async: function(callback) {
                $.ajax({
                    url: "http://twitter.com/status/user_timeline/jchris.json?count=1&callback=?",
                    dataType: "jsonp",
                    success: function(tweets) {
                        callback(tweets[0]);
                    }
                });
            },
            mustache: '<h4>Latest tweet from: {{name}}</h4><img src="{{image_src}}"/><p>{{text}}</p>',
            data: function(tweet) {
                return {
                    text: tweet.text,
                    name: tweet.user.name,
                    image_src: tweet.user.profile_image_url
                };
            }
        }
    });

What's new in this code? The ``async`` function is the main star,
which in this case makes an Ajax request (but it can do anything it wants).
Another important thing to note is that the first argument
to the async function is a ``callback`` which you use to tell Evently
when you are done with your asynchronous action.
In this case it is called with the first tweet in the list from Twitter.
Whatever you pass to the callback function
then becomes the first item passed to the ``data`` function.


Nested Evently Widgets with ``selectors``
+++++++++++++++++++++++++++++++++++++++++

So far, we've seen how you can use Evently to do regular jQuery events,
and how you can feed special event definitions to Evently,
that render Mustache templates and make asynchronous requests.
Now let's get recursive!

::

    $("#nestedEvently").evently({
        click: {
            mustache: '<p>Click or hover the word to change it: '
                       + '<span class="word">{{word}}</span></p>'
                       + '<p><a href="#win">Click here to win the lottery!</a></p>',
            data: {word:"melon"},
            selectors: {
                "span.word": {
                    click: {
                        mustache: '<strong>watermelon</strong>'
                    },
                    mouseenter: {
                        mustache: '<em>cantaloupe</em>'
                    },
                    congrats: {
                        mustache: "<strong>{{you}} won the lottery!</strong>",
                        data: function() {
                            return {
                                you : prompt("enter your name for a chance to win")
                            }
                        }
                    }
                },
                'a[href=#win]': {
                    click: function() {
                        $("span.word").trigger("congrats");
                        return false;
                    }
                }
            }
        }
    });

The ``selectors`` key points to a set of jQuery selectors,
which are used to find elements and add more evently widgets to them.
In this case the original Mustache template is just this HTML

.. code-block:: html

    <p>
        Click or hover the word to change it:
            <span class="word">{{word}}</span>
    </p>
    <p><a href="#win">Click here to win the lottery!</a></p>

Hovering over the ``span`` with class ``word`` will change the word,
as will clicking it.
Clicking the link with ``href=#win`` will trigger the ``congrats``
custom event on the ``span``.

What's neat about this is that you can draw a widget
that requires subqueries or loading data
from disparate sources on different events,
and avoid having to do all the loading work up front.
So the initial content is displayed immediately
and progressively enhanced with the results of running more code.

You can do anything inside a selector that you can do in a top-level Evently section.


A Little Bit of Magic
+++++++++++++++++++++

Evently has two magic event names: ``_init`` and ``_changes``.
The ``_init`` event runs whenever the Evently widget is initialized.
Here's a simple example::

    $("#initWidget").evently({
        _init: function() {
            var widget = $(this);
            $.ajax({
                url: "/_session",
                dataType: "json",
                success: function(resp) {
                    if (resp.userCtx && resp.userCtx.name) {
                        widget.trigger("loggedIn", [resp.userCtx]);
                    } else {
                        widget.trigger("loggedOut");
                    }
                }
            });
        },
        loggedIn: {
            data: function(e, user) {
                return user;
            },
            mustache: '<p>Hello {{name}}</p>'
        },
        loggedOut: {
            mustache: '<p>You are logged out</p>'
        }
    });

You can see in this example that ``_init`` is run,
and it decides whether to trigger the ``loggedIn`` or ``loggedOut`` events
based on the result of an Ajax call.

The ``_changes`` event name is CouchDB specific, at the moment.


.. _Mustache: https://github.com/janl/mustache.js
.. _CouchDB: http://couchdb.apache.org
.. _couchapp: https://github.com/couchapp/couchapp
